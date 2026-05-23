# HRM Flex Multitenant — Architecture & Flow

> Tài liệu kiến trúc cho hệ thống flex-multitenant mới (2026-05-23).
> Áp dụng cho cả 5 phase đã release qua PR nextx-hrm#5 + nextx-fe-tenant#374.

---

## 1. Bối cảnh — Tại sao phải làm

### Vấn đề cũ

NextX HRM ban đầu được build cho LSEV (1 công ty cụ thể). Toàn bộ business rule LSEV-specific đều **hardcode trong code**:

| Hardcode | File | Tác động |
|---|---|---|
| PIT giảm trừ 6.200.000đ | `EmployeeContact.cs:33,69` | Mọi workspace bắt buộc dùng mức LSEV |
| Default Nationality "Việt Nam" | `Employee.cs:57` | Không hỗ trợ công ty đa quốc gia |
| Phụ cấp 400k/600k EMPLOYEE/WORKER | `Position.cs` comment | Mỗi công ty mức khác nhau |
| 4 loại HĐ HD1/HD2/HD3/HD4 | `ContractType.cs` enum | Không thêm loại HĐ riêng |
| `const int = 7` tiêu chí đánh giá | `EvaluateContractCommandHandler.cs:19` | Bắt buộc đúng 7 tiêu chí |
| 7 RTF templates LSEV | `Templates/Contracts/*.rtf` | Không tự upload mẫu công ty khác |
| Auto-advance flow HD1→HD3→HD3→HD4 | `EvaluateContractCommandHandler.cs:92-96` | Không config được flow khác |

→ **Sản phẩm = demo LSEV, không phải SaaS multi-tenant.**

### Mục tiêu

> *"Code chỉ có engine. Template + Rule do HR config qua UI."*

Mỗi workspace = 1 tenant độc lập với:
- Tham số riêng (PIT, lương, công ty info)
- Mẫu HĐ riêng (variable substitution)
- Mẫu đánh giá riêng (số tiêu chí tùy ý)
- Custom field riêng (không cần migration)
- Flow HĐ riêng (state machine JSON)

---

## 2. Kiến trúc tổng thể

```
┌──────────────────────────────────────────────────────────────────┐
│                       Workspace LSEV                              │
│  ┌──────────────┬──────────────┬──────────────┬────────────────┐│
│  │ TenantSetting│ ContractTpl  │ EvalTemplate │ CustomFields   ││
│  │ PIT=6.2M     │ HD1..HD4 RTF │ 7 tiêu chí   │ BadgeNumber    ││
│  │ Bonus=400k   │              │ default      │                ││
│  └──────────────┴──────────────┴──────────────┴────────────────┘│
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ ContractFlowConfig: HD1→HD3→HD3→HD4 (LSEV)                 │ │
│  └────────────────────────────────────────────────────────────┘ │
├──────────────────────────────────────────────────────────────────┤
│                    Workspace Cty B (mới)                          │
│  ┌──────────────┬──────────────┬──────────────┬────────────────┐│
│  │ TenantSetting│ ContractTpl  │ EvalTemplate │ CustomFields   ││
│  │ PIT=11M      │ TT/HĐLĐ_6M  │ 5 tiêu chí   │ TaxCode        ││
│  │ Bonus=300k   │  /HĐLĐ_INF   │ default      │ VisaExpiry     ││
│  └──────────────┴──────────────┴──────────────┴────────────────┘│
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ ContractFlowConfig: TT→HĐLĐ_6M→HĐLĐ_INF (custom)          │ │
│  └────────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────────┘
                              ▲
                              │
              ┌───────────────┴──────────────────┐
              │       Shared HRM Engine          │
              │  - HrmTenantSettingService       │
              │  - ContractTemplateRenderer      │
              │  - EvaluationValidator           │
              │  - CustomFieldManager            │
              │  - ContractFlowEngine            │
              └──────────────────────────────────┘
```

---

## 3. 5 Component Mới

### 3.1 HrmTenantSetting (key-value config)

**Entity**: `HrmTenantSetting` — 1 record per (workspaceId, settingKey).

**Settings có sẵn (HrmSettingKey enum)**:
| Key | Type | Default LSEV | Purpose |
|---|---|---|---|
| PitDependentDeduction | decimal | 6,200,000 | Mức giảm trừ gia cảnh |
| DefaultNationality | string | "Việt Nam" | Quốc tịch mặc định |
| AttendanceBonusEmployee | decimal | 400,000 | Phụ cấp chuyên cần EMPLOYEE |
| AttendanceBonusWorker | decimal | 600,000 | Phụ cấp chuyên cần WORKER |
| CompanyLegalName | string | "" | Tên công ty in HĐ |
| CompanyLegalRepresentative | string | "" | Đại diện pháp luật |
| CompanyTaxCode | string | "" | MST |
| CompanyHeadquartersAddress | string | "" | Địa chỉ trụ sở |

**Service**: `IHrmTenantSettingService` với Redis cache 5 phút.

```csharp
var pit = await settingService.GetDecimalAsync(
    HrmSettingKey.PitDependentDeduction, 6_200_000m, ct);
// Caller pass `pit` vào EmployeeContact.Create() thay vì hardcode
```

**API**: `GET/PUT /api/v1/hrm/tenant-settings` (body: `Dictionary<string,string>`)

**UI**: `/workspace/hr/settings/tenant`

---

### 3.2 ContractTemplate (mẫu HĐ với variable substitution)

**Entity**: `ContractTemplate` — workspace-scoped, có Code (unique), Name, Format (HTML/MARKDOWN), ContentBody.

**Renderer**: `IContractTemplateRenderer` — regex-based engine, format `{{path.to.value}}`.

**Available variables**:
```
{{employee.full_name}}        {{contract.code}}           {{workspace.company_name}}
{{employee.code}}             {{contract.type}}           {{workspace.company_representative}}
{{employee.date_of_birth}}    {{contract.start_date}}     {{workspace.tax_code}}
{{employee.id_card}}          {{contract.end_date}}       {{workspace.address}}
{{employee.address}}          {{contract.basic_salary}}   {{today}}
{{employee.phone}}            {{contract.duration_months}}
{{employee.email}}            {{contract.signed_date}}
{{employee.nationality}}
{{employee.position}}
{{employee.department}}
```

**API**:
- `GET /contract-templates` — list
- `POST /contract-templates` — create
- `PUT /contract-templates/{id}` — update
- `PATCH /contract-templates/{id}/active` — toggle
- `GET /contract-templates/{templateId}/render/{contractId}` — render với data thực

**UI**: `/workspace/hr/settings/contract-templates`
- Variable Picker dropdown chèn `{{key}}` tại vị trí con trỏ
- Textarea cho body
- Form: code, name, format, targetType (HD1-4 optional), defaultDuration, defaultSalaryPercent

---

### 3.3 EvaluationTemplate (mẫu đánh giá configurable)

**Entities**:
- `EvaluationTemplate` — code, name, minPassScore, scaleMax, isDefault, isActive
- `EvaluationTemplateCriterion` — order, name, maxScore, weight, description (1-N với template)

**Validation**: `EvaluateContractCommandHandler` đọc default template, validate `criteria.Count == template.Criteria.Count`. Fallback 7 nếu chưa có template (backward compat).

**API**: `GET/POST /evaluation-templates`

**UI**: `/workspace/hr/settings/evaluation-templates`
- Dynamic add/remove criteria
- Auto-balance weight (100/n)
- Set default per workspace

---

### 3.4 CustomFields + EmployeeDocument

**Entities**:
- `CustomFieldDefinition` — workspace + resource (EMPLOYEE|CONTRACT) + code + labelVi/En + fieldType + options
- `CustomFieldValue` — definitionId + resourceId + value (string)
- `EmployeeDocument` — file metadata (uri, name, mime, expiry, verified)

**Field Types**: TEXT, NUMBER, DATE, ENUM (with optionsJson), BOOLEAN, MULTI_TEXT

**API**:
- `GET /custom-fields/definitions/{resource}` — list defs
- `POST /custom-fields/definitions` — create def
- `GET /custom-fields/values/{resource}/{resourceId}` — list values + def meta
- `PUT /custom-fields/values/{resource}/{resourceId}` — bulk upsert (Dict<code, value>)

**UI**: `/workspace/hr/settings/custom-fields`
- Tabs Employee / Contract
- Definition manager với form: code, label, type, required, displayOrder
- Sẽ render dynamic form trong Employee Detail (todo: tích hợp sau)

---

### 3.5 ContractFlowConfig (state machine JSON)

**Entity**: `ContractFlowConfig` — 1 per workspace, lưu JSON.

**JSON Schema**:
```json
{
  "transitions": [
    { "fromType": "HD1", "onDecision": "PASS", "toType": "HD3" },
    { "fromType": "HD2", "onDecision": "PASS", "toType": "HD3" },
    { "fromType": "HD3", "onDecision": "PASS", "toType": "HD3", "maxOccurrences": 2 },
    { "fromType": "HD3", "onDecision": "PASS", "toType": "HD4", "whenOccurrencesReached": 2 }
  ],
  "autoActivateOnCreate": ["HD3", "HD4"],
  "evaluationRequiredFor": ["HD1", "HD2", "HD3"]
}
```

**Engine**: `IContractFlowEngine.DetermineNextTypeAsync(workspaceId, currentType, decision, occurrenceCount)` — đọc config, fallback LSEV hardcode nếu chưa có.

**API**: `GET/PUT /contract-flow-config`

**UI**: `/workspace/hr/settings/contract-flow`
- JSON editor với validate
- Load default LSEV button
- Schema reference (collapsible)

---

## 4. Database Schema

Migration `20260523040949_AddHrmFlexMultitenantTables` tạo 8 tables:

```
hrm_tenant_settings
├── id, workspace_id, setting_key, setting_value
└── unique(workspace_id, setting_key)

hrm_contract_templates
├── id, workspace_id, code, name, description
├── format, content_body (text)
├── target_contract_type, default_duration_months, default_salary_percent
├── is_active
└── unique(workspace_id, code)

hrm_evaluation_templates
├── id, workspace_id, code, name
├── min_pass_score, scale_max
├── is_default, is_active
└── unique(workspace_id, code)
        │
        └── hrm_evaluation_template_criteria
            ├── id, evaluation_template_id, order
            ├── name, max_score, weight, description
            └── CASCADE DELETE

hrm_custom_field_definitions
├── id, workspace_id, resource (EMPLOYEE|CONTRACT)
├── code, label_vi, label_en
├── field_type, is_required, display_order
├── enum_options_json, default_value, is_active
└── unique(workspace_id, resource, code)

hrm_custom_field_values
├── id, definition_id, resource_id, value
└── unique(definition_id, resource_id)

hrm_employee_documents
├── id, workspace_id, employee_id
├── document_type, file_url, file_name, file_size_bytes, mime_type
├── issue_date, expiry_date, note
└── is_verified, verified_by, verified_at

hrm_contract_flow_configs
├── id, workspace_id, flow_json (jsonb)
└── unique(workspace_id)
```

---

## 5. Flow tích hợp với business hiện có

### Flow 1: Tạo Employee Contact (PIT auto)

```
[FE] Form Add Contact (isPit=true)
   │
   ▼
[BE] AddContactCommandHandler
   │  ① Inject IHrmTenantSettingService
   │  ② pitAmount = settings.GetDecimalAsync(PitDependentDeduction, fallback 6.2M)
   │  ③ EmployeeContact.Create(..., isPit, pitAmount, ...)
   │
   ▼
[Domain] EmployeeContact.PitDeductionAmount = pitAmount nếu isPit
   │
   ▼
[DB] hrm_employee_contacts.pit_deduction_amount = workspace-specific value
```

### Flow 2: Render Contract từ Template

```
[FE] User click "Xem trước HĐ" tại Contract Detail
   │
   ▼
[FE] GET /contract-templates/{templateId}/render/{contractId}
   │
   ▼
[BE] RenderContractFromTemplateQueryHandler
   │  ① Load template (workspace-scoped)
   │  ② Load contract + employee
   │  ③ Build context dict (employee, contract, workspace, today)
   │  ④ renderer.Render(template.ContentBody, context)
   │
   ▼
[BE] ContractTemplateRenderer
   │  Regex: \{\{([\w.]+)\}\}
   │  Resolve "employee.full_name" → context["employee"]["full_name"]
   │
   ▼
[FE] Hiển thị rendered HTML/Markdown
```

### Flow 3: Evaluate Contract → Auto-advance

```
[FE] HR đánh giá HĐ (5/7/10 tiêu chí tùy template)
   │
   ▼
[BE] EvaluateContractCommandHandler
   │  ① Load default EvaluationTemplate
   │  ② Validate cmd.Criteria.Count == template.Criteria.Count
   │  ③ contract.Evaluate(criteria, decision, notes, evaluatedAt, dt)
   │  ④ Nếu PASS → flowEngine.DetermineNextTypeAsync(workspace, currentType, decision, hd3Count)
   │
   ▼
[Engine] ContractFlowEngine
   │  ① Load ContractFlowConfig theo workspaceId
   │  ② Nếu có config → parse JSON, find matching transition
   │  ③ Nếu chưa có → fallback DefaultLsevFlow (HD1→HD3, HD3 sau 2 lần → HD4)
   │
   ▼
[BE] Tạo HĐ mới với nextType, set status=Draft cho HR điền chi tiết
```

---

## 6. Backward Compatibility

| Tình huống | Behavior |
|---|---|
| Workspace LSEV cũ chưa migrate | Tất cả setting GET trả default (giữ nguyên hardcode value) |
| Chưa tạo EvaluationTemplate | Fallback validate đúng 7 tiêu chí |
| Chưa tạo ContractFlowConfig | Fallback DefaultLsevFlow (HD1→HD3→HD3→HD4) |
| Chưa tạo ContractTemplate | Endpoint cũ vẫn dùng RTF tĩnh (GenerateMergedContractFileQueryHandler chưa thay) |
| Chưa định nghĩa CustomFields | Form Employee/Contract render bình thường, không có section custom |

→ **Zero-breaking change.** LSEV vẫn chạy như cũ, công ty mới opt-in qua UI.

---

## 7. Migration Path

### Bước 1: Apply DB migration
```bash
dotnet ef database update --project src/NextX.HRM.Infrastructure --startup-project src/NextX.HRM.Api
```
8 tables mới được tạo.

### Bước 2: Workspace setup
HR mở `/workspace/hr/settings`:
1. **Tenant Settings** → nhập PIT, Nationality, Bonus, Company info
2. **Contract Templates** → tạo 4 mẫu HĐ (copy từ RTF cũ + chèn variables)
3. **Evaluation Templates** → tạo 1 default template với N tiêu chí
4. **Custom Fields** → thêm field công ty cần (BadgeNumber, TaxId, ...)
5. **Contract Flow** → load default LSEV hoặc edit JSON cho flow riêng

### Bước 3: Test end-to-end
Tạo Employee mới → tạo Contact với isPit=true → check PIT amount = setting value (không phải 6.2M cứng nếu đã đổi).

---

## 8. Endpoints Reference

| Module | Method | Path | Mô tả |
|---|---|---|---|
| Tenant | GET | `/api/v1/hrm/tenant-settings` | List settings |
| Tenant | PUT | `/api/v1/hrm/tenant-settings` | Bulk upsert |
| Contract Tpl | GET | `/api/v1/hrm/contract-templates` | List templates |
| Contract Tpl | POST | `/api/v1/hrm/contract-templates` | Create |
| Contract Tpl | PUT | `/api/v1/hrm/contract-templates/{id}` | Update |
| Contract Tpl | PATCH | `/api/v1/hrm/contract-templates/{id}/active` | Toggle |
| Contract Tpl | GET | `/api/v1/hrm/contract-templates/{tplId}/render/{contractId}` | Render |
| Eval Tpl | GET | `/api/v1/hrm/evaluation-templates` | List |
| Eval Tpl | POST | `/api/v1/hrm/evaluation-templates` | Create with criteria |
| Custom Fields | GET | `/api/v1/hrm/custom-fields/definitions/{resource}` | List defs |
| Custom Fields | POST | `/api/v1/hrm/custom-fields/definitions` | Create def |
| Custom Fields | GET | `/api/v1/hrm/custom-fields/values/{resource}/{resourceId}` | List values |
| Custom Fields | PUT | `/api/v1/hrm/custom-fields/values/{resource}/{resourceId}` | Upsert |
| Flow Config | GET | `/api/v1/hrm/contract-flow-config` | Get config |
| Flow Config | PUT | `/api/v1/hrm/contract-flow-config` | Save (validate JSON) |

---

## 9. Out of scope (Phase tiếp theo)

- DOCX/PDF export từ ContractTemplate (chỉ HTML/Markdown)
- File upload UI cho EmployeeDocument (cần MinIO integration)
- Visual flow editor (chỉ có JSON editor — todo: graph editor như n8n)
- Update GenerateMergedContractFileQueryHandler dùng ContractTemplate thay RTF
- Dynamic form render Custom Fields trong Employee Detail tab
- e-signature integration với ContractTemplate
- Versioning + history cho templates
- Audit log thay đổi config

---

*HRM Flex Multitenant Architecture v1.0 · 2026-05-23 · NextX HRM Service*
