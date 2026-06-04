# Hướng dẫn HR: Flow chuẩn tính lương cho NV — từ Setup → Chấm công → Tính lương

> Cập nhật 2026-06-04 — sau 7 PR cải tiến trong sprint
> Workspace test: **LS Electronic-Devices Vietnam** (LSEV HQ)
> URL: `https://localhost-app.nextx.vn:3000` (local) hoặc dev server

---

## 0. Tổng quan flow

```
┌─────────────────┐    ┌──────────────────┐    ┌──────────────────┐
│ PHASE 1 (1 lần) │ →  │ PHASE 2 (đầu/th) │ →  │ PHASE 3 (đầu/th) │
│ Cấu hình hệ thg │    │ Setup tháng      │    │ Xếp ca tháng     │
└─────────────────┘    └──────────────────┘    └──────────────────┘
                                                        ↓
                       ┌──────────────────┐    ┌──────────────────┐
                       │ PHASE 5 (khi cần)│ ←  │ PHASE 4 (hàng/ng)│
                       │ Duyệt đơn nghỉ   │    │ Chấm công        │
                       └──────────────────┘    └──────────────────┘
                                ↓
                       ┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐
                       │ PHASE 6 (cuối/th)│ →  │ PHASE 7 (cuối/th)│ →  │ PHASE 8 (sau lock)│
                       │ Tính lại công    │    │ Tính lương       │    │ Phát hành phiếu  │
                       └──────────────────┘    └──────────────────┘    └──────────────────┘
                                                        ↓
                                               ┌──────────────────┐
                                               │ PHASE 9          │
                                               │ Export báo cáo   │
                                               └──────────────────┘
```

**Thời gian dự kiến** (LSEV ~50 NV):
- Setup tháng: 5 phút
- Xếp ca tháng: 10 phút (với Matrix paint mode)
- Chấm công hằng ngày: 1-2 phút/NV (nếu không có máy CC)
- Tính lại công: 10 giây cho 50 NV
- Tính lương: 5-10 giây cho 50 NV (async)
- 4 chữ ký + phát hành: 5 phút

---

## 1. Account để test

| Account | Mật khẩu | Vai trò | Permission |
|---|---|---|---|
| `hr.lsev@nextx.vn` | `Lsev@2026` | **HR Admin** (Owner) | Full HRM |
| `tbp.lsev@nextx.vn` | `Lsev@2026` | TBP (Trưởng bộ phận) | Approve L2 leaves |
| `tt.lsev@nextx.vn` | `Lsev@2026` | Tổ trưởng | Approve L1 leaves (Worker) |
| `nv.lsev@nextx.vn` | `Lsev@2026` | NLĐ văn phòng | Submit leave own |
| `cn.lsev@nextx.vn` | `Lsev@2026` | Công nhân | Submit leave own |

**Workspace**: chọn `LS Electronic-Devices Vietnam` sau khi login.

---

## 2. PHASE 1 — Cấu hình hệ thống (1 lần đầu)

### 2.1 Định mức pháp luật (LegalParameter)

**Login** `hr.lsev@nextx.vn` → **Module HR** → **Cấu hình**

**Cần khai báo 1 lần**:
- Lương tối thiểu vùng (LSEV Bắc Ninh = vùng I = 4.96M)
- Lương cơ sở Nhà nước (2.34M)
- Tỷ lệ BHXH/BHYT/BHTN NLĐ + Công ty (LSEV: 8%/1.5%/1%/0.5% NLĐ, 17.5%/3%/1%/2% Cty)
- 5 bậc thuế TNCN VN (5/10/20/30/35%) — **đã có default**
- 6 hệ số OT (150/200/200/270/300/390%)
- Giảm trừ cá nhân (15.5M), phụ thuộc (6.2M)

> 💡 **Mới**: Nếu Quốc hội đổi luật thuế 5 → 7 bậc, HR vào DB update field `pit_brackets_json` thay vì release lại code (PR #67 — P2 fix).

### 2.2 Master data (chỉ 1 lần khi onboard workspace)

| Tab | Nội dung | LSEV đã seed |
|---|---|---|
| **Phòng ban** | OrgUnit cây n-cấp | 9 đơn vị (Foreigner + LSEV → Admin/Production → Accounting/HR/Purchasing/Assembly/Plating) |
| **Chức danh** | Position | TBP, Tổ trưởng, NLĐ, Công nhân, etc. |
| **Ca làm việc** | Shift (code + giờ vào/ra + ca đêm overnight) | `S` (08:00-17:00 ca sáng), `D` (20:00-05:00 ca đêm) |
| **Ký hiệu nghỉ** | LeaveSymbol | 30 ký hiệu LSEV: F (phép), KP (không phép), SA (thứ 7), RO (refresh off), MP (LĐ nữ)... |
| **Ngạch lương** | SalaryRank | Theo nhóm chức danh + bậc |
| **Định mức lương** | SalaryRankRate | Số tiền theo ngạch + bậc |

### 2.3 Chính sách lương (PayrollPolicy)

**Module HR** → **Cấu hình** → **Chính sách lương** → "Tạo chính sách mới"

Khai báo:
- **Phụ cấp chuyên cần** (NLĐ 400k, Công nhân 600k LSEV)
- **Điều kiện hưởng chuyên cần** (vd: ≤ 3 lần trễ/tháng)
- **Hệ số phạt đi trễ** (vd: 1/4 lương giờ × ROUND((late+early)/15 phút))
- **Hiệu lực từ ngày** (effective_date) — quan trọng vì policy có lịch sử, không hồi tố

> 💡 LSEV setup mặc định: AttendanceBonusEmployee=400k, AttendanceBonusWorker=600k, LatePenaltyDivisor=4, LatePenaltyUnit=15 phút.

---

## 3. PHASE 2 — Setup tháng (đầu mỗi tháng)

**URL**: `https://localhost-app.nextx.vn:3000/vi/workspace/hr/attendance`

**HR Admin**: vào tab "**Cài đặt**"

### 3.1 Đường nhanh: Generate từ Year Template

1. Nếu workspace có Year Template năm 2026 → banner màu xanh hiện:
   > 🌟 "Workspace đã có template năm — bấm để áp dụng cho tháng này"
2. Click "**Áp dụng template**" → tự fill weekly-off (T7+CN) + holidays VN

### 3.2 Đường thủ công: Calendar paint mode

1. Click button **"Tạo mới"**
2. Calendar tháng hiện 30/31 ngày → click cells để toggle:
   - **Xanh** = ngày làm việc
   - **Xám** = ngày nghỉ tuần (weekly off)
   - **Đỏ** = ngày lễ
3. Toolbar "Quick Actions": **"Tick T7+CN"** / **"Tick CN only"** / **"Auto-fill holidays"** / **"Clear all"**
4. Chọn kiểu tính ngày công chuẩn:
   - **Type 1** (auto) = số ngày tháng − số ngày nghỉ → vd 30 − 8 = 22 ngày
   - **Type 2** (manual) = HR nhập tay (vd LSEV thường 26 ngày)
5. Click **"Lưu"** → status = `DRAFT`

### 3.3 Lock tháng

1. Khi setup OK → click **"Khoá tháng"**
2. Confirm dialog → status chuyển `LOCKED`
3. Mọi POST/PATCH vào tháng này sẽ trả 409 Conflict
4. Nếu cần sửa → click **"Mở khoá"** + nhập lý do (audit log ghi nhận)

### 3.4 Sao chép từ tháng trước

Nếu lazy: click **"Sao chép từ tháng nguồn"** → chọn năm/tháng nguồn → BE shift day-of-month tự động (vd tháng 30 ngày → tháng 31 ngày: clamp).

---

## 4. PHASE 3 — Xếp ca tháng

**URL**: `https://localhost-app.nextx.vn:3000/vi/workspace/hr/attendance/shift-matrix`

> 🆕 **Mới trong sprint** (PR #619): Matrix view kiểu Tanca — drag-paint nhanh.
> Vào tab "Cài đặt" → banner xanh "Mở bảng xếp ca" → trang Matrix.

### 4.1 Layout Matrix

```
        | T2 | T3 | T4 | T5 | T6 | T7 | CN | T2 | T3 | T4 | T5 | T6 | T7 | CN | ...
NV001   | S  | S  | S  | S  | S  | -  | -  | S  | S  | S  | S  | S  | -  | -  | ...
NV002   | D  | D  | D  | D  | D  | -  | -  | D  | D  | D  | D  | D  | -  | -  | ...
NV003   | -  | S  | S  | S  | S  | S  | -  | -  | S  | S  | S  | S  | S  | -  | ...
```

- Header sticky top: D1...D31 + tên thứ (T2/T3/.../CN)
- Cột sticky left: Mã NV + Họ tên
- Cell màu: mỗi shift code có màu riêng (S=xanh, D=tím, ...) + viền vàng = chưa lưu

### 4.2 Cách xếp ca nhanh

**Cách 1 — Paint mode (nhanh nhất)**:
1. Toolbar → dropdown **"Paint mode"** → chọn ca `S` (08:00-17:00)
2. Click cell đầu → giữ chuột kéo qua nhiều cells → mỗi cell auto fill `S`
3. Đổi paint sang `OFF` → click cells thứ 7+CN để bôi nghỉ
4. Click **"Sao chép tuần trước"** → tự duplicate D1-D7 sang D8-D14 cho mọi NV
5. Click **"Lưu thay đổi"** → batch save tất cả cell dirty

**Cách 2 — Click từng cell**:
1. Tắt Paint mode
2. Click cell → dropdown shift codes hiện ra → chọn ca
3. Cell chuyển màu, viền vàng đánh dấu chưa lưu
4. Lặp cho từng cell → "Lưu thay đổi"

**Cách 3 — Import Excel** (cho 100+ NV):
1. URL: `/vi/workspace/hr/attendance/shift-roster`
2. Tải template Excel mẫu LSEV
3. Fill file: cột Mã NV + Họ tên + D1...D31 (mỗi cell = shift code)
4. Upload → BE replace toàn bộ assignment tháng đó

### 4.3 Validate sau khi xếp

- Mỗi cell có shift code đúng với LSEV (S/D)
- Cuối tuần để trống (sẽ tự count vào weekly off)
- Bấm icon X góc dropdown để clear 1 cell

---

## 5. PHASE 4 — Chấm công hằng ngày

### 5.1 Tự động (máy ZKTeco / HikVision)

Nếu LSEV đã lắp máy chấm công:
- Máy ZKTeco push qua `POST /api/v1/hrm/attendance/iclock/cdata?SN=...` (ADMS)
- Máy HikVision FaceID push qua `POST /api/v1/hrm/attendance/hikvision/event`
- Cả 2 dùng header `X-Device-Serial` (không cần JWT)
- BE tự lookup employee theo `employeeCode` → tạo `AttendanceRaw` + `ClockRecord`

> 💡 **Cải tiến mới** (PR #66 — C10): HikVision phải gửi field `InOutType` rõ ràng (IN/OUT/0/1). Trước đây nếu thiếu sẽ fallback IN → có thể tạo sai data. Giờ bị reject với log warning.

### 5.2 Thủ công (khi máy lỗi hoặc chi nhánh không có máy)

**URL**: `/vi/workspace/hr/attendance` → tab **"Chấm công thô"**

1. Chọn ngày trên date picker
2. Bảng list NV đã có chấm công trong ngày
3. Click **"Nhập tay"** → dialog mở:
   - Chọn NV (autocomplete by code/name)
   - Type: `IN` hoặc `OUT`
   - Giờ: `08:05` (chỉ giờ:phút)
   - Date đã prefill
4. Click **"Lưu"** → tạo AttendanceRaw + ClockRecord source=`Manual`
5. **UX tip**: nếu NV đó đã có IN, dialog tự gợi ý type `OUT`

**Check-out nhanh 1-click**:
- NV trong bảng có cột "Hành động"
- Nếu NV có IN nhưng chưa có OUT → button **"Check out"** hiện → click 1 phát = OUT giờ hiện tại

### 5.3 Verify chấm công

- Cột "Vào/Ra" hiện IN/OUT badge + giờ
- Cột "Ký hiệu" hiện ký hiệu công đã resolve (nếu đã chạy Recompute)
- Cột "Nguồn" hiện `MANUAL` (có badge cam) hoặc `MACHINE`

---

## 6. PHASE 5 — Duyệt đơn nghỉ

### 6.1 NV gửi đơn

**URL**: `/vi/workspace/hr/leave`

NV (`nv.lsev@nextx.vn` hoặc `cn.lsev@nextx.vn`) login → vào trang Leave:

1. Click **"+ Đơn nghỉ"**
2. Form:
   - Từ ngày → Đến ngày
   - Loại nghỉ: `FullDay` hoặc `HalfDay` (HalfDay = 0.5 công)
   - Ký hiệu: `F` (phép có lương), `KP` (không phép), `MP` (chế độ nữ)...
   - Lý do (tuỳ chọn)
3. Submit → trả ra `id` + status `PENDING` + `nextApprover`

### 6.2 Quy trình duyệt (theo EmployeeType)

> 🆕 **Đã fix** (PR #66 — C4): Trước đây mọi NV vào flow Employee 2-cấp. Giờ tự động chọn:

| Loại NV | Cấp 1 | Cấp 2 | Final |
|---|---|---|---|
| **Worker** (Công nhân) | Tổ trưởng | TBP | HR |
| **Employee/Officer** (NLĐ văn phòng) | TBP | — | HR |
| **TBP** (TBP gửi) | — | — | HR |

### 6.3 Approve/Reject (TBP, HR)

1. `tbp.lsev@nextx.vn` login → tab "Inbox" hoặc trang Leave
2. Lọc status `PENDING` → list đơn chờ duyệt
3. Click đơn → modal chi tiết
4. **"Approve"** + ghi chú → đơn chuyển `APPROVED_L1` (hoặc `APPROVED` nếu là cấp cuối)
5. Hoặc **"Reject"** + lý do → đơn `REJECTED`

> 🆕 Webhook `hrm.leave.approved` chỉ dispatch khi đến cấp cuối (final approval).

### 6.4 Huỷ đơn (NV)

NV vào đơn mình đã submit → button **"Huỷ"** chỉ hiện khi status `PENDING` hoặc `APPROVED_L1` (chưa final).

---

## 7. PHASE 6 — Tính lại tổng hợp công (cuối tháng)

**URL**: `/vi/workspace/hr/attendance` → tab **"Tổng hợp"**

### 7.1 Chạy Recompute

1. Chọn tháng (vd `Tháng 6/2026`)
2. Click button **"Tính lại công"** (icon refresh)
3. BE chạy `RecomputeMonthSummary`:
   - Loop tất cả NV Active của workspace
   - Mỗi NV mỗi ngày: resolve symbol (`+`/`F`/`KP`/`H`/`CN`/`Out`/`In`/`CS`...)
   - Tính OT 6 bucket theo `OvertimeCalculator` (chia TV/CT theo `probationEnd`)
   - Đếm leave symbol → fDays, kpDays, saDays, roDays, mpDays
   - Tính phép F/MP còn lại từ YTD (12 - usedF, 30 - usedMp nếu nữ)
4. Sau ~10-30 giây (50 NV) → toast success
5. Bảng tổng hợp refresh

### 7.2 Review bảng tổng hợp

**Toggle view mode** (góc phải toolbar):
- **Gọn** = 8 cột chính (Mã NV, Họ tên, NgàyTC, NgàyTT, Trễ/Sớm, OT, Phép còn)
- **Đầy đủ LSEV** = 30 cột theo template Excel LSEV (chia CT/TV)

> 🆕 Workspace LSEV tự động default **Đầy đủ LSEV**; workspace khác default **Gọn**.

### 7.3 Verify số liệu

Click 1 dòng NV → modal "Chi tiết công":
- 6 metric tổng (Ngày TC, Ngày TT, Trễ/sớm số lần, OT thường/cuối tuần/lễ)
- **Bảng chi tiết 31 ngày** với cột:
  - Ngày + tag Holiday/WeeklyOff
  - Ca đã xếp
  - Ký hiệu công (badge)
  - Vào/Ra (format giờ:phút gọn)
  - **Giờ làm** (số giờ thực tế từ IN→OUT)
  - **OT** (số giờ OT)
  - **Đêm** (số giờ làm đêm 22h-5h)
  - **Trễ/Sớm** (vd "Trễ 15p · Sớm 5p")

### 7.4 Sửa lỗi nếu phát hiện

- NV thiếu chấm công 1 ngày → quay lại Phase 4, nhập tay → Recompute lại
- Đơn nghỉ chưa duyệt → quay lại Phase 5 → Recompute lại
- Phép F còn sai → check approved leave symbol = "F" chính xác chưa

### 7.5 Phiếu đối chiếu công (gửi NV ký)

Trong modal NV → button **"Phiếu đối chiếu PDF"** → tải PDF mẫu LSEV cho 1 NV.

Hoặc toolbar trang → **"Phiếu đối chiếu (ZIP cả tháng)"** → tải ZIP cho toàn bộ NV active. Header response `X-Skipped-Count` cho biết NV nào bị skip (thiếu summary).

**Quy trình LSEV**: HR gửi PDF qua email/in giấy → NV ký xác nhận → deadline phản hồi 09:00 ngày 02 tháng sau.

---

## 8. PHASE 7 — Tính lương

**URL**: `/vi/workspace/hr/payroll`

### 8.1 Trước khi tính

1. Đảm bảo PHASE 6 đã chạy xong (mọi NV có `AttendanceSummary`)
2. Vào tab **"Thu nhập khác"** → thêm thưởng/KPI bổ sung nếu có
3. Vào tab **"Khấu trừ khác"** → thêm khấu trừ custom (ngoài BH chuẩn)
4. Vào tab **"PIT Method"** → chọn `PT1` (OT phụ trội miễn thuế) hoặc `PT2` (toàn bộ OT chịu thuế)

### 8.2 Trigger tính lương

1. Chọn tháng 6/2026
2. Click **"Tính lương tháng"**
3. **PR #68 — P1 async**: BE tạo `PayrollMonth.JobStatus=Processing` + publish event qua RabbitMQ → trả 202 trong **<500ms**
4. Toast: "Đã enqueue job XXX — đang tính lương cho 50 NV (~1 phút)"
5. Status indicator `PROCESSING` (spinner) — KHÔNG block HR, có thể làm việc khác

### 8.3 Poll status (auto)

- FE poll GET `/payroll/{y}/{m}/calculate/status` mỗi 3s
- Khi `JobStatus=Completed` → toast success + auto refresh Preview
- Khi `Failed` → toast error + xem log server

> 💡 Server log Serilog/console: `[PayrollCalc] Start job xxx ... completed — processed=50 errors=0`

### 8.4 Xem Preview

Tab **"Xem trước kết quả"** → bảng 50 NV với:
- Lương cơ bản
- Phụ cấp
- Thưởng chuyên cần
- OT total (6 bucket cộng lại)
- Gross
- BH NLĐ
- PIT
- Khấu trừ khác
- **Lương thực nhận** (Net)

Click 1 NV → modal phiếu lương chi tiết 3 section (LSEV mẫu).

### 8.5 Lock + 4-Signature workflow

1. Click **"Chốt lương tháng"** + ghi chú → Status `LOCKED` (chốt sơ bộ)
2. **4-Signature** (PR #66 — P3/P4 đã fix enforce order):
   - HR Admin click **"Mark Prepared"** → record `PreparedBy/At`
   - HR Admin click **"Mark CheckedHr"** → record `CheckedHrBy/At` (chỉ chạy được khi đã Prepared)
   - Quản lý cấp cao click **"Mark CheckedMgmt"** → record `CheckedMgmtBy/At`
   - GĐ click **"Mark Approved"** → record `ApprovedBy/At` + **auto-Lock** (PR #66 P4)

> ⚠️ Nếu skip thứ tự (vd Approve trực tiếp khi chưa CheckedHr) → 400 error: `payroll.signature.must_be_checked_mgmt_first`.

### 8.6 Reopen nếu phát hiện sai

Click **"Mở lại tháng đã chốt"** + lý do → status về `Draft` + **clear 4 chữ ký** (PR #66 — P14 compliance fix). HR phải duyệt lại từ đầu.

---

## 9. PHASE 8 — Phát hành phiếu lương

### 9.1 Distribute

1. Sau khi đã Approved → click **"Phát hành phiếu lương"**
2. Chọn kênh:
   - **EMAIL** — gửi PDF qua email NLĐ
   - **PORTAL** — NLĐ tự xem qua `/me/payslip`
   - **ESIGN** — gửi link e-sign qua Contract service (Phase 4B)
3. Click **"Phát hành"** → BE đánh dấu `IsDistributed` cho từng payslip

### 9.2 Theo dõi acknowledge

Bảng "Distribution status" hiện:
- `NOT_SENT` | `SENT` | `ACKNOWLEDGED` | `REJECTED` | `EXPIRED`

NLĐ chưa ack quá X ngày → click **"Gửi lại link"** trên dòng đó.

---

## 10. PHASE 9 — Export báo cáo

URL: `/vi/workspace/hr/payroll` → tab **"Báo cáo"** (cuối page)

| Báo cáo | Mục đích | Format |
|---|---|---|
| **Summary** | Tổng hợp lương toàn cty | Excel |
| **OT** | Chi tiết OT theo NV | Excel |
| **PIT** | Tổng thuế TNCN nộp | Excel (gửi kế toán) |
| **Bank list** | Danh sách chuyển khoản | Excel (gửi ngân hàng) |
| **Payslip PDF** | Phiếu lương cá nhân mẫu LSEV | PDF per NV |

Param `?format=json` cho preview (JSON), default `format=excel` để download .xlsx.

**Tab attendance reports** (`/hr/attendance` → tab Tổng hợp):
- Attendance Record (D1-D31 symbols)
- OT Summary
- OT Record (gioNgay:gioDem từng ngày)
- Night Shift Record

---

## 11. Troubleshooting common errors

| Error | Nguyên nhân | Cách fix |
|---|---|---|
| `attendance.month.already_locked` | Tháng đã Lock, không sửa được | Mở khoá (Phase 2.3) hoặc Reopen Payroll |
| `attendance.month_setup.not_found` | Recompute trước khi Setup tháng | Quay lại Phase 2, setup tháng đó |
| `payroll.attendance_not_ready` | Calculate trước khi Recompute | Chạy Recompute Phase 6 trước |
| `payroll.month.locked` | Calculate khi tháng đã chốt | Reopen (Phase 7.6) trước |
| `payroll.signature.must_be_prepared_first` | Mark CheckedHr khi chưa Prepared | Mark Prepared trước |
| `payroll.signature.must_be_checked_hr_first` | Mark CheckedMgmt khi chưa CheckedHr | Đúng thứ tự 4-sig |
| `leave.request.already_exists` | Submit đơn overlap với đơn đã có | Xem lại lịch nghỉ NV trước khi submit |
| `me.employee_not_linked` | NV tự gửi nhưng `user_id` chưa link | HR vào Employee → link User account |
| Bảng tổng hợp chỉ hiện 1 NV | Lỗi cũ — đã fix PR #64 | Chạy lại Recompute (giờ hiện đủ NV) |
| Phép F còn = 0 cho mọi NV | Lỗi cũ — đã fix PR #67 (C8) | Chạy lại Recompute (giờ tính YTD đúng) |
| HTTP timeout khi Calculate >500 NV | Lỗi cũ — đã fix PR #68 (P1 async) | Giờ trả 202 ngay, poll status |
| HikVision device không tạo raw | Thiếu InOutType trong payload | Config device gửi field "InOutType":"IN/OUT" |

---

## 12. Checklist HR cuối tháng (cheatsheet 1 trang)

```
□ Phase 2: Setup tháng (Generate from template hoặc paint calendar)
□ Phase 3: Xếp ca tháng (Matrix paint mode, 10 phút)
□ Phase 4: Chấm công hằng ngày (auto từ máy + manual khi cần)
□ Phase 5: Duyệt đơn nghỉ pending mỗi ngày
□ Phase 6: Recompute cuối tháng → review summary 30 cột LSEV
□ Phase 7: Calculate lương → wait status Completed
□ Phase 7.5: Mark 4 signatures theo đúng thứ tự
□ Phase 7.5: Verify Status = LOCKED
□ Phase 8: Distribute phiếu lương
□ Phase 9: Export 4 báo cáo Excel + gửi kế toán/ngân hàng
□ (Optional) Phiếu đối chiếu công ZIP gửi NV ký
□ Theo dõi NV acknowledge phiếu lương trong 7 ngày
```

---

## 13. Tham khảo nhanh URL FE

| Trang | URL |
|---|---|
| Tab chấm công chính | `/vi/workspace/hr/attendance` |
| Shift Roster Matrix 🆕 | `/vi/workspace/hr/attendance/shift-matrix` |
| Shift Roster Excel import | `/vi/workspace/hr/attendance/shift-roster` |
| Year Template | `/vi/workspace/hr/attendance/year-template` |
| Máy chấm công | `/vi/workspace/hr/attendance/devices` |
| Đơn nghỉ | `/vi/workspace/hr/leave` |
| Import đơn nghỉ Excel | `/vi/workspace/hr/leave/import` |
| Bảng tổng hợp công (page riêng) | `/vi/workspace/hr/attendance-summary` |
| Của tôi (NV) | `/vi/workspace/hr/me/attendance` |
| Tính lương | `/vi/workspace/hr/payroll` |
| Chính sách lương | `/vi/workspace/hr/payroll-policy` |
| Đơn lương bổ sung | `/vi/workspace/hr/payroll-supplement` |

---

*Doc cập nhật 2026-06-04 — sau 7 PR sprint (PRs #64, #65, #66, #67, #68, FE #618, FE #619)*
