# Lỗi phần LƯƠNG còn tồn – cần fix (LSEV HRM, tháng 6/2026)

> Kết quả **test trực tiếp trên hệ thống** app-dev.nextx.vn – workspace *LSEV HR Test*, ngày kiểm tra sau đợt fix gần nhất.
> Case test: **Quyền (0003)** và **Hạnh (0028)**.

## Trạng thái tổng quan

| Lỗi | Nội dung | Trạng thái |
|---|---|---|
| L2 | Ngày công hưởng lương > 26 | ✅ Đã fix |
| L3 | Lương đóng BH thai sản/thử việc = 0 | ✅ Đã fix |
| **L1** | Trừ tiền đi trễ/về sớm | ❌ **CHƯA FIX** |
| **L8** | "BH Công ty (tổng)" gộp nhầm phí công đoàn | ❌ **CHƯA FIX** |
| **L9** | Thu nhập chịu thuế trừ nhầm phí công đoàn NLĐ | ❌ **CHƯA FIX** |
| L10 | Phương án OT không tính thuế | ⚠️ Cơ chế đã có, chờ khách chốt toggle |

> Đã loại trừ khả năng "cache cũ": trong cùng bản ghi payroll, L3 (lương đóng BH = 0) đã cập nhật → bảng lương đang tính lại được, nên L1/L8/L9 là **thật sự chưa sửa**.

---

## 🔴 L1 – Trừ tiền đi trễ / về sớm

**Công thức đúng:**
```
Trừ đi trễ/về sớm = (Lương giờ ÷ 4) × INT( (phút đi trễ + phút về sớm) ÷ 15 )
```
- `Lương giờ = Lương OT cơ sở ÷ 208`
- **Phải cộng cả phút về sớm** (không chỉ đi trễ).
- Lấy **phần nguyên INT** (bỏ phần lẻ), **KHÔNG** làm tròn ở bước trung gian – chỉ làm tròn kết quả cuối.

**2 lỗi hiện tại:** (1) bỏ sót phút về sớm; (2) con số ra quá thấp, không khớp công thức.

**Case test:**
| NV | Phút trễ + sớm | Hệ thống (SAI) | Đúng |
|---|---|---|---|
| Quyền 0003 | 254 (77 trễ + **177 về sớm**) | 54,689 | **710,962** |
| Hạnh 0028 | 159 (159 trễ) | 8,349 | **108,534** |

Chi tiết Quyền: `177,740 ÷ 4 × INT(254 ÷ 15 = 16) = 44,435 × 16 = 710,962đ`
(Quyền có 177 phút về sớm ngày 27/6 – hệ thống đang bỏ.)

---

## 🔴 L8 – "BH Công ty (tổng)" gộp nhầm 2% phí công đoàn công ty

**Đúng:** "BH Công ty (tổng)" **chỉ gồm 3 khoản bảo hiểm**: BHXH 17.5% + BHYT 3% + BHTN 1%. **Phí công đoàn 2% là khoản RIÊNG.**

**Sửa kèm:** cột **"Tổng chi phí Gross"** phải cộng công đoàn riêng:
```
Tổng chi phí Gross = Gross receiving + BH Công ty (3 khoản) + Phí CĐ Công ty (2%)
```
(Hiện "Tổng chi phí" đúng về con số tổng vì đang cộng cả cục, nhưng cột "BH tổng" hiển thị sai. Khi tách CĐ ra phải nhớ cộng lại CĐ vào Tổng chi phí, kẻo hụt.)

**Case test – Quyền 0003** (Lương đóng BH = 27,050,000):
| Khoản | Tỷ lệ | Số tiền |
|---|---|---|
| BHXH Cty | 17.5% | 4,733,750 |
| BHYT Cty | 3% | 811,500 |
| BHTN Cty | 1% | 270,500 |
| **BH Công ty (tổng) – ĐÚNG** | | **5,815,750** |
| Phí CĐ Cty | 2% | 541,000 *(để riêng)* |
| Hệ thống đang để (SAI) | | 6,356,750 *(= gộp cả CĐ)* |

---

## 🔴 L9 – Thu nhập chịu thuế trước giảm trừ trừ nhầm phí công đoàn NLĐ

**Đúng:** khi tính thu nhập chịu thuế **chỉ trừ 3 khoản bảo hiểm NLĐ** (BHXH 8% + BHYT 1.5% + BHTN 1% = 10.5%). **KHÔNG trừ phí công đoàn NLĐ (0.5%).**

**Hiện tại:** hệ thống trừ cả phí CĐ NLĐ → thu nhập chịu thuế thấp đi → **thu thiếu thuế**.

**Case test – Quyền 0003:**
| | Giá trị |
|---|---|
| Gross | 30,963,926 |
| Trừ BH NLĐ (3 khoản, 10.5%) | 2,840,250 |
| Phí CĐ NLĐ (0.5%) | 135,250 ← **KHÔNG được trừ** |
| **TN chịu thuế – ĐÚNG** | **28,123,676** |
| Hệ thống đang để (SAI) | 27,988,426 *(còn trừ CĐ)* |
| Thuế TNCN – đúng | **762,368** |
| Thuế TNCN – hệ thống (SAI) | 748,843 |

Lỗi này còn ảnh hưởng mọi nhân viên khác (VD Hạnh 0028: hệ thống 8,214,043 → đúng 8,257,693).

---

## ⚠️ L10 – Phương án OT không tính thuế (chờ khách chốt)

**Phát hiện khi test:** hệ thống **đã miễn toàn bộ tiền OT + toàn bộ lương đêm** khỏi thu nhập chịu thuế (đúng cơ chế "OT không tính thuế" – PA2). Kiểm trên Hạnh: `Gross − BH3 − toàn bộ OT − lương đêm` = đúng.

→ Phần "miễn toàn bộ OT" **không thiếu**. Sai lệch số của Hạnh **chỉ do L9** (trừ nhầm công đoàn).

**Việc cần làm:** hỏi khách để chốt – có cần thêm **công tắc chọn PA1** (OT có tính thuế – chỉ miễn phần OT trả thêm) hay chỉ dùng PA2 cố định? (Ví dụ 0144 trong tài liệu tính theo PA1.)

Tham chiếu 2 phương án:
- **PA1 (OT có thuế):** miễn = phần OT trả thêm `(Tổng OT − Lương giờ × giờ OT)` + toàn bộ lương đêm.
- **PA2 (OT không thuế):** miễn = **toàn bộ tiền OT** + toàn bộ lương đêm.
- Cả 2 đều trừ BH NLĐ, đều **KHÔNG** trừ công đoàn (theo L9).

---

## Checklist cho dev
- [ ] **L1**: đổi công thức trừ đi trễ = `Lương giờ/4 × INT((trễ+sớm)/15)`; cộng phút về sớm.
- [ ] **L8**: bỏ 2% CĐ khỏi "BH Công ty tổng"; "Tổng chi phí" = Gross + BH Cty + CĐ Cty.
- [ ] **L9**: khi tính thu nhập chịu thuế, bỏ phần trừ phí công đoàn NLĐ (chỉ trừ 3 khoản BH).
- [ ] **L10**: xác nhận với khách về công tắc PA1/PA2.
