# Đối chiếu & lỗi phần TÍNH LƯƠNG – LSEV HRM (tháng 6/2026)

> **Nguồn đối chiếu:** `2026.04.13. Yeu cau chi tiet he thong - Phan Luong.xlsx` (sheet *Bảng tổng hợp lương*, *Định mức theo luật*, *Định mức của công ty*) ⇄ Bảng tổng hợp lương hệ thống xuất (`tong-hop-luong-6-2026.xlsx`) + dữ liệu API `api-dev.nextx.vn/hrm` + các phiếu lương PDF.
> **Môi trường:** app-dev.nextx.vn – workspace *LSEV HR Test*. Tháng 6/2026: **ngày công tiêu chuẩn = 26**, giờ công chuẩn = 208 (26×8). Chủ nhật = ngày nghỉ tuần.

---

## A. TÓM TẮT – mức độ ưu tiên

| # | Lỗi | Ảnh hưởng | Ưu tiên |
|---|-----|-----------|---------|
| **L1** | Trừ tiền đi muộn/về sớm sai công thức + bỏ "về sớm" | Sai số tiền thực lãnh | 🔴 Cao |
| **L2** | Ngày công hưởng lương > ngày công tiêu chuẩn (cộng trùng Chủ nhật) | Trả lương cơ bản >100% + trùng OT 200%; mất chuyên cần oan | 🔴 Cao |
| **L3** | Lương đóng BH lấy = lương cơ bản, không đọc từ Quá trình đóng BH | Sai toàn bộ phí BH của NV thai sản/thử việc | 🔴 Cao |
| **L4** | Không tách Thử việc / Chính thức (lương & ngày công) | NV chuyển TV→CT giữa tháng bị tính sai | 🟠 Vừa |
| **L5** | Thiếu cột "Tăng ca tính thuế" → tách OT chịu thuế/miễn thuế cho PIT | Sai thu nhập tính thuế TNCN | 🟠 Vừa |
| **L6** | Thiếu loại "Phụ cấp khác" (mọi PC động trong Quá trình phụ cấp) | Bỏ sót phụ cấp của một số NV | 🟠 Vừa |
| **L7** | Định dạng & làm tròn (bỏ "đ", dấu ",", làm tròn hàng đơn vị) | Trình bày / đối chiếu | 🟢 Thấp |

---

## B. ĐỐI CHIẾU CỘT (tài liệu 51 cột ⇄ hệ thống 47 cột)

### B1. Cột KHỚP (giữ nguyên)
Emp.ID, Họ tên, Lương đóng BH*, Số lần trễ/về sớm, Giờ làm đêm, Lương đêm, Tổng phụ cấp, Tổng tiền OT, Thưởng chuyên cần*, Thưởng KPI, PC Nhà ở, PC Chức vụ, PC Năng suất, Thu nhập khác (chịu/miễn thuế), Khấu trừ khác, BHXH/BHYT/BHTN NLĐ + Cty, Số NPT, TN chịu thuế trước/sau GT, Thuế TNCN, Thực lãnh, Tổng chi phí Gross.
*(có lỗi giá trị – xem mục C)*

> Hệ thống **chi tiết hơn tài liệu** ở phần OT (tách 6 loại giờ: ngày/đêm × thường/cuối tuần/lễ), có thêm KPI cơ sở, Lương OT cơ sở → **giữ nguyên, tốt**.

### B2. Cột LỆCH so với yêu cầu

| Tài liệu yêu cầu | Hệ thống | Vấn đề |
|---|---|---|
| **Basic salary (Probation)** + **Basic salary (contractual)** – *2 cột* | `Lương cơ bản` – *1 cột gộp* | ❌ Không tách lương thử việc / chính thức → **L4** |
| **Paid days (contractual)** + **Paid days (probation)** – *2 cột* | `Ngày công hưởng lương` – *1 cột* | ❌ Không tách ngày công TV/CT → **L4** |
| **Số phút đi trễ + về sớm** | `Số phút trễ` | ❌ Chỉ lấy đi trễ, bỏ về sớm → **L1** |
| **Trừ tiền đi trễ về sớm** | `Phạt đi trễ` | ❌ Sai công thức → **L1** |
| **Insurance salary** (từ Quá trình đóng BH) | `Lương đóng BH` | ❌ Lấy = lương cơ bản → **L3** |
| **Tăng ca tính thuế** (cột riêng) | *(không có)* | ❌ Thiếu → PIT sai → **L5** |
| **Phụ cấp khác** (mọi PC động) | Chỉ 3 mã PC001/002/003 | ⚠️ Bỏ sót → **L6** |
| Bậc lương, Department, Position, Cost center | *(không có)* | ⚠️ Thiếu cột thông tin (nhẹ) |
| Giảm trừ bản thân, Tiền chi NPT, Lương sau thuế | *(ẩn trong tính PIT)* | ⚠️ Thiếu cột minh bạch (nhẹ) |

---

## C. CHI TIẾT CÁC LỖI

### 🔴 L1 – Trừ tiền đi muộn / về sớm

**Yêu cầu (sheet Bảng tổng hợp lương, cột "Trừ tiền đi trễ về sớm"):**
```
Trừ tiền = (Lương giờ ÷ 4) × round( Tổng số phút (đi trễ + về sớm) ÷ 15 , 0 )
```
- `Lương giờ = Lương OT cơ sở ÷ 208`.
- `Lương giờ ÷ 4` = tiền của 15 phút.
- Tổng số phút = **đi trễ + về sớm** (cột "Số phút đi trễ + về sớm").

**Hệ thống sai 2 chỗ:**
1. Chỉ lấy **phút đi trễ**, **bỏ hoàn toàn phút về sớm** (bảng chỉ có cột `Số phút trễ`).
2. Con số xuất ra **không khớp cả /15 lẫn /4** — thấp hơn nhiều lần thực tế.

**Ví dụ đối chiếu:**
| NV | Phút trễ+sớm | Hệ thống trừ | Đúng theo công thức |
|---|---|---|---|
| Phạm Mạnh Quyền (0003) | 254' | **54,689đ** | `44,435 × round(254/15=17)` = **755,395đ** |
| Nguyễn Thị Hạnh (0028) | 159' | **8,349đ** | `10,853 × round(159/15=11)` = **119,383đ** |

> ⚠️ Ghi chú tài liệu có chỗ ghi "/4" gây hiểu nhầm — bản chất là `(Lương giờ/4)` nhân với **số block 15 phút**, nên phải chia tổng phút cho **15**, không phải cho 4.

---

### 🔴 L2 – Ngày công hưởng lương > ngày công tiêu chuẩn (cộng trùng Chủ nhật)

**Hiện tượng:** 6 NV có "Ngày công hưởng lương" > 26 (điều không thể với lương cơ bản):

| Mã | Tên | Ngày công hưởng lương | Chuẩn | OT cuối tuần |
|---|---|---|---|---|
| 1460 | Lương Đức Cảnh | **28** | 26 | 21h |
| 2020 | Hà Thị Thu | 27 | 26 | 28h |
| 2194 | Hà Thị Kính | 27 | 26 | 33h |
| 1962 | Sùng A Tuấn | 27 | 26 | 6h |
| 2309 | Lò Thị Xuyến | 27 | 26 | 6h |
| 1356 | Nguyễn Thị Xuân | 26.5 | 26 | 33h |

**Nguyên nhân (chứng minh trên Cảnh 1460):** hệ thống **cộng ngày Chủ nhật đi làm vào "Ngày công hưởng lương"** dù ngày đó `isWeeklyOff=true`, `workedHours=0`:
```
26 ngày thường (T2–T7)                 = 26
+ CN 06-14 (đi làm, ca CaHanhChinh)     = +1
+ CN 06-21 (đi làm, ca CaHanhChinh)     = +1
                          paidWorkdays  = 28
```
→ 2 Chủ nhật này **đã được trả 200%** ở cột OT cuối tuần ⇒ **trả 2 lần**: (1) cộng vào ngày công (lương cơ bản × 28/26 = 107%) **+** (2) OT cuối tuần 200%.

**Phạm vi rộng hơn 6 người:** **235/309 NV** có làm ngày nghỉ. 6 người chỉ là phần *lộ ra >26* (vì đi đủ 26 ngày thường). Các NV khác vẫn bị cộng trùng Chủ nhật nhưng **bị che** do ngày thường <26 ⇒ **cần rà cả 235 NV**.

**Hệ quả kép:**
1. Lương cơ bản trả >100% + trùng OT 200%.
2. **Mất thưởng chuyên cần oan** — điều kiện chuyên cần = *"Ngày được trả lương = ngày công tiêu chuẩn (26)"*; để 27/28 ≠ 26 → rớt điều kiện (đúng note vàng của khách *"sửa ngày hưởng lương để test công thức chuyên cần"*).

**Quy tắc đúng:**
> "Ngày công hưởng lương" (tính lương cơ bản) **chỉ lấy từ ngày công tiêu chuẩn** (ngày làm việc theo lịch), **tối đa 26**. Ngày nghỉ tuần/lễ đi làm **KHÔNG cộng vào ngày công**, chỉ trả qua **OT cuối tuần (200%) / OT lễ (300%)**.

**Lưu ý dev:** cách gộp `paidWorkdays` hiện **không nhất quán** — 2 NV có bản ghi Chủ nhật y hệt nhau (cùng `CaHanhChinh` + có chấm) ra kết quả khác nhau ⇒ review lại hàm tổng hợp ngày công khi có xếp ca vào ngày nghỉ.

---

### 🔴 L3 – Lương đóng BH lấy sai nguồn

**Yêu cầu (cột BHXH):** *"Theo quá trình đóng BH và định mức/tỷ lệ theo luật. Có thể cộng/trừ mức đóng điều chỉnh ngoài phần tính theo quá trình (báo tăng/giảm bù tháng trước)."*
→ **Lương đóng BH lấy từ "Quá trình lương bảo hiểm"** của từng NV (theo ngày hiệu lực), **không tự tính từ lương cơ bản**.

**Các trường hợp Lương đóng BH = 0:**
- NV **nghỉ thai sản** → lịch sử BH trống.
- NV **thử việc chưa ký HĐ** → chưa nhập lịch sử BH.
- Có nhập nhưng **ngày hiệu lực sau tháng tính lương**.

→ Khi Lương đóng BH = 0 thì **mọi phí BH + Công đoàn (cả NLĐ lẫn Cty) = 0**.

**Hệ thống sai:** đang để **Lương đóng BH = Lương cơ bản** (VD 5,830,000) cho NV thai sản/thử việc ⇒ phần Cty tính ra tiền trong khi phần NLĐ = 0 → **bất nhất**. Đúng ra phải = 0 và mọi phí = 0.

**Công thức đúng:**
```
1. Lương đóng BH = giá trị trong Quá trình lương BH (theo ngày hiệu lực) ; không có → 0
2. Phí = tỷ lệ × Lương đóng BH   (NLĐ 8/1.5/1% + CĐ 0.5% ; Cty 17.5/3/1% + CĐ 2%)
3. + điều chỉnh bù/trừ hàng tháng (nếu có)
```

---

### 🟠 L4 – Không tách Thử việc / Chính thức

Tài liệu tách **Basic salary (Probation) / (contractual)** và **Paid days (Probation)/(contractual)**. NV chuyển thử việc→chính thức **giữa tháng** phải tính theo 2 đoạn ngày với 2 mức lương. Hệ thống chỉ 1 cột lương + 1 cột ngày công ⇒ sai với nhóm này. (Công thức OT cột "Night shift salary" của tài liệu cũng lưu ý: *"lấy mức lương giờ cuối tháng nếu trong tháng có thử việc"*.)

---

### 🟠 L5 – Thiếu cột "Tăng ca tính thuế" → sai base PIT

**Yêu cầu:** tách phần OT **chịu thuế** = `Lương giờ × tổng giờ OT`; phần vượt (0.5/1.0/2.0 lần) **miễn thuế**.
```
TN tính thuế trước GT = Gross − Tăng ca KHÔNG tính thuế − Bảo hiểm NLĐ − Thu nhập không tính thuế
```
Hệ thống gộp tất cả vào "Tổng tiền OT", **chưa có cột tách phần OT miễn thuế** ⇒ cần kiểm tra base tính thuế TNCN có trừ đúng phần OT miễn thuế không. **(Cần verify thêm trên 1 NV có OT lớn.)**

Tỷ lệ OT (tài liệu *Định mức theo luật*): TC ngày thường 1.5 · đêm thường 2.0 · **cuối tuần ngày 2.0 · cuối tuần đêm 2.7** · lễ ngày 3.0 · lễ đêm 3.9.

---

### 🟠 L6 – Thiếu "Phụ cấp khác"

Yêu cầu: *"Tính theo ngày trả lương đối với tất cả các phụ cấp khác trong Quá trình phụ cấp, ngoài các khoản đã liệt kê"*. Hệ thống chỉ có 3 mã cố định (PC001 Nhà ở, PC002 Năng suất, PC003 Chức vụ) ⇒ NV có phụ cấp loại khác bị **bỏ sót**.

---

### 🟢 L7 – Định dạng & làm tròn

Ghi chú tài liệu: *"Làm tròn đến hàng đơn vị tất cả số liệu"*. Áp dụng toàn bảng:
1. Bỏ ký hiệu `đ`.
2. Dùng dấu `,` ngăn cách hàng nghìn.
3. Làm tròn về **hàng đơn vị** (đồng).

---

## D. CÁC ĐỊNH NGHĨA CHUẨN (tham chiếu – sheet "Định mức của công ty")

| Khái niệm | Công thức |
|---|---|
| Lương OT cơ sở | Lương cơ bản + PC công việc + PC Năng suất + PC Nhà ở + Mức KPI cơ sở (nếu có) |
| Lương ngày | Lương OT ÷ Ngày công tiêu chuẩn (26) |
| Lương giờ | Lương OT ÷ Giờ công tiêu chuẩn (208) |
| Thưởng KPI | Theo loại: S 120% · A 110% · B 100% · C 90% · D 70% — tính theo ngày hưởng lương |
| Thưởng chuyên cần | ĐK: **ngày được trả lương = ngày công tiêu chuẩn** & số lần đi muộn+về sớm < 3; mức 600k (công nhân) / 400k (NV) |

---

## E. VIỆC CẦN LÀM TIẾP (đề xuất)

- [ ] Sửa **L1 → L2 → L3** trước (ảnh hưởng trực tiếp tiền thực lãnh).
- [ ] Quét toàn bộ **235 NV làm ngày nghỉ** → danh sách chính xác ai bị cộng trùng bao nhiêu ngày Chủ nhật.
- [ ] Verify **L5** (OT có bị tính vào thu nhập chịu thuế sai không) trên 1 NV có OT lớn.
- [ ] Sau khi sửa L2, kiểm lại **thưởng chuyên cần** có bật đúng cho nhóm đi đủ công không.
