# 01 — Problem Scan & Quick Problem Cards

> **Học viên:** Lê Văn Hoàng  ·  **MSV:** `2A202603357`  ·  **Branch:** `levanhoang`
> **Bối cảnh:** Tôi là AI Product Engineer tại **Vin Smart Future**. Bài toán lựa chọn để Deep-Dive: **Vinpearl — Tổng hợp & Phân tích review khách sạn**.
>
> ⚠️ Các con số thời gian/khối lượng trong tài liệu là **ước tính giả định** dùng cho mục đích scoping, cần xác thực lại với bộ phận vận hành Vinpearl trước khi triển khai.

---

# 🔍 Phase 1 — SCAN (4 Lenses)

| # | Subsidiary | Lens | Mô tả ngắn bài toán |
|---|------------|------|---------------------|
| 1 | **Vinpearl** | Pain từ người khác | Review tiêu cực của khách (*"phòng bẩn"*, *"nhân viên thái độ tệ"*) nằm rải rác trên Booking.com, Agoda, Google Maps, TripAdvisor; quản lý chỉ biết sau 1–2 ngày khi khách đã check-out. |
| 2 | **Vinpearl** | Tốn thời gian | Đọc email đặt phòng theo đoàn (Group Booking) phức tạp từ công ty lữ hành, kiểm tra quỹ phòng trống và soạn lệnh book thủ công. |
| 3 | **Vinhomes** | Lặp lại | Phân loại và chuyển phản ánh cư dân (mất nước, hỏng đèn, ồn ào) trên App Vinhomes Resident tới đúng ban quản lý tòa nhà. |
| 4 | **VinWonders** | AI có thể tốt hơn | Chatbot CSKH hỗ trợ đặt vé/combo vui chơi trả lời rập khuôn, không hiểu câu hỏi tự nhiên (*"đi 2 người lớn 1 bé 1m2 thì mua vé gì rẻ nhất?"*). |
| 5 | **VinFast** | AI có thể tốt hơn | Khách mô tả lỗi xe bằng tiếng Việt tự do (*"qua gờ giảm tốc kêu cụp cụp bánh trước"*), xưởng dịch vụ phải phân loại mã lỗi thủ công. |
| 6 | **Vinmec** | Tốn thời gian | Bác sĩ mất 20–30 phút/bệnh nhân để viết tóm tắt hồ sơ xuất viện. |

---

# 🃏 Phase 2 — QUICK-ASSESS: 3 Quick Problem Cards

Top 3 được chọn: **#1 (Vinpearl review)**, **#2 (Vinpearl Group Booking)**, **#3 (Vinhomes phản ánh cư dân)**.

## Card #1 — Vinpearl: Tổng hợp & Phân tích review khách sạn ✅ (được chọn)

```text
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #1                                       │
│                                                             │
│ Bài toán: Review khách sạn trên nhiều nền tảng không được   │
│ tổng hợp kịp thời, phàn nàn khẩn cấp bị phát hiện trễ.      │
│ Công ty thành viên: [x] Khác: Vinpearl                      │
│                                                             │
│ Ai đang đau? Nhân viên Guest Relations (GR) & Quản lý       │
│ khách sạn (Hotel Manager); gián tiếp: khách lưu trú.        │
│                                                             │
│ Workflow thủ công hiện tại:                                 │
│   1. GR đăng nhập từng nền tảng, đọc review mới             │
│   → 2. Copy vào Excel, tự gắn nhãn tích cực/tiêu cực + bộ   │
│        phận liên quan (Housekeeping, F&B, Lễ tân...)        │
│   → 3. Lọc phàn nàn khẩn cấp, gửi email/Zalo cho trưởng BP  │
│   → 4. Soạn phản hồi công khai cho review tiêu cực          │
│   → 5. Tổng hợp báo cáo tuần gửi General Manager            │
│                                                             │
│ Bước nào tốn nhất? Bước 2 & 4 (⏱ ~90 phút/ngày/khách sạn)   │
│ AI có thể nhảy vào: Bước 2 (phân loại), 3 (cảnh báo),       │
│ 4 (draft phản hồi), 5 (tóm tắt báo cáo)                     │
│                                                             │
│ Metric: Thời gian phát hiện phàn nàn khẩn cấp từ ~24h       │
│ ──> dưới 2h; thời gian xử lý review của GR từ ~2.5h/ngày    │
│ ──> dưới 30 phút/ngày.                                      │
│                                                             │
│ Quick Architecture: [ ] No AI [ ] Rule [x] LLM [ ] Agent    │
└─────────────────────────────────────────────────────────────┘
```

## Card #2 — Vinpearl: Xử lý email Group Booking

```text
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #2                                       │
│                                                             │
│ Bài toán: Email đặt phòng theo đoàn có cấu trúc lộn xộn     │
│ (nhiều loại phòng, ngày đến/đi lệch nhau, yêu cầu đặc biệt).│
│ Công ty thành viên: [x] Khác: Vinpearl                      │
│                                                             │
│ Ai đang đau? Nhân viên Reservation, đối tác lữ hành.        │
│                                                             │
│ Workflow thủ công hiện tại:                                 │
│   1. Đọc email ──> 2. Bóc tách số phòng/ngày/loại phòng     │
│   ──> 3. Kiểm tra quỹ phòng trên PMS ──> 4. Soạn báo giá    │
│                                                             │
│ Bước nào tốn nhất? Bước 2 (⏱ ~20 phút/email)                │
│ AI có thể nhảy vào: Bước 2 (trích xuất dữ liệu có cấu trúc) │
│                                                             │
│ Metric: Giảm thời gian bóc tách từ 20 phút ──> dưới 3 phút; │
│ độ chính xác trích xuất ≥ 95%.                              │
│                                                             │
│ Quick Architecture: [ ] No AI [ ] Rule [x] LLM [ ] Agent    │
└─────────────────────────────────────────────────────────────┘
```

## Card #3 — Vinhomes: Phân loại phản ánh cư dân

```text
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #3                                       │
│                                                             │
│ Bài toán: Phản ánh cư dân gửi qua App bị chuyển sai bộ phận,│
│ thời gian phản hồi kéo dài.                                 │
│ Công ty thành viên: [x] Vinhomes                            │
│                                                             │
│ Ai đang đau? Cư dân, nhân viên CSKH Ban quản lý tòa nhà.    │
│                                                             │
│ Workflow thủ công hiện tại:                                 │
│   1. Nhận phản ánh ──> 2. CSKH đọc & phân loại              │
│   ──> 3. Chuyển kỹ thuật/an ninh/vệ sinh ──> 4. Phản hồi    │
│                                                             │
│ Bước nào tốn nhất? Bước 2–3 (⏱ ~10 phút/phản ánh, chờ tới   │
│ vài giờ khi quá tải)                                        │
│ AI có thể nhảy vào: Bước 2 (phân loại & gợi ý bộ phận)      │
│                                                             │
│ Metric: 90% phản ánh được chuyển đúng bộ phận trong < 5 phút│
│                                                             │
│ Quick Architecture: [ ] No AI [x] Rule [x] LLM [ ] Agent    │
└─────────────────────────────────────────────────────────────┘
```

---

# 🗳️ Quyết định lựa chọn

Tôi chọn **Card #1 — Vinpearl Tổng hợp & Phân tích review khách sạn** để Deep-Dive vì:

* **Tác động trực tiếp tới doanh thu & thương hiệu:** điểm review ảnh hưởng tới thứ hạng hiển thị trên OTA (Booking, Agoda) và quyết định đặt phòng của khách mới.
* **Bài toán thuần ngôn ngữ:** review là văn bản tự do, đa ngôn ngữ (Việt, Anh, Hàn, Trung...) — rule-based keyword khó xử lý sắc thái, mỉa mai, đa ngữ → phù hợp LLM.
* **Rủi ro kiểm soát được:** AI chỉ đọc dữ liệu công khai, phân loại và soạn nháp; con người vẫn quyết định phản hồi công khai.

**Lý do loại các thẻ khác:**
* **Card #2 (Group Booking):** dính trực tiếp tới hệ thống PMS và giá — sai một con số gây thiệt hại tài chính; cần tích hợp PMS phức tạp hơn mức scope của lab.
* **Card #3 (Vinhomes):** phần lớn phản ánh có thể phân loại tốt bằng rule/form có sẵn danh mục; giá trị cộng thêm của LLM thấp hơn.

