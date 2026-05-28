# HRM UAT Flow Guide — Session 2026-05-28

> Tài liệu này hướng dẫn user/tester chạy thử toàn bộ luồng HRM sau khi 16 PRs LSEV được merge.
> Mục tiêu: nắm flow nghiệp vụ end-to-end để bàn giao cho LSEV và các tenant VN khác.
>
> **Phiên bản hệ thống đã deploy**:
> - BE `nextx-hrm` — image build từ `develop`, PR #28 (Import AttendanceRaw) là PR gần nhất.
> - FE `nextx-fe-tenant` — image `b11a78ac…`, PR #467 (Import Center) đã live trên sandbox.
>
> **Môi trường**: `https://sandbox.nextx.vn/vi/workspace/hr`

---

## 1. Tổng kết session — 16 PRs đã làm

### A. Core LSEV engine (5 PRs)
| PR | Repo | Nội dung |
|---|---|---|
| BE #22 | nextx-hrm | Symbol Resolver auto, OT Calculator, 2 PDF stubs (Phiếu lương + Phiếu đối chiếu), 4-signature lifecycle, schema migration `20260527_AddLsevSchema` |
| FE #455 | nextx-fe-tenant | Phiếu đối chiếu UI + Settings setup button + `/me/reconciliation` page |
| FE #458 | nextx-fe-tenant | `HrmDetailedEvaluationForm` — 7 nhóm × 14 sub-criteria (70 điểm), auto-rank Giỏi/Khá/TB/Kém |
| FE #462 | nextx-fe-tenant | Wire form vào `HrmContractsPage` + route `/evaluate-detailed` |
| FE #463 | nextx-fe-tenant | Wire FE form POST → BE `EvaluateContract` endpoint |

### B. Webhook M08 auto-dispatch (3 PRs)
| PR | Nội dung |
|---|---|
| BE #23 | `IWebhookEventDispatcher` + HMAC-SHA256 (`X-NextX-Signature`) + event `hrm.payroll.locked` |
| BE #24 | Auto-dispatch `hrm.employee.created` + `hrm.recruitment.application.hired` |
| BE #25 | Auto-dispatch `hrm.contract.evaluated` + `hrm.payslip.acknowledged` |

→ **5 lifecycle events** tự động bắn webhook (HMAC verify) ra HQ.

### C. Multi-tenant + Industry templates (6 PRs)
| PR | Nội dung |
|---|---|
| BE #26 | Enum `SetupPreset { VnFactory, VnOffice, Custom }` — foundation generic |
| BE #27 | Endpoint generic `POST /preset/{preset}` + Import SalaryRankRate từ Excel |
| BE #28 | Import AttendanceRaw từ máy CC (HikVision/ZKTeco/Ronald Jack) |
| FE #465 | De-LSEV-ify toàn bộ UI — zero LSEV user-facing strings |
| FE #466 | `HrmDetailedEvaluationForm` template-driven (load EvaluationTemplate) |
| FE #467 | Unified Import Center page — drag-drop + result summary |

### D. Industry compliance đạt được
| Tiêu chí | Before | After |
|---|---|---|
| Multi-tenant SaaS | ❌ Hardcoded "LSEV" UI | ✅ Zero LSEV user-facing |
| Industry templates | ❌ 1 preset hardcoded | ✅ 3 presets (Factory/Office/Custom) |
| Evaluation form | ❌ Hardcoded 14 sub-criteria | ✅ Template-driven (workspace tự config) |
| Bulk import | ❌ Chỉ Employees | ✅ Employees + SalaryRank + AttendanceRaw |
| Webhook lifecycle | ❌ Manual trigger | ✅ 5 events auto + HMAC |

---

## 2. Pre-requisites trước khi test

1. **Tài khoản**: HR Admin có quyền `hrm.tenant_settings.edit` + `hrm.attendance.record` + đầy đủ permission HRM.
2. **Workspace**: workspace mới (chưa setup) hoặc workspace test riêng để tránh đụng data thật.
3. **URL**: `https://sandbox.nextx.vn/vi/workspace/hr`
4. **JWT TTL**: 8h (Operator) hoặc 15p (IAM refresh).
5. **File mẫu Excel** để import (tester có thể tải template từ Import Center).

---

## 3. Flow 1 — Khởi tạo workspace bằng Industry Preset

### Mục đích
HR Admin của 1 công ty mới (nhà máy hoặc văn phòng VN) bấm 1 nút seed toàn bộ master data tham khảo: ca làm việc, ký hiệu nghỉ, chức danh, ApprovalFlow, LegalParameter 2026.

### Steps

1. **Vào Settings**: Menu trái → "Cài đặt HRM" → tab "Tổng quan".
2. **Quan sát**: thấy 3 button preset hoặc 1 button "Khởi tạo dữ liệu mẫu (Nhà máy VN)".
   - VnFactory: 7 ca · 30 ký hiệu · 12 chức danh · 3 ApprovalFlow
   - VnOffice: 1 ca · 20 ký hiệu · 10 chức danh · 2 ApprovalFlow
   - Custom: chỉ LegalParameter VN 2026 (không seed master)
3. **Bấm preset phù hợp** → confirm dialog → submit.
4. **Expected result**:
   - Toast "Đã khởi tạo thành công".
   - Response: `{ shiftsAdded, leaveSymbolsAdded, positionsAdded, approvalFlowsAdded, legalParameterAdded }`.
   - Vào tab "Ca làm việc", "Ký hiệu nghỉ", "Chức danh", "ApprovalFlow" → thấy dữ liệu vừa seed.
5. **Idempotent test**: bấm preset 2 lần → lần 2 không tạo duplicate (counts = 0).

### Backend endpoint
```
POST /api/v1/hrm/setup/preset/{VnFactory|VnOffice|Custom}
POST /api/v1/hrm/setup/lsev   (backward-compat alias = VnFactory)
```

---

## 4. Flow 2 — Bulk Import Master Data (Import Center)

### Mục đích
Tenant chuyển từ hệ cũ (Excel/Misa AMIS/in-house) sang NextX HRM — cần import hàng loạt dữ liệu thay vì gõ tay.

### Vị trí
Menu → "Cài đặt HRM" → "Import Center" hoặc trực tiếp URL `/workspace/hr/import-center`.

### Tabs

#### 4.1 Tab "Nhân viên"
- Upload Excel `.xlsx`.
- Template cột: `Code | FullName | Email | Phone | Gender | DateOfBirth | IdentityNumber | OrgUnitCode | PositionCode | EmployeeType | StartDate | …`.
- Result: `{ totalRows, insertedCount, updatedCount, errorCount, errors[] }`.

#### 4.2 Tab "Mức lương theo bậc" ✅ NEW (PR #27)
- Template cột: `SalaryRankCode | EffectiveYear | Amount`.
- VD:
  ```
  WK1, 2026, 5630000
  WK2, 2026, 6200000
  ST1, 2026, 7500000
  ```
- Idempotent: upsert theo (Rank, Year). Re-upload không tạo duplicate.
- Endpoint: `POST /api/v1/hrm/setup/import/salary-rank-rates`.

#### 4.3 Tab "Chấm công" ✅ NEW (PR #28)
- Template cột: `EmployeeCode | ClockedAt | Type`.
- VD (xuất từ máy HikVision):
  ```
  00045, 2026-05-15 07:55:00, IN
  00045, 2026-05-15 12:00:00, OUT
  00045, 2026-05-15 13:00:00, IN
  00045, 2026-05-15 17:05:00, OUT
  ```
- Type chấp nhận: `IN/OUT`, `I/O`, `VÀO/RA`.
- Idempotent: gộp nhiều clock cùng (NV, ngày) vào 1 `AttendanceRaw`, skip dòng trùng giờ + type.
- Default timezone: GMT+7 nếu Excel không có offset.
- Endpoint: `POST /api/v1/hrm/setup/import/attendance-raw`.

#### 4.4 Tab "Ngày lễ" (disabled — coming soon)

### Steps test
1. Vào Import Center → chọn tab "Mức lương theo bậc".
2. Bấm "Tải template" → mở file Excel → điền 3-5 dòng.
3. Drag-drop file vào vùng upload.
4. Bấm "Import".
5. Expected: card hiển thị summary `Thành công: 5 / Lỗi: 0` + danh sách `errors[]` nếu có.
6. Lặp lại với tab "Chấm công" — file 10 dòng chấm công 1 NV trong 3 ngày → expected gộp thành 3 raws.

---

## 5. Flow 3 — Quản lý Nhân viên (hồ sơ)

### Vị trí
Menu → "Nhân viên" → `/workspace/hr/employees`.

### Steps
1. **Tạo NV mới** (manual):
   - Bấm "Thêm nhân viên" → form đầy đủ: Code, FullName, Email, CCCD (unique), DOB, Gender, OrgUnit, Position, EmployeeType (Employee/Worker), StartDate.
   - Submit → Toast "Đã tạo".
   - Webhook auto-fire: `hrm.employee.created` (nếu workspace đã cấu hình outbound webhook).
2. **Edit NV**: bấm hàng → tab "Thông tin cơ bản" / "Hợp đồng" / "Lương" / "BHXH" / "Người liên hệ KC".
3. **Lịch sử**: bấm tab "Lịch sử" → thấy 4 loại:
   - History phòng ban
   - History chức danh
   - History lương cơ bản
   - History phụ cấp
   - Overlap detection: thêm 1 dòng overlap → expected lỗi 409.
4. **Search + Filter**: search theo Code/Name, filter theo OrgUnit/Position/Status.

### Endpoint
```
POST   /api/v1/hrm/employees
GET    /api/v1/hrm/employees?page=1&pageSize=20&search=
PUT    /api/v1/hrm/employees/{id}
```

---

## 6. Flow 4 — Hợp đồng & Đánh giá thử việc

### Vị trí
Menu → "Hợp đồng" → `/workspace/hr/contracts`.

### Steps
1. **Tạo HĐ thử việc HD1** cho NV vừa tạo:
   - Loại: HD1 (thử việc).
   - Lương thử việc: `BasicSalaryProbation` (85% lương chính thức theo luật VN).
   - Start/End date, vị trí, template song ngữ.
2. **Khi gần đến hạn HD1** → button "Chi tiết" (đổi tên từ "LSEV") xuất hiện trên hàng HĐ.
3. **Bấm "Chi tiết"** → route `/contracts/{id}/evaluate-detailed`:
   - Form **7 nhóm × 14 sub-criteria** = tổng 70 điểm.
   - Mỗi sub có điểm tối đa (6+4=10, 5+5=10, 5+5=10, 6+4=10, 5+5=10, 3+4+3=10, 6+4=10).
   - Auto-compute **Tổng điểm** + **Xếp loại** (Giỏi ≥56, Khá ≥42, TB ≥28, Kém <28).
   - Nếu workspace có `EvaluationTemplate` custom → form load template thay default.
4. **Chọn quyết định**: `sign_fixed_term` (Pass) / `renew_fixed_term` (Extend) / `sign_undefined` (Extend) / `reject` (Fail).
5. **Submit** → BE `POST /api/v1/hrm/contracts/{id}/evaluate`:
   - Notes format: `Tổng điểm: 62/70 (Giỏi) — Template: ABC — Thời hạn: 12 tháng — Comment`.
   - Webhook auto-fire: `hrm.contract.evaluated`.
6. **Tự động tạo HĐ tiếp** (HD2 hoặc HD3) theo decision.

---

## 7. Flow 5 — Chấm công

### 7.1 Setup tháng
1. Menu → "Chấm công" → "Setup tháng".
2. Chọn năm + tháng (VD: 2026-05) → bấm "Tạo".
3. System tạo `AttendanceMonth` skeleton cho tất cả NV active.

### 7.2 Symbol Resolver (auto) — từ PR #22
- Khi `AttendanceRaw` được tạo (manual hoặc import) → system tự resolve ký hiệu chấm công:
  - X = công thường đủ giờ
  - X1/2 = công nửa ngày
  - V = nghỉ phép
  - L = nghỉ lễ
  - O = nghỉ không lương
  - …và 25+ symbol khác theo cấu hình.

### 7.3 OT Calculator — từ PR #22
- Symbol resolver detect OT từ `ClockRecord` vượt ngoài ShiftDefinition.
- Auto-classify: OT thường (150%) / OT cuối tuần (200%) / OT lễ (300%) / OT đêm (130%).

### 7.4 Import AttendanceRaw từ máy CC (PR #28) — xem Flow 2.3
- Upload Excel xuất từ máy → system gộp + classify auto.

### 7.5 Sửa chấm công thủ công
- Bấm vào cell ngày của NV → modal sửa ClockRecord hoặc đổi Symbol.
- Audit log lưu lại user nào sửa lúc nào.

---

## 8. Flow 6 — Tính lương + 4-signature lifecycle

### Vị trí
Menu → "Tiền lương" → `/workspace/hr/payroll`.

### Steps
1. **Tạo tháng lương** từ `AttendanceMonth` đã chốt.
2. **Tính lương**: bấm "Tính lương" → system compute:
   - Lương cơ bản theo SalaryRankRate × số công.
   - Phụ cấp ăn / xăng / điện thoại / trách nhiệm.
   - Khấu trừ BHXH (8%) + BHYT (1.5%) + BHTN (1%) + Thuế TNCN.
   - Lương probation = `BasicSalaryProbation` (HD1).
3. **4-Signature lifecycle** (PR #22):
   - **Step 1 — HR Lập** (`PreparedAt` + `PreparedBy`): bấm "Hoàn thành lập bảng".
   - **Step 2 — TBP Duyệt** (`DeptApprovedAt`): trưởng bộ phận login → bấm duyệt.
   - **Step 3 — KT Duyệt** (`AccountingApprovedAt`): kế toán login → bấm duyệt.
   - **Step 4 — GĐ Duyệt** (`DirectorApprovedAt`): giám đốc login → bấm "Khoá tháng lương".
4. **Lock**: sau Step 4, `PayrollMonth.IsLocked = true`:
   - Không sửa được record.
   - Webhook auto-fire: `hrm.payroll.locked` (HMAC-SHA256 header `X-NextX-Signature`).

### Expected
- 4 timestamp + 4 user signature lưu DB.
- Audit trail đầy đủ.
- Webhook receiver bên LS Mtron HQ verify HMAC OK.

---

## 9. Flow 7 — Phiếu lương PDF (Payslip)

### Mục đích
HR phát phiếu lương cho NV — file PDF có chữ ký 4 cấp.

### Steps HR
1. Vào "Tiền lương" → chọn tháng đã lock.
2. Bấm "Xuất phiếu lương" trên hàng NV → tải PDF.
3. PDF gồm:
   - Header: Logo tenant + tên công ty + tháng/năm.
   - Bảng chi tiết: Mã NV, tên, chức danh, OrgUnit.
   - Cột thu nhập: Lương CB, phụ cấp, thưởng, OT.
   - Cột khấu trừ: BHXH, BHYT, BHTN, Thuế TNCN, Tạm ứng.
   - Thực lĩnh.
   - 4 chữ ký: Người lập / TBP / KT / GĐ + ngày ký.

### Steps NLĐ
1. NV login → vào `/me/payslip` → xem danh sách phiếu lương.
2. Bấm "Xác nhận đã nhận" → BE record `Acknowledged = true`.
3. Webhook auto-fire: `hrm.payslip.acknowledged`.

### Endpoint
```
GET /api/v1/hrm/payslips/{id}/pdf
POST /api/v1/hrm/payslips/{id}/acknowledge
```

---

## 10. Flow 8 — Phiếu Đối Chiếu Công (Reconciliation)

### Mục đích
NLĐ tự đối chiếu chấm công tháng trước (LSEV requirement). HR gửi PDF; NLĐ phải confirm trước **09:00 ngày 02 tháng sau**.

### Steps HR
1. Vào "Chấm công" → tháng đã chốt → bấm "Xuất phiếu đối chiếu hàng loạt".
2. System generate 1 PDF/NV gồm: Bảng công đầy đủ 31 ngày + 11 fields summary (TotalDays, RegularDays, OTHours, LeaveDays, …).

### Steps NLĐ (self-service)
1. NV login → menu "Của tôi" → "Phiếu đối chiếu công".
2. Page `/me/reconciliation`:
   - Month selector (mặc định tháng trước).
   - **Countdown deadline**: hiển thị "Còn X giờ Y phút" đến 09:00 ngày 02.
   - Bấm "Tải PDF" → xem.
   - Bấm "Xác nhận đồng ý" hoặc "Có khiếu nại" + nhập note.
3. Sau deadline → button confirm khoá → mặc định = OK.

---

## 11. Flow 9 — Webhook outbound (M08)

### Mục đích
LSEV/HQ muốn nhận realtime event từ NextX HRM về hệ ERP của họ.

### Setup
1. Vào Settings → "Webhooks" → "Thêm endpoint":
   - URL: `https://hq.lsev.com/webhook/nextx`
   - Secret: random string (dùng cho HMAC).
   - Events subscribe: chọn 5 events.
2. Submit → status `Active`.

### 5 Events auto-dispatch
| Event | Trigger | Payload chính |
|---|---|---|
| `hrm.employee.created` | Tạo NV | `{ employeeId, code, fullName, workspaceId, createdAt }` |
| `hrm.recruitment.application.hired` | Hire applicant | `{ applicationId, employeeId, hiredAt }` |
| `hrm.contract.evaluated` | Evaluate HĐ | `{ contractId, employeeId, decision, totalScore, ranking }` |
| `hrm.payroll.locked` | Lock PayrollMonth | `{ payrollMonthId, year, month, lockedBy, lockedAt }` |
| `hrm.payslip.acknowledged` | NLĐ confirm payslip | `{ payslipId, employeeId, acknowledgedAt }` |

### HMAC Signature
- Header: `X-NextX-Signature: sha256=<hex>`.
- Compute: `HMAC-SHA256(secret, raw_body)`.
- Receiver verify trước khi process.

### Test
1. Setup ngrok hoặc webhook.site → lấy URL test.
2. Đăng ký webhook trên Settings với URL đó.
3. Tạo 1 NV → kiểm tra webhook receiver nhận đúng event + verify HMAC khớp.

---

## 12. Smoke Test Checklist (cho LSEV UAT)

| # | Hành động | Expected |
|---|---|---|
| 1 | Login HR Admin | Vào được /workspace/hr |
| 2 | Setup preset "VnFactory" | 53 records seed thành công |
| 3 | Setup preset lần 2 | Idempotent, không tạo duplicate |
| 4 | Import 5 NV từ Excel | 5 inserted, 0 errors |
| 5 | Import SalaryRankRate (3 ranks × 2 năm) | 6 inserted |
| 6 | Import 30 dòng AttendanceRaw (3 NV × 10 ngày) | 30 imported, group thành 30 raws |
| 7 | Tạo HD1 cho 1 NV | Webhook `hrm.employee.created` fire (nếu đã setup) |
| 8 | Bấm "Chi tiết" HĐ → đánh giá 70/70 | Total = 70, Ranking = "Giỏi" |
| 9 | Submit evaluation → decision Pass | Webhook `hrm.contract.evaluated` fire |
| 10 | Setup tháng 2026-05 | Tạo skeleton AttendanceMonth |
| 11 | Tính lương tháng 2026-05 | PayrollMonth records compute đủ |
| 12 | 4-signature đầy đủ → Lock | `IsLocked=true`, webhook `hrm.payroll.locked` fire |
| 13 | Tải Phiếu lương PDF | PDF có đủ 4 chữ ký |
| 14 | NLĐ xem Phiếu đối chiếu `/me/reconciliation` | Countdown deadline hiển thị đúng |
| 15 | NLĐ confirm payslip | Webhook `hrm.payslip.acknowledged` fire |

---

## 13. Roadmap còn lại (lower priority, không block UAT)

- **HikVision FaceID ISAPI listener** — realtime push (workaround: Import Excel đã đủ ✅).
- **Onboarding Wizard 7-step** persistent progress + sample data mode.
- **HrmIndustryTemplate** entity (replace enum, marketplace share).
- **Import Holidays** + Import History audit page.
- **RLS OrgUnit filter** cho TBP scope.
- **C46 BHXH report** (defer — C45 cover most cases).

---

## 14. Known limitations

1. **Webhook retry**: chưa có exponential backoff — fail là fail (manual re-trigger).
2. **OT Calculator**: chưa hỗ trợ ca xoay 3-shift (đang dùng default 2-shift).
3. **PDF font**: dùng font default QuestPDF — tiếng Việt OK nhưng chưa custom theme tenant.
4. **AttendanceRaw timezone**: default GMT+7 — nếu Excel có offset thì dùng offset đó.
5. **EvaluationTemplate seed**: workspace mới chưa có template thì fallback default 14 sub-criteria.

---

## 15. Liên hệ

| Vấn đề | Channel |
|---|---|
| Bug UAT | Tạo issue trong repo `nextx-hrm` hoặc `nextx-fe-tenant` |
| Câu hỏi nghiệp vụ | BA NextX |
| Webhook receiver | LS Mtron HQ — verify HMAC theo doc trên |
| Sandbox down | DevOps NextX |

---

*Tài liệu UAT — NextX HRM · session 2026-05-28 · 16 PRs merged*
