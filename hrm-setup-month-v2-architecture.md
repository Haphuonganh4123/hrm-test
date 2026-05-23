# HRM Setup Tháng v2 — Architecture & Flow

> Tài liệu kiến trúc cho UI Setup Tháng chấm công v2 (2026-05-23).
> Áp dụng cho PR nextx-hrm#3 + nextx-fe-tenant#370.

---

## 1. Bối cảnh — Tại sao phải làm v2

### UI v1 — form thủ công cơ bản

UI cũ của Setup Tháng (`/workspace/hr/attendance` tab Setup):

| Vấn đề v1 | Tác động |
|---|---|
| Mỗi ngày CN/T7/lễ phải bấm date picker + nút "Thêm" | Tháng 31 ngày = bấm 16-18 lần |
| Chỉ có chip list "2026-05-04, 2026-05-11..." | HR không nhìn ra trực quan đâu là weekend/lễ |
| Không có quick-pick "Tick tất cả CN" | Phải nhập tay |
| Không có template lễ VN | HR tự nhớ 30/4, 1/5, 2/9, Tết âm — dễ sót |
| Mỗi tháng setup lại từ đầu | 5-10 phút/tháng/workspace |
| Lock một chiều (không có Unlock) | Lỡ tay khóa → kẹt, phải gọi dev |
| Không có audit | Không biết ai khóa/sửa khi nào |
| Không có Year Template | Setup lặp lại 12 lần/năm |

### So sánh competitor

| HRM | Approach hay nhất |
|---|---|
| **MISA AMIS** | Calendar grid visual + auto-detect lễ VN theo Nghị định |
| **Tanca** | Wizard 3 bước |
| **Base HRM** | Drag-to-select nhiều ngày |
| **BambooHR** | Tự generate cả năm |
| **Deel/Rippling** | Holiday library theo country |
| **Bravo HR** | Paste từ clipboard |

### Mục tiêu v2

> *"HR không phải nhập gì cả nếu hệ thống đã đoán đúng. HR chỉ confirm hoặc override."*

3 layer:
- **Year Template** — config 1 lần/năm
- **Month Setup** — auto-generate từ template, HR review
- **Daily Exception** — click/drag thay đổi từng ngày

---

## 2. Kiến trúc tổng thể

```
┌─────────────────────────────────────────────────────────────┐
│                     Year Template 2026                       │
│   defaultWeekOffDows: [0]    (Chủ nhật)                     │
│   defaultHolidays: [11 ngày lễ VN]                          │
│   standardWorkdayType: TYPE1                                │
└──────────────────────────────┬──────────────────────────────┘
                               │ auto-generate
                ┌──────────────┼──────────────┐
                ▼              ▼              ▼
        Setup T1/2026   Setup T2/2026   Setup T3/2026 ...
        DRAFT → OPEN     LOCKED          OPEN
        ↑                ↓
        Smart banner     Audit log
        "Auto-fill?"     "Locked by HR
                          at 02/03 14:30"

         ┌─────────────────────────────────────┐
         │      HrmHolidayLibrary (system)     │
         │  VN 2026: 11 lễ | VN 2027: 6 lễ     │
         │  (seed sẵn từ Nghị định CP)         │
         └─────────────────────────────────────┘
```

---

## 3. Component Mới

### 3.1 Calendar Grid (UI nổi bật nhất)

**Component**: `HrmMonthCalendarGrid.tsx`

**Tương tác**:
- **Click 1 ô** → toggle Làm việc ↔ Nghỉ tuần
- **Drag từ ô A → ô B** → bulk toggle weeklyOff cho range
- **Shift+Drag** → toggle Lễ thay vì weeklyOff
- **Hover ô lễ** → tooltip "Quốc khánh 02/09 — auto từ VN Holiday Library"

**Layout**: 7×n grid, tuần bắt đầu T2 (chuẩn VN).

**Color-coded**:
- ⬜ Bình thường (làm việc)
- 🟥 Nghỉ tuần (rose tint)
- 🟨 Ngày lễ (amber tint, có label tên lễ)

### 3.2 Summary Panel (realtime)

**Component**: `HrmMonthSummaryPanel.tsx` — sticky right 2/5 column.

**Hiển thị realtime khi user tick/untick**:
- Status badge (Draft/Open/Locked/Closed)
- Ngày làm việc (highlight emerald, lớn)
- Tổng ngày tháng / Nghỉ tuần / Ngày lễ
- Giờ chuẩn = workdays × 8

### 3.3 Quick Actions

**Component**: `HrmQuickActions.tsx`

4 nút:
- **Tick CN** — chọn tất cả CN trong tháng
- **Tick T7+CN** — chọn tất cả T7+CN
- **Tự động lễ VN** — auto-fill từ HrmHolidayLibrary
- **Bỏ chọn tất cả** — clear

### 3.4 Copy From Month Dialog

**Component**: `HrmCopyFromMonthDialog.tsx`

- Modal chọn tháng nguồn (default = tháng trước)
- Hệ thống shift ngày giữ day-of-month, clamp nếu tháng đích ngắn hơn
- Toast confirm

### 3.5 Audit History Dialog

**Component**: `HrmMonthAuditDialog.tsx`

- Hiển thị danh sách hành động (Created/Updated/Opened/Locked/Unlocked/Copied/Generated)
- Mỗi entry: timestamp, action badge, actor user, note
- Lazy load (chỉ fetch khi mở dialog)

### 3.6 Year Template Page

**Page**: `/workspace/hr/attendance/year-template`
**Component**: `HrmYearTemplatePage.tsx`

- Chọn DOW nghỉ tuần mặc định (T2-CN button toggle)
- Import lễ VN từ HrmHolidayLibrary 1 click
- Chọn StandardWorkdayType (TYPE1 auto / TYPE2 manual)
- Save → áp dụng cho mọi tháng generate sau này

### 3.7 Smart Suggestion Banner

Hiển thị ở đầu MonthSetupTab khi:
- Workspace có Year Template
- Tháng hiện tại chưa có setup

```
✨ Đã có Year Template 2026. Auto-fill cấu hình tháng từ template?
                                                  [Áp dụng template]
```

1-click apply → tự tạo setup tháng từ template.

---

## 4. Backend — 5 entities + state machine

### 4.1 `AttendanceMonthSetup` (mở rộng)

Thêm field `Status` enum:
```csharp
public enum MonthSetupStatus
{
    Draft,    // Hệ thống auto-tạo, chưa HR review
    Open,     // HR đã mở, chấm công bình thường
    Locked,   // HR khóa, có thể Unlock
    Closed,   // Chốt lương xong, hard-lock
}
```

Methods mới: `Open()`, `Reopen()`, `Close()`. Giữ `Lock()` cũ (backward compat).

### 4.2 `AttendanceMonthSetupAudit`

Append-only audit log. Action enum:
```csharp
public enum MonthSetupAuditAction
{
    Created, Updated, Opened, Locked, Unlocked, Closed, CopiedFrom, GeneratedFromTemplate
}
```

### 4.3 `HrmYearTemplate`

```csharp
public class HrmYearTemplate
{
    Year, DefaultWeekOffDows: List<int>, DefaultHolidays: List<DateOnly>,
    StandardWorkdayType, ManualStandardWorkdays
}
```

Unique (WorkspaceId, Year).

### 4.4 `HrmHolidayLibrary`

System-wide reference data (KHÔNG có WorkspaceId filter):
```csharp
public class HrmHolidayLibrary
{
    CountryCode (VN/...), Year, Date, Name, IsOfficial
}
```

Seed sẵn: VN 2026 + 2027 (Tết, 30/4, 1/5, 2/9...).

### 4.5 5 Commands + 3 Queries mới

| Command/Query | Mô tả |
|---|---|
| `CopyMonthFromCommand` | Copy setup từ tháng nguồn, shift giữ day-of-month |
| `UnlockMonthCommand` | Mở lại tháng đã khóa (audit + reason) |
| `BulkUpdateDaysCommand` | PATCH weeklyOff+holidays từ drag-select |
| `GenerateFromTemplateCommand` | Expand Year Template thành Month Setup cụ thể |
| `SetYearTemplateCommand` | Upsert Year Template |
| `GetMonthAuditQuery` | Lịch sử thay đổi tháng |
| `GetYearTemplateQuery` | Lấy Year Template hiện có |
| `GetVnHolidaysQuery` | Lấy lễ VN từ library theo year |

---

## 5. Database Schema (3 tables mới + 1 column)

```
attendance_month_setups
  └── + status (varchar(20), default 'Draft')
       Backfill: is_locked=true → 'Locked', else → 'Open'

attendance_month_setup_audits
├── id, workspace_id, year, month
├── action (string), actor_user_id, note
├── created_at, updated_at
└── INDEX (workspace_id, year, month, created_at DESC)

hrm_year_templates
├── id, workspace_id, year
├── default_week_off_dows (integer[])
├── default_holidays (date[])
├── standard_workday_type (string), manual_standard_workdays
└── UNIQUE (workspace_id, year)

hrm_holiday_library
├── id, country_code (2 chars), year, date
├── name, is_official
└── UNIQUE (country_code, year, date)
   INDEX (country_code, year)

Seed VN holidays:
  2026: 1/1, Tết 5 ngày 16-20/2, 26/4 (Giỗ Tổ), 30/4, 1/5, 2/9, 3/9 — 11 lễ
  2027: 1/1, Tết 3 ngày, 30/4, 1/5, 2/9 — 6 lễ
```

Migration: `20260523031820_AddMonthSetupV2Tables`

---

## 6. State Machine — Vòng đời 1 tháng setup

```
                           ┌──────────────┐
                           │   (Initial)  │
                           └──────┬───────┘
                  Create / Generate│
                  (POST setup)     │
                  ┌────────────────┴──────────────┐
                  ▼                                ▼
            ┌─────────┐  Open()              ┌────────┐
            │  Draft  ├─────────────────────▶│  Open  │
            └─────────┘                      └────┬───┘
                                                  │
                                            Lock()│
                                                  ▼
                                            ┌────────┐
                                            │ Locked │
                                            └────┬───┘
                                                 │
                              Reopen() ──────────┤─── Close()
                                                 │      (payroll done)
                                                 ▼
                                            ┌────────┐
                                            │ Closed │ ◄── hard-lock vĩnh viễn
                                            └────────┘
```

**Quy tắc**:
- Draft → Open: HR mở tháng sau khi review
- Open → Locked: HR khóa khi chốt công xong
- Locked → Open: Reopen (cần audit + reason)
- Locked → Closed: sau khi payroll lock
- Closed: KHÔNG mở lại được nữa

---

## 7. Flow tích hợp

### Flow 1: Auto-generate tháng từ Year Template
```
HR setup Year Template 2026 (DOW=[0] CN, holidays=11 VN)
   │
   ▼
HR mở tháng 6/2026 (chưa có setup)
   │
   ▼  Banner: "Đã có Year Template 2026. Auto-fill?"
   │
   ▼  HR click "Áp dụng template"
   │
[BE] GenerateFromTemplateCommandHandler
   ① Load Year Template 2026
   ② Expand DOW=[0] → tất cả CN trong tháng 6 (4 ngày)
   ③ Lọc holidays về tháng 6 (rỗng, không có lễ tháng 6)
   ④ Create AttendanceMonthSetup với weeklyOff=4 CN, holidays=[]
   ⑤ Status=Draft, ghi audit GeneratedFromTemplate
   │
   ▼  HR review calendar, có thể tick thêm/bỏ ngày bất kỳ
   ▼  HR click "Lưu setup"
   │
[BE] SetupMonthCommandHandler update bình thường
```

### Flow 2: Copy from Previous Month
```
Tháng 7/2026 chưa setup. HR click "Sao chép tháng"
   │
   ▼  Modal: chọn tháng nguồn = 6/2026
   │
[BE] CopyMonthFromCommandHandler
   ① Load AttendanceMonthSetup 6/2026 (weeklyOff=4 ngày, holidays=[])
   ② Shift dates: giữ day-of-month, clamp nếu tháng đích ngắn hơn
       Vd: 2026-06-07 → 2026-07-07 (cùng day-of-month=7)
   ③ Create setup 7/2026 với data đã shift
   ④ Ghi audit CopiedFrom: "From 2026/6"
   │
   ▼  HR review calendar grid mới, có thể adjust
```

### Flow 3: Lock + Unlock cycle
```
End of month — HR đối soát chấm công xong
   │
   ▼  Click "Khoá tháng"
   │
[BE] LockMonthCommand → setup.Lock() → Status=Locked, IsLocked=true
   ▼  Audit: Locked
   │
HR phát hiện cần sửa 1 ngày (vd: nhân viên báo nhầm ngày nghỉ)
   │
   ▼  Click "Mở lại tháng" → AlertDialog → nhập reason
   │
[BE] UnlockMonthCommandHandler
   ① Validate Status != Closed (nếu Closed → 409 Conflict)
   ② setup.Reopen() → Status=Open, IsLocked=false
   ③ Audit: Unlocked với reason
   │
   ▼  HR sửa, lock lại
   ▼  Sau khi payroll xong → Close() (hard-lock)
```

---

## 8. Backward Compatibility

| Tình huống | Behavior |
|---|---|
| Tháng cũ chưa có Status column | Migration backfill `IsLocked=true → 'Locked'`, else `'Open'` |
| Workspace chưa setup Year Template | UI không hiện Smart Banner, HR tạo setup tay như cũ |
| Workspace chưa có HrmHolidayLibrary | "Auto-fill lễ VN" trả mảng rỗng — không crash |
| GET /setup/{y}/{m} tháng chưa setup | 404 NotFound (giữ behavior cũ) |
| API cũ `POST /setup` + `POST /lock` | Hoạt động bình thường, sau lock Status thành `Locked` |

→ **Zero-breaking change.** Tháng cũ vẫn hiển thị + thao tác như cũ.

---

## 9. Endpoints Reference

| Method | Path | Mô tả |
|---|---|---|
| GET | `/api/v1/hrm/attendance/setup/{year}/{month}` | Xem cấu hình tháng (đã có) |
| POST | `/api/v1/hrm/attendance/setup/{year}/{month}` | Lưu cấu hình (đã có) |
| POST | `/api/v1/hrm/attendance/setup/{year}/{month}/lock` | Khóa tháng (đã có) |
| **POST** | **`/api/v1/hrm/attendance/setup/{y}/{m}/unlock`** | **Mở lại tháng (mới)** |
| **POST** | **`/api/v1/hrm/attendance/setup/{y}/{m}/copy-from/{srcY}/{srcM}`** | **Sao chép tháng (mới)** |
| **PATCH** | **`/api/v1/hrm/attendance/setup/{y}/{m}/days`** | **Bulk update days (drag-select)** |
| **POST** | **`/api/v1/hrm/attendance/setup/{y}/{m}/from-template`** | **Generate từ Year Template (mới)** |
| **GET** | **`/api/v1/hrm/attendance/setup/{y}/{m}/audit`** | **Lịch sử thay đổi (mới)** |
| **GET** | **`/api/v1/hrm/year-template/{year}`** | **Lấy Year Template (mới)** |
| **PUT** | **`/api/v1/hrm/year-template/{year}`** | **Upsert Year Template (mới)** |
| **GET** | **`/api/v1/hrm/holidays/{countryCode}/{year}`** | **Lấy lễ VN từ library (mới)** |

---

## 10. UI Layout

```
┌─────────────────────────────────────────────────────────────────┐
│  [Tháng 5 ▼]  [2026 ▼]   🟢 Open       [Sao chép] [Lịch sử]   │
├─────────────────────────────────────────────────────────────────┤
│  ✨ Đã có Year Template 2026. Auto-fill?     [Áp dụng template]│
├──────────────────────────────────────┬──────────────────────────┤
│  Calendar Grid (3/5 cols)             │ Summary Panel (2/5)     │
│                                       │  ┌────────────────────┐ │
│  T2 T3 T4 T5 T6 T7 CN                 │  │  Tổng kết tháng   │ │
│              1  2 ●3 ●4               │  │  🟢 Open           │ │
│   5  6  7  8  9●10●11                 │  │                    │ │
│  12 13 14 15 16●17●18                 │  │  Ngày làm việc: 22 │ │
│  19 20 21 22 23●24●25                 │  │  Tổng tháng:    31 │ │
│  26 27 28 29 ★30★1*                   │  │  Nghỉ tuần:     8  │ │
│                                       │  │  Ngày lễ:       1  │ │
│  Quick: [Tick CN] [T7+CN] [Auto lễ VN]│  │                    │ │
│         [Bỏ chọn]                     │  │  ⏰ Giờ chuẩn:176h │ │
│  Hint: Click/kéo để chọn nhiều ngày   │  └────────────────────┘ │
│        Shift+click = ngày lễ          │                          │
│                                       │                          │
│  Loại ngày công: [TYPE1 Auto] [22 ngày]│                         │
│                                       │                          │
│  [Lưu setup] [Khoá tháng]             │                          │
└──────────────────────────────────────┴──────────────────────────┘
```

---

## 11. Out of Scope (Phase tiếp theo)

- **Cron auto-tạo Draft** ngày 25 tháng trước
- **Department-level overrides** (CN này CN nghỉ ở CN A, làm ở CN B)
- **Lunar new year auto-calculation** (hiện hardcode trong seed)
- **Mobile-responsive calendar grid** (P1 desktop-first)
- **Bulk lock nhiều tháng** cùng lúc
- **Export setup tháng** ra Excel
- **Recurring monthly setup** (vd: mọi tháng tự khóa ngày 5 tháng sau)

---

*HRM Setup Tháng v2 Architecture v1.0 · 2026-05-23 · NextX HRM*
