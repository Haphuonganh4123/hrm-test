# ĐẶC TẢ LỖI & YÊU CẦU — TÍNH CÔNG + QUẢN LÝ PHÉP (NextX HRM — LSEV)

> **Mục đích:** tài liệu giao dev, tổng hợp từ file nghiệm thu *"Check Tính công"* + file *"Bổ sung Quản lý phép 09/06"*, đối chiếu SRS v2.1 và **dữ liệu thật trên app-dev** (workspace LSEV HR Test, kỳ 6/2026).
> **Nguyên tắc xuyên suốt:** Sheet 2 (Giờ làm) · Sheet 3 (OT) · Sheet 5 (Giờ đêm) đều là **kết quả dẫn xuất từ MỘT bước gốc — xác định Giờ vào / Giờ ra**. Sửa bước gốc → các phần kia đúng theo.

---

## PHẦN 0 — THUẬT TOÁN GỐC: Xác định Giờ vào / Giờ ra / Ký hiệu

> ⚠️ Chỉ áp dụng cho ngày **CÓ XẾP CA**. Ngày **không xếp ca** → KHÔNG tự gán +/In/Out/CS/KP, và **tuyệt đối không tự sinh RO** (xem Sheet 1).

Với ca có **giờ bắt đầu S**, **giờ kết thúc E** (ca vắt: E sang ngày hôm sau):

- **Cửa sổ VÀO** = `[S − 4h, S + 3h]` → **Giờ vào = lần quẹt SỚM NHẤT** trong cửa sổ.
- **Cửa sổ RA**  = `[E − 3h, E + 4h]` → **Giờ ra = lần quẹt MUỘN NHẤT** trong cửa sổ.

**Bảng quyết định ký hiệu (ưu tiên đơn nghỉ đã duyệt trước):**

| Điều kiện | Ký hiệu | Giờ công | Lương |
|---|---|---|---|
| Có đơn nghỉ cả ngày (đã duyệt) | Ký hiệu nghỉ đã đăng ký (F/S/M/SA/RO/…) | theo danh mục | theo danh mục |
| Có Giờ vào **và** Giờ ra | **+** | tính ở Phần 2 | 100% |
| Có Giờ vào, thiếu Giờ ra | **Out** | 4h | 50% |
| Thiếu Giờ vào, có Giờ ra | **In** | 4h | 50% |
| Thiếu cả 2, có quẹt trong `[S−4h, E+4h]` nhưng ngoài 2 cửa sổ | **CS** | 4h | 50% |
| Không có quẹt nào trong `[S−4h, E+4h]` | **KP** | 0 | 0% |
| Nửa ngày nghỉ + nửa ngày làm | Ghép `½X` + kết quả nửa còn lại | 4h + … | … |

**Ví dụ cửa sổ:**
- CaHanhChinh (08:00–17:00): vào `04:00–11:00`, ra `14:00–21:00`.
- CAB (20:00–05:00⁺¹): vào `16:00–23:00`, ra `02:00⁺¹–09:00⁺¹`.

---

## PHẦN 1 — SHEET "Tổng hợp ngày nghỉ"

**Khách báo:** loại ngày nghỉ tổng hợp sai (13/6 hiện **RO**); double nghỉ cả ngày, sai nửa ngày; ký hiệu chưa đủ.

**Lỗi đã xác minh (NV 0003, 6/2026):**
- Cột RO ở backend (`roDays`) bị **gán cứng = 2** cho 25/300 NV, **không khớp** số ngày trống thật (3 hay 8), **không xuất hiện** trong Chi tiết công.
- Ngày 13/6 (không xếp ca, không chấm công, không đơn nghỉ) bị tự quy thành **RO** — trong khi Chi tiết công để trống.

**Yêu cầu sửa:**
1. Ký hiệu ở Tổng hợp phải **lấy đúng từ Chi tiết công / đơn nghỉ đã duyệt**, không tự suy diễn.
2. **Bỏ tự sinh RO.** RO chỉ phát sinh từ **ngày nghỉ bù được xếp/đăng ký** (RO là ký hiệu hợp lệ, nhưng phải có nguồn thật).
3. Ngày làm việc không xếp ca + không chấm công + không đơn → để trống hoặc **KP** (theo quy tắc), KHÔNG phải RO.
4. Đếm: **cả ngày = 1 · nửa ngày = 0.5** (hết double).
5. Ký hiệu nửa ngày hiển thị rõ `½X / ½Y` (đang dính chuỗi `1/21/2SA`).

---

## PHẦN 2 — SHEET "Tổng hợp giờ làm" (Sheet 2)

**Khách báo:** đang tính cả số lẻ và thời gian ngoài ca.

**Đại lượng dẫn xuất (chỉ cho ngày "+"):**
- `Đi muộn (phút) = max(0, Giờ vào − S)` → trừ giờ làm
- `Về sớm (phút) = max(0, E − Giờ ra)` → trừ giờ làm
- (Đi sớm / Về muộn → KHÔNG cộng vào giờ làm — xem Phần 3 OT)

**Công thức:**
```
Giờ làm = Hca − round_0.25( Đi muộn + Về sớm )
```
- `Hca` = số giờ chuẩn của ca (đã trừ nghỉ): CaHanhChinh/CAB/CA1/2/3 = **8h**, CaTS/CaBTS = **7h**.
- `round_0.25` = quy phút về bội số **0.25h (15 phút)**.
- **Hiển thị 2 chữ số thập phân.** Ngày In/Out/CS → giờ công = **4h** (cố định).

**Bằng chứng lỗi (NV 0003):** vào 07:54 ra 17:01 → app cho **9.12h** (không trừ 1h trưa, cộng cả đến sớm/về trễ). **Đúng = 8.00h.**

**⚠️ Double-count:** ngày 11/6 vào 07:08 → app cho giờ làm **9.9h + OT 0.75h** (52 phút đến sớm bị đếm 2 lần). Đúng: giờ làm **8.00h**, OT **0.75h** — phần đến sớm chỉ thuộc OT.

---

## PHẦN 3 — SHEET "Tổng hợp OT" (Sheet 3)

**Đại lượng dẫn xuất (chỉ cho ngày "+"):**
- `Đi sớm (phút) = max(0, S − Giờ vào)` → OT trước ca
- `Về muộn (phút) = max(0, Giờ ra − E)` → OT sau ca

**Công thức:**
```
OT trước ca = floor_block15( Đi sớm );   nếu < 30 phút → 0
OT sau ca   = floor_block15( Về muộn );  nếu < 30 phút → 0
OT/ngày     = OT trước + OT sau           (trước & sau tính RIÊNG)
```
- `floor_block15` = làm tròn **XUỐNG** theo block 15 phút (0.25h).
- Đi sớm/về muộn bị **chặn tối đa 4h** mỗi đầu (do cửa sổ vào/ra đã giới hạn 4h).
- **Hiển thị 2 chữ số thập phân.**

**Phân loại OT ngày / đêm — theo khung ban đêm hợp pháp 22:00–06:00:**
- Giờ OT rơi vào **22:00–06:00** → **OT đêm**; rơi vào **06:00–22:00** → **OT ngày**.
- VD: OT 05:00–08:00 → 05:00–06:00 = **OT đêm (1h)**, 06:00–08:00 = **OT ngày (2h)**.

**Phân loại theo loại ngày** (thường / Chủ nhật / lễ) × (ngày/đêm) = 8 loại, hệ số 150%→390%.

**🆕 Ngoại lệ theo chức danh — KHÔNG tự động tính OT:**
- **Manager · Senior Manager · Group Leader · Team Leader** → OT = 0 **trừ khi có Đơn đăng ký OT được duyệt**.
- (Hệ thống đã có "Đăng ký OT" + duyệt; cần bổ sung **logic chỉ bắt buộc 4 chức danh này**.)

**🆕 Điều chỉnh OT thủ công (YÊU CẦU MỚI):** cần chỗ để HR **tăng/giảm OT từng người, từng ngày** — giống cơ chế điều chỉnh bảo hiểm (cộng/trừ bù).

---

## PHẦN 4 — SHEET "Phiếu đối chiếu PDF" (Sheet 4)

**Khách báo:** PDF chưa tổng hợp ký hiệu nghỉ, giờ tăng ca, giờ đêm từng ngày.

**Yêu cầu:** theo mẫu Word, mỗi ngày D1–D31 có **3 dòng**:
- **Ngày công**: 1 ký hiệu/ngày (`+`, `CN`, `RO`, `F`…) HOẶC một số (**dương = phút đi muộn, âm = phút về sớm**).
- **Giờ làm đêm**: số giờ đêm.
- **Tăng ca**: dương = OT ngày, âm = OT đêm.
- Dữ liệu PDF phải **khớp 100%** với Chi tiết công (cùng nguồn). Mục này phụ thuộc Phần 1–3–5.

---

## PHẦN 5 — SHEET "Tính giờ làm ban đêm / Ca vắt" (Sheet 5) — GỐC RỄ

**Khách báo:** nhận sai giờ vào/ra ca đêm, tính sai giờ đêm; cần ưu tiên In/Out/CS trước trễ/sớm.

**Bằng chứng lỗi (NV 0028, ca CAB):** app ghép **vào 08:02 / ra 19:57** (lấy nhầm quẹt sáng làm "vào"), đẻ ra **"về sớm 542 phút"**, giờ đêm **0h**. → do gom theo ngày dương lịch thay vì dùng cửa sổ vắt ca.

**Yêu cầu sửa:**
1. **Ghép vào/ra theo cửa sổ (Phần 0)** — ca vắt: giờ ra nằm ở ngày hôm sau. Ưu tiên xác định +/In/Out/CS **trước**, có cặp hợp lệ **rồi mới** tính trễ/sớm (không bung "về sớm 5xx phút").
2. **Giờ làm đêm:**
   ```
   Giờ đêm = (phần [Giờ vào, Giờ ra] giao với 22:00–05:00, trong ca) − 1h nghỉ giữa ca
   ```
   - VD CAB ra đúng 05:00: 22:00–05:00 = 7h − 1h = **6h** (app đang để 7h ❌).
   - Phải tính theo **Giờ ra thực tế**: ra 04:00 → đêm = (22:00–04:00 = 6h) − 1h = **5h**, đồng thời cột **Trễ/Sớm = 60 phút** (về sớm 1h).

3. **Hiển thị OT của ca vắt → dồn sang NGÀY HÔM SAU**, và **phân loại (thường/CN/lễ) theo ngày hôm sau:**

| Ca bắt đầu | Vào → Ra | Giờ làm/đêm ghi ngày | OT hiển thị ngày | Loại OT |
|---|---|---|---|---|
| 12/6 (T6) | 19:55 → 08:05 (13/6) | **12/6**: 8h + 6h đêm | **13/6** | thường: 2h ngày + 1h đêm |
| 13/6 (T7) | 19:50 → 07:00 (14/6) | **13/6**: 8h + 6h đêm | **14/6 (CN)** | **CN: ngày CN + đêm CN** |
| 15/6 (T2) | 18:58 → 07:55 (16/6) | **15/6**: 8h + 6h đêm | **16/6** | thường |

> Lưu ý: ca làm **đêm thứ Bảy** kéo sang sáng **Chủ nhật** → phần OT đó hưởng **giá Chủ nhật (200%/270%)**.

---

## PHẦN 6 — QUẢN LÝ PHÉP NĂM (file Bổ sung 09/06) — 5 ĐIỂM CHƯA LÀM

**Quy tắc trong file:**
- **Quỹ phép = F1 (theo tháng) + F2 (HR nhập thêm)**; F1 theo ngày vào: trước 1/1 = 12; vào ngày 1–15 tháng X = (12−X+1); vào ngày ≥16 = (12−X+1) − 1.
- **Tích lũy:** số phép được nghỉ trong tháng = phép tồn tháng trước + 1.
- **Phép tồn = Quỹ − đã nghỉ (T1→nay) − số tháng còn lại (tháng sau→T12)** ⇒ **không cho ứng phép tháng chưa tới**.
- **Đơn duyệt tách ký hiệu:** 1 khoảng nghỉ → HR tách thành nhiều ký hiệu (F→MP→**O** khi hết phép).

**Đã xác minh trên live (kỳ 6/2026):** có **61 NV vào công ty trong năm 2026** nhưng vẫn được cấp **12 ngày phép** (kể cả người vào tháng 6) → quỹ đang **gán cứng 12**.

| # | Yêu cầu | Trạng thái | Bằng chứng |
|---|---|---|---|
| 1 | Quỹ phép **prorate theo ngày vào** (1–15 vs ≥16) | ❌ Chưa | 45/61 NV vào 2026 vẫn = 12 |
| 2 | Ô **F2 — phép cộng thêm (HR nhập)** | ❌ Chưa | Không có ô nhập |
| 3 | **Phép tồn theo tháng** (không ứng phép tương lai; T6 ≈ 6 ngày) | ❌ Chưa | Hiện cả quỹ năm 12 |
| 4 | **Tích lũy +1/tháng** | ❌ Chưa | Cùng gốc #3 |
| 5 | **Đơn duyệt tách ký hiệu** F→MP→O, tự chuyển O khi hết phép | ❌ Chưa | Form chỉ 1 ký hiệu/đơn |

**Đã có (không cần làm lại):** tạo đơn nghỉ + duyệt 3 cấp, trừ phép F khi nghỉ, hiển thị phép còn năm, đăng ký OT + duyệt.

---

## PHẦN 7 — TEST CASE (dữ liệu thật 6/2026 — dùng nghiệm thu)

| # | NV / Ca | Quẹt | KH | Giờ làm | OT | Đêm | App đang ra (SAI) |
|---|---|---|---|---|---|---|---|
| TC1 | 0003 · CaHanhChinh 1/6 | 07:54 → 17:01 | + | **8.00h** | 0 | 0 | 9.12h, Out |
| TC2 | 0003 · CaHanhChinh 11/6 | 07:08 → 17:01 | + | **8.00h** | **0.75h** | 0 | 9.9h + 0.75h (đếm 2 lần) |
| TC3 | 0003 · CaHanhChinh 17/6 | 07:56 → (không ra) | **Out** | 4h | 0 | 0 | Out ✓ |
| TC4 | 0028 · CAB 1/6 (vắt ca) | 19:56 → 08:02 (2/6) | + | **8.00h** | **3h** → ghi 2/6 | **6h** | giờ làm 0, đêm 0, "sớm 542'" |
| TC5 | CAB | chỉ quẹt 23:30 (vùng giữa) | **CS** | 4h | 0 | — | — |
| TC6 | có xếp ca, 0 quẹt, 0 đơn | — | **KP** | 0 | 0 | — | đang tự gán RO |
| TC7 | NV vào 01/06/2026 | — | — | — | — | — | phép F = 12 (đúng phải ~7) |

---

## PHẦN 8 — TIÊU CHÍ NGHIỆM THU (Acceptance Criteria)

1. Có đủ Giờ vào + Giờ ra trong 2 cửa sổ → ký hiệu **"+"** (không còn "Out" sai).
2. Giờ làm = Hca − (đi muộn + về sớm), làm tròn 0.25, 2 thập phân; **không** cộng đi sớm/về muộn.
3. OT trước & sau tính riêng, block 15' làm tròn xuống, ≥0.5h/phía; **không** double-count với giờ làm.
4. OT phân loại ngày/đêm theo **22:00–06:00**; theo loại ngày (thường/CN/lễ).
5. Ca vắt: ghép đúng giờ ra hôm sau; **OT dồn sang hôm sau, phân loại theo ngày hôm sau**.
6. Giờ đêm = giao 22:00–05:00 trong ca **− 1h nghỉ**, theo giờ ra thực tế.
7. Đi trễ/về sớm chỉ tính cho ngày "+"; ngày In/Out/CS/nghỉ → không tính.
8. Ngày không xếp ca → không tự sinh ký hiệu (**không tự đẻ RO**).
9. 4 chức danh quản lý: OT = 0 nếu không có đơn duyệt; có chỗ **điều chỉnh OT tay**.
10. Phép: quỹ prorate theo ngày vào + F2; áp **trần phép tồn theo tháng**; đơn duyệt **tách nhiều ký hiệu** + tự chuyển O khi hết phép.

---

## PHẦN 9 — CẦN KHÁCH/LSEV XÁC NHẬN
1. Hướng làm tròn **đi muộn/về sớm** (trừ giờ làm): làm tròn **gần nhất** hay **xuống**? (OT đã chốt: làm tròn **xuống**.)
2. Ngày vừa có đơn nghỉ nửa ngày vừa đi làm nửa ngày → quy tắc ghép ký hiệu `½X + (+/In/Out)`.
3. Ngày OFF (nghỉ tuần/CN) mà NV vẫn đi làm → toàn bộ tính **OT ngày nghỉ tuần (200%/270%)**, đúng không?
4. Vị trí **1h nghỉ giữa ca** đối với ca đêm (để xác nhận luôn trừ vào phần đêm).

---
*Tài liệu lập bởi BA dựa trên dữ liệu app-dev (LSEV HR Test) kỳ 6/2026, đối chiếu SRS v2.1 + file yêu cầu gốc LSEV.*
