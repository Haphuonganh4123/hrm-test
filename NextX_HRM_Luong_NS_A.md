# NextX HRM — Luồng Gia nhập → Hợp đồng → Chấm công → Tính lương

> Tài liệu minh hoạ nghiệp vụ theo **SRS NextX HRM v2.1 (LSEV, 17/04/2026)**.
> Số liệu nhân sự, mức lương trong file là **dữ liệu giả định** để minh hoạ công thức — không phải số liệu thật.
> Kỳ tính: **Tháng 3/2026** (31 ngày). Múi giờ GMT+7.

---

## 0. Luồng tổng thể (End-to-end)

```
[NV/CN mới gia nhập]
        │
        ▼
┌─ PHASE A · HỒ SƠ ──────────────────────────────────────────┐
│ HR tạo hồ sơ → sinh Mã NV tự động                          │
│ Phân loại tự động theo Chức danh:                          │
│   • Nhân viên → trạng thái THỬ VIỆC (cam)                  │
│   • Công nhân → trạng thái HỌC VIỆC (cam)                  │
└────────────────────────────────────────────────────────────┘
        │
        ▼
┌─ PHASE B · HỢP ĐỒNG ───────────────────────────────────────┐
│ Tự động gán Thoả thuận TV/HV (không mã HĐ)                 │
│ Cảnh báo hết TV/HV: 30 / 15 / 7 ngày                       │
│ Đánh giá TV/HV (7 tiêu chí) → Ký HĐLĐ lần 1                │
│ Ngày ký = Ngày hết TV/HV + 1 · Mã = MãNV/HĐLĐ/Năm          │
│ → XĐTH 1/2/3 năm → (lần 2) → HĐLĐ Không XĐTH               │
└────────────────────────────────────────────────────────────┘
        │
        ▼
┌─ PHASE C · CHẤM CÔNG (hàng tháng) ─────────────────────────┐
│ Cài đặt tháng (nghỉ tuần, lễ, kiểu công TC)                │
│ Xếp ca → Máy CC gửi dữ liệu → Tự nhận ký hiệu +/In/Out/CS  │
│ OT tự động · Đăng ký nghỉ · Tổng hợp công cuối tháng       │
└────────────────────────────────────────────────────────────┘
        │
        ▼
┌─ PHASE D · TÍNH LƯƠNG ─────────────────────────────────────┐
│ Lấy số liệu công → Engine lương → GROSS → trừ BH → trừ PIT │
│ = ACTUAL RECEIVING → Chốt → Phiếu lương + Bank list        │
└────────────────────────────────────────────────────────────┘
```

---

## 1. LÀM RÕ PHẦN HỢP ĐỒNG (Module 03)

### 1.1. 4 loại hợp đồng & khi nào dùng

| Mã | Loại | Áp dụng | Mã HĐ | Cảnh báo hết hạn |
|----|------|---------|-------|------------------|
| HĐ-1 | Thoả thuận **Thử việc** | Phân loại = **Nhân viên** | Không có | — |
| HĐ-2 | Thoả thuận **Học việc** | Phân loại = **Công nhân** | Không có | — |
| HĐ-3 | **HĐLĐ Xác định thời hạn (XĐTH)** | Tất cả NV sau TV/HV | `MãNV/HĐLĐ/Năm` | 30 / 15 / 7 ngày |
| HĐ-4 | **HĐLĐ Không XĐTH** | Sau 2 lần XĐTH | Có | Không (vô thời hạn) |

### 1.2. Quy tắc tự động hoá quan trọng

- **Tự động gán HĐ đầu vào:** ngay khi tạo hồ sơ, hệ thống nhìn Phân loại (theo Chức danh) để gán HĐ-1 hoặc HĐ-2 — HR không phải chọn tay.
- **Ngày ký HĐLĐ lần 1 = Ngày kết thúc TV/HV + 1** (đúng theo Module 02, trường "Ngày kết thúc thử việc").
- **Sinh mã HĐ tự động:** `[MãNV]/HĐLĐ/[Năm ký]` → VD `00791/HĐLĐ/2026`.
- **Thời hạn XĐTH:** HR chọn 1 / 2 / 3 năm (tuỳ chỉnh). Ngày hết hạn = ngày ký + thời hạn − 1.
- **Cảnh báo tự động** trước hết hạn TV/HV và trước hết hạn HĐLĐ: **30 / 15 / 7 ngày**.
- **HĐLĐ Không XĐTH** chỉ phát sinh sau khi đã ký **2 lần** HĐLĐ XĐTH.
- **Không cho xoá HĐ đã ký** → chỉ thêm HĐ mới thay thế (ràng buộc validation).
- **Template:** 7 mẫu song ngữ VI-EN, **xuất được cả PDF (ký chính thức) và Word .docx (chỉnh sửa)** — cập nhật v2.1 mục "In PDF Word".

### 1.3. Vòng đời HĐ — Ví dụ cụ thể

**Nhân sự A — Công nhân (Worker), Mã NV `01742`**

| Mốc | Ngày | Hệ thống xử lý |
|-----|------|----------------|
| Gia nhập | 01/12/2025 | Phân loại Công nhân → tự gán **HĐ-2 Học việc** (không mã) |
| Hết học việc | 28/02/2026 | Cảnh báo 30/15/7 ngày trước; HR đánh giá HV (7 tiêu chí) → Đạt |
| Ký HĐLĐ lần 1 | **01/03/2026** | = 28/02 + 1. Mã **`01742/HĐLĐ/2026`**, XĐTH **1 năm** → hết 28/02/2027 |
| (Sau này) lần 2 | 01/03/2027 | XĐTH lần 2 |
| (Sau này) | 01/03/2028 | **HĐLĐ Không XĐTH** (vô thời hạn) |

**Nhân sự B — Nhân viên văn phòng (Staff), Mã NV `00305`**

| Mốc | Ngày | Hệ thống xử lý |
|-----|------|----------------|
| Gia nhập | 01/09/2025 | Phân loại Nhân viên → tự gán **HĐ-1 Thử việc** (không mã) |
| Hết thử việc | 30/11/2025 | Cảnh báo + đánh giá TV → Đạt |
| Ký HĐLĐ lần 1 | 01/12/2025 | Mã **`00305/HĐLĐ/2025`**, XĐTH 2 năm → hết 30/11/2027 |

---

## 2. NHÂN SỰ A (CÔNG NHÂN) — BẢNG CHẤM CÔNG THÁNG 3/2026

### 2.1. Thông tin gốc & cấu hình tháng

| Mục | Giá trị |
|-----|---------|
| Mã NV / Tên | `01742` / Nguyễn Văn A |
| Phân loại / Chức danh | Công nhân / Worker |
| Nhóm–Bậc lương (2026) | WK · **WK3** |
| Đánh giá KPI hiệu lực 01/01/2026 | Loại **A → 110%** |
| **Ngày nghỉ tuần (CN)** tháng 3/2026 | 01, 08, **15**, 22, 29 (HR set thủ công, **không mặc định** CN) |
| Ngày nghỉ lễ | Không có trong tháng |
| Kiểu công tiêu chuẩn | **Kiểu 1** = Số ngày tháng − Số CN |
| **Ngày công TC** | 31 − 5 = **26 ngày** |
| **Giờ công TC** | 26 × 8 = **208 giờ** |

### 2.2. Bảng chấm công chi tiết (đủ tình huống)

> Ký hiệu In/Out/CS/+/KP do hệ thống **tự nhận dạng** từ máy CC. `Giờ vào/ra TT` = giờ chấm công thực tế.

| Ngày | Thứ | Ca xếp | Giờ vào TT | Giờ ra TT | Ký hiệu | OT | Giờ đêm | Tình huống minh hoạ |
|------|-----|--------|-----------|-----------|---------|-----|---------|---------------------|
| 01 | CN | — | — | — | **CN** | — | — | Nghỉ tuần |
| 02 | T2 | CA1 06:00–14:00 | 05:20 | 14:00 | + | **0.5h** ngày thường | — | OT trước ca: 40' → làm tròn xuống 30' = 0.5h ✅ |
| 03 | T3 | CA1 | 05:32 | 14:00 | + | 0h | — | OT trước ca: 28' < 30' tối thiểu → **0h** ❌ |
| 04 | T4 | CA1 | 06:00 | 15:47 | + | **1.75h** ngày thường | — | OT sau ca: 107' → làm tròn xuống block 15' = 105' = 1.75h |
| 05 | T5 | CA1 | 06:25 | 14:00 | + | — | — | **Đi trễ 25'** (1 lần) — ảnh hưởng thưởng chuyên cần |
| 06 | T6 | CA1 | — | — | **F** | — | — | Nghỉ phép năm (100% lương, 0h CC) |
| 07 | T7 | CA1 | 05:58 | 14:05 | + | 0h | — | Bình thường (lệch <30' cả 2 đầu) |
| 08 | CN | — | — | — | **CN** | — | — | Nghỉ tuần |
| 09 | T2 | CA1 | 06:00 | — | **Out** | — | — | Thiếu giờ ra: không có CC cuối ca trong vùng 4h → 50%, 4h CC |
| 10 | T3 | CA1 | — | 14:00 | **In** | — | — | Thiếu giờ vào: CC đầu sau giờ đầu ca ≥4h → 50%, 4h CC |
| 11 | T4 | CA1 | 05:59 | 14:02 | + | 0h | — | Bình thường |
| 12 | T5 | CA1 | 09:30 | 10:15 | **CS** | — | — | Có CC nhưng không xác định +/In/Out, không có ký hiệu nghỉ → 50%, 4h |
| 13 | T6 | CA1 | — | — | **KP** | — | — | **Có xếp ca nhưng không có dữ liệu CC** → nghỉ không phép (0%) |
| 14 | T7 | **CAB 20:00–05:00+1** | 19:55 | 08:10 (15/3) | + | **1h** OT đêm CN + **2h** OT ngày CN | **7h** (đêm thường) | **Ca vắt qua đêm** — xem mục 2.3 |
| 15 | CN | — | — | — | **CN** | — | — | Nghỉ tuần |
| 16 | T2 | CA1 | 05:57 | 14:01 | + | 0h | — | Bình thường |
| 17 | T3 | CA1 | 06:00 | 14:00 | + | — | — | Bình thường |
| 18 | T4 | CA1 | 06:00 | 14:00 | + | — | — | Bình thường |
| 19 | T5 | CA1 | 06:00 | 14:00 | + | — | — | Bình thường |
| 20 | T6 | **CA3 22:00–06:00+1** | 22:00 | 06:00 (21/3) | + | 0h | **7h** (đêm thường) | **Làm đêm 130%**: 22:00–05:00 = 7h đêm; ra đúng ca, không OT |
| 21 | T7 | CA1 | 06:00 | 14:00 | + | — | — | Bình thường |
| 22 | CN | — | — | — | **CN** | — | — | Nghỉ tuần |
| 23 | T2 | CA1 | 06:00 | 14:00 | + | — | — | Bình thường |
| 24 | T3 | CA1 | 06:00 | 14:00 | + | — | — | Bình thường |
| 25 | T4 | CA1 | 06:00 | 14:00 | + | — | — | Bình thường |
| 26 | T5 | CA1 | 06:00 | 14:00 | + | — | — | Bình thường |
| 27 | T6 | CA1 | 06:00 | 10:00 | **1/2F** | — | — | Nghỉ phép **nửa ngày** (làm 4h + phép 4h) → ngày công ×0.5 |
| 28 | T7 | CA1 | 06:00 | 14:00 | + | — | — | Bình thường |
| 29 | CN | — | — | — | **CN** | — | — | Nghỉ tuần |
| 30 | T2 | CA1 | 06:00 | 14:00 | + | — | — | Bình thường |
| 31 | T3 | CA1 | 06:00 | 14:00 | + | — | — | Bình thường |

### 2.3. Giải thích chi tiết ca vắt 14/3 (theo ví dụ SRS)

NV làm **CAB 20:00 → 05:00+1**, vào 19:55, ra 08:10 (sáng 15/3). Ngày 15/3 là CN (nghỉ tuần).
**Ngày công ghi nhận vào 14/3 (ngày thường).** Phân rã thời gian:

| Khoảng | Số giờ | Phân loại | Tỷ lệ |
|--------|--------|-----------|-------|
| 20:00 – 22:00 | 2h | Làm trong ca (ngày thường) | 100% (trong LCB) |
| **22:00 – 05:00** | **7h** | **Làm đêm ngày thường** | **+30% (130%)** |
| 05:00 – 06:00 | 1h | **OT đêm Chủ nhật** (15/3 là CN) | **270%** |
| 06:00 – 08:10 → làm tròn xuống **2h** | 2h | **OT ngày Chủ nhật** | **200%** |
| OT trước ca (19:55, sớm 5') | 0h | < 30' tối thiểu | **0h** ❌ |

---

## 3. NHÂN SỰ A — TỔNG HỢP CÔNG CUỐI THÁNG

### 3.1. Đếm ký hiệu

| Nhóm | Ngày | Số | Quy đổi công |
|------|------|-----|--------------|
| Làm đủ (+) | 02,03,04,05,07,11,14,16,17,18,19,20,21,23,24,25,26,28,30,31 | **20** | 20.0 |
| Nửa ngày (1/2F) | 27 | 1 | 0.5 |
| In / Out / CS (4h = ½ ngày) | 09,10,12 | 3 | 1.5 |
| Nghỉ phép F (100%) | 06 | 1 | (paid, 0h CC) |
| KP (0%) | 13 | 1 | 0.0 |
| Nghỉ tuần CN | 01,08,15,22,29 | 5 | — |
| **Tổng ngày** | | **31** | |

### 3.2. Các chỉ số tổng hợp

| Chỉ số | Giá trị | Cách tính |
|--------|---------|-----------|
| Ngày công TC | 26 | 31 − 5 CN |
| Giờ công TC | 208 | 26 × 8 |
| **Ngày công thực tế** | **22** | 20 (đủ) + 0.5 (½F) + 1.5 (In/Out/CS) |
| **Ngày hưởng lương** | **23.5** | 26 − KP(1) − phần không lương In/Out/CS (3 × 0.5 = 1.5) |
| Đi trễ | 25 phút / **1 lần** | Ngày 05 |
| **Giờ làm đêm** | **14h** | 14/3 (7h) + 20/3 (7h) |
| OT ngày thường (ngày) | **2.25h** | 0.5h (02) + 1.75h (04) |
| OT ngày nghỉ tuần (đêm) | **1h** | 14/3 |
| OT ngày nghỉ tuần (ngày) | **2h** | 14/3 |
| Phép F đã dùng | 1.5 ngày | 06 (1) + 27 (0.5) |

> **Lưu ý thưởng chuyên cần:** điều kiện = *Ngày hưởng lương = Ngày TC VÀ số lần trễ/về sớm < 3*.
> Vì Ngày hưởng lương 23.5 ≠ 26 (do có KP + In/Out/CS) → **KHÔNG đủ điều kiện → thưởng chuyên cần = 0**.

---

## 4. NHÂN SỰ A — BẢNG TÍNH LƯƠNG THÁNG 3/2026

### 4.1. Mức lương & phụ cấp gốc (hiệu lực 2026)

| Khoản | Mức/tháng |
|-------|-----------|
| LCB (WK3) | 6,200,000 |
| PC Position (PC003) | 0 |
| PC Productivity (PC002) | 500,000 |
| PC Nhà ở (PC001) — *quản lý theo quá trình PC* | 300,000 |
| KPI cơ sở | 1,000,000 |
| **LƯƠNG OT (cơ sở tính TC)** | **8,000,000** |

- **Lương ngày** = 8,000,000 / 26 = **307,692đ**
- **Lương giờ** = 8,000,000 / 208 = **38,462đ**

### 4.2. Các khoản thu nhập (prorate theo Ngày hưởng lương 23.5/26)

| # | Khoản | Công thức | Số tiền |
|---|-------|-----------|---------|
| 1 | LCB hưởng | 6,200,000 × 23.5/26 | 5,603,846 |
| 2 | PC Productivity | 500,000 × 23.5/26 | 451,923 |
| 3 | PC Nhà ở | 300,000 × 23.5/26 | 271,154 |
| 4 | **Tổng PC** | (2)+(3) | **723,077** |
| 5 | Thưởng KPI | 1,000,000 × **110%** × 23.5/26 | 994,231 |
| 6 | Lương làm đêm | 38,462 × 30% × 14h | 161,538 |
| 7a | OT thường (ngày) 150% | 38,462 × 1.5 × 2.25h | 129,808 |
| 7b | OT nghỉ tuần (đêm) 270% | 38,462 × 2.7 × 1h | 103,846 |
| 7c | OT nghỉ tuần (ngày) 200% | 38,462 × 2.0 × 2h | 153,846 |
| 7 | **Tổng Lương OT** | (7a)+(7b)+(7c) | **387,500** |
| 8 | Thưởng chuyên cần | Không đủ điều kiện | 0 |
| 9 | Thu nhập khác | — | 0 |
| 10 | **Trừ đi muộn/về sớm** | (38,462/4) × ROUND(25/15) = 9,615 × 2 | −19,231 |

### 4.3. Kết quả lương (GROSS → ACTUAL)

```
GROSS RECEIVING = LCB(1) + Lương đêm(6) + Thưởng CC(8) + Tổng PC(4)
                + Thưởng KPI(5) + Lương OT(7) + Thu nhập khác(9) − Trừ đi muộn(10)
```

| Khoản | Số tiền |
|-------|---------|
| LCB hưởng | 5,603,846 |
| Lương làm đêm | 161,538 |
| Tổng PC | 723,077 |
| Thưởng KPI | 994,231 |
| Lương OT | 387,500 |
| Thưởng chuyên cần | 0 |
| (−) Trừ đi muộn | −19,231 |
| **GROSS RECEIVING** | **7,850,961** |

**Bảo hiểm & Thuế** (giả định Lương đóng BH = 6,500,000):

| Khoản | Công thức | Số tiền |
|-------|-----------|---------|
| BH NLĐ (BHXH 8% + BHYT 1.5% + BHTN 1%) | 10.5% × 6,500,000 | 682,500 |
| Phí CĐ NLĐ | 0.5% × 6,500,000 | 32,500 |
| TNCT sau giảm trừ | 7,168,461 − 15,500,000 (bản thân) < 0 → 0 | 0 |
| **PIT** | TNCT = 0 | **0** |

```
ACTUAL RECEIVING = GROSS − BH NLĐ − Phí CĐ NLĐ − PIT − Khấu trừ khác
                 = 7,850,961 − 682,500 − 32,500 − 0 − 0
```

| | |
|---|---|
| **THỰC LĨNH (ACTUAL RECEIVING)** | **7,135,961đ** |

> *Toàn bộ số làm tròn đến hàng đơn vị (0 chữ số thập phân) theo quy định Module 05.*

---

## 5. NHÂN SỰ B (NHÂN VIÊN) — MINH HOẠ PIT & NPT

> Ví dụ thứ 2 để thấy luồng ca hành chính + PIT thực sự phát sinh + giảm trừ NPT.

### 5.1. Thông tin & chấm công (tóm tắt)

| Mục | Giá trị |
|-----|---------|
| Mã NV / Tên | `00305` / Trần Thị B |
| Chức danh / Bậc | Manager / SF2.3 |
| Ca | CaHanhChinh 08:00–17:00 (−1h nghỉ = 8h) |
| Người phụ thuộc (NPT) | **2 người** |
| Đánh giá KPI | Loại A (110%) |
| Chấm công T3/2026 | Đi làm **đủ 26 ngày**, không trễ, không OT → Ngày hưởng lương = 26 |

### 5.2. Mức lương gốc

| Khoản | Mức |
|-------|-----|
| LCB (SF2.3) | 25,000,000 |
| PC Position | 5,000,000 |
| PC Productivity | 2,000,000 |
| PC Nhà ở | 300,000 |
| KPI cơ sở | 5,000,000 |
| Thưởng chuyên cần NV | 400,000 |

### 5.3. Tính lương

| Khoản | Công thức | Số tiền |
|-------|-----------|---------|
| LCB hưởng | full 26/26 | 25,000,000 |
| Tổng PC | 5,000,000 + 2,000,000 + 300,000 | 7,300,000 |
| Thưởng KPI | 5,000,000 × 110% | 5,500,000 |
| Thưởng chuyên cần | đủ điều kiện (26=26, trễ 0) | 400,000 |
| **GROSS RECEIVING** | | **38,200,000** |

**BH & PIT** (giả định Lương đóng BH = 30,000,000):

| Bước | Công thức | Số tiền |
|------|-----------|---------|
| BH NLĐ | 10.5% × 30,000,000 | 3,150,000 |
| Phí CĐ NLĐ | 0.5% × 30,000,000 | 150,000 |
| TNCT trước giảm trừ | 38,200,000 − 3,150,000 (BH) | 35,050,000 |
| Giảm trừ bản thân | | −15,500,000 |
| Giảm trừ NPT | 6,200,000 × 2 | −12,400,000 |
| **TNCT sau giảm trừ** | | **7,150,000** |
| **PIT** (bậc 1: 5%) | 7,150,000 × 5% + 0 | **357,500** |

```
ACTUAL RECEIVING = 38,200,000 − 3,150,000 − 150,000 − 357,500 − 0
```

| | |
|---|---|
| **THỰC LĨNH** | **34,542,500đ** |

---

## 6. CÁC ĐIỂM CẦN LSEV XÁC NHẬN (ảnh hưởng số liệu thật)

- **L-Q1:** Bảng mức lương cụ thể từng bậc (WK1–WK10, SF1.x, SF2.x) theo từng năm → cần để ra LCB & Lương OT thật.
- **Lương đóng BH:** trong file giả định (6,500,000 cho A; 30,000,000 cho B) — cần lấy từ Quá trình Lương bảo hiểm thực tế (ràng buộc ≥ LTT 4,730,000).
- **Quy đổi In/Out/CS & KP vào "Ngày hưởng lương":** file đang quy In/Out/CS = ½ ngày (50%). Cần LSEV chốt cách quy đổi chính xác vào công thức Ngày hưởng lương.
- **Phương thức tính OT vào thuế (PT1/PT2):** ví dụ NS A có TNCT = 0 nên không ảnh hưởng; với thu nhập cao cần chọn PT theo từng đợt.
- **PC Nhà ở:** quản lý theo Quá trình Phụ cấp (không hard-code 300k) — mức có thể đổi theo ngày hiệu lực.
