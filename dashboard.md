# OPERATING DASHBOARD — AutoResolve AI

**Loại mô hình:** B2B (Partner-Led qua Haravan) · **Cập nhật:** 09/10/2026 · **Đàm Quang Trung – 2A202602525**  
**NORTH STAR:** **Median TTFV** (từ lúc cài app đến 20 ticket đêm đạt QA) — **Hiện tại: Chưa đo** · **Mục tiêu: ≤ 7 ngày** (khi cohort pilot ≥ 5 shop)

### Đèn báo sớm (Leading — nhìn hằng tuần)

| Đèn | Hiện | 🟢 / 🟡 / 🔴 — ngưỡng | Nguồn | Báo trước cho |
|---|:---:|---|:---:|---|
| **Median TTFV** ⭐ | Chưa đo | 🟢 ≤ 7 ngày · 🟡 8–14 ngày · 🔴 > 14 ngày | [TB] baseline | POC→Paid (O) & NRR (G) |
| **Containment Rate ca đêm** | Chưa đo | 🟢 ≥ 80,50% · 🟡 77,41%–<80,50% · 🔴 < 77,41% | [MH] hòa vốn | Chi phí HITL (L) & GM (G) |
| **Chi phí AI & HITL / Resolution** | Chưa đo | 🟢 ≤ $0,3168 · 🟡 >$0,3168–$0,3960 · 🔴 > $0,3960 | [MH] GM 60/50% | Gross Margin ròng (G) |

### Đèn vận hành (Operating — nhìn hằng tuần/tháng)

| Đèn | Hiện | 🟢 / 🟡 / 🔴 — ngưỡng | Nguồn | Báo trước cho |
|---|:---:|---|:---:|---|
| **Pilot Shop Activation Rate** | Chưa đo | 🟢 ≥ 60,00% · 🟡 40,00%–<60,00% · 🔴 < 40,00% | [TB] baseline | POC→Paid (O) & NRR (G) |
| **POC → Paid Conversion** | Chưa đo | 🟢 ≥ 48,58% · 🟡 24,29%–<48,58% · 🔴 < 24,29% | [MH] payback 3/6m | New ARR & Payback (G) |

### Đèn kết quả (Lagging — nhìn hằng tháng/quý)

| Đèn | Hiện | 🟢 / 🟡 / 🔴 — ngưỡng | Nguồn | Tác động |
|---|:---:|---|:---:|---|
| **Gross Margin ròng** (sau 20% rev-share) | Chưa đo | 🟢 ≥ 60,00% · 🟡 50,00%–<60,00% · 🔴 < 50,00% | [BM] ICONIQ 2026 | Runway sống còn |
| **Net Revenue Retention (NRR)** | Chưa đo | 🟢 ≥ 105,00% · 🟡 95,00%–<105,00% · 🔴 < 95,00% | [BM] Benchmarkit 2025 | Tăng trưởng dài hạn |

### 5 luật quyết định (⏹ = luật dừng)

1. **⏹ [DỪNG ONBOARDING] NẾU** Median TTFV > 14 ngày **TRONG** 2 cohort tuần liên tiếp **VÀ** mỗi cohort ≥ 5 shop mới **THÌ** dừng nhận shop mới 14 ngày, tinh gọn onboarding còn 1 kịch bản FAQ chuẩn; **KHÔNG THÌ** cấm giảm giá $0,99 hoặc tặng ticket để bù cài đặt chậm.
2. **⏹ [DỪNG ADS/SCALE] NẾU** Containment ca đêm < 77,41% **TRONG** 2 tuần liên tiếp **VÀ** volume ≥ 300 ticket **THÌ** đóng băng chiến dịch trên Haravan App Store, giới hạn bot ở 3 intent chuẩn (>90% conf), chuyển ca khó về tin nhắn chờ; **KHÔNG THÌ** cấm cắt giảm QA/HITL để ép giảm chi phí.
3. **⏹ [DỪNG QUOTA KHÁCH] NẾU** Chi phí AI & HITL / job > $0,3960 (> 10.296 ₫) **TRONG** 2 tuần liên tiếp **VÀ** ≥ 5 shop hoạt động **THÌ** bật hard-cap tối đa 5 turn/session, cắt context < 1.500 token, chuyển câu hỏi phức tạp sang ticket chờ; **KHÔNG THÌ** cấm tăng giá đại trà lên khách hiện hữu.
4. **NẾU** POC → Paid < 24,29% **TRONG** 2 cohort kết thúc 14 ngày pilot **VÀ** mỗi cohort ≥ 10 shop **THÌ** phỏng vấn trực tiếp 5 chủ shop rời bỏ, hoàn thiện Evidence Pack v2 và làm lại bản tổng kết giá trị sau pilot; **KHÔNG THÌ** cấm kéo dài thời gian dùng thử miễn phí quá 14 ngày.
5. **⏹ [DỪNG TÙY BIẾN RIÊNG] NẾU** 1 shop chiếm > 30% tổng doanh thu **TRONG** 2 tháng đối soát liên tiếp **THÌ** đóng băng mọi yêu cầu custom feature từ shop này, dồn 100% sprint tích hợp thêm 10 shop vừa và nhỏ; **KHÔNG THÌ** cấm ký cam kết SLA độc quyền hoặc sửa core pipeline theo ý một khách.

### Cổng gác 90 ngày (Mốc ngày gốc: 09/10/2026)

| Mốc ngày | Đúng một metric | Ngưỡng qua cổng | Bằng chứng vật lý | Quyết định nếu trượt |
|---|---|---|---|---|
| **30 · 08/11/2026** | Tỷ lệ ticket ca đêm ghi nhận đủ webhook log & phân loại đúng intent | ≥ 95,00% ticket ca đêm có log chuẩn xác trên ≥ 5 shop pilot | Bảng audit log webhook, query SQL, biên bản nghiệm thu kỹ thuật | **FIX** luồng tích hợp webhook trong 14 ngày; chưa mở rộng listing |
| **60 · 08/12/2026** | Median TTFV trên cohort pilot | ≤ 7 ngày trên cohort ≥ 10 shop SME | Dashboard cohort Haravan, đối soát timestamp `20th_resolved` | **PIVOT** sang dạng kịch bản rule-based + AI hybrid nếu FIX 1 lần thất bại |
| **90 · 07/01/2027** | Containment Rate ca đêm & POC→Paid | Containment ≥ 77,41% VÀ POC→Paid ≥ 24,29% (trên ≥ 30 shop) | Báo cáo thanh toán Haravan Billing, đối soát hóa đơn đối tác | **KILL** nếu < 77,41% sau 1 vòng FIX; nếu đạt 77–80%: PIVOT gói giá |

**KILL CRITERIA:** Đến ngày **07/01/2027**, nếu Containment Rate ca đêm vẫn **< 77,41%** trên **≥ 500 ticket** hoặc POC → Paid Conversion **< 24,29%** trên **≥ 30 shop pilot** sau khi đã FIX một lần ở ngày 60, **dừng triển khai dự án AutoResolve AI trên Haravan**.  
**CHƯA ĐO ĐƯỢC:** Hiện chưa có log production thật, chưa có hóa đơn chia sẻ doanh thu từ Haravan và bảng lương HITL thực tế. Cần kết nối Haravan Webhook (hoàn thành trước 16/10/2026), baseline TTFV dự kiến chốt ngày 30/10/2026, baseline Containment dự kiến chốt ngày 06/11/2026. Mọi ô trạng thái để **Chưa đo**, tuân thủ nguyên tắc trung thực.  
**Đối chiếu phép tính [MH], định nghĩa và lý do ngưỡng:** Xem chi tiết trong `worksheet.md` và Trang 2 của `dashboard.pdf`.
