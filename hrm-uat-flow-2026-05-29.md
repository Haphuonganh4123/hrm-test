# HRM UAT Flow Guide v3 — Session 2026-05-29

> Tài liệu này thay thế v2 (`hrm-uat-flow-2026-05-28.md`). v3 reflect chính xác sidebar tiếng Việt thực tế đang ship + bổ sung toàn bộ flow Employee và Attendance còn thiếu.
>
> **Builds đã deploy**: 26 PRs LSEV merged (15 wave 1 + 11 wave 2-6 trong continuation 2026-05-29) — gồm cả Onboarding Wizard, HrmIndustryTemplate entity, HikVision listener, BHXH C46, RLS OrgUnit, Reconciliation ZIP, Holidays import.
>
> **Môi trường UAT**: `https://sandbox.nextx.vn/vi/workspace/hr` hoặc `app-dev.nextx.vn/vi/workspace/hr`.

---

## 1. Sidebar tiếng Việt — KHỚP THỰC TẾ

Sidebar HRM thực tế nhóm theo 10 group (xem `src/messages/vi/hr.json` → `layout.workspaceSidebar.hr`):

| Group | Items | Route |
|---|---|---|
| **Tổng quan** | Dashboard · Trang cá nhân · Hộp duyệt · Lịch sự kiện | `/`, `/me`, `/inbox`, `/calendar` |
| **Cá nhân** | Thông tin liên hệ · Tăng ca · Bổ sung lương · Khóa học của tôi · Phản hồi · Báo tai nạn | `/me`, `/me/overtime`, `/me/salary-supplement`, `/me/training`, `/me/feedback`, `/me/health-accident` |
| **Hồ sơ nhân viên** | Nhân viên trong công ty · Phòng ban & Tổ chức · Tuyển dụng · Onboarding · Hiệu suất (OKR/Review) · Dự án | `/employees`, `/org-chart`, `/recruitment`, `/onboarding`, `/performance` |
| **Nghỉ phép** | Quản lý nghỉ phép · Cấu hình ký hiệu nghỉ phép · Báo cáo & Thống kê phép · Chuyển phép tồn · Chính sách nghỉ phép | `/leave`, `/leave-config`, `/leave-report`, `/leave-carry`, `/leave-policy` |
| **Chấm công** | Bảng công · Hành trình · Thống kê công · Bảng tổng hợp · Cấu hình chấm công · Ca làm việc | `/attendance` (5 tabs), `/attendance-summary`, `/shift` |
| **Tiền lương** | Tổ hợp bảng lương · Tính lương tháng · Đăng ký bổ sung · Thống kê · Thiết lập · Chính sách lương | `/payroll`, `/payroll-processing`, `/payroll-supplement`, `/payroll-policy` |
| **Đào tạo** | Chương trình đào tạo · Nhân viên tham gia · Đánh giá sau đào tạo | `/training`, `/training-employees`, `/training-evaluation` |
| **Phúc lợi** | Y tế & An toàn · Sự cố lao động · Bảo hiểm BHXH · Hợp đồng lao động | `/health-accident`, `/insurance`, `/contracts` (hoặc `/contract`) |
| **Báo cáo** | Báo cáo tổng quan · Báo cáo lao động · Báo cáo BHXH | `/reports`, `/reports-labor`, `/reports-bhxh` |
| **Cài đặt** | Cài đặt chung (toàn bộ danh mục) · Tham số công ty (PIT, lương, lễ) · Mẫu HĐ · Mẫu đánh giá · Trường tùy chỉnh · Luồng HĐ · Luồng phê duyệt · Phân quyền · Webhooks · API Tokens · **Onboarding Wizard** ✨ | `/settings`, `/settings/tenant`, `/approval-flows`, `/roles`, `/webhooks`, `/api-tokens`, `/onboarding-wizard` |

> ⚠️ **Routes alias**: `/staff` redirect → `/employees`. `/staff/[id]` mở chi tiết NV. `/contract` và `/contracts` đều tới Contracts hub. `/settings`, `/org-chart`, `/shift`, `/leave-config` là các "view" khác nhau của cùng 1 page `HrmMasterDataPage` với tab được lock khác nhau.

---

## 2. Pre-requisites

1. **Tài khoản HR Admin** đủ permission: `hrm.employees.*`, `hrm.attendance.*`, `hrm.leave.*`, `hrm.payroll.*`, `hrm.contracts.*`, `hrm.config.edit`, `hrm.tenant_settings.edit`, `hrm.webhooks.manage`, `hrm.masterdata.edit`, `hrm.onboarding.*`.
2. **Workspace test riêng**.
3. URL UAT: `app-dev.nextx.vn/vi/workspace/hr`.

---

## 3. Flow 0 — Onboarding Wizard (✨ MỚI — bắt đầu từ đây cho workspace mới)

### Vị trí
Sidebar **Cài đặt → Onboarding Wizard** → `/workspace/hr/onboarding-wizard`. Page: `HrmOnboardingWizardPage`.

### 7 bước (BE định nghĩa, FE render theo Order)
| # | Step | Mô tả | Hành động ở đâu |
|---|---|---|---|
| 1 | Thông tin công ty | Tên, MST, địa chỉ, đại diện pháp luật | Cài đặt → Tham số công ty |
| 2 | Chọn industry preset | VnFactory / VnOffice / Custom | Cài đặt → Tham số công ty (card "Khởi tạo dữ liệu mẫu") |
| 3 | Xem lại master data | Ca làm việc · Ký hiệu nghỉ · Chức danh | Cài đặt → Cài đặt chung |
| 4 | Cấu hình luồng phê duyệt | ApprovalFlow nghỉ phép / OT / lương | Cài đặt → Luồng phê duyệt |
| 5 | Nhập NV đầu tiên | Import Excel 5-10 NV | Cài đặt → Import Center (tab Nhân viên) |
| 6 | Tạo HĐLĐ mẫu | HD1 thử việc cho 1 NV để test | Phúc lợi → Hợp đồng lao động |
| 7 | Hoàn tất | Quick tour 4 menu chính | Bấm "Hoàn tất ✓" |

### Behavior
- Progress lưu trong `tenant_settings` (`OnboardingCurrentStep`, `OnboardingCompletedAt`).
- Mỗi step active có CTA Link mở trang tương ứng + button "Đã làm xong" để advance.
- Có "← Lùi 1 bước" và "Reset wizard".
- Step 7 dispatch `complete: true` → BE ghi `CompletedAt` → banner xanh hiển thị thời gian.

### Endpoint
```
GET  /api/v1/hrm/onboarding/progress
POST /api/v1/hrm/onboarding/progress { step, complete }
```

---

## 4. Flow 1 — Industry Preset (chi tiết)

### Vị trí
Sidebar **Cài đặt → Tham số công ty (PIT, lương, lễ)** → `/workspace/hr/settings/tenant`. Page: `HrmTenantSettingsPage`.

### Setup form trên đầu trang
6 trường tham số kế toán/HR:
- Giảm trừ gia cảnh (VNĐ/người phụ thuộc) — default 6,200,000
- Quốc tịch mặc định — default "Việt Nam"
- Tên công ty (in trên HĐ)
- Đại diện pháp luật
- Mã số thuế
- Địa chỉ trụ sở chính

Bấm "Lưu cấu hình" → ghi vào `tenant_settings`.

### Card "Khởi tạo dữ liệu mẫu (theo Industry)"
3 PresetCard side-by-side (Wave 2 PR #481):
- **Nhà máy VN (Factory)** *(Gợi ý)* — 7 ca · 30 ký hiệu nghỉ · 12 chức danh · 3 ApprovalFlow
- **Văn phòng VN (Office)** — 1 ca · 20 ký hiệu nghỉ · 10 chức danh · 2 ApprovalFlow
- **Tự cấu hình (Custom)** — chỉ seed LegalParameter VN 2026

Mỗi card có tooltip hover (Wave 4 PR #494) giải thích chi tiết. Bấm "Khởi tạo" → toast + card xanh summary số records vừa seed.

Idempotent — bấm lần 2 counts = 0.

### Endpoint
```
POST /api/v1/hrm/setup/preset/{VnFactory|VnOffice|Custom}
POST /api/v1/hrm/setup/lsev   (BC alias = VnFactory)
GET  /api/v1/hrm/industry-templates   (Wave 6 — list templates marketplace)
```

---

## 5. Flow 2 — Import Center (4 tabs)

### Vị trí
Direct URL `/workspace/hr/import-center` (chưa có trong sidebar mặc định — tester gõ URL hoặc click từ Onboarding Wizard step 5).

### Tabs
| Tab | Cột Excel | Endpoint | Idempotent |
|---|---|---|---|
| Nhân viên | Code · FullName · Email · Phone · Gender · DOB · CCCD · OrgUnitCode · PositionCode · EmployeeType · StartDate · … | `POST /employees/import` | Upsert by Code |
| Mức lương theo bậc | SalaryRankCode · EffectiveYear · Amount | `POST /setup/import/salary-rank-rates` | Upsert (Rank, Year) |
| Chấm công | EmployeeCode · ClockedAt · Type (IN/OUT/VÀO/RA/0/1) | `POST /setup/import/attendance-raw` | Skip dòng trùng (giờ, type) |
| **Ngày lễ** ✨ | Date · Name · IsOfficial | `POST /setup/import/holidays?replace={bool}` | Merge mặc định; Replace nếu tick |

Mỗi tab: drag-drop upload → result card hiển thị `totalRows / inserted / updated / skipped / errorCount` + list errors max 100.

---

## 6. Flow 3 — Quản lý Nhân viên (DEEP DIVE — yêu cầu của user)

### 6.1 Trang danh sách
Sidebar **Hồ sơ nhân viên → Nhân viên trong công ty** → `/employees`. Page: `HrmEmployeesPage`.

#### 5 tabs lọc nhanh
- **Tất cả** (có count) — toàn bộ NV active + probation + trainee + terminated
- **Đang làm** — Status = Active
- **Thử việc** — Status = Probation
- **Học việc** — Status = Trainee
- **Đã nghỉ** — Status = Terminated

#### Toolbar
- Search box (Code, FullName, Email)
- Filter: OrgUnit · Position · EmployeeType (Employee/Worker)
- Sort: Code / FullName / JoinDate (asc/desc)
- Button **"Thêm nhân viên"** — primary
- Button **"Export Excel"** — tải `.xlsx` toàn bộ NV theo filter hiện tại
- Button **"Import Excel"** — link tới `/import-center`

#### Bảng cột
| STT | Mã NV | Họ tên | DOB | Phone | Email | Phòng ban | Chức danh | Loại | Ngày vào | Hết thử việc | Trạng thái |

Click row → mở `/staff/{id}` chi tiết. Mỗi NV có nút action ẩn: Edit · Terminate · Delete.

### 6.2 Trang chi tiết — `/staff/{id}` (8 tabs)
Page: `HrmEmployeeDetailPage`. Header gồm avatar · code · fullName · position · orgUnit · status pill · button "Sửa" · banner IAM provision (nếu chưa link user).

#### Tab 1 — Thông tin cá nhân (Personal)
- DOB · Gender · Place of birth · CCCD (số + ngày cấp + nơi cấp) · Quốc tịch · Dân tộc · Tôn giáo · Tình trạng hôn nhân
- Địa chỉ thường trú · Tạm trú
- Học vấn · Chuyên ngành · Ngoại ngữ
- Phone · Email · Bank account · Bank name
- Section "Cập nhật" — form sửa, validate CCCD unique workspace-scope (return 409 nếu trùng).

#### Tab 2 — Công việc (Job)
- OrgUnit hiện tại + Position hiện tại
- ProbationEndDate · JoinDate
- **Lịch sử công việc (Job History)**: bảng OrgUnit + Position + Notes + FromDate + ToDate. Button "Thêm dòng" mở dialog input.
  - Overlap detection: thêm dòng trùng kỳ → 409.
- Endpoint: `GET/POST /employees/{id}/job-history`.

#### Tab 3 — Lương (Salary)
- Lương cơ bản hiện tại · BasicSalaryProbation (85% khi probation)
- Phụ cấp hiện tại (chuyên cần, ăn, xăng, điện thoại, trách nhiệm)
- **Lịch sử lương cơ bản (Salary History)** — bảng Amount + EffectiveDate + Reason + DecisionNumber.
- **Lịch sử phụ cấp (Allowance History)** — bảng AllowanceType + Amount + EffectiveDate + Reason.
- Endpoint: `GET/POST /employees/{id}/salary-history`, `GET/POST /employees/{id}/allowance-history`.

#### Tab 4 — Bảo hiểm (Insurance)
- Mã BHXH (SocialInsuranceNumber)
- Mức lương đóng BH hiện tại
- **Lịch sử lương BH** — bảng InsuranceSalary + EffectiveDate + Reason + DecisionNumber.
- Dùng làm input cho báo cáo C45 / C46 / D02 / TK1.
- Endpoint: `GET/POST /employees/{id}/insurance-history`.

#### Tab 5 — Đánh giá KPI (Evaluations)
- **Lịch sử đánh giá KPI** — bảng Period + Score + Ranking + Notes.
- Button "Thêm đánh giá" — chọn template + nhập điểm + comment.
- Endpoint: `GET/POST /employees/{id}/evaluations`.

#### Tab 6 — Liên hệ & NPT (Contacts)
- Người thân (Family) + Người liên hệ khẩn cấp (Emergency)
- Người phụ thuộc thuế (Dependents) — số NPT này feed vào tính TNCN.
- Cột: Quan hệ · Họ tên · DOB · CCCD · Phone · IsDependent · NPT effective dates.
- Endpoint: `GET/POST/PUT/DELETE /employees/{id}/contacts/{contactId?}`.

#### Tab 7 — Đào tạo (Training)
- **Lịch sử đào tạo** — Course + Provider + StartDate + EndDate + Score + Certificate.
- Endpoint: `GET/POST /employees/{id}/training`.

#### Tab 8 — Checklist thôi việc (chỉ hiện khi Status = TERMINATED)
- 10+ mục checklist (thu hồi thiết bị, hoàn tất bàn giao, xóa quyền IAM, payslip cuối, …)
- Button "Update" từng item.
- Endpoint: `GET/PUT /employees/{id}/termination-checklist`.

### 6.3 Tạo NV mới (Add Employee dialog)
Form sections:
- Personal: FullName · Email · CCCD · Phone · DOB · Gender
- Job: OrgUnit · Position · EmployeeType (Employee/Worker) · JoinDate · ProbationEndDate
- Salary: BasicSalary
- Auto-generate Code nếu không truyền (theo pattern `00001`, `00002`...)
- Helper note: "Hồ sơ nhân viên sẽ được tạo ở trạng thái Probation. Bạn có thể bổ sung thông tin chi tiết ngay sau khi tạo."

Submit → BE `POST /employees` → webhook **`hrm.employee.created`** fire (Wave 1 PR #24).

### 6.4 Sửa NV (Edit) — `PUT /employees/{id}`
- Update full profile.
- Webhook **`hrm.employee.updated`** fire (Wave 5 PR #30).
- Auto-link IAM user nếu email vừa thay đổi và Employee chưa có UserId.

### 6.5 Cho nghỉ việc (Terminate)
- Button "Thôi việc" → dialog input: TerminationDate · Reason · DecisionNumber.
- BE `POST /employees/{id}/terminate`:
  - `Status = Terminated`
  - Tự tạo `TerminationChecklist` (10+ items)
  - Webhook **`hrm.employee.terminated`** fire (Wave 5 PR #30)

### 6.6 IAM link/unlink
- Banner trên đầu page detail: "NV này chưa có tài khoản đăng nhập" nếu UserId = null.
- Button:
  - **"Link IAM User"** — nhập email user IAM → match → link `Employee.UserId`.
  - **"Provision IAM"** — tạo IAM user mới từ email NV + invite link.
  - **"Unlink"** — xóa link (giữ NV nhưng mất quyền login).
- Endpoint: `POST /employees/{id}/link-user`, `/unlink-user`, `/provision-iam`.

### 6.7 Documents (tab riêng — `EmployeeDocumentsController`)
Quản lý CV, HĐLĐ scan, chứng chỉ, ảnh CCCD, … qua MinIO storage.
- `GET /employee-documents?employeeId={id}`
- `POST /employee-documents` (upload)
- `GET /employee-documents/{id}/download-url` (presigned URL)
- `DELETE /employee-documents/{id}`

### 6.8 Self-service "Trang cá nhân" — `/me` (HrmMyPortalPage)
4 sub-tab:
- **Tổng quan** — KPI cards (phép còn lại, đơn chờ, lương tháng, công tháng)
- **Đơn của tôi** — list yêu cầu nghỉ/OT/bổ sung lương + tiến trình duyệt
- **Phiếu lương** — list payslip + pill trạng thái xác nhận (Wave 2 PR #481) + button download PDF + pagination 12/page (Wave 4 PR #494)
- **Tài liệu** — payslip PDF, HĐLĐ, chứng chỉ

Self-service form khác (sidebar **Cá nhân**):
- `/me/overtime` — đăng ký tăng ca
- `/me/salary-supplement` — đề xuất bổ sung lương
- `/me/feedback` — gửi phản hồi
- `/me/health-accident` — báo tai nạn lao động
- `/me/training` — khóa học của tôi
- `/me/reconciliation` — phiếu đối chiếu công self-service (Wave 1)

---

## 7. Flow 4 — Tổ chức (Org Chart) + Master Data

### Vị trí
Sidebar **Hồ sơ nhân viên → Phòng ban & Tổ chức** → `/org-chart`. Sidebar **Cài đặt → Cài đặt chung** → `/settings`. **Cùng 1 page** `HrmMasterDataPage` chỉ khác tab default.

### 6 tabs
| Tab | Mô tả | Lock route |
|---|---|---|
| **Phòng ban & Cơ cấu tổ chức** | Cây OrgUnit 6 cấp · drag-drop reorder · cost center · manager assign. Có view "Sơ đồ" + "Danh sách". | `/org-chart` |
| **Chức danh nhân viên** | Position CRUD · Code · Name · Salary group · Attendance bonus | `/settings/positions` |
| **Cấu hình Ca làm việc** | Shift CRUD · start/end · break · night-hours · standard hours | `/shift` |
| **Cấu hình Nghỉ phép** | LeaveSymbol CRUD · code (X/V/P/O/...) · paid % · counts as workday | `/leave-config` |
| **Ngạch lương** | SalaryRank CRUD + import từ Import Center | — |
| **Định mức lương** | LegalParameter · BHXH/BHYT/BHTN rate · giảm trừ TNCN · tier thuế | — |

### Endpoint
- `GET/POST/PUT/DELETE /master-data/org-units`
- `GET/POST/PUT/DELETE /master-data/positions`
- `GET/POST/PUT/DELETE /master-data/shifts`
- `GET/POST/PUT/DELETE /master-data/leave-symbols`
- `GET/POST/PUT/DELETE /master-data/salary-ranks`
- `GET/PUT /master-data/legal-parameters`

---

## 8. Flow 5 — Chấm công (DEEP DIVE — yêu cầu của user)

### 8.1 Setup tháng — Bước đầu tiên mỗi tháng
Sidebar **Chấm công → Bảng công** → `/attendance` → tab **"Cấu hình chấm công"** (5th tab).

Tab này gồm:
- Chọn năm + tháng
- Hiển thị `StandardWorkdays` (mặc định Auto = số ngày trong tháng - Chủ Nhật, hoặc Manual)
- `StandardWorkHours` per ngày (mặc định 8)
- Grid 31 ngày — mỗi ngày có flag: WeeklyOff · Holiday · CustomNote
- Buttons:
  - **"Tạo từ template năm"** — nếu workspace đã setup YearTemplate, generate tự động
  - **"Sao chép từ tháng trước"** — clone month setup từ src month
  - **"Lock tháng"** — sau khi chốt (không cho sửa)
  - **"Mở khóa"** — reopen tháng đã lock (audit log)
  - **"Xem audit"** — list lịch sử thay đổi setup

Endpoint:
- `POST /attendance/setup/{year}/{month}` (init)
- `GET /attendance/setup/{year}/{month}`
- `POST /attendance/setup/{year}/{month}/lock`
- `POST /attendance/setup/{year}/{month}/unlock`
- `POST /attendance/setup/{year}/{month}/copy-from/{srcYear}/{srcMonth}`
- `PATCH /attendance/setup/{year}/{month}/days` (bulk update days)
- `POST /attendance/setup/{year}/{month}/from-template`
- `GET /attendance/setup/{year}/{month}/audit`

### 8.2 Year Template (1 lần đầu năm)
Sidebar **Chấm công → Bảng công** → `/attendance/year-template`. Page sub-route.
- Cấu hình mặc định cho cả năm: ngày trong tuần nào nghỉ (Sat/Sun), list ngày lễ year, StandardWorkdayType (Type1=Auto / Type2=Manual).
- 1 workspace có tối đa 1 template/năm.
- DefaultHolidays update qua Import Center tab Ngày lễ (Wave 5 PR #31).
- Endpoint: `GET/PUT /attendance/year-template`.

### 8.3 Gán ca làm việc — `Shift Assignment`
Sidebar **Chấm công → Bảng công** → `/attendance` → tab... (không có riêng — qua bulk assign dialog).
- Mở dialog "Bulk assign shift": chọn NV (multi-select) · range ngày · shift code (CA1/CA2/CA3/HC).
- Endpoint: `GET /attendance/shifts/{year}/{month}` (list), `POST /attendance/shifts/{year}/{month}` (bulk assign).

### 8.4 Tab "Chấm công thô" (Raw)
Hiển thị `/attendance` → tab **"Chấm công thô"** (3rd tab).
- Bảng: NV · Ngày · ClockRecords (list IN/OUT giờ) · ResolvedSymbol · Source (Machine/Manual/Import).
- Filter: date · employee · orgUnit.
- Button "Thêm thủ công" → modal input clock record (chọn NV + giờ + type IN/OUT).
- Hỗ trợ chỉnh sửa từng dòng (audit log lưu user nào sửa).

#### Cách system tạo AttendanceRaw
| Nguồn | Endpoint | Auth | Webhook fire |
|---|---|---|---|
| **Thủ công** (HR sửa) | `POST /attendance/raw` | JWT HR | `hrm.attendance.checked_in` (Wave 5) trên IN đầu ngày |
| **ZKTeco/Virdi máy push** | `POST /attendance/iclock/cdata?SN=...` text/plain | X-Device-Serial | — (push raw không fire webhook) |
| **HikVision FaceID push** ✨ | `POST /attendance/hikvision/event` JSON | X-Device-Serial | — |
| **Import Excel từ máy** | `POST /setup/import/attendance-raw` | JWT HR | — |

#### Symbol Resolver (auto sau insert raw)
Khi `AttendanceRaw` có ClockRecord IN+OUT đủ, hệ thống auto-resolve `ResolvedSymbol`:
- `X` = công đủ (đủ giờ trong shift)
- `X1/2` = nửa ngày
- `V` = nghỉ phép có lương
- `L` = nghỉ lễ
- `O` = nghỉ không lương
- `P` = nghỉ thai sản
- 25+ symbol khác theo Cấu hình ký hiệu nghỉ phép

#### OT Calculator (auto sau resolve symbol)
- Detect ClockRecord vượt ngoài Shift definition
- Phân loại: OT thường (150%) · cuối tuần (200%) · lễ (300%) · đêm (130%)
- 6 cột summary: `OtWeekdayDay/Night`, `OtWeekendDay/Night`, `OtHolidayDay/Night`

### 8.5 Tab "Máy chấm công" (Devices)
Page `/attendance` → tab **"Máy chấm công"** (4th tab).
- List device: Serial · Model · OrgUnit · LastPing · Status (Online/Offline)
- Button "Thêm máy" — register device serial vào workspace.
- Endpoint: `GET /attendance/devices`, `POST /attendance/devices`, `DELETE /attendance/devices/{id}`.

### 8.6 Tab "Tổng hợp tháng" (Summary)
Page `/attendance` → tab **"Tổng hợp"** (1st tab) hoặc full page `/attendance-summary`.
- Bảng tổng hợp theo NV + tháng: 20+ cột số liệu (StandardWorkdays · ActualWorkdaysFull · LateMinutes · NightShiftHours · OT 6 cột · LeaveDays · RemainingAnnualLeave · …).
- Button **"Tính lại"** (recompute) — re-run symbol resolver + OT calculator + aggregate.
- Buttons báo cáo (Wave 1):
  - "Báo cáo chi tiết chấm công" (Excel)
  - "Báo cáo tổng OT" (Excel)
  - "Báo cáo chi tiết OT" (Excel)
  - "Báo cáo ca đêm" (Excel)
  - ✨ **"Phiếu đối chiếu (ZIP cả tháng)"** (Wave 5 PR #30/494) — gom PDF từng NV vào ZIP, surface skipped count qua header.
- Click 1 row NV → mở dialog chi tiết tháng + grid 31 ngày + button "Tải Phiếu đối chiếu PDF" (per-employee).

#### RLS (Wave 5 PR #32)
- HR Admin (Full) thấy tất cả.
- TBP (Team) chỉ thấy NV trong OrgUnit subtree mình quản.
- Worker (Own) chỉ thấy chính mình.

### 8.7 Tab "Nghỉ phép" (Leaves) — duyệt 2/3 cấp
Page `/attendance` → tab **"Nghỉ phép"** (2nd tab).
- List leave request: NV · From · To · LeaveType · Symbol · Status · ApprovalFlow level
- Filter: status (Pending/ApprovedL1/L2/Approved/Rejected/Cancelled) · date range · orgUnit
- Button cho TBP/HR: "Duyệt cấp X" · "Từ chối"
- Endpoint:
  - `GET /attendance/leaves`
  - `POST /attendance/leaves` (NV submit từ /me)
  - `PUT /attendance/leaves/{id}/approve`
  - `PUT /attendance/leaves/{id}/reject`
  - `PUT /attendance/leaves/{id}/cancel`

#### Webhook lifecycle
- `hrm.leave.approved` (Wave 5 PR #30) — chỉ fire khi đạt status final `Approved` (không fire L1/L2 trung gian)
- `hrm.leave.rejected` (Wave 5 PR #30)

### 8.8 Trang riêng "Quản lý nghỉ phép" — `/leave`
Sidebar **Nghỉ phép → Quản lý nghỉ phép**. Page `HrmLeavePage` — full page version của tab "Nghỉ phép" với UI rộng hơn + filter advanced + bulk export.

### 8.9 Cấu hình nghỉ phép — 4 trang riêng
| Sidebar | Route | Mô tả |
|---|---|---|
| Cấu hình ký hiệu nghỉ phép | `/leave-config` | LeaveSymbol CRUD (X/V/P/O/...) — paid rate, count as workday |
| Báo cáo & Thống kê phép | `/leave-report` | Pie chart by type, bar chart by month |
| Chuyển phép tồn | `/leave-carry` | Carry over annual leave qua năm sau |
| Chính sách nghỉ phép | `/leave-policy` | LeavePolicy: số ngày/năm theo seniority, cấp bậc, region |

---

## 9. Flow 6 — Tiền lương + 4-Signature (xem v2 doc — không đổi)

Trang `/payroll-processing` (M01-M04) → `/payroll` (xem bảng + PDF + 4 chữ ký).

Updates kể từ v2:
- Confirm dialog cho mỗi signature button (Wave 4 PR #494) — irreversible action.
- Payslip pagination 12/page trong `/me` (Wave 4).
- Status badge e-sign acknowledge trên row payslip.

---

## 10. Flow 7 — Hộp duyệt (Inbox) — TBP/Manager dashboard

### Vị trí
Sidebar **Tổng quan → Hộp duyệt** → `/inbox`. Page: `HrmInboxPage`.

### 3 tabs
- **Chờ duyệt** (có count badge) — toàn bộ request đang chờ user duyệt
- **Tất cả** — toàn bộ request liên quan
- **Đã xử lý** — lịch sử đã duyệt/từ chối

### Filter
- Loại: Nghỉ phép · Tăng ca · Bổ sung lương · Hợp đồng
- Người gửi · OrgUnit
- Bước duyệt hiện tại (L1/L2/Final)

### Cột
| Loại | Người gửi | Nội dung | Thời gian | Bước duyệt | Action |

Click row → mở slide-over chi tiết → buttons "Duyệt" + "Từ chối" + textarea note.

### KPI
- Chờ duyệt (số đơn)
- Trung bình thời gian duyệt (giờ)
- Đã duyệt tuần này / Từ chối tuần này

---

## 11. Flow 8 — Calendar (Lịch sự kiện)

Sidebar **Tổng quan → Lịch sự kiện** → `/calendar`. Page: `HrmCalendarPage`.
- Hiển thị workspace-shared events (nghỉ lễ, training, all-hands, …)
- Filter: my events (NV ở org subtree) vs all
- Permission `hrm.calendar.view` (scope team available)
- Button "Thêm sự kiện" cho user có `hrm.calendar.manage`

---

## 12. Flow 9 — Recruitment + Onboarding (per-employee)

### Recruitment — `/recruitment`
Sidebar **Hồ sơ nhân viên → Tuyển dụng**. 3 tabs:
- **Tin tuyển dụng (Jobs)** — Code · Title · EmploymentType · Headcount · Status (Draft/Open/Closed)
- **Pipeline** — Kanban board: NEW → SCREENING → INTERVIEW → OFFER → HIRED/REJECTED
- **Ứng viên (Candidates)** — list ứng viên với CV + history move

KPI: Tin đang mở · Ứng viên tuần · Time-to-hire · Conversion%.

Action "Hire" trên candidate → tự tạo Employee + checklist Onboarding + fire webhook **`hrm.recruitment.application.hired`** (Wave 1 PR #24).

### Onboarding (per-employee) — `/onboarding`
Sidebar **Hồ sơ nhân viên → Onboarding**. Page: `HrmOnboardingPage`.
- List checklists của NV đang onboarding
- Mỗi checklist gồm 10+ task (sign HĐ, nhận laptop, training an toàn LĐ, link IAM, …)
- Progress per checklist
- Button "Đánh dấu hoàn thành" từng task
- ⚠️ **KHÔNG nhầm với Onboarding Wizard** (`/onboarding-wizard`) — Wizard là setup workspace, Onboarding (no -wizard) là per-NV checklist.

---

## 13. Flow 10 — Performance (OKR + Reviews)

Sidebar **Hồ sơ nhân viên → Hiệu suất (OKR/Review)** → `/performance`. 3 tabs:
- **Mục tiêu (OKR)** — Goal · Key Results · Weight · Due date · Progress %
- **Đánh giá** — review cycle quarterly/half-year/yearly per NV
- **1-1 Meetings** — schedule + agenda + notes 1-on-1

KPI: Active goals · Avg progress · Pending reviews · Upcoming meetings.

---

## 14. Flow 11 — Self-service workflows (`/me/...`)

| Route | Mô tả | Permission |
|---|---|---|
| `/me/overtime` | NV đăng ký tăng ca → flow duyệt L1 (TBP) → L2 (HR) | `hrm.overtime.submit_own` |
| `/me/salary-supplement` | NV đề xuất bổ sung lương (thưởng, hoàn ứng) | `hrm.salary_supplement.submit_own` |
| `/me/feedback` | NV gửi phản hồi, ý kiến (M07) | `hrm.feedbacks.submit_own` |
| `/me/health-accident` | NV báo tai nạn LĐ → HR xử lý | `hrm.health_accident.view_own` |
| `/me/reconciliation` | NV xem + xác nhận phiếu đối chiếu công (deadline 09:00 ngày 02 tháng sau) | `hrm.attendance.view_own` |
| `/me/training` | Khóa học của NV đó | `hrm.training.enrollments.view` (scope own) |
| `/me/profile` | Sửa thông tin liên hệ (phone, personal email, contact KC) | `hrm.me.profile.update` |
| `/me/payslip` (qua tab Payslips trong /me) | Xem lịch sử + status xác nhận e-sign | `hrm.me.payslips.view` |

---

## 15. Flow 12 — Phúc lợi (Benefits)

### Sức khoẻ — `/health-accident`
Sidebar **Phúc lợi → Y tế & An toàn** + **Sự cố lao động**. 
- Health check lịch sử per NV
- Accident report — incident date + severity + medical leave days + insurance claim

### Bảo hiểm BHXH (overview) — `/insurance`
Sidebar **Phúc lợi → Bảo hiểm BHXH**. Admin-only.
- Tổng quan workspace: tổng NV đang đóng · tổng lương BH · breakdown by status
- Link tới Báo cáo BHXH (C45/C46/D02/TK1).

### Hợp đồng lao động — `/contracts`
Sidebar **Phúc lợi → Hợp đồng lao động**. Xem flow chi tiết v2.

---

## 16. Flow 13 — Đào tạo (Training)

Sidebar **Đào tạo**:
- `/training` — Khóa học (program/course catalog)
- `/training-employees` — NV tham gia mỗi khóa
- `/training-evaluation` — Đánh giá sau khoá học

---

## 17. Flow 14 — Báo cáo

| Sidebar | Route | Báo cáo |
|---|---|---|
| Báo cáo tổng quan | `/reports` | Dashboard tổng hợp metrics |
| Báo cáo lao động | `/reports-labor` | 3 tabs: Form 01/PLI · Form 05/PLI · Nghỉ việc (Nghị định 145) |
| Báo cáo BHXH | `/reports-bhxh` | 4 tabs: C45 (tăng giảm) · D02 (đóng quý) · TK1 (tham gia lần đầu) · **C46 (bảng kê tháng)** ✨ |

✨ **C46-TS** (Wave 6 PR #33) — Bảng kê thông tin đóng BHXH/BHYT/BHTN. Liệt kê toàn bộ NV đang đóng + số ngày tham gia + breakdown NLĐ (10.5%) vs NSDLĐ (21.5%).

---

## 18. Flow 15 — Cài đặt (Admin-only)

| Sidebar | Route | Chức năng |
|---|---|---|
| Cài đặt chung (toàn bộ danh mục) | `/settings` | HrmMasterDataPage 6 tabs (xem mục 7) |
| Tham số công ty (PIT, lương, lễ) | `/settings/tenant` | HrmTenantSettingsPage — Settings form + 3 PresetCard (xem mục 4) |
| Mẫu hợp đồng | `/settings/contract-templates` | CRUD ContractTemplate |
| Mẫu đánh giá | `/settings/evaluation-templates` | CRUD EvaluationTemplate (LSEV-style 70 điểm) |
| Trường tùy chỉnh | `/settings/custom-fields` | CRUD CustomFieldDefinition cho Employee |
| Luồng hợp đồng | `/settings/contract-flow` | ContractFlowConfig — auto-progress HD1→HD2→HD3→HD4 |
| Luồng phê duyệt | `/approval-flows` | ApprovalFlow CRUD — sub-tabs: Leave/OT/Salary/Contract |
| Phân quyền | `/roles` | Role + Permission assignment (delegate to IAM) |
| Webhooks | `/webhooks` | Outbound webhook config — 12 events groups 4 nhóm (Wave 2) |
| API Tokens | `/api-tokens` | M2M tokens cho integration |
| **Onboarding Wizard** ✨ | `/onboarding-wizard` | 7-step setup wizard (mục 3) |

---

## 19. Smoke Test Checklist — Comprehensive (cho LSEV UAT)

### Setup (5 step)
| # | Action | Expected |
|---|---|---|
| 1 | Login HR Admin → `/workspace/hr` | Dashboard hiển thị |
| 2 | Open Onboarding Wizard → start | Step 1 active, progress 0/7 |
| 3 | Cài đặt → Tham số công ty → bấm preset VnFactory | Toast `Đã khởi tạo 53 dữ liệu mẫu`, card xanh summary |
| 4 | Bấm lần 2 | Idempotent counts = 0 |
| 5 | Cài đặt → Cài đặt chung → các tab | Thấy data vừa seed (8 OrgUnit · 12 chức danh · 7 ca · 30 ký hiệu · …) |

### Employees (10 step)
| # | Action | Expected |
|---|---|---|
| 6 | Import Center → tab Nhân viên → upload 5 NV | `inserted: 5, errors: 0` + webhook `hrm.employee.created` × 5 |
| 7 | `/employees` → tab "Đang làm" → search 1 NV | Filter đúng |
| 8 | Click row → vào `/staff/{id}` → 7 tabs hiện đầy đủ | Personal/Job/Salary/Insurance/Evaluations/Contacts/Training |
| 9 | Sửa Phone NV → Lưu | Webhook `hrm.employee.updated` fire |
| 10 | Tab Lương → thêm dòng lịch sử lương | Bảng update + audit log |
| 11 | Tab Liên hệ → thêm NPT (Người phụ thuộc thuế) | Used trong tính TNCN |
| 12 | Tab Bảo hiểm → thêm InsuranceSalary | Báo cáo C45/C46 capture được |
| 13 | Tạo HĐ thử việc HD1 cho NV qua `/contracts` | Webhook fire khi evaluate |
| 14 | Thôi việc NV → input TerminationDate | Status = TERMINATED, tab Checklist hiện, webhook `hrm.employee.terminated` |
| 15 | `/employees` → tab "Đã nghỉ" | Thấy NV vừa terminate |

### Attendance (12 step)
| # | Action | Expected |
|---|---|---|
| 16 | `/attendance` → tab "Cấu hình" → Setup tháng 2026-05 | AttendanceMonth created |
| 17 | Import Center → tab Chấm công → 30 dòng (3 NV × 10 ngày) | `imported: 30, raws: 30`, symbol auto-resolved |
| 18 | `/attendance` → tab "Chấm công thô" → filter NV | Bảng có clock records |
| 19 | Bulk assign shift CA1 cho 5 NV cả tháng | 150 assignments created |
| 20 | NV gửi đơn nghỉ qua `/me/overtime` (hoặc admin trong tab Leaves) | Status = Pending |
| 21 | `/inbox` → tab "Chờ duyệt" → duyệt L1 → L2 → Final | Webhook `hrm.leave.approved` chỉ fire ở Final |
| 22 | Setup HikVision device serial trong tab Devices | Device list có item mới |
| 23 | Post fake JSON HikVision AcsEvent với `X-Device-Serial` header | Raw inserted + webhook `hrm.attendance.checked_in` fire (Wave 5) |
| 24 | `/attendance-summary` → "Tính lại" tháng 2026-05 | 6 cột OT + LateMinutes + RemainingAnnualLeave populate |
| 25 | Bấm "Phiếu đối chiếu (ZIP cả tháng)" | ZIP download, toast "gồm N NV, bỏ qua M" |
| 26 | Click 1 row NV → grid 31 ngày → tải Phiếu đối chiếu PDF | PDF có đủ 6 OT + 30 ký hiệu chú thích |
| 27 | NV login → `/me/reconciliation` → countdown deadline 09:00 ngày 02 | Pill timer hiển thị, button confirm |

### Payroll (8 step) — Wave 1 + 2 + 4
| # | Action | Expected |
|---|---|---|
| 28 | `/payroll-processing` → chọn 2026-05 → "Tính lương" | Job kicked off, status polling |
| 29 | `/payroll` → bấm cell → PayslipDialog hiện | 4 sections (lương/công/thu nhập/khấu trừ) |
| 30 | 4-Signature: bấm "Bước 1 — HR Lập" → confirm dialog | Toast `Đã đánh dấu chữ ký bước HR Lập bảng` (Wave 4) |
| 31 | Tiếp các bước 2-3-4 → confirm mỗi bước | 4 timestamp signature lưu DB |
| 32 | Bấm "Khoá tháng" | Status LOCKED, webhook `hrm.payroll.locked` fire |
| 33 | Bấm "Xuất Excel" 4 báo cáo (summary/OT/PIT/bank-list) | 4 file `.xlsx` |
| 34 | NLĐ login → `/me` → tab Phiếu lương → tải PDF | PDF 4 chữ ký + pill ack status |
| 35 | Pagination payslip — nếu > 12 phiếu → trang 2 | Wave 4 — paginator OK |

### Reports (4 step)
| # | Action | Expected |
|---|---|---|
| 36 | `/reports-bhxh` → tab C45 → tháng 2026-05 → Xuất Excel | File `BHXH-C45-2026-05.xlsx` |
| 37 | Tab D02 → Q2-2026 → Xuất Excel | File `BHXH-D02-Q2-2026.xlsx` |
| 38 | Tab C46 → tháng 2026-05 → Xuất Excel ✨ | File `BHXH-C46-2026-05.xlsx` (Wave 6) |
| 39 | `/reports-labor` → Form 01/PLI → Xuất Excel | File theo Nghị định 145 |

### Webhooks (3 step)
| # | Action | Expected |
|---|---|---|
| 40 | `/webhooks` → Add endpoint URL webhook.site + 5 events ✅ wired | Subscribe OK |
| 41 | Trigger 5 events (1 cho mỗi action ở trên) | Receiver verify HMAC `X-NextX-Signature` |
| 42 | Tab Deliveries → list | 5 entries success |

### Onboarding Wizard advancement
| # | Action | Expected |
|---|---|---|
| 43 | `/onboarding-wizard` → step 1 "Đã làm xong" | Step 2 active, progress 1/7 = 14% |
| 44 | Lặp các bước 2-6 | Wizard advance, CTA links đúng |
| 45 | Step 7 → bấm "Hoàn tất ✓" | CompletedAt ghi, banner xanh hiển thị timestamp |

---

## 20. So sánh v2 → v3

| Mục | v2 | v3 |
|---|---|---|
| Sidebar | Mention 9 flow chính | Map 10 groups + 40+ menu items khớp i18n |
| Employee detail | 4 tabs (mention) | 8 tabs chi tiết + IAM link/unlink + Documents |
| Attendance | "Setup tháng + Import + Symbol Resolver" tóm tắt | 5 tabs `/attendance` + Year Template + Shift Assignment + 5 leave-related sidebar items |
| Webhooks | 5 events đã wire | 5 events ✅ + 7 placeholder (giờ đã wire 8 events tổng) |
| Self-service | `/me/reconciliation` | `/me` 4 tabs + 7 sub-routes `/me/*` |
| Cài đặt | 6 mục | 11 mục bao gồm Onboarding Wizard ✨ |
| Smoke Test | 16 step | 45 step phân category |
| Báo cáo BHXH | C45 + D02 + TK1 | + **C46-TS** ✨ |
| Wave history | 16 PRs | 28 PRs (wave 1 → 8 — xem mục 22) |

---

## 21. Known limitations / Coming (rút gọn — đã đối chiếu LSEV SRS v2.2)

Em đã audit lại §21 v3 ban đầu cho khớp 5 items với LSEV SRS — chỉ còn 2 items không phải LSEV requirement:

- **"Hành trình"** trong menu Chấm công → đó là DMS GPS journey check-in cho Sales/Field force. **KHÔNG nằm trong LSEV SRS.** Để placeholder trong menu cho future DMS integration.
- **Performance (OKR)** sub-tab "Đánh giá" + "1-1 Meetings" còn skeleton. **KHÔNG nằm trong LSEV SRS.** LSEV chỉ cần KPI evaluation (Score → S/A/B/C/D → 120/110/100/90/70%) — đã DONE ở `EmployeeEvaluation.CalculateGrade/CalculateKpiRatio` + áp dụng trong `CalculatePayrollCommandHandler`.
- **HrmIndustryTemplate** (Wave 6 PR #34) — entity + 2 system seeds OK; chưa có endpoint `POST /setup/preset/from-template/{id}` apply workspace-owned template + FE template catalog. **KHÔNG nằm trong LSEV SRS** — feature marketplace cho multi-tenant tương lai.

### Đã đóng trong Wave 8 (sau audit lần cuối)

- ✅ **Onboarding per-employee UI** — em ghi nhầm trong v3 lần trước. Code thực tế đã có:
  - `HrmOnboardingChecklistDetail.tsx:121` — function `toggleTask` đổi status complete/pending
  - `HrmStartOnboardingDialog` — start từ template
  - `AddAdHocOnboardingTask` command — thêm task tự do
  - `HrmOnboardingTemplateForm` — CRUD template

- ✅ **Webhook retry exponential backoff** (Wave 8 PR #38) — `WebhookEventDispatcher` rewrite:
  - 3 attempts · backoff 1s → 5s
  - Retry chỉ trên transient (HttpRequestException, timeout, HTTP 5xx/429)
  - HTTP 4xx ≠ 429 → abort ngay (manual re-trigger qua /webhooks Deliveries tab)
  - `WebhookDelivery` ghi đầy đủ `RetryCount` + `ErrorMessage` per attempt + `ResponsePreview` 512 chars + payload sample 2KB
  - Header mới `X-NextX-Attempt: {n}` cho receiver dedupe

### CC-Q1 chỉ chờ LSEV side
- 🔴 **HikVision FaceID 3000 khuôn mặt realtime** (SRS §755) — listener endpoint `/attendance/hikvision/event` (Wave 6 PR #33) đã sẵn sàng nhận push. **Chờ LSEV cấp network info (IP, subnet)** để Bên B cấu hình device push tới server NextX. Không phải gap phía em.

---

## 22. Wave history (8 waves · 28 PRs total)

| Wave | Date | PRs | Scope |
|---|---|---|---|
| 1 | trước 2026-05-28 | 15 (BE #22-28 + FE #455-467) | LSEV core: Symbol Resolver, OT, PDF, 4-signature, contracts evaluate, webhook 3, multi-tenant, Import Center |
| 2 | 2026-05-28 | BE #29 + FE #481 | UAT doc v2 + 3-button preset selector + 4-sig panel + payslip ack badge + webhook event sync |
| 3 | 2026-05-28 | BE #30 | Reconciliation bulk ZIP + 5 missing webhook events wire |
| 4 | 2026-05-28 | BE #31 + FE #494 + FE #496 | Import Holidays + UX polish (bulk ZIP button + 4-sig confirm + payslip pagination + preset tooltips) |
| 5 | 2026-05-28 | BE #32 | RLS OrgUnit scope cho GetSummary + GetContracts + bulk-zip |
| 6 | 2026-05-29 | BE #33 + #34 + #35 + FE #502 | HikVision listener + BHXH C46 + IndustryTemplate entity + Onboarding Wizard (BE+FE) |
| 7 | 2026-05-29 | BE #36 | UAT doc v3 — Employee + Attendance deep dive |
| 8 | 2026-05-29 | BE #38 + BE #39 (this doc cleanup) | Webhook retry exponential backoff + doc §21 cleanup |

---

*Tài liệu UAT v3.1 · NextX HRM · session 2026-05-29 · 28 PRs LSEV merged (wave 1 → 8)*
