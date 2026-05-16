# NextX HRM — Hướng dẫn UAT cho End-user v4.0

> **Dành cho:** End user / tester — HR Manager, HR Staff, Trưởng phòng, Tổ trưởng, Nhân viên thường, Công nhân
> **Mục đích:** Đọc xong là biết bấm nút nào, không cần biết code
> **Cập nhật:** 2026-05-15 v4.0 — Wave 2-5 FE features lên sandbox: HrmGlobalUI (Ctrl+K + Bell), OrgChart Tree view, Sidebar "Cá nhân" Self-Service (6 menu /me/*), 4 Picker search components, Mobile card layout, Auto-save draft, Success animation, Role-based view, Calendar Add Event, Payroll Step 6 publish, Report preview, Hire→Employee flow, IAM Provisioning UI, Inbox bulk + delegation, 7 canonical routes mới + i18n 1300+ keys.
> **Cách dùng:** Tick vào ô `[ ]` sau mỗi bước đã test. Ghi "❌ LỖI" và mô tả ngắn nếu có vấn đề.
> **Số liệu mặc định:** dùng định dạng Việt Nam — `15.500.000đ`, ngày `13/05/2026`, giờ `14:30`.

---

## 🆕 Changelog v4.0 / Cập nhật mới nhất (2026-05-15)

### Tổng quan 16 nhóm tính năng mới (Wave 2-5)

> **Mục đích:** Tester đọc nhanh để biết FE đã bổ sung gì kể từ v3.3 (2026-05-14). Mỗi mục có section test riêng trong các phần persona bên dưới.

#### 1. HrmGlobalUI — Ctrl+K Command Palette + Notification Bell (MỚI)

- **Ctrl/Cmd+K** từ bất kỳ trang nào trong `/workspace/hr/*` → mở Command Palette giữa màn hình.
- Palette có **26 navigation commands** chia **7 nhóm**: Cá nhân · Nhân sự · Chấm công & Nghỉ phép · Tiền lương · Hiệu suất · Đào tạo · Báo cáo & Cấu hình.
- Gõ keyword (vd: "hồ sơ", "phiếu lương", "OT") → fuzzy match → Enter chạy lệnh.
- **Floating bell** góc dưới-phải (chỉ hiện trong `/workspace/hr/*`) → click → mở **HrmNotificationPanel** dạng SlideOver. Badge số notification chưa đọc, click mark-as-read.
- ESC đóng palette / panel.
- **Troubleshooting:** Nếu `Ctrl+K` không mở palette → hard refresh trình duyệt (Ctrl+Shift+R) để load HrmGlobalUI wrapper.

#### 2. OrgChart Tree view (MỚI) — tab Phòng ban

- Đường connector kiểu file explorer: `├─ └─ │` thay vì layout phẳng.
- Mỗi node có button **▶ / ▼** expand/collapse riêng. Toolbar có 2 nút: `<Mở rộng tất cả>` / `<Thu gọn>`.
- Icon theo level: **🏢 Công ty / 💼 Phòng ban / 📁 Tổ**.
- Color badge **tier 1-4**: purple (L1) · blue (L2) · green (L3) · amber (L4+).
- **Employee count badge** mỗi node — click → filter danh sách NV theo phòng ban đó.
- **Search inline**: gõ tên/mã → highlight `<mark>` match + tự expand parent nodes chứa kết quả.
- **Quick stats** trên toolbar: `{units} đơn vị · {employees} NV · {levels} cấp`.

#### 3. Sidebar Group "Cá nhân" (MỚI từ Wave 2)

Section riêng trong sidebar dành cho NV, 6 menu Self-Service:

| Menu | URL canonical |
|---|---|
| Hồ sơ của tôi | `/me/profile` |
| Tăng ca | `/me/overtime` |
| Bổ sung lương | `/me/salary-supplement` |
| Khóa học của tôi | `/me/training` |
| Phản hồi | `/me/feedback` |
| Báo tai nạn | `/me/health-accident` |

> NV chỉ thấy data của mình (BE enforce `currentUser.EmployeeId`). NV002 đã linked → mọi `/me/*` hoạt động ngay.

#### 4. Picker components có search (MỚI — 4 components)

Mọi form HR khi cần chọn entity giờ dùng picker có search thay vì native `<select>`:

| Picker | Dùng cho | Search theo |
|---|---|---|
| `HrmEmployeeSelect` | Chọn nhân viên | Mã NV / Họ tên |
| `HrmOrgUnitSelect` | Chọn phòng ban (flatten tree) | Tên đơn vị |
| `HrmPositionSelect` | Chọn chức danh | Tên / mã |
| `HrmContractSelect` | Chọn HĐLĐ theo employee | Số HĐ |

#### 5. Mobile responsive — Card layout < 768px (MỚI 6 pages)

Các pages chuyển từ table sang card stack ở mobile breakpoint:

`/hr/leave` · `/hr/feedback` · `/hr/insurance` · `/hr/contracts` · `/hr/training/courses` · `/hr/recruitment/job-posting`

- Sidebar trên mobile → hamburger drawer (slide từ trái).
- Tự switch table ↔ card tại **md breakpoint (768px)**.

#### 6. Form features mới

- **Auto-save draft** (3 forms): Review form · Onboarding template · Job Posting — gõ vài giây tự lưu localStorage. Đóng tab → mở lại → toast "Đã khôi phục bản nháp".
- **HrmSuccessCheck animation** (5 forms): sau submit thành công hiện icon `CheckCircle2 + pulse` 1.5s rồi toast xanh.
- **HrmQueryError retry banner** (11 pages): khi load data lỗi (5xx / network) hiện banner đỏ + nút `<Thử lại>` thay vì màn trắng.
- **Skeleton fade-in** thay text "Đang tải..." — shimmer placeholder giống Linear/Notion.

#### 7. Role-based view (MỚI — 3 pages split HR vs NV)

| Page | HR thấy | NV thấy |
|---|---|---|
| `/hr/leave` | Tất cả đơn của workspace | Chỉ đơn của mình; ẩn cột Employee + nút Duyệt/Từ chối |
| `/hr/feedback` | Tất cả feedback | Chỉ feedback do mình gửi |
| `/hr/health-accident` | Tất cả báo cáo | Chỉ báo cáo của mình |

- Detect qua **`useHrmRole`** (FE) — group code có `hr_admin` / `hr_manager` → HR view.
- BE enforce qua **`HrmRoles.IsHrAdmin`** trên handler — NV cố call API list-all → 403.

#### 8. Calendar "Thêm sự kiện" (MỚI từ Wave 5)

- Nút **`<+ Thêm sự kiện>`** trên toolbar Calendar.
- 5 categories user-created: `CUSTOM` · `MEETING` · `TRAINING` · `COMPANY_EVENT` · `OTHER`.
- 7 categories **auto-aggregated** (read-only): `LEAVE` · `BIRTHDAY` · `CONTRACT_END` · `PROBATION_END` · `HOLIDAY` · `PAYROLL_LOCK` · `INTERVIEW`.
- **Click ô ngày trống** trong tháng → dialog Add Event mở với `startDate` pre-fill.

#### 9. Payroll Wizard Step 6 — Phát hành phiếu lương (MỚI)

- Step 6 sau Lock kỳ — chọn **channel**: `EMAIL` · `PORTAL` · `ESIGN`.
- Bảng per-employee distribution status: ✅ Sent / ⏳ Pending / ❌ Failed.
- Channel `ESIGN` → publish event `hrm.payslip.esign.requested` tới Contract service (DMS sign flow).

#### 10. Reports — Preview trước khi tải Excel (MỚI 4 reports)

- Mỗi card báo cáo nay có **2 button**: `<Xem trước>` + `<Tải Excel>` (thay cho 1 nút `<Tải xuống>` cũ).
- `<Xem trước>` → dialog table render full data trong app.
- `<Tải Excel>` → ClosedXML export với header **Linear accent color** + currency format `#,##0 đ`.

#### 11. Hire & Create Employee flow (MỚI — link Recruitment ↔ Employee)

- Trong **Recruitment Kanban**: drag candidate card vào **terminal stage HIRED** → tự trigger **Hire dialog**.
- Dialog pre-fill từ application: `fullName`, `email`, `phone`.
- Submit → tạo Employee record + link `application.HiredEmployeeId = newEmployeeId`.
- Toast thành công có **action button** `<Tạo HĐLĐ ngay>` → redirect tới `/hr/contracts/new?employeeId=...`.

#### 12. IAM Provisioning UI (MỚI)

- Trên **Employee detail page** — nếu NV chưa có user account → banner vàng: "Chưa có tài khoản IAM" + button `<Cấp tài khoản>`.
- Dialog form: Email + Họ tên (pre-fill từ employee).
- Submit → call IAM Provisioning API. Nếu IAM service unreachable → **stub fallback** (lưu intent, hiện toast "Sẽ tạo khi IAM service sẵn sàng").

#### 13. Inbox — Bulk approve + Delegation (MỚI)

- Mỗi row trong **`/hr/inbox`** có **checkbox column**. Tick 2+ → floating **action bar** ở bottom: `<Duyệt N>` / `<Từ chối N>`.
- Quick row actions: **✓ / ✗ inline** ở cuối row — duyệt/từ chối 1 đơn không cần mở detail.
- **`<Uỷ quyền duyệt>`** trong UserProfileDropdown → form chọn người delegate + date range + scope (loại đơn).

#### 14. Tài khoản test (KHÔNG ĐỔI)

- Email: **`leanhluong2001@gmail.com`** · Password: **`Luong2001@`**
- Workspace: **Nextx-Demo** (`019d8f70-3dac-73fa-9a9e-d1583a966e8d`)
- NV002 — Trần Thị B linked với account này → mọi `/me/*` Self-Service hoạt động.

#### 15. Routes mới — 7 canonical + redirect aliases (MỚI)

URL cũ vẫn redirect 301 sang URL mới, không break bookmark:

| Canonical (mới) | Alias (cũ — vẫn redirect) |
|---|---|
| `/hr/employees` | `/staff` |
| `/hr/contracts` | `/contract` |
| `/hr/salary-supplement` | `/payroll-supplement` |
| `/hr/reports/bhxh` | `/reports-bhxh` |
| `/hr/reports/labor` | (mới) |
| `/hr/training/employees` | (mới) |
| `/hr/training/evaluations` | (mới) |

#### 16. i18n đầy đủ vi/en

- **1300+ keys**, **17 namespaces** consolidated vào root `hr.*` (giảm nesting, dễ tra cứu).
- Toggle locale ở header → tất cả label/menu/toast/validation message đổi ngôn ngữ tức thì, không reload trang.

### Sandbox image sau Wave 2-5

| Mục | Trạng thái |
|---|---|
| HRM service image | `ghcr.io/nextx-organization/nextx-hrm:5404b7f...` (chưa đổi từ v3.3 — Wave 2-5 chủ yếu FE) |
| FE tenant build | ✅ Đã deploy với 16 nhóm tính năng trên |
| FE truy cập | `https://localhost-app.nextx.vn:3000` (dev) hoặc tenant URL sandbox |

### Troubleshooting nhanh — v4.0

| Triệu chứng | Nguyên nhân & cách fix |
|---|---|
| `Ctrl+K` không mở palette | Hard refresh (Ctrl+Shift+R). HrmGlobalUI chỉ wrap routes `/workspace/hr/*` — không hoạt động ở `/login`, `/workspace/admin` |
| Floating bell không hiện | Đang ở route ngoài `/workspace/hr/*` — bell ẩn theo design (không show ở admin/iam pages) |
| Picker `<HrmEmployeeSelect>` không gợi ý kết quả | Kiểm tra workspace có > 0 employee. Thử gõ ≥ 2 ký tự — debounce 200ms |
| Mobile card layout không xuất hiện | Test < 768px width (DevTools responsive mode iPhone 12 — 390px). Một số page (insurance/contracts) chỉ chuyển card ở < 640px |
| `/me/profile` báo 403 | Account chưa link với employee — yêu cầu HR vào `/hr/employees/{id}` → tab IAM → `<Cấp tài khoản>` |
| Auto-save draft không khôi phục | Draft lưu trong **localStorage cùng browser** — đổi browser/máy sẽ mất. Mở Inspect → Application → Local Storage tìm key `hrm.draft.*` |
| Bulk approve báo "Một số đơn không thuộc bước duyệt của bạn" | Bình thường — chỉ approve đơn ở approval step mà current user là approver. Skip các đơn còn lại |
| Calendar Add Event báo 422 ở mobile | Lỗi date picker mobile native — nhập tay format `YYYY-MM-DD` |

---

## 🆕 Changelog v3.3 / Cập nhật mới nhất (2026-05-14)

### Sandbox trạng thái — READY TO TEST

| Mục | Trạng thái |
|---|---|
| Image HRM trên sandbox | `ghcr.io/nextx-organization/nextx-hrm:5404b7f6147993fe4546823f60bc5a2fa0980825` ✅ healthy |
| Tất cả endpoints mới (Dashboard, Calendar, LeaveCarry, Insurance, Contract, Payroll, Onboarding, Performance, Recruitment, OT, Salary Supplement, Health Accident, Feedback, Training) | ✅ Đã deploy |
| Docker Desktop local | ❌ KHÔNG còn cần — CI/CD tự build + push GHCR + deploy sandbox khi commit lên `develop` |
| FE truy cập | `https://localhost-app.nextx.vn:3000` (dev) hoặc gateway tenant URL sandbox |

### Tài khoản test sẵn sàng (Nextx-Demo workspace)

| Mục | Giá trị |
|---|---|
| Workspace ID | `019d8f70-3dac-73fa-9a9e-d1583a966e8d` |
| Tên workspace | Nextx-Demo |
| Username | `leanhluong2001@gmail.com` |
| Password | `Luong2001@` |
| Linked employee | NV002 — Trần Thị B — Phòng Nhân Sự — Nhân Viên Nhân Sự |
| Số màn FE có data | **40/41 màn** sẵn sàng test |

### Test data đã seed sẵn

- **3 employees**: NV001, NV002 (linked với account test), NV003
- **4 phòng ban**: Kinh Doanh, Nhân Sự, Kế Toán, Ban Giám Đốc
- **2 hợp đồng**: HD1 (không thời hạn) + HD2 (12 tháng)
- **3 insurance histories**, **2 attendance setups + summaries**
- **Payroll tháng 05/2026** đã tính: gross **11.300.000đ**, net **9.950.000đ**, 2 payslip
- **Performance Cycle Q2/2026** Active, 2 OKR, 2 1-on-1 scheduled
- **Recruitment**: 6 stages, 2 jobs, 4 candidates, 1 interview lịch 20/05
- **1 OT request**, **1 Salary Supplement**, **1 Health Accident**, **2 Feedback**
- **Approval Flows**: 6 luồng (Employee, Worker, Tbp)

### Bug fixes user thấy được (Phase 0)

| # | Trước | Sau (đã fix) |
|---|---|---|
| 1 | Sidebar "Tổng quan" stuck expanded khi đang ở page bên trong | Đã thu gọn được bình thường |
| 2 | Trang "Trang cá nhân" lỗi 500 "Invalid time value" khi có salary supplement | Render OK, không còn lỗi |
| 3 | Dashboard chỉ có period filter cố định (Hôm nay/Tuần/Tháng) | Thêm bộ lọc **"Tùy chọn (CUSTOM)"** với 2 date picker (từ ngày → đến ngày, max 366 ngày) |
| 4 | Calendar events có thể 500 nếu thiếu params | Load OK, validation rõ ràng |
| 5 | Leave-carry preview trả 500 khi thiếu params | Trả 400 + message rõ ràng |

### Đổi tên menu

- **vi**: `Của tôi` → **`Trang cá nhân`**
- **en**: `My space / My portal` → **`My page`**
- Mọi đoạn hướng dẫn "Vào menu Của tôi" / "Click vào Của tôi" trong doc đã được cập nhật.
- Lưu ý: cụm "Đơn của tôi", "Phiếu lương của tôi" là tên **tab/section bên trong**, KHÔNG đổi.

### Phase 20 — Security hardening

- Upgrade **Scriban 5.12.1 → 7.2.0** — loại bỏ 11 vulnerability advisories (1 Critical + 7 High + 3 Moderate).
- Không ảnh hưởng UX, không cần test riêng — chỉ note để tester biết bản test đã được security-hardened.

---

## Mục lục

- [Changelog v4.0 / Cập nhật mới nhất (2026-05-15)](#-changelog-v40--cập-nhật-mới-nhất-2026-05-15)
- [Changelog v3.3 (2026-05-14)](#-changelog-v33--cập-nhật-mới-nhất-2026-05-14)
- [Bảng tiến độ tổng quan](#bảng-tiến-độ-tổng-quan)
- [Thông tin truy cập](#thông-tin-truy-cập)
- [Phần 1: Bắt đầu](#-phần-1-bắt-đầu)
- [Phần 2: Persona Nhân viên văn phòng (Self-Service)](#-phần-2-persona-nhân-viên-văn-phòng-self-service)
- [Phần 3: Persona Trưởng phòng (Manager)](#-phần-3-persona-trưởng-phòng-manager)
- [Phần 4: Persona HR Manager](#-phần-4-persona-hr-manager)
- [Phần 5: Persona Công nhân (Worker)](#-phần-5-persona-công-nhân-worker)
- [Phần 6: Module-by-module checks](#-phần-6-module-by-module-checks)
- [Phần 7: Bilingual checks (vi/en)](#-phần-7-bilingual-checks-vien)
- [Phần 8: Theme & Design System checks](#-phần-8-theme--design-system-checks)
- [Phần 9: Acceptance criteria](#-phần-9-acceptance-criteria)
- [Phần 10: Known limitations + workarounds](#-phần-10-known-limitations--workarounds)

---

## 📊 Bảng tiến độ tổng quan

### Tổng quan 15 module

| # | Module | Vai trò chính | Số TC | Trạng thái | Lỗi phát hiện |
|---|--------|---------------|-------|------------|---------------|
| 1 | Đăng nhập & Chọn workspace | Mọi tester | 6 | ⬜ Chưa test | |
| 2 | Master Data (Dữ liệu gốc) | HR Manager | 12 | ⬜ Chưa test | |
| 3 | Hồ sơ nhân viên | HR + NV | 18 | ⬜ Chưa test | |
| 4 | Hợp đồng lao động | HR Manager | 14 | ⬜ Chưa test | |
| 5 | Chấm công | HR + TBP + Tổ trưởng | 16 | ⬜ Chưa test | |
| 6 | Nghỉ phép | NV + Manager + HR | 11 | ⬜ Chưa test | |
| 7 | Bảng lương | HR Manager | 13 | ⬜ Chưa test | |
| 8 | Bổ sung lương | HR Manager | 6 | ⬜ Chưa test | |
| 9 | Tuyển dụng (ATS) | HR + Manager | 9 | ⬜ Chưa test | |
| 10 | Onboarding | HR + Manager | 7 | ⬜ Chưa test | |
| 11 | Performance (OKR + 360 + 1-1) | NV + Manager | 12 | ⬜ Chưa test | |
| 12 | Phúc lợi (BHXH + TNLĐ) | HR Manager | 6 | ⬜ Chưa test | |
| 13 | Báo cáo | HR Manager | 8 | ⬜ Chưa test | |
| 14 | Cấu hình chính sách | HR Manager | 10 | ⬜ Chưa test | |
| 15 | Design System (dark/light/Ctrl+K/Bell) | Mọi tester | 11 | ⬜ Chưa test | |
| 16 | **(v4.0)** Self-Service `/me/*` (Profile/OT/Supplement/Training/Feedback/Accident) | NV | 6 | ⬜ Chưa test | |
| 17 | **(v4.0)** Mobile Card Layout (6 pages) | QA | 6 | ⬜ Chưa test | |
| 18 | **(v4.0)** OrgChart Tree view + search + tier badges | HR Manager | 7 | ⬜ Chưa test | |
| 19 | **(v4.0)** Calendar + Add Event | HR + Manager | 4 | ⬜ Chưa test | |
| 20 | **(v4.0)** Payroll Step 6 Phát hành phiếu lương | HR Manager | 4 | ⬜ Chưa test | |
| 21 | **(v4.0)** Hire → Employee + IAM Provisioning | HR Manager | 5 | ⬜ Chưa test | |
| 22 | **(v4.0)** Inbox bulk + quick actions + delegation | Manager | 3 | ⬜ Chưa test | |
| 23 | **(v4.0)** Pickers (Employee/OrgUnit/Position/Contract search) | HR | 4 | ⬜ Chưa test | |

> **Tổng test case:** ~205 case (154 cũ + ~51 mới từ v4.0). Thời gian ước tính cho tester thực hiện đầy đủ: **10–12 giờ**.

### Tổng quan theo 4 persona

| Persona | Module test | Thời gian |
|---|---|---|
| **NV văn phòng (Self-Service)** | Module 1, 11 + xem hồ sơ + đơn nghỉ + phiếu lương | 1.5h |
| **Trưởng phòng (Manager)** | Module 1, 6, 9, 10, 11 + Unified Inbox | 2.5h |
| **HR Manager** | Tất cả module | 6-8h |
| **Công nhân (Worker)** | Module 1 + xem phiếu lương + gửi feedback | 1h |

---

## 📌 Ghi chú ID — điền sau khi tạo

> **Quan trọng:** Sau khi tạo xong mỗi đối tượng, ghi lại ID/mã để dùng cho các bước sau.

| Đối tượng | Mã / ID |
|-----------|---------|
| Workspace test | `___________________________` |
| Phòng ban — Phòng Kinh doanh | Mã: `KD` / ID: ______________ |
| Phòng ban — Phòng Nhân sự | Mã: `NS` / ID: ______________ |
| Chức danh — Trưởng phòng | ______________ |
| Chức danh — Nhân viên | ______________ |
| Chức danh — Công nhân kỹ thuật | ______________ |
| Bậc lương WK1 — 2026 | 5.630.000đ |
| Bậc lương SF1.0 — 2026 | 8.500.000đ |
| Ca CA1 — 06:00-14:00 | ______________ |
| Ca Hành chính — 08:00-17:00 | ______________ |
| Ký hiệu nghỉ F (phép năm) | màu xanh |
| Ký hiệu nghỉ S (ốm) | màu vàng |
| NV Nguyễn Văn A — Phòng KD | Mã: NV-_______ |
| NV Trần Thị B — Phòng NS (Manager) | Mã: NV-_______ |
| NV Lê Văn C — Công nhân | Mã: NV-_______ |
| HĐLĐ HD-1 của NV A | ______________ |
| HĐLĐ HD-3 của NV B | ______________ |
| Đơn nghỉ phép test (3 ngày) | ID: __________ |
| Kỳ lương Tháng 04/2026 | ______________ |

---

## 🔑 Thông tin truy cập

| Mục | Giá trị |
|-----|---------|
| URL Local Dev (FE) | `https://localhost-app.nextx.vn:3000` |
| URL Tenant Portal | `https://app-dev.nextx.vn/vi/workspace/hr` |
| URL Sandbox | `https://nextx-tenant.sandbox.nextx.vn/vi/workspace/hr` |
| Workspace test sẵn sàng | **Nextx-Demo** — ID `019d8f70-3dac-73fa-9a9e-d1583a966e8d` |
| Tài khoản test chính (NV002 — HR Staff) | `leanhluong2001@gmail.com` / `Luong2001@` |
| Tài khoản HR Manager | `<hr-manager@workspace.test>` / `<password>` *(yêu cầu admin cấp riêng)* |
| Tài khoản Manager | `<manager@workspace.test>` / `<password>` *(yêu cầu admin cấp riêng)* |
| Tài khoản NV thường | `<staff@workspace.test>` / `<password>` *(yêu cầu admin cấp riêng)* |
| Tài khoản Công nhân | `<worker@workspace.test>` / `<password>` *(yêu cầu admin cấp riêng)* |

> **Trước khi bắt đầu:**
> - Tài khoản `leanhluong2001@gmail.com` đã linked với NV002 (Trần Thị B — Phòng Nhân Sự) và có đủ data để test **40/41 màn FE** — dùng được ngay.
> - Nếu cần test multi-persona (HR Manager / Manager / Worker riêng biệt) — liên hệ admin IAM cấp thêm tài khoản trong cùng workspace Nextx-Demo.

---

## 🧭 Thứ tự test (phải tuân theo — có dependency)

```
1  → Đăng nhập                  (vào hệ thống)
2  → Master Data                 (cấu hình trước tiên: org, position, shift, leave symbol, salary rank, legal params)
3  → Cấu hình chính sách (M14)   (Payroll Policy, Leave Policy, Approval Flow — BẮT BUỘC trước payroll/leave)
4  → Hồ sơ nhân viên             (tạo ít nhất 3 NV: HR, Manager, Staff, Worker)
5  → Hợp đồng                    (tạo HĐ-1/HĐ-2 cho NV mới, sau đó HĐ-3 cho NV đã pass thử việc)
6  → Chấm công                   (setup tháng, gán ca, nhập CC test)
7  → Nghỉ phép                   (NV gửi đơn, Manager duyệt, xem trong Unified Inbox)
8  → Bảng lương                  (tính lương sau khi có CC + HĐ)
9  → Bổ sung lương               (Other Income + Deductions)
10 → Tuyển dụng (ATS)            (độc lập)
11 → Onboarding                  (độc lập)
12 → Performance                 (cycle + OKR + review + 1-1)
13 → Phúc lợi                    (BHXH history, TNLĐ)
14 → Báo cáo                     (xem sau khi có data)
15 → Design System               (test dark mode, Ctrl+K cuối cùng)
```

---

# 📍 PHẦN 1: Bắt đầu

## 1.1 Đăng nhập thành công (happy path)

**Mục tiêu:** Đăng nhập bằng email/password đúng, vào được workspace HR.
**Data chuẩn bị:** Tài khoản HR Manager đã có sẵn workspace HRM.

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Mở trình duyệt → truy cập `https://app-dev.nextx.vn/vi/login` | Trang login hiển thị: logo NextX, ô `<Email>`, ô `<Mật khẩu>`, nút `<Đăng nhập>` |
| 2 | Nhập `<Email>` = `<hr-manager@workspace.test>` | Ô email hiển thị đúng |
| 3 | Nhập `<Mật khẩu>` | Ô mật khẩu hiển thị dạng ●●●●●●●● |
| 4 | Bấm nút `<Đăng nhập>` | Hiện loading spinner 1–2s |
| 5 | Quan sát URL sau khi load xong | Tự chuyển sang `/vi/workspace/hr` |
| 6 | Quan sát header trên cùng | Hiện tên user, tên workspace, icon avatar |
| 7 | Quan sát sidebar trái | Hiện đầy đủ menu: Dashboard, **Trang cá nhân** (en: My page), Hộp duyệt, Nhân viên, Sơ đồ tổ chức, Tuyển dụng, Onboarding, Performance, Hợp đồng, Chấm công, Nghỉ phép, Bảng lương, Phúc lợi, Báo cáo, Cài đặt. Menu "Tổng quan" thu gọn được bình thường khi đang ở page con (đã fix). |
| 8 | **(v4.0)** Quan sát sidebar có section **"Cá nhân"** với 6 menu: Hồ sơ của tôi · Tăng ca · Bổ sung lương · Khóa học của tôi · Phản hồi · Báo tai nạn | Section "Cá nhân" hiện ngay sau Dashboard |
| 9 | **(v4.0)** Quan sát góc dưới-phải | Floating bell icon hiện — chưa click → unread badge nếu có notification |
| 10 | **(v4.0)** Bấm `Ctrl+K` (Mac: `Cmd+K`) | Modal Command Palette mở giữa màn hình với 26 commands chia 7 nhóm |
| 11 | **(v4.0)** Gõ "phiếu lương" → chọn suggestion → Enter | Tự navigate tới `/me/profile` (tab Phiếu lương) |

**Kết quả tổng hợp:** ✅ Test PASS khi 11 bước đều đạt.

- [ ] 1.1 PASS

---

## 1.2 Sai mật khẩu

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Truy cập trang login | Trang login hiển thị |
| 2 | Nhập email đúng + mật khẩu sai | — |
| 3 | Bấm `<Đăng nhập>` | Toast đỏ: "Email hoặc mật khẩu không đúng" |
| 4 | Quan sát URL | Vẫn ở `/vi/login`, không redirect |
| 5 | Ô email giữ nguyên, ô mật khẩu bị reset rỗng | ✅ |

- [ ] 1.2 PASS

---

## 1.3 Email không tồn tại

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Nhập email lạ `notexist@test.com` + mật khẩu bất kỳ | — |
| 2 | Bấm `<Đăng nhập>` | Toast đỏ: "Email hoặc mật khẩu không đúng" (KHÔNG được lộ email không tồn tại) |

- [ ] 1.3 PASS

---

## 1.4 Validate trường rỗng

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Để trống cả 2 ô → bấm `<Đăng nhập>` | Dưới ô email "Vui lòng nhập email", dưới password "Vui lòng nhập mật khẩu" |
| 2 | Nhập `abc` (không @) → bấm `<Đăng nhập>` | Dưới ô email "Email không đúng định dạng" |

- [ ] 1.4 PASS

---

## 1.5 Chọn workspace (nếu user có nhiều)

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Đăng nhập tài khoản có **>1 workspace** | Sau login → trang chọn workspace |
| 2 | Bấm chọn workspace HRM | Redirect vào `/vi/workspace/hr` |
| 3 | Bấm avatar → `<Đổi workspace>` | Quay về trang chọn |

- [ ] 1.5 PASS

---

## 1.6 Đăng xuất

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Bấm avatar góc phải trên | Dropdown: Hồ sơ, Đổi workspace, Đăng xuất |
| 2 | Bấm `<Đăng xuất>` | Redirect `/vi/login`, toast "Đã đăng xuất" |

- [ ] 1.6 PASS

---

# 👤 PHẦN 2: Persona Nhân viên văn phòng (Self-Service)

> **Đối tượng:** Nguyễn Văn A — NV Phòng Kinh doanh, không có quyền quản lý.
> **Tài khoản:** `<staff@workspace.test>`
> **Thời gian test:** ~1.5h
> **Mục tiêu:** Test các tính năng dành cho NV thường — xem hồ sơ, phiếu lương, gửi đơn nghỉ.

---

## 2.1 Xem dashboard cá nhân

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Đăng nhập tài khoản NV → vào `/workspace/hr` | Dashboard cá nhân hiển thị (KHÔNG phải dashboard HR) |
| 2 | Quan sát các thẻ KPI | Có: Số ngày phép F còn lại, Số đơn pending của tôi, Phiếu lương tháng gần nhất |
| 3 | Quan sát ô Action Items | "Bạn có 0 đơn cần bổ sung thông tin" hoặc tương tự |
| 4 | Quan sát bộ lọc thời gian (period filter) | Có các option: **Hôm nay**, **Tuần này**, **Tháng này**, **Quý này**, **Năm nay**, **Tùy chọn (CUSTOM)** *(mới — v3.3)* |
| 5 | Chọn `<Tùy chọn>` | Hiện 2 ô date picker: **Từ ngày** và **Đến ngày** |
| 6 | Chọn range 01/05/2026 → 14/05/2026 → bấm `<Áp dụng>` | KPI cập nhật theo range custom; không lỗi |
| 7 | Thử chọn range > 366 ngày | Validation lỗi: "Khoảng thời gian không được vượt quá 366 ngày" |

- [ ] 2.1 PASS

---

## 2.2 Xem hồ sơ của tôi (`/hr/me`)

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Bấm menu **Trang cá nhân** (en: *My page*, trước đây là "Của tôi") | Trang `/workspace/hr/me` mở với 4 tabs: Hồ sơ, Phiếu lương, Đơn của tôi, Số dư phép. KHÔNG còn lỗi 500 "Invalid time value" khi có salary supplement (đã fix). |
| 2 | Tab **Hồ sơ** | Hiển thị: Mã NV, Họ tên, Phòng ban, Chức danh, Ngày vào làm, Email, Số ĐT |
| 3 | Quan sát các trường nhạy cảm (LCB, lương đóng BH) | KHÔNG hiển thị — vì NV không có quyền xem lương của mình ngoài phiếu lương |
| 4 | Bấm nút `<Yêu cầu sửa thông tin>` (nếu có) | Mở form Feedback → chọn loại `INFO_CHANGE` |

- [ ] 2.2 PASS

---

## 2.3 Xem phiếu lương tháng

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Tab **Phiếu lương** | Danh sách phiếu lương các tháng đã có |
| 2 | Bấm tháng gần nhất | Phiếu lương HTML 3 section: |
| 3 | Section 1 — Thông tin NV + Mức lương/PC | Hiện đủ: Mã NV, Họ tên, Bộ phận, Chức danh, Bậc lương, LCB, PC003, PC002, PC001, KPI base |
| 4 | Section 2 — Chấm công | Ngày TC, Ngày hưởng lương, Đi trễ (phút/lần), Về sớm (phút/lần), Giờ đêm, OT1-OT6, Phép F còn lại |
| 5 | Section 3 — Thu nhập − Giảm trừ = Thực nhận | LCB tháng, PC tháng, Lương đêm, Thưởng CC, Thưởng KPI, OT 6 loại, Other Income, **GROSS** |
| 6 | | BH NLĐ (BHXH/BHYT/BHTN/CĐ), Phạt trễ/sớm, Khấu trừ khác, PIT |
| 7 | | **THỰC NHẬN** in đậm |
| 8 | Bấm nút `<Tải PDF>` (nếu có) | Tải file PDF (chưa có thì hiện "Coming soon") |

- [ ] 2.3 PASS

---

## 2.4 Xem số dư phép

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Tab **Số dư phép** | Hiển thị: Phép F còn lại (X ngày), MP còn (nếu nữ, max 2.5/năm), RO, SA |
| 2 | Lịch tích lũy phép | Mỗi tháng từ khi ký HĐLĐ tự +1 ngày F |

- [ ] 2.4 PASS

---

## 2.5 Gửi đơn nghỉ phép

**Mục tiêu:** NV gửi đơn nghỉ 3 ngày (15-17/05/2026) với ký hiệu F (phép năm).

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Vào menu **Nghỉ phép** (`/workspace/hr/leave`) | Trang Nghỉ phép mở: danh sách đơn của tôi |
| 2 | Bấm `<+ Tạo đơn nghỉ>` | Form mở |
| 3 | Chọn `<Từ ngày>` = `15/05/2026` | — |
| 4 | Chọn `<Đến ngày>` = `17/05/2026` | — |
| 5 | Chọn `<Loại nghỉ>` = `Nguyên ngày (FULL_DAY)` | — |
| 6 | Chọn `<Ký hiệu nghỉ>` = `F (Phép năm)` | — |
| 7 | Nhập `<Lý do>` = `Đi du lịch gia đình` | — |
| 8 | Quan sát ô **Phép F còn lại** | Readonly — hiển thị số dư hiện tại |
| 9 | Bấm `<Gửi đơn>` | **HrmSuccessCheck** animation (icon ✅ pulse 1.5s) → Toast xanh: "Đã gửi đơn nghỉ phép" |
| 10 | Quan sát danh sách | Đơn vừa tạo hiện với trạng thái `PENDING` |
| 11 | Bấm vào đơn | Mở chi tiết — thấy luồng duyệt: NLĐ → TBP → HR (2 bước chờ) |
| 12 | **(v4.0) Test mobile card layout** — mở DevTools chế độ responsive 390px → vào `/hr/leave` | Bảng đơn nghỉ chuyển sang **card stack** (mỗi đơn 1 card), sidebar thu thành hamburger drawer |
| 13 | **(v4.0) Test HrmEmployeeSelect search** — nếu là HR mở form đơn nghỉ thay NV | Picker `<Nhân viên>` cho gõ "trần" → match NV002 Trần Thị B với highlight |
| 14 | **(v4.0) Test role-based view** — đăng nhập NV thường → vào `/hr/leave` | Chỉ thấy đơn của mình; **không có cột "Nhân viên"** + **không có nút "Duyệt/Từ chối"** |

- [ ] 2.5 PASS

---

## 2.6 Xem đơn của tôi

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Vào `/workspace/hr/me` → tab **Đơn của tôi** | Danh sách tất cả đơn đã gửi (nghỉ phép, OT, bổ sung lương) |
| 2 | Filter theo status `PENDING` | Chỉ hiện đơn chờ duyệt |
| 3 | Bấm 1 đơn | Mở chi tiết — xem được lịch sử duyệt từng bước |

- [ ] 2.6 PASS

---

## 2.7 Gửi đơn vượt phép — phải báo lỗi

**Mục tiêu:** Verify hệ thống không cho gửi đơn vượt số phép tích lũy.

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Tạo đơn nghỉ 30 ngày liên tục (vượt phép F còn) | Khi bấm `<Gửi>` → toast đỏ: "Còn X ngày phép, đăng ký vượt" |
| 2 | Đơn không được tạo | — |

- [ ] 2.7 PASS

---

## 2.8 Gửi đơn ngày kết thúc < ngày bắt đầu

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Form đơn: từ `20/05/2026` đến `15/05/2026` | — |
| 2 | Bấm `<Gửi>` | Validation: "Ngày kết thúc phải sau ngày bắt đầu" (422) |

- [ ] 2.8 PASS

---

## 2.9 Gửi feedback đến HR

**Mục tiêu:** NV thắc mắc về công tháng → gửi feedback PAYROLL_QUERY.

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Vào menu **Feedback** hoặc nút `<Gửi ý kiến>` | Form feedback mở |
| 2 | Chọn `<Loại>` = `Thắc mắc lương (PAYROLL_QUERY)` | Hiện thêm ô chọn tháng tham chiếu |
| 3 | Chọn `<Tháng tham chiếu>` = `04/2026` | — |
| 4 | Nhập `<Nội dung>` = `Tôi thấy số OT tháng 4 thiếu 2 buổi, vui lòng kiểm tra` | — |
| 5 | Bấm `<Gửi>` | **HrmSuccessCheck** animation → Toast xanh: "Đã gửi ý kiến đến HR" |
| 6 | Vào danh sách feedback của tôi (menu **Cá nhân → Phản hồi** = `/me/feedback`) | Feedback vừa gửi hiện với status `PENDING`. Đây là role-based view — NV chỉ thấy feedback của mình |
| 7 | **(v4.0)** Đăng nhập HR Manager → vào `/hr/feedback` | HR thấy TẤT CẢ feedback của workspace, bao gồm feedback NV vừa gửi |

- [ ] 2.9 PASS

---

## 2.10 Tính lương sai/đúng — đối chiếu với phiếu

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Tính tay theo công thức để verify GROSS:<br>`GROSS = LCB_tháng + Lương_đêm + Thưởng_CC + Σ_PC + Thưởng_KPI + Σ_OT + Other_Income − Phạt_trễ_sớm` | Kết quả khớp với phiếu lương trong hệ thống |
| 2 | Tính BH NLĐ:<br>`BH = (8% + 1.5% + 1% + 0.5%) × Lương_BH = 11% × Lương_BH` | Khớp |
| 3 | Tính PIT (lũy tiến 5 bậc) | Khớp |
| 4 | Thực nhận = GROSS - BH - PIT - Khấu trừ | Khớp |

> ⚠️ Sai lệch dù chỉ 1 đồng cũng phải báo bug — đây là số liệu pháp lý.

- [ ] 2.10 PASS

---

# 👨‍💼 PHẦN 3: Persona Trưởng phòng (Manager)

> **Đối tượng:** Trần Thị B — Trưởng phòng Nhân sự.
> **Tài khoản:** `<manager@workspace.test>`
> **Thời gian test:** ~2.5h
> **Mục tiêu:** Test workflow duyệt đơn qua Unified Inbox, quản lý team.

---

## 3.1 Truy cập Unified Inbox

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Đăng nhập Manager → vào `/workspace/hr` | Dashboard Manager |
| 2 | Bấm menu **Hộp duyệt (Inbox)** | Trang `/workspace/hr/inbox` mở |
| 3 | Quan sát danh sách | Hiển thị TẤT CẢ loại đơn: Nghỉ phép, OT, Bổ sung lương, Yêu cầu sửa thông tin |
| 4 | Filter mặc định | Hiển thị `PENDING` — chỉ đơn chưa duyệt |
| 5 | Số badge bên cạnh menu Inbox | Khớp với số đơn pending |

- [ ] 3.1 PASS

---

## 3.2 Duyệt đơn nghỉ phép

**Mục tiêu:** Manager duyệt đơn của NV A (đã gửi ở 2.5).

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Trong Inbox → tìm đơn của NV A | Hiện đơn 15-17/05/2026, ký hiệu F |
| 2 | Bấm vào đơn | Mở detail panel bên phải |
| 3 | Quan sát thông tin | NV, ngày, loại, lý do, số dư phép trước đơn |
| 4 | Bấm `<Duyệt>` | Dialog confirm + ô ghi chú |
| 5 | Nhập note `Đã trao đổi qua điện thoại, OK` → bấm `<Xác nhận>` | Toast xanh: "Đã duyệt đơn" |
| 6 | Đơn chuyển sang status `APPROVED` (bước 2/2 — Manager) | — |
| 7 | Đơn còn 1 bước duyệt HR — chờ HR duyệt cuối | — |

- [ ] 3.2 PASS

---

## 3.3 Từ chối đơn nghỉ với lý do

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | NV A gửi 1 đơn khác (giả lập) → manager mở | — |
| 2 | Bấm `<Từ chối>` | Dialog yêu cầu lý do |
| 3 | Để trống lý do → bấm `<Xác nhận>` | Validation: "Vui lòng nhập lý do từ chối" |
| 4 | Nhập lý do `Tuần đó có cuộc họp quan trọng` | — |
| 5 | Bấm `<Xác nhận>` | Toast: "Đã từ chối đơn" |
| 6 | NV A nhận notification | (kiểm tra sau ở tài khoản NV) |

- [ ] 3.3 PASS

---

## 3.4 Yêu cầu bổ sung thông tin

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Mở đơn pending | — |
| 2 | Bấm `<Yêu cầu bổ sung>` | Dialog yêu cầu mô tả |
| 3 | Nhập `Vui lòng đính kèm vé máy bay` → bấm `<Gửi>` | Đơn quay về NV với note bổ sung |
| 4 | NV nhận notification → bổ sung → đơn quay lại Inbox Manager | (verify sau) |

- [ ] 3.4 PASS

---

## 3.5 Bulk approve + Quick row actions + Delegation (cập nhật v4.0)

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Trong Inbox — tick **checkbox** ở cột đầu cho 3 đơn cùng loại (vd: 3 đơn nghỉ phép) | **Floating action bar** ở bottom: "Đã chọn 3 đơn" với 2 nút `<Duyệt 3>` / `<Từ chối 3>` |
| 2 | Bấm `<Duyệt 3>` | Dialog confirm |
| 3 | Bấm `<Xác nhận>` | Toast: "Đã duyệt 3 đơn" |
| 4 | Refresh list | Cả 3 đơn chuyển sang APPROVED |
| 5 | **(v4.0) Quick row action** — hover row đơn → cuối row hiện inline **✓ / ✗** | Click ✓ duyệt nhanh 1 đơn không cần mở detail; ✗ mở dialog từ chối |
| 6 | **(v4.0) Delegation** — bấm avatar góc phải → `<Uỷ quyền duyệt>` | Form delegate: chọn người + date range + scope (loại đơn: nghỉ phép/OT/...) |
| 7 | Lưu delegation → đăng nhập delegate user → Inbox | Delegate thấy đơn của Manager trong khoảng date range |

- [ ] 3.5 PASS

---

## 3.6 Xem hồ sơ NV trong team

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Vào menu **Nhân viên** | Hiện danh sách NV — Manager chỉ thấy NV trong phòng/bộ phận của mình (không thấy NV phòng khác) |
| 2 | Bấm 1 NV trong team | Mở detail hồ sơ — 9 tab |
| 3 | Tab "Lương" | Manager CÓ thể xem (hoặc KHÔNG tùy permission setup) |
| 4 | Tab "Liên hệ & NPT" | Hiển thị NPT (nếu có) |

> ⚠️ Permission quy định Manager xem lương hay không tùy workspace setup — hỏi HR trước.

- [ ] 3.6 PASS

---

## 3.7 Xem OKR của team

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Vào menu **Performance** | Hiện cycle hiện tại |
| 2 | Chọn tab `<OKR>` | Cây OKR: Company → Department (của mình) → Individual |
| 3 | Bấm vào OKR của 1 NV team | Xem chi tiết: Objective, Key Results (% progress) |

- [ ] 3.7 PASS

---

## 3.8 Tạo 1-1 meeting với NV

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Vào `/workspace/hr/performance` → tab **1-1** | — |
| 2 | Bấm `<+ Lên lịch 1-1>` | Form mở |
| 3 | Chọn NV (trong team) | — |
| 4 | Chọn ngày + giờ | — |
| 5 | Quan sát template | 4 section: Highlights, Challenges, Action Items, Next Steps |
| 6 | Lưu | NV nhận notification lịch hẹn |

- [ ] 3.8 PASS

---

## 3.9 Đánh giá 360 cho NV trong team

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Trong cycle review hiện hành → bấm `<Đánh giá>` cho 1 NV team | Form 7 tiêu chí (mỗi cái 0–100) |
| 2 | Cho điểm từng tiêu chí + ghi chú | — |
| 3 | **(v4.0) Test auto-save draft** — gõ vài ghi chú → đóng tab → quay lại form | Toast vàng "Đã khôi phục bản nháp" + giá trị đã gõ vẫn còn |
| 4 | Bấm `<Lưu>` | **HrmSuccessCheck** animation → Toast: "Đã lưu đánh giá" |
| 5 | Đánh giá đã lưu KHÔNG sửa được — chỉ tạo mới | Nếu cố sửa → ô input disabled |

- [ ] 3.9 PASS

---

# 👩‍💼 PHẦN 4: Persona HR Manager

> **Đối tượng:** HR Manager — quyền cao nhất trong workspace.
> **Tài khoản:** `<hr-manager@workspace.test>`
> **Thời gian test:** ~6-8h
> **Mục tiêu:** Test toàn bộ workflow setup workspace + vận hành HR.

> ⚠️ **TUÂN THEO THỨ TỰ** dependency: Master Data → Config → Employee → Contract → Attendance → Payroll.

---

## 4.1 Setup Master Data — Cây tổ chức (Tree view MỚI v4.0)

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Vào `/workspace/hr/org-chart` → tab **Phòng ban** | **(v4.0)** Hiển thị Tree view với connector lines `├─ └─ │` (giống file explorer) thay vì layout phẳng |
| 2 | Quan sát toolbar | Có nút `<Mở rộng tất cả>` / `<Thu gọn>` + **Quick stats** `{units} đơn vị · {employees} NV · {levels} cấp` |
| 3 | Quan sát icons & badges | **🏢** company L1 (badge **purple**) · **💼** department L2 (badge **blue**) · **📁** team L3 (badge **green**) · L4+ **amber** |
| 4 | Click button **▶** trên 1 node có con | Node expand hiện con; icon đổi sang **▼** |
| 5 | Bấm `<Mở rộng tất cả>` | Toàn bộ tree expand đệ quy |
| 6 | Gõ "kinh doanh" trong **search inline** | Match node có chữ "Kinh doanh" → highlight `<mark>` + tự expand parent chứa kết quả |
| 7 | Click **employee count badge** trên 1 node (vd "5 NV") | Filter danh sách Nhân viên theo phòng ban đó |
| 8 | Bấm `<+ Tạo đơn vị>` → cha = NULL (root) | Form tạo đơn vị |
| 9 | Tạo "Chi nhánh Hà Nội" (level 1) | Xuất hiện trong cây với icon 🏢 + badge purple |
| 10 | Bấm vào node Hà Nội → `<+ Thêm con>` | — |
| 11 | Tạo "Khối Vận hành" (level 2) | — |
| 12 | Tiếp tục tạo 6 cấp: Khối → Ban → Phòng → Bộ phận → Tổ | Mỗi cấp đều xuất hiện đúng vị trí với connector line + tier badge |
| 13 | Bộ phận (level 5) — nhập **cost center** = `CC-001` | Lưu được |
| 14 | Tạo 1 đơn vị trùng tên ở chi nhánh khác | Cảnh báo nhưng cho phép |
| 15 | Tạo 1 đơn vị trùng MÃ | Block 409 Conflict |

- [ ] 4.1 PASS

---

## 4.2 Setup Master Data — Chức danh

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Vào `/workspace/hr/settings` → tab **Chức danh** | — |
| 2 | Bấm `<+ Tạo chức danh>` | Form |
| 3 | Tạo "Trưởng phòng" — employeeType = STAFF | Lưu thành công |
| 4 | Tạo "Nhân viên" — STAFF | — |
| 5 | Tạo "Công nhân kỹ thuật" — WORKER | — |
| 6 | Sửa thuộc tính `attendanceBonus` = `400.000` cho STAFF, `600.000` cho WORKER | Lưu |

- [ ] 4.2 PASS

---

## 4.3 Setup Master Data — Bậc lương

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Tab **Bậc lương** | — |
| 2 | Tạo bậc `WK1` (group = WK) — entry năm 2025: `5.260.000` | — |
| 3 | Thêm entry năm 2026 cho WK1: `5.630.000` | Bậc lưu 2 năm |
| 4 | Tạo bậc `SF1.0` (group = SF1) — 2025: `8tr`, 2026: `8.5tr` | — |
| 5 | Cố tạo trùng `(WK1, 2026)` | Block 409 |

- [ ] 4.3 PASS

---

## 4.4 Setup Master Data — Ca làm việc

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Tab **Ca làm việc** | — |
| 2 | Tạo ca `CaHanhChinh` — 08:00 - 17:00, nghỉ trưa 60p | — |
| 3 | Tạo ca `CA1` — 06:00 - 14:00 | — |
| 4 | Tạo ca `CA2` — 14:00 - 22:00 | — |
| 5 | Tạo ca `CA3` — 22:00 - 06:00 hôm sau (overnight) | Tick `isOvernight` |
| 6 | Tạo ca `CaTS` (thai sản — 8:00-15:30) | — |
| 7 | Sửa giờ ca | Cập nhật được |

- [ ] 4.4 PASS

---

## 4.5 Setup Master Data — Ký hiệu nghỉ

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Tab **Ký hiệu nghỉ** | — |
| 2 | Tạo `F` (Phép năm, paid, màu xanh) | — |
| 3 | Tạo `MP` (Kinh nguyệt, paid, màu hồng) — chỉ áp dụng giới tính Nữ | — |
| 4 | Tạo `S` (Ốm, sick, màu vàng) | — |
| 5 | Tạo `M` (Thai sản, paid, màu tím) — chỉ Nữ | — |
| 6 | Tạo `B` (Bệnh nghề nghiệp) / `T` (Tử thân) / `W` (Cưới) / `D` (Việc riêng) | — |
| 7 | Tạo `RO` (Nghỉ bù) / `SA` (Nghỉ T7) | — |

- [ ] 4.5 PASS

---

## 4.6 Setup Master Data — Định mức pháp luật

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Tab **Định mức pháp luật** (hoặc `/payroll-settings`) | — |
| 2 | Quan sát các giá trị mặc định 2026 | LTT vùng I: `4.730.000` · BHXH 8/17.5% · BHYT 1.5/3% · BHTN 1/1% · CĐ 0.5/2% |
| 3 | Quan sát giảm trừ PIT | Bản thân: `15.500.000` · NPT: `6.200.000` |
| 4 | Quan sát tỷ lệ OT | OT1=150% · OT2=200% · OT3=200% · OT4=270% · OT5=300% · OT6=390% · Đêm: 130% |
| 5 | Bấm `<Sửa>` → đổi LTT vùng I = `5.000.000` (giả lập) → `<Lưu>` | Toast xanh + audit log mới |
| 6 | Quan sát: tháng đã chốt lương KHÔNG bị hồi tố | — |

- [ ] 4.6 PASS

---

## 4.7 Setup chính sách Payroll Policy

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Vào `/workspace/hr/payroll-policy` | Form policy |
| 2 | Cấu hình `nightShiftBonusPercent` = `30%` | — |
| 3 | Cấu hình `attendanceBonusStaff` = `400.000` / `Worker` = `600.000` | — |
| 4 | Cấu hình `latePenaltyMinutesBlock` = `15` (phút) | — |
| 5 | Cấu hình `otCapPerYear` = `300` (giờ — Luật LĐ mới) | — |
| 6 | Bấm `<Lưu>` | Toast xanh |
| 7 | Nếu chưa lưu policy → KHÔNG tính lương được | — |

- [ ] 4.7 PASS

---

## 4.8 Setup Leave Policy

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Vào `/workspace/hr/leave-policy` | — |
| 2 | Tạo policy "Default" áp dụng cho mọi NV | — |
| 3 | `phep_F_per_year` = `12` (1/tháng) | — |
| 4 | `phep_MP_per_year` = `2.5` (chỉ nữ) | — |
| 5 | `carry_over_max` = `5` (chuyển tồn cuối năm) | — |
| 6 | Lưu | — |

- [ ] 4.8 PASS

---

## 4.9 Setup Approval Flow

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Vào `/workspace/hr/approval-flows` | Danh sách flow |
| 2 | Tạo flow "Đơn nghỉ — Staff" với 2 bước: TBP → HR | — |
| 3 | Tạo flow "Đơn nghỉ — Worker" với 3 bước: Tổ trưởng → TBP → HR | — |
| 4 | Tạo flow "Đơn nghỉ — TBP" với 2 bước: BOD → HR | — |
| 5 | Mỗi bước có thể chọn `approverType` (Role hoặc Specific User) | — |
| 6 | Lưu | — |

- [ ] 4.9 PASS

---

## 4.10 Tạo NV mới (Staff)

**Mục tiêu:** Tạo NV Nguyễn Văn A — Phòng KD, chức danh NV, Staff.

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Vào `/hr/employees` (canonical mới — `/staff` cũ tự redirect) → `<+ Thêm nhân viên>` | Form mở |
| 2 | Nhập **Họ tên** = `Nguyễn Văn A` | — |
| 3 | **Giới tính** = Nam, **Ngày sinh** = `01/01/1990` | — |
| 4 | **Phòng ban** — picker `<HrmOrgUnitSelect>` cho gõ "kinh doanh" → chọn Phòng KD; **Chức danh** — picker `<HrmPositionSelect>` cho search → chọn Nhân viên | **(v4.0)** Cả 2 picker có search box + highlight match |
| 5 | **Loại NV** tự gán = STAFF (từ chức danh) | — |
| 6 | **Ngày vào làm** = `01/05/2026` | — |
| 7 | **CCCD** = `001090000001`, **Bank** = `0123456789`, **MST** = `8512345678` | — |
| 8 | Bấm `<Lưu>` | **HrmSuccessCheck** animation → Toast xanh + sinh mã NV `NV-202605-00001` |
| 9 | Đồng thời tự tạo HĐ-1 (Thỏa thuận thử việc) | Verify trong tab Hợp đồng của NV |
| 10 | **(v4.0) IAM Provisioning UI** — mở Employee detail | Banner vàng phía trên: "Chưa có tài khoản IAM" + button `<Cấp tài khoản>` |
| 11 | Click `<Cấp tài khoản>` | Dialog form — Email + Họ tên pre-fill từ employee |
| 12 | Submit | Toast "Đã cấp tài khoản"; banner biến mất. Nếu IAM service unreachable → toast vàng "Sẽ tạo khi IAM service sẵn sàng" (stub fallback) |

- [ ] 4.10 PASS

---

## 4.11 Tạo NV Manager + NV Công nhân

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Tạo **NV B** — Trần Thị B — Phòng NS — Chức danh = Trưởng phòng | Sinh mã NV-202605-00002 |
| 2 | Tạo **NV C** — Lê Văn C — Tổ Xưởng A — Chức danh = Công nhân kỹ thuật (WORKER) | Sinh mã NV-202605-00003 |
| 3 | NV B đồng thời tạo HĐ-1 (Staff Thử việc) | — |
| 4 | NV C đồng thời tạo HĐ-2 (Worker Học việc) | — |

- [ ] 4.11 PASS

---

## 4.12 Thêm quá trình lương & phụ cấp cho NV

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Mở chi tiết NV A → tab **Lương** → `<Thêm lịch sử lương>` | Form |
| 2 | Ngày hiệu lực = `01/05/2026`, LCB = `8.500.000`, Bậc lương = SF1.0 | — |
| 3 | Lưu | Lịch sử hiện trong timeline |
| 4 | Tab **Phụ cấp** → `<Thêm phụ cấp>` | Form |
| 5 | Thêm `PC002 — Nhà ở` = `1.500.000`, hiệu lực `01/05/2026` | — |
| 6 | Tab **Bảo hiểm** → `<Thêm lịch sử BHXH>` | — |
| 7 | Mức lương đóng BH = `7.000.000` (≥ LTT vùng I) | — |
| 8 | Cố nhập BH = `3.000.000` (< LTT) | 422 Validation |

- [ ] 4.12 PASS

---

## 4.13 Thêm NPT (Người phụ thuộc)

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Tab **Liên hệ & NPT** → `<+ Thêm liên hệ>` | — |
| 2 | Họ tên = `Nguyễn Thị D`, Quan hệ = `Con`, `isPit` = ✅ | — |
| 3 | Lưu | NPT có isPit → mỗi tháng giảm 6.2tr khi tính PIT |
| 4 | Verify tại phiếu lương tháng sau | Giảm trừ = 15.5tr + 6.2tr × 1 NPT = 21.7tr |

- [ ] 4.13 PASS

---

## 4.14 Đánh giá thử việc + Tạo HĐ-3

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | NV A đã thử việc 2 tháng → vào tab **Hợp đồng** → HĐ-1 | — |
| 2 | Bấm `<Đánh giá>` | Form 7 tiêu chí |
| 3 | Cho điểm: 85, 90, 88, 85, 92, 87, 80 (avg ≈ 86.7 — A) | — |
| 4 | Quyết định = `PASSED` | — |
| 5 | Lưu | Tự gợi ý tạo HĐ-3 |
| 6 | Tạo HĐ-3 — `12 tháng`, ngày bắt đầu = ngày tiếp theo của TV | — |
| 7 | HĐ-1 tự chuyển `EXPIRED`, HĐ-3 `ACTIVE` | — |

- [ ] 4.14 PASS

---

## 4.15 Setup chấm công tháng 05/2026

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Vào `/workspace/hr/attendance-settings` | — |
| 2 | Chọn tháng `05/2026` | — |
| 3 | Pick ngày nghỉ tuần — tick toàn bộ Chủ Nhật (5 CN) | — |
| 4 | Pick ngày lễ — `30/04` + `01/05` (Quốc tế LĐ) | — |
| 5 | StandardWorkdayType = `TYPE1` | Tự tính = 31 - 5 (CN) - 2 (lễ) = 24 ngày TC |
| 6 | Lưu | — |
| 7 | Cố sửa tháng đã chốt → 409 | — |

- [ ] 4.15 PASS

---

## 4.16 Xếp ca tháng 05/2026

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Vào `/workspace/hr/shift` → tab Assignment | Bảng NV × Ngày |
| 2 | Chọn NV A (Staff) → click ô D1 → chọn `CaHanhChinh` | Cell hiện "Hành chính" |
| 3 | Bulk: chọn D1-D31 (trừ CN + lễ) → gán hàng loạt CaHanhChinh | — |
| 4 | NV C (Worker) → CN+: xếp luân phiên CA1/CA2/CA3 | — |
| 5 | Import Excel format LSEV | Test với file mẫu |
| 6 | Cố gán 2 ca cùng ngày 1 NV → 409 | — |

- [ ] 4.16 PASS

---

## 4.17 Nhập chấm công thủ công

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Vào `/workspace/hr/attendance` | Danh sách CC theo ngày |
| 2 | Chọn ngày `13/05/2026` | — |
| 3 | Tìm NV A → bấm `<Thêm record>` | Form |
| 4 | Nhập IN = `07:55`, OUT = `17:30` | — |
| 5 | Source = `MANUAL` (vì không có máy CC test) | — |
| 6 | Lưu | Record hiện với màu xám (source manual) |
| 7 | Cố xóa raw record | Block — không cho xóa, chỉ đánh dấu invalid |

- [ ] 4.17 PASS

---

## 4.18 Tính tổng hợp công tháng

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Vào `/workspace/hr/attendance-summary` | — |
| 2 | Chọn tháng `05/2026` → bấm `<Tính tổng hợp>` | Async job — vài giây |
| 3 | Done → bảng tổng hợp hiện 30+ chỉ số/NV | Ngày TC, Giờ TC, Ngày thực tế, OT1-6, Giờ đêm, Phép F còn... |
| 4 | Bấm chi tiết NV A | Trang per-employee với từng ngày |
| 5 | Kiểm tra symbol auto-detection | NV A đi đúng giờ → ký hiệu `+` (100%, 8h) |
| 6 | Export Excel 4 loại báo cáo | Tải về thành công |

- [ ] 4.18 PASS

---

## 4.19 Duyệt đơn nghỉ (HR — bước cuối)

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Đăng nhập HR Manager → Inbox | Hiện đơn nghỉ của NV A đã được Manager duyệt, chờ HR |
| 2 | Mở đơn → `<Duyệt>` | Đơn chuyển `APPROVED` (3/3) |
| 3 | Hệ thống tự update attendance summary tháng 05 — 3 ngày 15-17/05 gán ký hiệu `F` | — |

- [ ] 4.19 PASS

---

## 4.20 Nhập Other Income (thu nhập khác)

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Vào `/workspace/hr/payroll-supplement` → chọn tháng `05/2026` | — |
| 2 | Tab **Thu nhập khác** → `<+ Thêm>` | — |
| 3 | NV A — type = `REFERRAL_BONUS` — `1.000.000đ` — `isTaxable = true` | — |
| 4 | NV B — type = `ANNIVERSARY_5` — `5.000.000đ` — `isTaxable = false` (luật miễn thuế) | — |
| 5 | Lưu | Hiện trong danh sách |

- [ ] 4.20 PASS

---

## 4.21 Nhập Deductions (khấu trừ)

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Tab **Khấu trừ khác** → `<+ Thêm>` | — |
| 2 | NV A — type = `ADVANCE` (tạm ứng) — `2.000.000đ` | — |
| 3 | Lưu | — |

- [ ] 4.21 PASS

---

## 4.22 Chọn PIT method

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Vào `/workspace/hr/payroll-settings` → tab PIT | — |
| 2 | Chọn method cho tháng `05/2026` = `PT2` (toàn bộ OT không tính thuế) | — |
| 3 | Lưu | — |

- [ ] 4.22 PASS

---

## 4.23 Tính lương tháng 05/2026

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Vào `/workspace/hr/payroll-processing` → chọn tháng `05/2026` | — |
| 2 | Tick "Overwrite kết quả cũ" (nếu có) | — |
| 3 | Bấm `<Tính lương>` | Spinner loading → vài giây sau done |
| 4 | Đi sang `/workspace/hr/payroll` | Bảng preview 50+ cột |
| 5 | Verify NV A: GROSS, BH NLĐ, PIT, THỰC NHẬN | Khớp với công thức tính tay |
| 6 | Quan sát Thưởng KPI | Nếu chưa có evaluation → KPI = 0 |
| 7 | Thưởng chuyên cần | Đủ ngày + dưới 3 lần trễ/sớm → có 400k (Staff) |

- [ ] 4.23 PASS

---

## 4.24 Chốt lương (Lock) + Reopen + Step 6 Phát hành (v4.0)

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Trong preview → bấm `<Chốt lương tháng 05>` | Dialog confirm |
| 2 | Tick "Tôi xác nhận" → `<Xác nhận>` | Toast xanh: "Đã chốt kỳ lương" |
| 3 | Cố sửa attendance_summary của tháng đã chốt → 409 | — |
| 4 | Bấm `<Mở lại (Reopen)>` | Dialog yêu cầu lý do |
| 5 | Nhập `Sửa thông tin OT của NV X` → `<Xác nhận>` | Audit log lưu reason + before/after |
| 6 | Kỳ lương quay về `unlocked` | — |
| 7 | **(v4.0) Step 6 — Phát hành phiếu lương** — chốt lại + bấm `<Tiếp tục Step 6>` | Wizard mở step Phát hành |
| 8 | Chọn channel `EMAIL` / `PORTAL` / `ESIGN` | — |
| 9 | Chọn `EMAIL` + bấm `<Phát hành>` | Bảng per-employee distribution status: ✅ Sent / ⏳ Pending / ❌ Failed cho từng NV |
| 10 | Chọn `ESIGN` + bấm `<Phát hành>` | Publish event `hrm.payslip.esign.requested` tới Contract service; kiểm tra RabbitMQ console hoặc Contract Inbox |

- [ ] 4.24 PASS

---

## 4.25 Export báo cáo lương (Preview trước Excel — v4.0)

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Vào `/workspace/hr/reports` hoặc `/payroll-statistics` | **(v4.0)** Mỗi card báo cáo nay có **2 button**: `<Xem trước>` + `<Tải Excel>` (thay cho 1 nút `<Tải xuống>` cũ) |
| 2 | Báo cáo `Bảng tổng hợp lương` → bấm `<Xem trước>` | Dialog mở table preview full data — verify rồi mới tải |
| 3 | Trong dialog preview → bấm `<Tải Excel>` | File `.xlsx` 50+ cột tải về với header **Linear accent color** + currency format `#,##0 đ` |
| 4 | Báo cáo `Bảng tính PIT` — preview + tải | — |
| 5 | Báo cáo `Bảng tính OT` — preview + tải | — |
| 6 | Báo cáo `Bank list Vietinbank` | File có STT, mã NV, họ tên, CCCD, số TK, thực lĩnh, tổng bằng chữ VI+EN |
| 7 | **(v4.0) Route mới** — vào `/hr/reports/bhxh` (canonical) — URL cũ `/reports-bhxh` tự redirect | Page Reports BHXH mở (placeholder Q3/2026) |
| 8 | **(v4.0) Route mới** — vào `/hr/reports/labor` | Page Reports Lao động (placeholder Q3/2026) |
| 9 | Xuất `Phiếu lương PDF cá nhân` cho NV A | PDF 3 section đẹp (hoặc 501 nếu chưa wire) |

- [ ] 4.25 PASS

---

## 4.26 Tạo Contract HĐLĐ in template

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Vào `/workspace/hr/contract` → NV A → HĐ-3 | Detail HĐ |
| 2 | Bấm `<In hợp đồng>` | Preview HTML |
| 3 | Bấm `<Xuất PDF>` | PDF song ngữ VI-EN |
| 4 | Bấm `<Xuất DOCX>` | 501 Not Implemented (coming soon) |
| 5 | Tab **Sắp hết hạn** → filter 30 ngày | Chỉ HĐ-3 hiện, không HĐ-4 |

- [ ] 4.26 PASS

---

## 4.27 Thôi việc + Checklist

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | NV A → bấm `<Thôi việc>` | Form |
| 2 | Ngày thôi việc = `15/12/2026`, Lý do = `Resigned`, Số QĐ = `QĐ-001` | — |
| 3 | Lưu | NV status → `TERMINATED`, tự sinh checklist |
| 4 | Tab **Checklist thu hồi** | 8 items: thẻ ID, đồng phục, laptop, bàn giao, đóng email, lương cuối, sổ BHXH, QĐ thôi việc |
| 5 | Tick từng item | Progress bar |
| 6 | Khi đủ 100% → nút `<Đóng tài khoản hệ thống>` enable | — |
| 7 | Cố đóng tài khoản khi checklist chưa đủ → Error | — |

- [ ] 4.27 PASS

---

## 4.28 Phản hồi Feedback

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Vào danh sách feedback | Hiện feedback của NV A (đã gửi ở 2.9) |
| 2 | Mở feedback `PAYROLL_QUERY` | — |
| 3 | Nhập response: `Đã kiểm tra, OT tháng 4 thiếu 2 buổi do quên gán ca, đã update.` | — |
| 4 | Bấm `<Gửi phản hồi>` | Status → `RESPONDED`. NV A nhận notification |

- [ ] 4.28 PASS

---

# 👷 PHẦN 5: Persona Công nhân (Worker)

> **Đối tượng:** Lê Văn C — Công nhân kỹ thuật Tổ Xưởng A
> **Tài khoản:** `<worker@workspace.test>`
> **Thời gian test:** ~1h
> **Mục tiêu:** Test các tính năng tối thiểu mà công nhân cần — chấm công, xem phiếu lương, gửi đơn qua Tổ trưởng.

> 💡 Công nhân thường ít dùng máy tính — chủ yếu chấm vân tay/face. Tới Q4/2026 sẽ có PWA mobile cho công nhân.

---

## 5.1 Đăng nhập (lần đầu — đổi mật khẩu)

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | NV C nhận email mật khẩu tạm từ HR | — |
| 2 | Đăng nhập lần đầu | Yêu cầu đổi mật khẩu |
| 3 | Đổi mật khẩu | Đăng nhập thành công |

- [ ] 5.1 PASS

---

## 5.2 Xem phiếu lương qua điện thoại

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Mở Chrome/Cốc Cốc trên điện thoại → truy cập URL portal | Layout mobile-friendly |
| 2 | Đăng nhập | — |
| 3 | Vào **Trang cá nhân** → tab **Phiếu lương** | Phiếu lương hiển thị rõ trên màn hình nhỏ |
| 4 | Bấm tháng mới nhất | Section 3 — THỰC NHẬN hiện in đậm to |

- [ ] 5.2 PASS

---

## 5.3 Xem chấm công của mình

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Vào `/hr/me` → tab `Số dư phép` hoặc `Chấm công của tôi` | — |
| 2 | Hiển thị: ngày làm trong tháng, ký hiệu công từng ngày | + (đủ), - (thiếu), F (phép), KP (vắng) |

- [ ] 5.3 PASS

---

## 5.4 Gửi đơn nghỉ qua Tổ trưởng

> 💡 NV C là công nhân — luồng duyệt **3 bước**: NLĐ → Tổ trưởng → TBP → HR.

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Tạo đơn nghỉ tương tự 2.5 — 1 ngày `20/05/2026` ký hiệu F | — |
| 2 | Gửi đơn | — |
| 3 | Trong tab `Đơn của tôi` → mở đơn | Luồng duyệt hiện: NV → **Tổ trưởng (chờ)** → TBP (chờ) → HR (chờ) |
| 4 | Đăng nhập tài khoản Tổ trưởng → duyệt | Đơn chuyển bước 2 |
| 5 | Đăng nhập TBP → duyệt | Đơn chuyển bước 3 |
| 6 | Đăng nhập HR → duyệt | APPROVED — attendance summary update F cho ngày 20/05 |

- [ ] 5.4 PASS

---

## 5.5 Gửi feedback "Thắc mắc về công"

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Gửi feedback type `ATTENDANCE_QUERY` tháng 04/2026: "Tôi đi làm ngày 15/04 mà không thấy trong phiếu" | — |
| 2 | HR nhận notification → phản hồi → NV nhận lại | Lịch sử 2 chiều |

- [ ] 5.5 PASS

---

# 📋 PHẦN 6: Module-by-module checks

Phần này bổ sung các trường hợp **edge case** cho từng module — dành cho QA test sâu.

---

## M02 — Master Data: Edge cases

| ID | Test case | Expected |
|---|---|---|
| M02-E01 | Xóa Phòng còn Bộ phận con | Block: "Không thể xóa — còn 3 đơn vị con" |
| M02-E02 | Xóa Bộ phận còn 5 NV | Block: "Không thể xóa — đang liên kết với 5 nhân viên" |
| M02-E03 | Tạo cây 11 cấp (>10) | Cảnh báo nhưng cho phép |
| M02-E04 | parentId trỏ về chính nó | 422 — circular reference |
| M02-E05 | Đổi `employeeType` của chức danh đang dùng | Cảnh báo về ảnh hưởng tính lương |
| M02-E06 | Tăng LTT vùng vào giữa tháng | Tháng đã chốt không bị hồi tố — chỉ áp tháng chưa chốt |

- [ ] M02 Edge cases PASS

---

## M03 — Hồ sơ NV: Edge cases

| ID | Test case | Expected |
|---|---|---|
| M03-E01 | Tạo NV với email trùng | 409 Conflict |
| M03-E02 | Tạo NV chưa có HĐ-1 → tự tạo HĐ-1 cùng transaction | OK |
| M03-E03 | NV chuyển bộ phận giữa tháng → tab Công việc lưu lịch sử | OK |
| M03-E04 | Thêm NPT trùng (cùng người 2 lần) | Cảnh báo |
| M03-E05 | NPT đăng ký ở 2 NV (vợ chồng đều khai con) | Cảnh báo — cần policy ngoài hệ thống |
| M03-E06 | Soft delete NV còn HĐ active | Block — phải TERMINATE trước |
| M03-E07 | Cố sửa thông tin NV đã TERMINATED | Block một số trường |

- [ ] M03 Edge cases PASS

---

## M04 — Hợp đồng: Edge cases

| ID | Test case | Expected |
|---|---|---|
| M04-E01 | HĐ-3 lần 3 cho 1 NV (đã có 2 HĐ-3 trước) | Cảnh báo — yêu cầu HĐ-4 |
| M04-E02 | Tạo HĐ-3 với endDate < startDate | 422 |
| M04-E03 | HĐ-3 với durationMonths = 18 (không thuộc {12,24,36}) | 422 |
| M04-E04 | Ký HĐ mới khi đã có HĐ active | Tự EXPIRE HĐ cũ |
| M04-E05 | Sửa HĐ đã ACTIVE | 409 — chỉ DRAFT mới sửa được |
| M04-E06 | Xóa HĐ đã ký (signedDate ≠ NULL) | Block — chỉ TERMINATE |
| M04-E07 | Đánh giá NV đã TERMINATED | 409 |
| M04-E08 | Đánh giá đã lưu cố sửa | Disabled — tạo mới |

- [ ] M04 Edge cases PASS

---

## M05 — Chấm công: Edge cases

| ID | Test case | Expected |
|---|---|---|
| M05-E01 | 1 NV có 10+ lần CC/ngày | Cho phép, thuật toán xử lý lần đầu/cuối |
| M05-E02 | Timestamp CC trong tương lai | 422 |
| M05-E03 | CC trùng giây (double tap) | Giữ 1 record duy nhất |
| M05-E04 | Setup tháng đã chốt cố sửa | 409 |
| M05-E05 | standardWorkdays > số ngày tháng | 422 |
| M05-E06 | Trùng ngày `weeklyOffDays` và `holidays` | Cảnh báo |
| M05-E07 | Xếp 2 ca CA1+CA2 cùng ngày 1 NV | 409 |
| M05-E08 | Import Excel sai format | Trả danh sách lỗi từng dòng |
| M05-E09 | Xếp ca cho NV đã TERMINATED trong tháng | Cảnh báo |
| M05-E10 | Symbol Auto-Detection: NV không có CC nào | Gán `KP` (0%, 0h) |
| M05-E11 | NV đầu giờ trễ > 4h | Gán `In` (50%, 4h) |
| M05-E12 | NV cuối giờ về sớm > 4h | Gán `Out` (50%, 4h) |
| M05-E13 | OT làm tròn block 15p, min 30p | Verify công thức |
| M05-E14 | Ngày có CC và đơn nghỉ | Đơn nghỉ priority cao hơn (manual > auto) |
| M05-E15 | Tính lại summary tháng đã chốt | 409 |
| M05-E16 | Ca vắt qua đêm (CA3 22:00-06:00) | OT/giờ đêm tính cho ngày hôm sau |

- [ ] M05 Edge cases PASS

---

## M06 — Nghỉ phép: Edge cases

| ID | Test case | Expected |
|---|---|---|
| M06-E01 | NV gửi đơn vượt số phép tích lũy | 409 "Còn X ngày phép, đăng ký vượt" |
| M06-E02 | NV gửi đơn trùng ngày với đơn khác | 409 |
| M06-E03 | LĐ nam gửi đơn `M` (thai sản) | 422 |
| M06-E04 | LĐ nữ gửi đơn `MP` > 2.5 ngày/năm | 409 |
| M06-E05 | Hủy đơn đã APPROVED | Cần workflow riêng (chưa hỗ trợ) |
| M06-E06 | HR override (xóa nghỉ không qua đơn) | Cho phép + audit log |
| M06-E07 | Approval chain bỏ bước (Manager skip qua HR) | Block |
| M06-E08 | Phép F cho NV còn TV (chưa ký HĐLĐ) | 0 ngày phép (chưa tích lũy) |

- [ ] M06 Edge cases PASS

---

## M07 — Lương: Edge cases

| ID | Test case | Expected |
|---|---|---|
| M07-E01 | NV nghỉ giữa tháng → tính prorate theo ngày làm thực tế | Verify |
| M07-E02 | NV chuyển TV → ACTIVE giữa tháng → tách 2 phần | Verify |
| M07-E03 | NV có HĐ-1/HĐ-2 → lương TV (thường thấp hơn) | Áp dụng đúng |
| M07-E04 | Calculate gặp lỗi giữa chừng | Ghi log, không lưu kết quả dở |
| M07-E05 | Thiếu thành phần OT base | Tính bằng 0, không lỗi (BR-M05-02) |
| M07-E06 | TNCT ≤ 0 | PIT = 0 (BR-M05-08) |
| M07-E07 | Phạt trễ/sớm dùng ROUND toán học | Verify (không FLOOR) |
| M07-E08 | Chưa có evaluation KPI → KPI = 0 | OK |
| M07-E09 | Thưởng chuyên cần — đủ ngày nhưng trễ >= 3 lần | Không có thưởng |
| M07-E10 | LegalParameter đổi sau khi lock kỳ | Không hồi tố |
| M07-E11 | Reopen kỳ lương bắt buộc reason | 422 nếu không nhập |
| M07-E12 | 2 HR cùng tính lương 1 tháng đồng thời | Redis Lock — 1 HR thấy "Đang được tính bởi user X" |

- [ ] M07 Edge cases PASS

---

## M08 — Recruitment ATS: Test cases

| ID | Test case | Expected |
|---|---|---|
| M08-01 | Vào `/workspace/hr/recruitment` | Trang ATS với 5 stage Kanban |
| M08-02 | Tạo Job Posting mới | Form: tên job, JD, kênh đăng, salary range |
| M08-03 | Thêm Candidate vào stage `Applied` | Card xuất hiện |
| M08-04 | Kéo Candidate sang `Screening` (dnd-kit drag) | Card chuyển stage |
| M08-05 | Schedule Interview cho Candidate | Form: ngày giờ, interviewer, type (HR/Tech/Final) |
| M08-06 | Đánh giá Interview với feedback | Lưu |
| M08-07 | Move sang `Offer` → `Hired` | — |
| M08-08 | Khi Hired → publish event `hrm.recruitment.application.hired` | Verify trong RabbitMQ |
| M08-09 | **(v4.0)** Drag candidate vào **terminal stage HIRED** | Tự trigger **Hire dialog** với form pre-fill `fullName`, `email`, `phone` từ application |
| M08-10 | **(v4.0)** Submit Hire dialog | Tạo Employee record + link `application.HiredEmployeeId = newEmployeeId`. Toast có **action button** `<Tạo HĐLĐ ngay>` |
| M08-11 | **(v4.0)** Click `<Tạo HĐLĐ ngay>` trong toast | Redirect tới `/hr/contracts/new?employeeId={newEmployeeId}` với form pre-fill |
| M08-12 | **(v4.0) Job Posting auto-save draft** — `/hr/recruitment/job-posting/new` → gõ JD → đóng tab → quay lại | Toast vàng "Đã khôi phục bản nháp" + content vẫn còn |

- [ ] M08 Recruitment PASS

---

## M09 — Onboarding: Test cases

| ID | Test case | Expected |
|---|---|---|
| M09-01 | Vào `/workspace/hr/onboarding` → tạo template "NV mới Phòng KD" | — |
| M09-01b | **(v4.0) Auto-save draft template** — gõ tên + thêm 2 items → đóng tab → quay lại | Toast vàng "Đã khôi phục bản nháp" + items vẫn còn |
| M09-02 | Thêm checklist items (IT/HR/Manager/Finance) | — |
| M09-03 | Tạo NV mới → tự gán checklist | Verify |
| M09-04 | Owner check item của mình | Progress update |
| M09-05 | Khi 100% items đã check | Status `Onboarded` + publish event `hrm.employee.onboarded` |
| M09-06 | Báo cáo onboarding tiến độ | Hiển thị tổng quan các NV đang onboarding |
| M09-07 | Sửa template không ảnh hưởng checklist đang chạy | Verify |

- [ ] M09 Onboarding PASS

---

## M10 — Performance: Test cases

| ID | Test case | Expected |
|---|---|---|
| M10-01 | Tạo Performance Cycle Q2/2026 | — |
| M10-02 | Cycle có 3 component: OKR, Review, 1-1 | — |
| M10-03 | Tạo OKR ở 3 cấp: Company → Department → Individual | Cây OKR hiển thị đúng |
| M10-04 | Set Key Result với progress % | — |
| M10-05 | Update progress KR | Tự rollup lên parent OKR |
| M10-06 | Tạo Review form 7 tiêu chí | — |
| M10-07 | Self + Manager + Peer + Subordinate đánh giá | Multi-rater |
| M10-08 | Tổng kết Review | Score trung bình + comments |
| M10-09 | Schedule 1-1 meeting | Form 4 section |
| M10-10 | Update 1-1 meeting với action items | Sau meeting |
| M10-11 | Lịch sử 1-1 hiển thị timeline | — |
| M10-12 | Cycle đóng → KPI grade tự gán cho NV | Verify trong tab Đánh giá |

- [ ] M10 Performance PASS

---

## M11 — Reports: Test cases

| ID | Test case | Expected |
|---|---|---|
| M11-01 | Báo cáo tổng quan HR | Headcount, turnover, attendance rate |
| M11-02 | Báo cáo lương Excel | 50+ cột |
| M11-03 | Báo cáo OT Excel | — |
| M11-04 | Báo cáo PIT Excel | — |
| M11-05 | Bank list Vietinbank Excel | Tổng bằng chữ VI+EN |
| M11-06 | Báo cáo BHXH C45/C46/D02 | 🟡 Q3/2026 |
| M11-07 | Báo cáo Lao động Sở LĐTBXH | 🟡 Q3/2026 |
| M11-08 | Xuất Excel có dấu tiếng Việt đúng UTF-8 | Mở file không lỗi font |

- [ ] M11 Reports PASS

---

# 🌐 PHẦN 7: Bilingual checks (vi/en)

## 7.1 Đổi ngôn ngữ tức thì (v4.0 — 1300+ keys, 17 namespaces consolidated `hr.*`)

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Header trên cùng → bấm nút ngôn ngữ | Dropdown: Tiếng Việt · English |
| 2 | Chọn English | Toàn bộ UI chuyển sang English — KHÔNG reload trang |
| 3 | **(v4.0)** Menu sidebar hiển thị English (bao gồm section "Personal" mới) | "Employees", "Contracts", "Attendance", "Payroll", "Personal" (My profile · Overtime · Salary supplement · My training · Feedback · Accident report)... |
| 4 | Form labels hiển thị English | "Full Name", "Date of Birth", "Department"... |
| 5 | Toast messages hiển thị English | — |
| 6 | **(v4.0)** Ctrl+K palette tiếng Anh | 26 commands hiển thị English names + nhóm names |
| 7 | Quay về Tiếng Việt | Tức thì |
| 8 | Setting ngôn ngữ lưu trong profile | Đăng nhập lại vẫn giữ |
| 9 | **(v4.0) Spot check** — vào ≥ 5 page bất kỳ + switch locale | KHÔNG còn key bị lộ (vd "hr.leave.title_missing"); 1300+ keys đều có vi + en |

- [ ] 7.1 PASS

---

## 7.2 Data song ngữ (name + nameEn)

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Tạo chức danh "Trưởng phòng" với nameEn = "Department Manager" | Lưu cả 2 |
| 2 | Khi UI tiếng Việt → hiển thị "Trưởng phòng" | — |
| 3 | Khi UI English → hiển thị "Department Manager" | — |
| 4 | Tạo Department + Position + Shift + Leave Symbol đều có nameEn | Verify |

- [ ] 7.2 PASS

---

## 7.3 Template HĐLĐ song ngữ

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | In HĐ-3 → xuất PDF | PDF có 2 cột: Tiếng Việt | English |
| 2 | Mọi field (họ tên, vị trí, lương) cùng giá trị, label 2 ngôn ngữ | — |

- [ ] 7.3 PASS

---

## 7.4 Bank list — Tổng bằng chữ song ngữ

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Export Bank list Vietinbank | — |
| 2 | Cột "Tổng bằng chữ" — VD: `Mười lăm triệu năm trăm nghìn đồng / Fifteen million five hundred thousand VND` | Đúng 2 ngôn ngữ |

- [ ] 7.4 PASS

---

# 🎨 PHẦN 8: Theme & Design System checks

## 8.1 Dark mode

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Header → nút theme | Dropdown: Light · Dark · System |
| 2 | Chọn Dark | UI chuyển dark mode tức thì — bg #0F1115, text #E5E7EB |
| 3 | Mọi component (table, card, form, modal) đều dark | Không có "ô trắng lệch" |
| 4 | Charts đổi màu theo dark theme | — |
| 5 | Chọn Light | Quay về light |
| 6 | Chọn System | Theo OS setting (Mac/Win) |

- [ ] 8.1 PASS

---

## 8.2 Ctrl+K Command Palette + Notification Bell (v4.0)

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Bấm `Ctrl+K` (Cmd+K trên Mac) ở bất kỳ trang `/workspace/hr/*` | Modal palette mở giữa màn hình |
| 2 | **(v4.0)** Quan sát 26 commands chia 7 nhóm | Nhóm: **Cá nhân · Nhân sự · Chấm công & Nghỉ phép · Tiền lương · Hiệu suất · Đào tạo · Báo cáo & Cấu hình** |
| 3 | Gõ "tạo nhân viên" | Suggest: "Tạo nhân viên mới" → Enter |
| 4 | Tự navigate đến form tạo NV | — |
| 5 | Ctrl+K → gõ "phiếu lương" → Enter | Vào trang phiếu lương |
| 6 | Esc đóng palette | — |
| 7 | **(v4.0) Notification Bell** — quan sát góc dưới-phải | Floating bell icon, badge số notification chưa đọc |
| 8 | Click bell | **HrmNotificationPanel** mở dạng SlideOver từ phải |
| 9 | Click 1 notification | Mark-as-read; badge count giảm |
| 10 | Bấm Esc / click ngoài | Panel đóng |
| 11 | Vào route ngoài `/workspace/hr/*` (vd `/workspace/admin`) | Bell ẩn (HrmGlobalUI chỉ wrap HR routes) |

- [ ] 8.2 PASS

---

## 8.3 Density compact

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Settings → Density: Compact / Comfortable / Spacious | Toggle 3 option |
| 2 | Compact: 1.500 NV xem hết trên 1 màn hình | — |
| 3 | Spacious: dễ đọc hơn nhưng ít row hơn | — |

- [ ] 8.3 PASS

---

## 8.4 Accessibility AAA

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Test contrast với Chrome DevTools → Lighthouse | Score ≥ 95 |
| 2 | Tab navigation: chỉ dùng phím Tab | Tab đi qua mọi nút theo thứ tự logic |
| 3 | Enter submit form, Esc đóng modal | OK |
| 4 | Screen reader (VoiceOver/NVDA) đọc đúng label | OK |

- [ ] 8.4 PASS

---

## 8.5 Mobile responsive (Card layout MỚI v4.0 — 6 pages)

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Test trên điện thoại (Chrome mobile) hoặc DevTools responsive < 768px | Layout adapt đúng |
| 2 | Sidebar thu nhỏ thành hamburger drawer (slide từ trái) | — |
| 3 | **(v4.0)** Mở 6 pages sau ở mobile breakpoint, kiểm tra table chuyển sang **card stack**: | — |
| 3a | `/hr/leave` | Mỗi đơn nghỉ 1 card |
| 3b | `/hr/feedback` | Mỗi feedback 1 card |
| 3c | `/hr/insurance` | Mỗi insurance history 1 card |
| 3d | `/hr/contracts` | Mỗi HĐ 1 card |
| 3e | `/hr/training/courses` | Mỗi khóa học 1 card |
| 3f | `/hr/recruitment/job-posting` | Mỗi job 1 card |
| 4 | Bảng dài (không trong 6 pages trên) cuộn ngang | — |
| 5 | Form lấp đầy width | — |

- [ ] 8.5 PASS

---

## 8.6 Toast notifications

| Bước | Thao tác chi tiết | Kết quả mong đợi |
|---|---|---|
| 1 | Tạo entity thành công | Toast xanh "Đã tạo" góc phải dưới |
| 2 | Lỗi 422 | Toast đỏ với từng error message |
| 3 | Lỗi 500 | Toast đỏ chung "Có lỗi xảy ra. Vui lòng thử lại" |
| 4 | Tự ẩn sau 3-5s | — |
| 5 | Có thể bấm X đóng | — |

- [ ] 8.6 PASS

---

# ✅ PHẦN 9: Acceptance criteria

## Tiêu chí Pass UAT

Để UAT pass và go-live, các module sau **bắt buộc** phải hoạt động:

### Critical (BẮT BUỘC — fail 1 cái thì không go-live)

- [ ] **M01 Đăng nhập** — Login + Logout + Đổi workspace
- [ ] **M02 Master Data** — Tạo đủ Cây tổ chức + Chức danh + Bậc lương + Ca + Ký hiệu nghỉ + Định mức pháp luật
- [ ] **M03 Hồ sơ NV** — Tạo NV + sửa + xem 9 tab
- [ ] **M04 Hợp đồng** — Tạo HĐ-1/HĐ-2 + đánh giá + tạo HĐ-3 + cảnh báo hết hạn
- [ ] **M05 Chấm công** — Setup tháng + xếp ca + nhập CC + tính tổng hợp + export báo cáo
- [ ] **M06 Nghỉ phép** — NV gửi đơn + Manager duyệt + HR duyệt + đơn áp vào summary
- [ ] **M07 Lương** — Tính lương 8 bước + GROSS + BH + PIT + THỰC NHẬN khớp công thức tay
- [ ] **M07 Lock/Reopen** — Lock kỳ + Reopen với audit reason
- [ ] **M07 Export báo cáo** — 5 báo cáo Excel + Bank list Vietinbank
- [ ] **M14 Cấu hình** — Payroll Policy + Leave Policy + Approval Flow lưu được
- [ ] **Bilingual** — Đổi vi/en tức thì, không reload
- [ ] **RLS** — Workspace A không thấy data workspace B

### Important (Nên có)

- [ ] M03 Self-Service `/me` cho NV
- [ ] M03 Unified Inbox `/inbox` cho Manager
- [ ] M08 Recruitment ATS với Kanban
- [ ] M09 Onboarding template + checklist
- [ ] M10 Performance OKR + Review + 1-1
- [ ] M11 Feedback 2 chiều NV ↔ HR
- [ ] Dark mode hoàn chỉnh
- [ ] Ctrl+K Command Palette

### Nice-to-have (Có thì tốt, không có cũng OK)

- [ ] Báo cáo BHXH C45/C46/D02 (Q3/2026)
- [ ] Báo cáo Lao động Sở LĐTBXH (Q3/2026)
- [ ] Mobile PWA cho công nhân (Q4/2026)
- [ ] DOCX export HĐ (Q4/2026)
- [ ] Payslip PDF (đẹp) (Q3/2026)

---

# 🚧 PHẦN 10: Known limitations + workarounds

| ID | Hạn chế | Workaround | Plan fix |
|---|---|---|---|
| L01 | Phiếu lương PDF chưa generate (501) | Export Excel thay tạm | Q3/2026 |
| L02 | Xuất DOCX HĐLĐ trả 501 | Xuất PDF rồi convert | Q4/2026 |
| L03 | Báo cáo BHXH C45/C46/D02 chưa có | Tự làm Excel manual | Q3/2026 |
| L04 | Báo cáo Lao động Sở LĐTBXH chưa có | — | Q3/2026 |
| L05 | Hủy đơn nghỉ đã APPROVED chưa hỗ trợ workflow | HR Admin override + audit | Q3/2026 |
| L06 | ~~Hire → Employee auto chưa có~~ | **(v4.0 — DONE)** Drag candidate vào terminal stage HIRED → Hire dialog pre-fill → tạo Employee + link `HiredEmployeeId`. Toast có action `<Tạo HĐLĐ ngay>` | ✅ Done v4.0 |
| L07 | Mobile chấm công cho công nhân chưa có | Vẫn dùng máy CC vân tay | Q4/2026 — PWA |
| L08 | Notification SignalR real-time chưa hoàn thiện | Polling mỗi 30s | Q3/2026 |
| L09 | API Tokens vẫn ở HRM (v2.x) — sắp chuyển sang IAM | Vẫn dùng được, sẽ migrate sang `/iam/api-keys` | Q3/2026 |
| L10 | Roles UI ở `/hr/roles` là placeholder | Vào `/workspace/admin/roles-permissions` | v3.0 |
| L11 | ~~Calendar tổng hợp chưa có~~ | **(v4.0 — DONE)** Calendar `/hr/calendar` đã có nút `<+ Thêm sự kiện>` với 5 user categories (CUSTOM/MEETING/TRAINING/COMPANY_EVENT/OTHER) + 7 auto-aggregated (LEAVE/BIRTHDAY/CONTRACT_END/PROBATION_END/HOLIDAY/PAYROLL_LOCK/INTERVIEW). Click ô ngày trống → dialog pre-fill startDate. | ✅ Done v4.0 |
| L12 | Leave Carry-over cuối năm chưa tự động | HR tạm thời reset thủ công | Q4/2026 |
| L13 | Cột "Số NV" trong Cây tổ chức luôn 0 | Lỗi hiển thị — đếm đúng từ trang Nhân viên | Đã có bug ticket |
| L14 | OT cap chưa enforce tự động | HR kiểm tra manual khi duyệt OT | Q3/2026 |
| L15 | Multi-country (Thailand/Indonesia) chưa hỗ trợ | Workspace riêng cho từng country với policy tự định nghĩa | 2027 |

---

# 📅 LỊCH ONBOARDING UAT (gợi ý 7 ngày)

Đối với khách hàng triển khai NextX HRM mới — lộ trình UAT chuẩn:

| Ngày | Người | Nội dung | Thời lượng |
|---|---|---|---|
| **Ngày 1** | HR Manager | M01 Đăng nhập + M02 Master Data + M14 Cấu hình chính sách | 4h |
| **Ngày 2** | HR Manager | M03 Tạo NV + M04 Hợp đồng + M12 Phúc lợi | 6h |
| **Ngày 3** | HR + Tổ trưởng | M05 Chấm công: setup + xếp ca + nhập CC + tính summary | 6h |
| **Ngày 4** | NV + Manager | M06 Nghỉ phép: NV gửi đơn + Manager duyệt qua Inbox | 4h |
| **Ngày 5** | HR Manager | M07 Lương: tính + verify công thức + lock + export | 6h |
| **Ngày 6** | HR + Manager | M08 Recruitment + M09 Onboarding + M10 Performance | 5h |
| **Ngày 7** | Cả team | Mock 1 ngày vận hành thực tế + verify acceptance criteria | 8h |

> 💡 **Ngày 7 quan trọng nhất** — mock vận hành 1 ngày HR thực tế: tạo NV mới, ký HĐ, NV gửi đơn nghỉ, Manager duyệt, tính lương tháng test, in phiếu, đối chiếu với Excel cũ. Khớp → go-live.

---

# ❓ FAQ — Câu hỏi thường gặp

### Q1: Tôi quên mật khẩu, làm sao đăng nhập?
**A:** Trang login → `<Quên mật khẩu>` → nhập email → nhận link reset qua email. Nếu không nhận được sau 5 phút, kiểm tra Spam, hoặc liên hệ HR/Admin.

### Q2: Tôi không thấy menu "Bảng lương" / "Cài đặt"?
**A:** Bạn chưa có permission. Nhờ Admin cấp `hrm.payroll.view` / `hrm.config.view`. Vào `/workspace/admin/roles-permissions` để cấp.

### Q3: Sao tính lương xong 1 NV không có thưởng KPI?
**A:** Có 2 nguyên nhân:
- NV chưa có evaluation → KPI = 0 (BR-M05-04). Vào hồ sơ NV → tab Đánh giá → thêm
- Ngày hưởng lương < Ngày TC → thưởng giảm tỉ lệ

### Q4: Sao đơn nghỉ của tôi vẫn pending sau 2 ngày?
**A:** Kiểm tra luồng duyệt: NV → Manager → HR. Có thể đang ở bước Manager — gọi Manager nhắc duyệt. Nếu Manager đã duyệt mà vẫn pending → HR cuối cùng.

### Q5: Xuất Excel ra file cột tiếng Việt bị lỗi?
**A:** Mở Excel → Data → From Text/CSV → chọn file → encoding **UTF-8**. Hoặc dùng Google Sheets.

### Q6: Tôi muốn xem ai đã sửa giá trị lương của NV này?
**A:** Vào hồ sơ NV → tab Lịch sử / Audit log. Hiển thị: ai sửa, lúc nào, trước/sau.

### Q7: Sao tôi không tạo được HĐ-4 cho NV này?
**A:** Hệ thống bắt buộc 2 lần HĐ-3 trước HĐ-4. Verify lịch sử HĐ của NV.

### Q8: Có thể chấm công bằng điện thoại NV không?
**A:** Hiện chưa — phải qua máy CC. Q4/2026 sẽ có PWA mobile chấm bằng wifi check-in.

### Q9: Engine tính lương có chính xác đồng nào không?
**A:** Có. Mọi bước round to integer (đồng). Tổng kết khớp tay. Nếu phát hiện sai 1 đồng — báo bug ngay.

### Q10: Workspace tôi có 1500 NV — tính lương mất bao lâu?
**A:** 3-5 phút. Chạy async — HR không phải ngồi chờ, có notification khi xong.

### Q11: Sao tôi không xóa được phòng ban này?
**A:** Phòng còn 5 NV → không cho xóa. Phải termination/move NV trước, sau đó xóa.

### Q12: Sao báo cáo "BHXH C45" chưa có?
**A:** Đang trong roadmap Q3/2026. Tạm thời tải báo cáo "Bảng tính PIT" làm input.

### Q13: NV cũ nghỉ việc rồi quay lại — làm sao tạo NV mới?
**A:** Hiện tại tạo Employee mới với CCCD trùng → cảnh báo nhưng cho phép. Q4/2026 sẽ có workflow "Reopen termination".

### Q14: Có hỗ trợ chuyển phép tồn cuối năm sang đầu năm sau không?
**A:** Q4/2026. Hiện tại HR phải làm thủ công.

### Q15: Tôi đăng nhập mà thấy "Phiên đã hết hạn"?
**A:** Bình thường — sau 8h không hoạt động hệ thống tự logout. Đăng nhập lại.

---

# 🆘 Khi gặp sự cố

## Lỗi đăng nhập
1. Caps Lock?
2. Thử lại 2 lần — nếu vẫn sai → Quên mật khẩu
3. Vẫn không vào được → liên hệ Admin

## Trang trắng / Lỗi 500
1. F5 reload
2. Nếu vẫn lỗi → tab ẩn danh / browser khác
3. Vẫn lỗi → screenshot + URL + thao tác → gửi support

## Tính lương chạy mãi không xong
1. Đợi thêm 5 phút (1500 NV mất 3-5 phút)
2. Vẫn loading → F5 → quay lại preview xem có data chưa
3. Báo support nếu > 10 phút

## Không nhận email/SMS notification
1. Kiểm tra Spam
2. Đợi 5 phút
3. Vào danh sách feedback / đơn của tôi xem có notification trong app không
4. Báo HR check Notification service

---

# 📋 Phụ lục

## A. Bảng phím tắt

| Phím | Hành động |
|---|---|
| **Ctrl + K** (Mac: Cmd + K) | Command Palette (26 commands / 7 nhóm) — v4.0 |
| **F5** / **Ctrl + Shift + R** | Reload / Hard reload (fix nếu Ctrl+K không hoạt động) |
| **Ctrl + F** | Tìm trong trang |
| **Esc** | Đóng modal / Notification Panel / Command Palette |
| **Tab** | Navigate qua các nút |
| **Enter** | Submit form / chạy command đang highlight trong palette |
| **↑ / ↓** | Di chuyển trong Command Palette |

## B. Bảng phân quyền 4 vai trò × module

| Module | NV | Tổ trưởng | TBP / Manager | HR Manager |
|---|---|---|---|---|
| Dashboard | ✅ Cá nhân | ✅ Team | ✅ Phòng | ✅ Toàn workspace |
| Trang cá nhân (Self-Service, en: *My page*) | ✅ | ✅ | ✅ | ✅ |
| Hộp duyệt (Inbox) | ❌ | ✅ Bước 1 | ✅ Bước 2 | ✅ Bước cuối |
| Master Data | ❌ | ❌ | ❌ | ✅ |
| Hồ sơ NV | 👀 Mình | 👀 Tổ | 👀 Phòng | ✅ Toàn workspace |
| Hợp đồng | 👀 HĐ mình | ❌ | 👀 Team | ✅ |
| Chấm công | 👀 Mình | ✅ Xếp ca tổ | 👀 Phòng | ✅ |
| Nghỉ phép | ✅ Gửi đơn | ✅ Duyệt bước 1 | ✅ Bước 2 | ✅ Bước cuối |
| Bảng lương | 👀 Phiếu mình | ❌ | 👀 (tùy permission) | ✅ |
| Tuyển dụng | ❌ | ❌ | ✅ Đề xuất | ✅ Quản lý |
| Onboarding | ✅ Checklist mình | ❌ | ✅ Phân công | ✅ |
| Performance | ✅ OKR mình | 👀 Team | ✅ Đánh giá team | ✅ |
| Phúc lợi | 👀 Mình | ❌ | ❌ | ✅ |
| Báo cáo | ❌ | ❌ | 👀 Team | ✅ |
| Cài đặt | ❌ | ❌ | ❌ | ✅ |

**Chú thích:** ✅ = Toàn quyền · 👀 = Chỉ xem · ❌ = Không có quyền

> 💡 Bảng trên là **mặc định**. Admin có thể tùy chỉnh trong `/workspace/admin/roles-permissions`.

## C. Trạng thái thường gặp

### Trạng thái Nhân viên
| Trạng thái | Ý nghĩa |
|---|---|
| 🟢 ACTIVE | Đang làm việc |
| 🟡 PROBATION | Thử việc (HĐ-1/HĐ-2) |
| ❄️ ON_LEAVE | Đang nghỉ thai sản / dài hạn |
| 🔴 TERMINATED | Đã nghỉ việc |

### Trạng thái Hợp đồng
| Trạng thái | Ý nghĩa |
|---|---|
| 📝 DRAFT | Nháp — sửa được |
| ✅ ACTIVE | Đang hiệu lực |
| ⏰ EXPIRED | Đã hết hạn |
| 🚫 TERMINATED | Kết thúc sớm |

### Trạng thái Đơn nghỉ phép
| Trạng thái | Ý nghĩa |
|---|---|
| ⏳ PENDING | Chờ duyệt |
| ✅ APPROVED | Đã duyệt |
| ❌ REJECTED | Bị từ chối |

### Trạng thái Kỳ lương
| Trạng thái | Ý nghĩa |
|---|---|
| 🟡 DRAFT | Đang tính — chưa xong |
| 🟢 CALCULATED | Tính xong — có thể preview |
| 🔒 LOCKED | Đã chốt — không sửa được |
| 🔓 REOPENED | Đã mở lại — có audit |

---

## Lời cuối

Tài liệu UAT này không thay thế được kinh nghiệm vận hành thực tế — nhưng nó là **checklist đầy đủ** giúp bạn không bỏ sót case nào trước khi go-live NextX HRM.

Hãy đọc → test → tick checkbox → ghi bug nếu có → quay lại sau khi bug fix → retest → khi 100% Critical PASS thì go-live.

Chúc bạn UAT thành công và doanh nghiệp của bạn vận hành nhân sự ngày càng chuyên nghiệp 🎯.

---

*NextX HRM — Hướng dẫn UAT cho End-user v4.0 · 2026-05-15*
*Customer Success Team · NextX Vietnam*
*Mọi góp ý vui lòng gửi về: `<support@nextx.vn>`*

### Changelog version

- **v4.0 (2026-05-15)** — Thêm 16 nhóm tính năng Wave 2-5: HrmGlobalUI (Ctrl+K + Bell), OrgChart Tree view, Sidebar "Cá nhân" 6 menu Self-Service, 4 Picker components search, Mobile card layout 6 pages, Auto-save draft 3 forms, HrmSuccessCheck animation, Role-based view 3 pages, Calendar Add Event, Payroll Step 6 Publish, Reports preview before Excel, Hire → Employee flow, IAM Provisioning UI, Inbox bulk + delegation, 7 canonical routes mới, i18n 1300+ keys. +51 test cases mới.
- **v3.3 (2026-05-14)** — Sandbox deploy + tài khoản test sẵn sàng + đổi tên menu "Của tôi" → "Trang cá nhân" + filter Dashboard "Tùy chọn" + Phase 0 bug fixes.
- **v2.0 (2026-05-13)** — Phiên bản ban đầu cho End-user UAT.
