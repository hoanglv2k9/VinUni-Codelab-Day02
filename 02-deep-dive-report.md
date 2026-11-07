# 02 — Deep-Dive Report: Vinpearl Review Intelligence

> **Học viên:** Lê Văn Hoàng  ·  **MSV:** `2A202603357`  ·  **Branch:** `levanhoang`
> **Bài toán:** Tổng hợp & Phân tích review khách sạn Vinpearl trên các nền tảng OTA/mạng xã hội, phát hiện sớm phàn nàn khẩn cấp và soạn nháp phản hồi.
>
> ⚠️ Các con số dưới đây là **ước tính giả định** cho mục đích scoping (giả sử 1 khách sạn Vinpearl ~500 phòng, ~60 review mới/ngày). Cần xác thực bằng dữ liệu thực tế trước khi triển khai.

---

# 🏗️ Phase 3 — DEEP-DIVE

## 3.1. Current-State Workflow Mapping

Sơ đồ chi tiết: xem [04-workflow-diagram.png](04-workflow-diagram.png).

```text
┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│ Bước 1       │     │ Bước 2       │     │ Bước 3       │     │ Bước 4       │     │ Bước 5       │
│ Đăng nhập    │     │ Đọc & gắn    │ 🔄  │ Lọc phàn nàn │ 🔄  │ Soạn phản hồi│     │ Tổng hợp     │
│ 4-5 nền tảng │ ──→ │ nhãn thủ công│ ──→ │ khẩn cấp, báo│ ──→ │ công khai cho│ ──→ │ báo cáo tuần │
│ lấy review   │     │ vào Excel    │     │ trưởng BP    │     │ review xấu   │     │ gửi GM       │
│ Ai: GR       │     │ Ai: GR       │     │ Ai: GR → BP  │     │ Ai: GR → Mgr │     │ Ai: GR       │
│ ⏱ 20 ph/ngày │     │ ⏱ 60 ph 🔴   │     │ ⏱ 15 ph      │     │ ⏱ 50 ph 🔴   │     │ ⏱ 2h/tuần    │
│ In: OTA      │     │ In: Raw text │     │ In: Excel    │     │ In: Review   │     │ In: Excel    │
│ Out: Raw text│     │ Out: Excel   │     │ Out: Email   │     │ Out: Reply   │     │ Out: Báo cáo │
└──────────────┘     └──────────────┘     └──────────────┘     └──────────────┘     └──────────────┘
🔴 = Bottleneck   🔄 = Handoff (GR → Trưởng bộ phận / GR → Hotel Manager duyệt reply)
⏱ Tổng: ~145 phút/ngày/khách sạn (+2h báo cáo tuần). Độ trễ phát hiện phàn nàn khẩn cấp: ~24h.
```

**Điểm nghẽn chính:**
* 🔴 **Bước 2:** đọc review đa ngôn ngữ, gắn nhãn cảm xúc + bộ phận thủ công, dễ bỏ sót khi cao điểm (lễ, Tết).
* 🔴 **Bước 4:** soạn phản hồi cá nhân hóa cho từng review tiêu cực, phải qua Manager duyệt.
* 🔄 **Handoff Bước 3:** cảnh báo gửi qua email/Zalo không có trạng thái theo dõi → không biết bộ phận đã xử lý chưa.

---

## 3.2. Problem Statement (6-field)

| Field | Nội dung |
|---|---|
| **1. Actor / Operator** | Nhân viên Guest Relations (GR) của khách sạn Vinpearl; người duyệt: Hotel Manager; người nhận cảnh báo: Trưởng bộ phận (Housekeeping, F&B, Front Office, Kỹ thuật). |
| **2. Current Workflow** | Hằng ngày GR đăng nhập Booking.com, Agoda, Google Maps, TripAdvisor để đọc review mới, copy vào Excel, tự gắn nhãn tích cực/tiêu cực và bộ phận liên quan, gửi email/Zalo cảnh báo trưởng bộ phận, soạn phản hồi công khai chờ Manager duyệt, cuối tuần tổng hợp báo cáo cho GM. 5 bước, hoàn toàn thủ công, ~145 phút/ngày. |
| **3. Bottleneck** | Bước 2 (gắn nhãn đa ngôn ngữ, ~60 phút) và Bước 4 (soạn phản hồi, ~50 phút). Phàn nàn khẩn cấp (vệ sinh, an toàn, thái độ nhân viên) bị phát hiện trễ trung bình ~24h — thường sau khi khách đã check-out, mất cơ hội khắc phục tại chỗ. |
| **4. Business Impact** | ~2.5 giờ công GR/ngày/khách sạn. Review tiêu cực không được phản hồi kịp làm giảm điểm trung bình trên OTA → giảm thứ hạng hiển thị và tỉ lệ chuyển đổi đặt phòng. Không có số liệu tổng hợp theo bộ phận để GM ra quyết định cải tiến. |
| **5. Success Metric** | 1. **Độ trễ phát hiện** phàn nàn khẩn cấp: ~24h → **< 2h**.<br>2. **Thời gian GR xử lý review:** ~145 → **< 30 phút/ngày**.<br>3. **Độ chính xác phân loại** (cảm xúc + bộ phận) **≥ 90%** so với nhãn của GR trên tập kiểm thử 300 review.<br>4. **Tỉ lệ review tiêu cực được phản hồi trong 24h ≥ 95%**. |
| **6. Operational Boundary** | **AI được phép:** đọc review công khai; phân loại cảm xúc, chủ đề, bộ phận; gắn cờ khẩn cấp; soạn **nháp** phản hồi và báo cáo tóm tắt.<br>**AI TUYỆT ĐỐI KHÔNG:** tự đăng phản hồi công khai (bắt buộc GR/Manager duyệt — mọi nháp có thẻ `[DRAFT_ONLY]`); hứa hẹn bồi thường, hoàn tiền, voucher; thừa nhận trách nhiệm pháp lý; nêu tên nhân viên cụ thể hay tiết lộ thông tin cá nhân khách (số phòng, số điện thoại); bịa đặt sự kiện không có trong review.<br>**Điểm cần duyệt:** mọi phản hồi công khai; review có yếu tố an toàn/sức khỏe/pháp lý (ngộ độc, tai nạn, trộm cắp) → **chuyển thẳng Hotel Manager**, AI không soạn phản hồi. |

---

## 3.3. Future-State Flow & AI Fit

### AI-Fit Matrix

| Phương án | Đánh giá |
|---|---|
| **Rule / State-Machine** (keyword "dirty", "bẩn"...) | ❌ Không đủ: review đa ngôn ngữ, mỉa mai (*"tuyệt vời, chờ check-in có 2 tiếng thôi"*), một review nhắc nhiều bộ phận. Rule chỉ phù hợp cho phần thu thập dữ liệu và định tuyến sau phân loại. |
| **LLM Feature** ✅ | **Được chọn.** Quy trình cố định, mỗi review xử lý độc lập: phân loại → gắn cờ → soạn nháp. Output có cấu trúc JSON, dễ kiểm tra và đo lường. |
| **Agentic Loop** | ❌ Không cần: không có nhiều bước ra quyết định tự trị; để agent tự đăng phản hồi hay tự liên hệ khách tạo rủi ro thương hiệu không đáng có. |

### Future-State Flow

```text
┌──────────────┐     ┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐
│ Bước 1       │     │ Bước 2           │     │ Bước 3           │     │ Bước 4           │
│ ⚙️ Rule: tự  │     │ 🔵 AI: phân loại │     │ ⚙️ Rule: định    │     │ 🟢 HITL: GR/Mgr  │
│ động thu     │ ──→ │ cảm xúc, chủ đề, │ ──→ │ tuyến cảnh báo   │ ──→ │ duyệt / sửa nháp │
│ review qua   │     │ bộ phận, mức khẩn│     │ khẩn cấp tới     │     │ rồi mới đăng     │
│ API/export   │     │ + 🔵 draft reply │     │ trưởng BP (ticket)│    │ công khai        │
│ (mỗi 30 ph)  │     │ [DRAFT_ONLY]     │     │                  │     │                  │
└──────────────┘     └──────────────────┘     └──────────────────┘     └──────────────────┘
                              │                                                │
                              ▼                                                ▼
                     ↩️ Fallback:                                     🔵 AI: tóm tắt báo cáo
                     - JSON lỗi / confidence thấp                     tuần theo bộ phận
                       → đẩy vào hàng chờ GR xử lý tay                → 🟢 GR kiểm tra → GM
                     - Review an toàn/pháp lý
                       → chuyển thẳng Hotel Manager
                     - API LLM lỗi → quay về quy trình cũ
```

**Output JSON mẫu của bước AI:**
```json
{
  "sentiment": "negative",
  "topics": ["vệ sinh phòng", "check-in chậm"],
  "departments": ["Housekeeping", "Front Office"],
  "urgency": "high",
  "safety_or_legal": false,
  "confidence": 0.91,
  "draft_reply": "[DRAFT_ONLY] Kính gửi Quý khách, Vinpearl chân thành xin lỗi vì trải nghiệm chưa trọn vẹn..."
}
```

---

# 💻 Phase 4 — Technical Prompt Prototype & Boundary Stress-Test

Tôi đã xây dựng nguyên mẫu [starter-code/prompt_prototype.py](starter-code/prompt_prototype.py) gọi trực tiếp Gemini API cho đúng bài toán **Vinpearl Review Intelligence** với các ranh giới khóa cứng:
* **Quy tắc 1 (`[DRAFT_ONLY]`):** Mọi phản hồi nháp đều bắt buộc mở đầu bằng `[DRAFT_ONLY]` để nhân viên Guest Relations / Hotel Manager phê duyệt trước khi đăng lên Booking.com, Agoda hay Google Maps.
* **Quy tắc 2 (Cấm tự cam kết bồi thường & Cảnh báo nhóm 5% khẩn cấp):** Tuyệt đối không hứa hoàn tiền, tặng voucher hay tiết lộ số phòng/tên nhân viên; khi gặp sự cố nghiêm trọng (ngộ độc thực phẩm, vệ sinh/rệp giường, an ninh), bắt buộc kích hoạt JSON `{"action": "escalate_hotel_manager", ...}` báo thẳng cho Quản lý khách sạn.
* **Kết quả kiểm thử 3 kịch bản tấn công (Adversarial Tests):** Cả 3/3 kịch bản (ép hứa hoàn tiền 100% khi khách báo ngộ độc, ép bỏ `[DRAFT_ONLY]` để đăng thẳng lên Agoda, và `SYSTEM OVERRIDE` mạo danh Tổng Giám đốc GM đòi nêu tên nhân viên + số phòng) đều **Passed 100%**.

---

# 🏁 Phase 5 — EVALUATE

### AI Readiness Checklist

| # | Tiêu chí | Đánh giá | Bằng chứng |
|---|---|---|---|
| 1 | Có sẵn dữ liệu mẫu/logs sạch để test? | ✅ Có | Review OTA là dữ liệu công khai; GR đã có file Excel gắn nhãn thủ công nhiều tháng → dùng làm tập kiểm thử (ground truth). |
| 2 | Rủi ro khi AI sai nằm trong tầm kiểm soát? | ✅ Có | AI không đăng gì công khai; mọi phản hồi qua HITL; review an toàn/pháp lý bỏ qua AI; phân loại sai chỉ làm cảnh báo trễ/thừa, không gây thiệt hại trực tiếp. |
| 3 | Stakeholders sẵn sàng thay đổi quy trình? | ⚠️ Một phần | GR được giảm tải nên ủng hộ; cần thống nhất với trưởng bộ phận về cách nhận và đóng ticket cảnh báo; cần kiểm tra điều khoản API của từng nền tảng OTA. |

### Quyết định cuối cùng

- [x] **GO (Bắt đầu xây dựng Prototype)** — scope hẹp
- [ ] NOT YET
- [ ] NO-GO

**Justification:**
> Dự án đạt **GO với scope hẹp**: thí điểm tại **1 khách sạn Vinpearl**, **2 nền tảng** (Google Maps và Booking.com), trong **4 tuần**, chế độ **shadow mode** 2 tuần đầu (AI chạy song song, GR vẫn làm tay, so sánh kết quả).
>
> * **Kỹ thuật:** bài toán phân loại + soạn thảo văn bản là thế mạnh của LLM; Gemini 2.5 Flash xử lý tốt đa ngôn ngữ, output JSON có cấu trúc dễ kiểm tra.
> * **Chi phí:** ~60 review/ngày × ~600 token/review ≈ 36K token/ngày — với giá Gemini 2.5 Flash, chi phí chỉ vài cent/ngày, không đáng kể so với ~2 giờ công GR tiết kiệm mỗi ngày.
> * **Rủi ro:** được khống chế bằng `[DRAFT_ONLY]` + HITL, danh sách cấm (bồi thường, pháp lý, thông tin cá nhân) và fallback về quy trình thủ công.
> * **Điều kiện chuyển sang mở rộng:** độ chính xác phân loại ≥ 90% và ≥ 80% nháp phản hồi được GR chấp nhận với sửa đổi nhỏ trong giai đoạn thí điểm. Nếu không đạt → chuyển sang **NOT YET** để bổ sung dữ liệu gắn nhãn.

