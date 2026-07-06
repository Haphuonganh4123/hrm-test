# YÊU CẦU TIỀN LƯƠNG — tổng hợp từ 2 file test

Nguồn: **`Test_Bang_tong_hop_luong_6_2026.xlsx`** (khách audit, có cột "sai/tính lại" + comment ẩn HR TEAM) đối chiếu **`tong-hop-luong-6-2026.xlsx`** (hệ thống xuất). Kỳ 6/2026, 305 NV.

> Đã bóc **đầy đủ**: cột đánh dấu sai/ok + **comment ẩn của Đỗ Thị Bích Lâm/HR TEAM** + ghi chú định dạng + ghi chú "chưa test".

---

## A. LỖI CÔNG THỨC — cần sửa (khách chấm "sai")

| # | Cột | Hiện trạng | Công thức ĐÚNG | Bằng chứng |
|---|---|---|---|---|
| A1 | **Tổng phụ cấp** | ❌ Sai | Prorate: **PC × (Ngày hưởng ÷ Ngày công TC)** | 0045: →1,903,846 (15/26) |
| A2 | **Thưởng chuyên cần** | ❌ Sai — không thưởng dù đủ đk | Đủ đk (**Ngày hưởng = Ngày TC** VÀ **số lần trễ/sớm < 3**) → **CN 600k / NV 400k** | 0028 đủ đk mà = 0 |
| A3 | **Thưởng KPI** | ❌ Sai | **KPI cơ sở × Tỷ lệ đánh giá gần nhất ÷ Ngày công TC × Ngày hưởng lương** *(comment R11)* | 0003: 6.66tr→9.62tr |
| A4 | **Trừ đi trễ/về sớm** | ❌ Sai — quá thấp | **Đổi tên "Phạt" → "Trừ"**. Tiền trừ = **Lương giờ/4 × số nguyên(Số phút trễ+sớm ÷ 4)** *(comment T11)* ⚠️ xem mục E | 0012: 97k→1.5tr (~15 lần) |
| A5 | Lương đêm | ✅ OK | — | — |
| A6 | Tổng tiền OT | ✅ OK (khách chấm) | ⚠️ nhưng **OT theo giờ vẫn lỗi làm tròn 15'** (xem tài liệu Chấm công) | — |

---

## B. THÊM / SỬA CỘT — comment ẩn của HR TEAM

| # | Ô | Cột | Yêu cầu |
|---|---|---|---|
| B1 | E11 | Lương CB thử việc | **Thêm cột** tính lương thời gian thử việc; **ẩn** nếu tháng đó không có ngày thử việc |
| B2 | L11 | Lương CB theo ngày công | **Chưa có — cần thêm cột**: = LCB ÷ Ngày công TC × Ngày hưởng (chia thử việc/chính thức) |
| B3 | W11 | Ngày hưởng lương | **Tách 2 cột**: Ngày hưởng **chính thức** + Ngày hưởng **thử việc** |
| B4 | Y11 | Số phút / số lần trễ+sớm | Đảm bảo là **tổng chính xác** phút trễ+sớm và số lần trong tháng (lấy đúng từ bảng công) |

---

## C. ĐỊNH DẠNG BẢNG — "Áp dụng toàn bộ bảng tính"

| # | Yêu cầu | Lưu ý (BA đề xuất) |
|---|---|---|
| C1 | Bỏ ký hiệu **"đ"** ở tất cả các cột | → để ô là SỐ, kế toán tính/SUM được |
| C2 | Định dạng dấu **","** ngăn cách hàng nghìn | ⚠️ **Loại trừ cột ID** (Số TK / Mã NV / CCCD / MST) — không thêm dấu phẩy |
| C3 | **Làm tròn đến hàng đơn vị** | ⚠️ Làm tròn **giá trị thật** (số nguyên đồng), không chỉ hiển thị — để tổng khớp |

> Kỹ thuật: áp định dạng `#,##0` cho **các cột tiền** (đã gộp cả C1–C3).

---

## D. GHI CHÚ / TRẠNG THÁI (không phải lỗi)

| # | Ghi chú | Ý nghĩa |
|---|---|---|
| D1 | *"Màu vàng: sửa ngày hưởng lương để test công thức chuyên cần"* | Khách sửa Ngày hưởng = 26 cho **0003, 0028, 0183** (ô vàng) để dựng ca test chuyên cần |
| D2 | *"Không check lại, cần đảm bảo lấy đúng dữ liệu như bảng công"* | Cột ngày công / OT / trễ-sớm phải **lấy đúng từ Bảng công**, không tính lại |
| D3 | *"Chưa test, chờ mẫu import"* | Cột **Thu nhập khác** — chưa test, chờ mẫu file import |
| D4 | *"Chưa test, chờ tính lại các thành phần"* | Chưa test cột này, chờ các thành phần khác tính đúng |

---

## E. CẦN KHÁCH LÀM RÕ (mâu thuẫn/nghi vấn)

| # | Điểm | Chi tiết |
|---|---|---|
| E1 | Công thức **Trừ trễ/sớm** — chia /4 hay /15? | **SRS (đã ký):** (Lương giờ/4) × ROUND(phút **/15**). **Comment T11 (mới):** Lương giờ/4 × số nguyên(phút **/4**). → Hai cái khác nhau; xét về logic /15 mới hợp lý (trễ 60' → trừ 1h lương), /4 sẽ trừ quá nặng → **nhiều khả năng /15, cần khách xác nhận**. |

---

## TÓM TẮT
- **4 lỗi công thức** cần sửa: Phụ cấp, Chuyên cần, KPI, Trừ trễ (+ đổi tên "Phạt"→"Trừ").
- **4 việc cột**: thêm Lương CB thử việc, thêm Lương CB theo ngày công, tách Ngày hưởng CT/TV, đảm bảo phút/lần trễ đúng.
- **3 định dạng**: bỏ "đ", dấu ",", làm tròn đơn vị (loại trừ cột ID).
- **Lương đêm + Tổng OT: OK** (nhưng OT giờ còn lỗi làm tròn 15' — thuộc phần Chấm công).
- **1 điểm làm rõ**: công thức Trừ trễ /4 hay /15.

*Gốc rễ: cột ngày công / OT / trễ-sớm phải đúng từ Bảng công — nên sửa Chấm công trước thì Lương mới chuẩn.*
