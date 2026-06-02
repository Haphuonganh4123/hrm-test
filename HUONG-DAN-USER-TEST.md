# Hướng dẫn User Test — NextX HRM (LSEV Demo)

> Tài liệu gửi user test. 3 flow chính: **Setup workspace → Hợp đồng → Chấm công**.

---

## 0. Tài khoản test

URL chính thức: `https://app-dev.nextx.vn`
URL local (nếu chạy FE local): `https://localhost-app.nextx.vn:3000`
Workspace: **LS Electronic-Devices Vietnam** (`@lsev-vn`)

| Vai trò | Email | Password | Link với NV | Mục đích test |
|---|---|---|---|---|
| **HR Admin** | `hr.lsev@nextx.vn` | `Lsev@2026` | — | Setup, master data, duyệt đơn cuối cùng, tính lương, xuất báo cáo |
| **Tổ trưởng / Manager** | `tl.test@nextx.vn` | `TlTest@2026` | LSEV001 — Nguyễn Văn An | Duyệt đơn nghỉ cấp 1, xem CC team, đánh giá NV |
| **Nhân viên (Công nhân)** | `nv.test@nextx.vn` | `NvTest@2026` | LSEV003 — Lê Hoàng Cường | Self-service: chấm công, xem CC bản thân, gửi đơn nghỉ |

> ⚠️ Đăng nhập lần đầu sẽ thấy chỉ **1 module HR** (CRM/Sales/Chat đã ẩn theo SRS LSEV). Chọn HR để vào.

---

## 1. Flow SETUP — Khởi tạo hệ thống (đã hoàn tất sẵn)

### Đã seed sẵn cho LSEV — user test KHÔNG cần làm lại
HR Admin login vào sẽ thấy đã có:

| Hạng mục | Số lượng | Vị trí xem |
|---|---|---|
| Chi nhánh (IAM) | 2 (LSEV HQ, Foreigner) | Header → "Chuyển chi nhánh" |
| Cây tổ chức (HRM) | 8 unit (2 chi nhánh + 2 khối + 6 phòng) | Sidebar → Hồ sơ NV → Phòng ban & Tổ chức |
| Chức danh | 12 (Senior Manager → Worker) | Cài đặt → Chức danh |
| Bậc lương | 26 (WK1-10, SF1.0-1.9, SF2.0-2.5 × 2025+2026) | Cài đặt → Bậc lương |
| Ca làm việc | 7 (CA1, CA2, CA3, CaHanhChinh, CAB, CaTS, CaBTS) | Cài đặt → Ca làm việc |
| Ký hiệu nghỉ | 31 (+, F, MP, S, KP, H, SA, RO, ...) | Cài đặt → Ký hiệu CC |
| Định mức pháp luật 2026 | LTT 4.730k, BH (8/1.5/1%), PIT 5 bậc, OT 6 loại | Cài đặt → Định mức |
| Phụ cấp (PC001 Housing, PC003 Position) | Gán qua quá trình PC từng NV | — |
| Nhân viên | 8 (LSEV001–LSEV008) + 4 HD1/HD2 auto + 8 HD3 ACTIVE | Hồ sơ NV |
| Approval flows | 3 (Worker / Staff / Department Head) | Cài đặt → Luồng phê duyệt |
| Year Template 2026 | 12 ngày lễ VN (Tết DL, Tết Âm 6 ngày, Giỗ tổ, 30/4, 1/5, 2/9, 3/9) | Cài đặt → Year Template |
| Setup chấm công T5/2026 | 26 ngày TC, weekly off CN | Chấm công → Setup tháng |
| Shift assignments + Clock T5/2026 | 8 NV × 20 ngày | Chấm công → Bảng công |

### Khi cần thêm tháng mới (T6/2026 chẳng hạn)
HR Admin → Chấm công → **Setup tháng**:
1. Chọn năm 2026, tháng 6
2. Ngày nghỉ tuần: Chủ nhật (0)
3. Ngày lễ: (tự load từ Year Template, có thể override)
4. Số ngày công chuẩn: Type1 (auto: ngày tháng - CN) hoặc Type2 (manual)
5. Lưu → tháng đã active, bắt đầu xếp ca + chấm công

---

## 2. Flow HỢP ĐỒNG LAO ĐỘNG

### Tổng quan 4 loại HĐ

| Loại | Tên | Đối tượng | Cách tạo |
|---|---|---|---|
| **HD1** | Thỏa thuận Thử việc | Nhân viên (Staff) | 🤖 **Tự động** khi tạo NV mới |
| **HD2** | Thỏa thuận Học việc | Công nhân (Worker) | 🤖 **Tự động** khi tạo NV mới |
| **HD3** | HĐLĐ Xác định thời hạn | Sau TV/HV đạt | ✋ HR tạo qua "+ Tạo HĐ mới" |
| **HD4** | HĐLĐ Không XĐTH | Sau 2 lần HD3 | ✋ HR tạo qua "+ Tạo HĐ mới" |

> ❓ **"Tại sao form Tạo HĐ chỉ có HD3/HD4?"** Vì HD1/HD2 hệ thống tự sinh khi tạo NV mới — không cho HR tạo tay (tránh trùng).
> ❓ **"Hợp đồng thử việc/học việc từ đâu mà có?"** Tự động sinh khi `POST /employees` với `employeeType=Employee` (→ HD1) hoặc `Worker` (→ HD2).

### Vòng đời

```
[Tạo NV mới] → HD1/HD2 (DRAFT, auto) → Đánh giá TV (7 tiêu chí) → Quyết định
                                                                    │
                                                  ┌─────────────────┼─────────────────┐
                                                  │                 │                 │
                                              "Đạt"            "Không đạt"      "Cần xem xét"
                                                  │                 │                 │
                                                  ▼                 ▼                 ▼
                                          Tạo HD3 XĐTH      Thôi việc          Gia hạn TV
                                          (1/2/3 năm)
                                                  │
                                                  ▼
                                          Trước hết hạn 30/15/7 ngày
                                                  │
                                          Đánh giá gia hạn
                                                  │
                                          ┌───────┴───────┐
                                          ▼               ▼
                                      "Đạt" lần 1     "Đạt" lần 2
                                          │               │
                                          ▼               ▼
                                      HD3 lần 2       HD4 Không XĐTH
                                                      (vĩnh viễn)
```

### Trạng thái HĐ

| Status | Hiển thị | Ý nghĩa |
|---|---|---|
| `DRAFT` | Nháp | Vừa tạo, chưa ký |
| `SIGNING` | Đang ký | Đợi NV ký |
| `ACTIVE` | Đang hiệu lực | HĐ chính thức |
| `EXPIRED` | Đã hết hạn | Cần tạo HĐ mới |
| `TERMINATED` | Đã chấm dứt sớm | — |

### Test scenarios gợi ý

**Scenario 1: Đánh giá thử việc → ký HD3**
1. Login `hr.lsev`. Mở danh sách HĐ. Lọc loại = HD1 hoặc HD2.
2. Chọn 1 HĐ status DRAFT (vd: LSEV008 — Bùi Thị Lan). Bấm "Đánh giá".
3. Nhập 7 tiêu chí (chuyên môn, thái độ, kỷ luật, hiệu suất, hợp tác, sáng kiến, tuân thủ) — mỗi tiêu chí 0-100.
4. Quyết định "Đạt" → hệ thống gợi ý tạo HD3.
5. Bấm "+ Tạo HĐ mới" → form đã pre-fill NV + StartDate = ngày kết thúc TV + 1.
6. Chọn duration 24 tháng, nhập basicSalary theo bậc lương SF1.4 (10.5M).
7. Lưu → contract HD3 trạng thái DRAFT → SIGNING → ACTIVE.

**Scenario 2: Cảnh báo HĐ sắp hết hạn**
- Vào Tổng quan HR (Dashboard) → widget "HĐ sắp hết hạn (30 ngày tới)".
- Hiện 8 HD3 LSEV001-008 vì cùng hết hạn 15/01/2026 (~7 tháng tới).
- Bấm vào tên NV → đi tới HĐ → Đánh giá gia hạn.

**Scenario 3: Tải mẫu HĐ song ngữ**
- Trong list HĐ → "Tải mẫu HĐ" → chọn template (7 mẫu: Thỏa thuận TV/HV, Đánh giá TV/HV, Đánh giá gia hạn, HĐLĐ XĐTH, HĐLĐ Không XĐTH, PDPA).
- Xuất Word/PDF song ngữ Việt-Anh.

---

## 3. Flow CHẤM CÔNG

> **Quy tắc vàng:** Recompute là **MANUAL trigger**. Sau mỗi lần thay đổi (chấm công, xếp ca, duyệt đơn) → **bấm "Tính lại công"** để cập nhật bảng tổng hợp.

### Pipeline đầy đủ

```
PHASE 1: SETUP THÁNG (đầu tháng — HR Admin)
  ├── Chấm công → Setup tháng → chọn năm/tháng
  ├── Nhập: weeklyOffDays (CN), holidays (auto-load Year Template), workdayType
  └── Số ngày công chuẩn = 26 (auto Type1) → Giờ công TC = 26 × 8 = 208h

PHASE 2: XẾP CA (đầu tháng — Tổ trưởng / HR)
  ├── Cách 1: Bulk assign — chọn NV → chọn ca CA1/CA2/CA3/CAHANHCHINH → range ngày
  ├── Cách 2: Import Excel: cột MaNV | TenNV | D1...D31 (mã ca per ngày)
  └── Cách 3: Generate từ template (đã save shift pattern recurring)

PHASE 3: CHẤM CÔNG HÀNG NGÀY
  ├── Cách 1: Máy CC Ronald Jack/HikVision/ZKTeco gửi real-time qua ADMS
  ├── Cách 2: Upload Excel/Text từ máy → POST /attendance/import-raw
  ├── Cách 3: Manual — Chấm công → "Chấm công thủ công" → chọn NV + ngày + IN/OUT time
  └── Hệ thống tự nhận diện symbol: + (đủ), In (thiếu IN), Out (thiếu OUT), CS (lỗi), KP (vắng)

PHASE 4: ĐĂNG KÝ NGHỈ (NLĐ tự gửi, các cấp duyệt)
  ├── NLĐ login → Đơn nghỉ → "+ Tạo đơn" — chọn loại nghỉ (F/MP/S/B/T/W/D/...)
  ├── Worker: Tổ trưởng (PD1) → TBP (PD2) → HR (final)
  ├── Staff:   TBP → HR
  ├── Manager: BOD (PD1) → HR
  └── HR có thể tự thêm/sửa/xóa nghỉ không qua đơn (override)

PHASE 5: TÍNH LẠI CÔNG (cuối tháng / khi cần refresh)
  ├── Chấm công → "Bảng công" → nút "Tính lại công"
  ├── BE chạy POST /attendance/summary/{y}/{m}/recompute
  └── Tính: Ngày TT, OT thường (D/N), OT cuối tuần (D/N), OT lễ (D/N),
            Giờ đêm, Đi trễ/về sớm, F còn lại, MP còn lại

PHASE 6: BÁO CÁO (4 báo cáo Excel + Phiếu PDF)
  ├── Attendance Record — ký hiệu CC từng ngày toàn NV, song ngữ
  ├── OT Summary — tổng OT 6 bucket
  ├── OT Record — chi tiết từng ngày (số dương = ngày, âm = đêm)
  ├── Night Shift Record — giờ làm đêm
  └── Phiếu Đối chiếu công PDF — cá nhân (NLĐ ký xác nhận trước 09:00 ngày 02 tháng sau)
```

### Vai trò & hành động tương ứng

| Vai trò | Hành động chấm công |
|---|---|
| **HR Admin** (hr.lsev) | Setup tháng, xếp ca bulk, import máy CC, override nghỉ, tính lại công, xuất báo cáo, khóa tháng |
| **Tổ trưởng** (tl.test → LSEV001) | Xếp ca cho tổ mình, duyệt đơn nghỉ CN cấp 1, xem CC team |
| **Nhân viên** (nv.test → LSEV003) | Xem CC bản thân, gửi đơn nghỉ, xem phép F còn lại |

### Test scenarios gợi ý

**Scenario 1: NLĐ tự chấm công + gửi đơn nghỉ**
1. Login `nv.test@nextx.vn` / `NvTest@2026` (Công nhân — Lê Hoàng Cường).
2. Vào "Cá nhân" → "Chấm công của tôi" → xem T5/2026 (đã có sẵn 20 ngày clock IN/OUT).
3. Vào "Nghỉ phép" → "+ Tạo đơn":
   - Từ ngày: 03/06/2026 (thứ 4) — Đến ngày: 04/06/2026
   - Loại: F (Nghỉ phép năm)
   - Lý do: "Công việc gia đình"
4. Gửi → đơn ở status "Chờ Tổ trưởng duyệt".

**Scenario 2: Tổ trưởng duyệt đơn cấp 1**
1. Login `tl.test@nextx.vn` / `TlTest@2026` (Senior Manager — Nguyễn Văn An).
2. Vào "Inbox" hoặc "Phê duyệt" → thấy đơn của LSEV003.
3. Bấm "Duyệt" với note "OK" → đơn chuyển sang "Chờ Trưởng BP duyệt".
4. Hiện tại không có TBP riêng → HR sẽ duyệt nốt (login `hr.lsev`).

**Scenario 3: HR duyệt cuối + tính lại công**
1. Login `hr.lsev@nextx.vn`. Vào Inbox.
2. Duyệt đơn nghỉ pending của LSEV003 (kết quả cuối).
3. Vào "Chấm công" → "Bảng công" → chọn tháng 6/2026.
4. Bấm **"Tính lại công"** → đợi 2-5 giây → bảng cập nhật:
   - LSEV003 — Ngày TT giảm 2 (nếu loại F không tính làm việc)
   - Hoặc Ngày TT giữ nguyên + phép F còn lại giảm 2 (vì F = pay rate 100%, vẫn tính công nhưng dùng phép)
5. Click vào dòng LSEV003 → xem chi tiết từng ngày: 3/6 + 4/6 có ký hiệu "F".

**Scenario 4: OT tự động**
1. NV clock IN 5:20 (trước ca 06:00 = 40 phút) → bấm Tính lại công → OT thường ngày = 0.5h (làm tròn xuống block 30 phút).
2. Demo case có sẵn: 30% ngày của LSEV003-007 đã được seed có clock OUT muộn 18:00 (4h OT sau ca 14:00).

**Scenario 5: Báo cáo cuối tháng**
1. HR → Chấm công → "Tính lại công" T5/2026 lần cuối.
2. Bấm "BC Chấm công" → xuất Excel Attendance Record cho cả 8 NV.
3. Bấm "BC OT chi tiết" → xuất chi tiết OT.
4. Click 1 NV → "Phiếu đối chiếu PDF" → xuất phiếu cá nhân để gửi NLĐ ký xác nhận.

---

## 4. Flow TIỀN LƯƠNG (bonus)

Đã chạy tính lương T5/2026 sẵn. Vào "Tiền lương" → "Tính lương tháng" → 2026/5:

| Mã NV | Họ tên | Gross | BH | PIT | Thực lĩnh |
|---|---|---:|---:|---:|---:|
| LSEV001 | Nguyễn Văn An | 25.378k | — | — | 23.210k |
| LSEV002 | Trần Thị Bình | 16.273k | — | — | 14.953k |
| LSEV003 | Lê Hoàng Cường | 9.371k | — | — | 8.711k |
| ... | (5 NV còn lại) | ... | ... | ... | ... |

Click 1 NV → xem phiếu lương PDF chi tiết.

---

## 5. Bug đã biết / Lưu ý

| # | Vấn đề | Trạng thái | Workaround |
|---|---|---|---|
| 1 | List HĐ hiện `basicSalary=0` | 🔧 Đã fix BE — PR #49 chờ deploy | Mở "Chi tiết" HĐ vẫn xem được lương |
| 2 | Ngày TT = 0 sau chấm công thủ công | ✅ Expected behavior | Bấm nút **"Tính lại công"** sau mỗi lần thay đổi |
| 3 | Webhook event tới webhook.site/lsev-demo | ⚠️ URL placeholder | Demo OK, không có receiver thật |
| 4 | Salary-history NV dùng format cũ (group="ST"/level="1") | ⚠️ Pre-existing data | Tạo "+ Quá trình lương mới" với group=WK/SF1/SF2 |
| 5 | Recruitment menu ẩn | ✅ Đúng theo SRS | Out of scope LSEV |

---

## 6. Quick reference URL

| Module | Direct URL |
|---|---|
| Tổng quan HR | `/vi/workspace/hr` |
| Nhân viên | `/vi/workspace/hr/employees` |
| Phòng ban & Tổ chức | `/vi/workspace/hr/org-units` |
| Hợp đồng | `/vi/workspace/hr/contracts` |
| Chấm công — Bảng công | `/vi/workspace/hr/attendance/board` |
| Chấm công — Setup tháng | `/vi/workspace/hr/attendance/setup` |
| Chấm công — Đơn nghỉ | `/vi/workspace/hr/attendance/leaves` |
| Tiền lương | `/vi/workspace/hr/payroll` |
| Cài đặt → Bậc lương | `/vi/workspace/hr/settings/salary-ranks` |
| Cài đặt → Ca làm việc | `/vi/workspace/hr/settings/shifts` |
| Cài đặt → Year Template | `/vi/workspace/hr/settings/year-template` |

---

## 7. Cần hỗ trợ?

- **Bug functional:** chụp screenshot + URL + step reproduce → gửi qua Zalo.
- **Câu hỏi nghiệp vụ:** đối chiếu với SRS `docs/LSEV/NextX_HRM_SRS_LSEV_v2.md`.
- **Quên password:** "Quên mật khẩu" trên màn login (nếu SMTP setup) hoặc báo HR Admin reset.

---

*Tài liệu cập nhật: 02/06/2026 · LSEV demo workspace · v1.0*
