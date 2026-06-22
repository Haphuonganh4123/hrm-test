# QUẢN LÝ NGHỈ PHÉP — HỆ THỐNG CẦN BỔ SUNG GÌ

> Nguồn: file *"2026.06.09. Bổ sung phần chấm công — Quản lý phép"* (2 sheet: **Quy định về phép năm** + **Mẫu duyệt xin nghỉ**).
> Đối chiếu dữ liệu thật app-dev (LSEV HR Test, kỳ 6/2026).

---

## A. SHEET 1 — Quy định quỹ phép & phép tồn (tóm tắt)

1. **Quỹ phép năm = F1 (theo tháng) + F2 (HR nhập thêm)**, F1 theo **ngày vào công ty**:
   - Vào **trước 1/1** năm nay → **12 ngày**.
   - Vào ngày **1–15** tháng X → = số tháng từ X → 12 (được tính tháng vào).
   - Vào ngày **≥16** tháng X → = số tháng từ X → 12 **− 1**.
2. **Tích lũy:** mỗi tháng +1 ("số phép được nghỉ trong tháng = phép tồn tháng trước + 1").
3. **Phép tồn** (số được dùng tại 1 thời điểm):
   ```
   Phép tồn = Quỹ phép − đã nghỉ (T1→nay) − số tháng còn lại (tháng sau→T12)
   ```
   ⇒ **Không cho ứng phép của tháng chưa tới.** VD quỹ 12, tháng 6, chưa nghỉ → chỉ được dùng **6 ngày** (không phải 12).

---

## B. SHEET 2 — Mẫu duyệt xin nghỉ (TRỌNG TÂM)

Sheet 2 mô tả **đơn xin nghỉ + duyệt** gồm **2 phần**:

**① Phần Người lao động nộp:** một **khoảng ngày** (Từ ngày – Đến ngày) · Loại nghỉ (Cả ngày / Nửa ngày) · Lý do · *nút Gửi đơn*.

**② Phần duyệt:** Bộ phận duyệt → chuyển Nhân sự. Khi **Nhân sự duyệt**, được **TÁCH khoảng nghỉ đó thành nhiều đoạn, mỗi đoạn gán 1 ký hiệu khác nhau**.

**Ví dụ trong file (NV 2309 — Lò Thị Xuyến):**
- NV xin nghỉ **08/06 → 11/06** (4 ngày, cả ngày).
- Nhân sự duyệt **tách thành:**

| Từ ngày | Đến ngày | Ký hiệu |
|---|---|---|
| 08/06 | 09/06 | **F** (phép năm) |
| 10/06 | 10/06 | **MP** (chế độ nữ) |
| 11/06 | 11/06 | **O** (hết phép) |

→ Ý nghĩa nghiệp vụ: **dùng hết phép F thì TỰ CHUYỂN sang O**; trong một đơn có thể có **nhiều loại ký hiệu**. Form còn hiển thị **"Số ngày phép được nghỉ trong tháng"** để biết khi nào hết.

---

## C. HỆ THỐNG ĐANG CÓ (không cần làm lại)
- ✅ Tạo đơn nghỉ + **duyệt 3 cấp** (Tổ trưởng → TBP → Nhân sự).
- ✅ **Trừ phép F** khi nghỉ; hiển thị "Phép còn lại năm".
- ✅ Đăng ký OT + duyệt.

---

## D. HỆ THỐNG CẦN BỔ SUNG

| # | Cần bổ sung | Thuộc | Bằng chứng đang thiếu |
|---|---|---|---|
| **1** | **Đơn duyệt TÁCH KÝ HIỆU**: khi Nhân sự duyệt 1 đơn (khoảng ngày), cho **tách thành nhiều đoạn — mỗi đoạn 1 ký hiệu** (F / MP / O…) | Sheet 2 | Form hiện chỉ chọn **1 ký hiệu / đơn** |
| **2** | **Tự chuyển sang O khi hết phép F**: hệ thống tự đề xuất F cho tới khi cạn quỹ, phần dư → **O** | Sheet 2 | Chưa có logic chuyển O |
| **3** | **Hiển thị "Số phép được nghỉ trong tháng"** ngay trên form duyệt/đơn | Sheet 2 | Form không hiện số phép tồn |
| **4** | **Quỹ phép tính theo NGÀY VÀO** (1–15 vs ≥16) + ô **F2 (HR nhập thêm)** | Sheet 1 | 61 NV vào năm 2026 vẫn được **12 ngày** (kể cả vào tháng 6) |
| **5** | **Trần "phép tồn theo tháng"** — không cho ứng phép tháng chưa tới (T6 ≈ 6 ngày), tích lũy +1/tháng | Sheet 1 | Trang cá nhân hiện **"0/12"** (cả năm), không có trần tháng |

---

## E. TIÊU CHÍ NGHIỆM THU
1. Khi duyệt 1 đơn nghỉ khoảng nhiều ngày → Nhân sự **tách được** thành nhiều dòng ký hiệu khác nhau (F/MP/O…), mỗi dòng một đoạn ngày.
2. Hết quỹ F → các ngày sau **tự gán O** (hoặc cảnh báo để HR gán O).
3. Form đơn/duyệt **hiển thị số phép được nghỉ trong tháng** (phép tồn).
4. NV vào công ty **giữa năm** → quỹ phép **prorate đúng** (vào 1/6 → ~7 ngày, không phải 12); có ô **F2**.
5. Trong 1 tháng, NV **không nghỉ vượt phép tồn** (T6 không cho nghỉ >6 nếu chưa tích đủ).

---

## F. CẦN KHÁCH XÁC NHẬN
- Khi tách ký hiệu, hệ thống **tự đề xuất** (F→O) hay để Nhân sự **chọn tay** từng đoạn?
- Thứ tự ưu tiên dùng ký hiệu khi tách (F trước, rồi mới O? MP do NV ghi rõ?).

---
*Lập bởi BA — đối chiếu file Quản lý phép 09/06 với app-dev (LSEV HR Test) kỳ 6/2026.*
