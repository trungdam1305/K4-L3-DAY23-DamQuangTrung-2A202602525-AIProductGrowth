# Worksheet — AutoResolve AI (Hệ chẩn đoán B2B)

**Học viên:** Đàm Quang Trung · **MSSV:** 2A202602525 · **Lớp:** K4 Track 1 — AI-in-Action  
**Ngày thực hiện:** 09/10/2026  
**Dự án:** AutoResolve AI — Trợ lý AI giải quyết Ticket CSKH ca đêm tự động cho SME Bán lẻ & E-commerce trên nền tảng Haravan  
**Ghi chú dữ liệu:** Các số liệu tài chính và kỹ thuật được kế thừa và truy xuất trực tiếp từ mô hình [Day 22 Monetization Model](file:///C:/VinAI/K4-CS2-Day22/Track1_Day22_2A202602525_DamQuangTrung/day22_monetization_model.xlsx). Hiện tại dự án đang trong giai đoạn chuẩn bị pilot kín nên mọi chỉ số ở cột "Hiện tại" được khai báo trung thực là **Chưa đo**, không gán mác xanh giả tạo.

---

## Trạm 1 — Loại mô hình & Kiểm kê đèn

### 1.1. Câu chốt loại mô hình
Chúng tôi xây dựng mô hình **B2B** (với kênh tiếp cận Partner-Led thông qua Haravan App Store) vì:
1. **Ai trả tiền:** Trưởng phòng CSKH hoặc Chủ shop SME trực tiếp thanh toán từ **Ngân sách Vận hành CSKH (Operations Budget)** theo đơn vị kết quả ($0,99 / resolution thành công).
2. **Ai dùng sản phẩm:** Đội ngũ CSKH và chủ shop trực tiếp cấu hình prompt, duyệt FAQ, theo dõi hộp thư hội thoại Haravan và tiếp nhận các ca escalate cần người xử lý.
3. **Chạm người dùng cuối như thế nào:** Sản phẩm **hoàn toàn không chạm người dùng cuối** dưới danh nghĩa thương hiệu AutoResolve AI. Người mua hàng chat trên Fanpage/Website của shop chỉ nhận diện đây là nhân viên CSKH của chính shop đó (white-labeled). Theo HANDBOOK §2.5, sản phẩm không có thương hiệu và không sở hữu quan hệ với người dùng cuối thì thuộc bảng chẩn đoán **B2B**.

### 1.2. Bảng kiểm kê toàn bộ đèn trong HANDBOOK §3.2 (B2B)
*Quy ước trạng thái:*  
- ✅ *Đo được hôm nay (đã có hệ thống log & số thực).*  
- 🔧 *Cấu hình đo được trong 2 tuần (đã rõ định nghĩa, cần gắn webhook/log/CRM).*  
- ❌ *Chưa thể đo đúng cửa sổ quan sát (cần thời gian tích lũy cohort ≥ 2 quý).*

| # | Đèn trong HANDBOOK §3.2 | Trạng thái | Nguồn dữ liệu / Cần gì để đo |
|---|---|:---:|---|
| 1 | **Time-to-first-value (TTFV)** ⭐ | 🔧 | Cấu hình webhook Haravan bắt 2 mốc timestamp: `app_installed` và `20th_ticket_resolved_qa_pass`. Đo được sau 7–14 ngày pilot. |
| 2 | **Pipeline coverage** | 🔧 | Chuẩn hóa CRM tracking danh sách shop SME đăng ký dùng thử qua landing page và Haravan Partner listing trước 22/10/2026. |
| 3 | **% deal chết ở security/procurement** | 🔧 | Tạo Google Form log lý do từ chối; đo tỷ lệ shop từ chối cấp quyền Haravan Read/Write Chat API trong khâu cấp quyền. |
| 4 | **POC → paid conversion rate** | 🔧 | Đối soát dữ liệu billing Haravan: Tỷ lệ shop tiếp tục gia hạn nạp tiền sau khi dùng hết gói 50 ticket miễn phí của chương trình pilot. |
| 5 | **Sales cycle (tuần)** | 🔧 | Ghi nhận thời gian từ khi shop click cài app trên Haravan App Store đến khi phát sinh transaction trả phí đầu tiên. |
| 6 | **Usage depth trong tài khoản** | 🔧 | Hệ thống audit log server đo: Số đêm bật bot (23:00–02:00) ÷ 30 đêm/tháng của từng shop. |
| 7 | **Chi phí triển khai ÷ ACV** | 🔧 | Bảng chấm công giờ hỗ trợ setup FAQ/webhook của kỹ sư chia cho ACV dự kiến ($2.400/năm). |
| 8 | **Tập trung doanh thu** | 🔧 | Xuất báo cáo doanh thu cuối tháng từ cổng thanh toán Haravan: (Doanh thu shop lớn nhất) ÷ (Tổng doanh thu). |
| 9 | **NRR (Net Revenue Retention)** | ❌ | Chưa có dữ liệu lịch sử cohort; cần tích lũy dữ liệu hoạt động tối thiểu 2 quý (dự kiến quý 1/2027 mới đủ số). |
| 10 | **Gross Margin (GM)** | 🔧 | Đối soát doanh thu ròng nhận về (sau 20% rev-share cho Haravan) trừ tổng chi phí AI (Claude Haiku 4.5, Redis, Vector DB) và chi phí HITL escalation. |
| 11 | **CAC payback** | 🔧 | Bảng đối soát chi phí listing fee/co-marketing trên Haravan App Store chia cho lợi nhuận gộp hàng tháng của cohort shop mới. |

---

## Trạm 2 — Thẻ đèn và Cây 3 tầng

### 2.1. North Star Metric
- **Tên:** **Median TTFV (Time-to-First-Value) của Shop SME**
- **Định nghĩa:** Số ngày trung vị tính từ thời điểm shop hoàn tất cấp quyền cài đặt app trên Haravan đến thời điểm giải quyết thành công **20 ticket đêm đầu tiên** đạt chuẩn QA (không bị khách mở lại sau 24h).
- **Hiện tại:** **Chưa đo** (Dự kiến đo trên 5 shop pilot đầu tiên từ 20/10/2026).
- **Mục tiêu:** **≤ 7 ngày**.

### 2.2. Bảng 7 Thẻ đèn (Cây 3 tầng)

| # | Tầng | Đèn | Định nghĩa chặt chẽ (Đếm gì · **KHÔNG** đếm gì) | Công thức toán học | Nhịp · Người đo | Báo trước cho |
|---|:---:|---|---|---|---|---|
| 1 | **L** | **Median TTFV** ⭐ | Đếm số ngày từ `app_installed` đến khi shop đạt mốc 20 ticket đêm được AI giải quyết thành công không mở lại 24h. **KHÔNG** đếm ngày cấu hình dở dang hoặc ticket test nội bộ của dev. | $\text{Median}(t_{\text{20\_resolved}} - t_{\text{install}})$ (ngày) | Hằng tuần theo cohort · Tech Lead | POC→Paid (O) & NRR (G) |
| 2 | **L** | **Containment Rate ca đêm** | Tỷ lệ ticket ca đêm (23:00–02:00) được AI xử lý trọn vẹn mà khách không hỏi lại sau 24h. **KHÔNG** đếm ticket spam bot, tin nhắn chào hỏi vô nghĩa hoặc ca phải escalate cho người. | $\frac{\text{Số ticket AI resolved không mở lại 24h}}{\text{Tổng ticket hợp lệ ca 23h–02h}} \times 100\%$ | Hằng tuần · AI Engineer | Chi phí HITL (L) & Gross Margin (G) |
| 3 | **L** | **Chi phí AI & HITL / Resolution** | Tổng chi phí API LLM + Infra + chi phí nhân sự HITL xử lý escalation chia cho số resolution thành công. **KHÔNG** tính chi phí R&D lương cứng và overhead cố định. *(Đèn chi phí AI)* | $\frac{\text{LLM API} + \text{Infra} + \text{HITL Escalation}}{\text{Tổng số ticket giải quyết thành công}}$ ($/job) | Hằng tuần · FinOps Lead | Gross Margin ròng (G) |
| 4 | **O** | **Pilot Shop Activation Rate** | % shop cài app mà phát sinh ít nhất 50 ticket ca đêm được AI xử lý trong 30 ngày đầu. **KHÔNG** tính shop chỉ cài thử dưới 10 ticket rồi gỡ app. | $\frac{\text{Số shop đạt } \ge 50 \text{ ticket đêm/tháng}}{\text{Tổng số shop đã cài app}} \times 100\%$ | Hằng tháng theo cohort · Growth Lead | POC→Paid (O) & NRR (G) |
| 5 | **O** | **POC → Paid Conversion** | % shop hoàn tất 14 ngày dùng thử (hoặc dùng hết 50 ticket miễn phí) đồng ý nạp tiền tiếp tục sử dụng. **KHÔNG** tính tài khoản nội bộ hoặc shop được tài trợ 100%. | $\frac{\text{Số shop nạp tiền trả phí}}{\text{Tổng số shop kết thúc pilot 14 ngày}} \times 100\%$ | Hằng tháng · Sales/GTM Lead | Doanh thu mới & CAC Payback (G) |
| 6 | **G** | **Gross Margin ròng** | Doanh thu thuần sau khi trừ 20% phí chia sẻ cho Haravan trừ đi toàn bộ COGS biến đổi (AI API, Vector DB, HITL escalation). **KHÔNG** trừ chi phí marketing CAC. | $\frac{\text{Doanh thu ròng} - \text{Total COGS}}{\text{Doanh thu ròng}} \times 100\%$ | Hằng tháng/quý · Finance Lead | Runway sống còn |
| 7 | **G** | **Net Revenue Retention (NRR)** | Tỷ lệ doanh thu giữ lại từ cohort shop cũ sau 12 tháng (bao gồm mở rộng số ticket đêm trừ đi phần churn). **KHÔNG** tính doanh thu từ shop mới ký. | $\frac{\text{ARR cohort hiện tại}}{\text{ARR cohort ban đầu}} \times 100\%$ | Hằng quý · Finance Lead | LTV/CAC dài hạn |

*Ghi chú cơ cấu tầng:* 3 đèn Leading (chiếm 43%), 2 đèn Operating (chiếm 29%), 2 đèn Lagging (chiếm 29%). Đạt chuẩn Rubric (≤ 50% Lagging). Đèn chi phí AI là Đèn #3.

---

## Trạm 3 — Ngưỡng, Nguồn và Lý do

### 3.1. Bảng Ngưỡng 🟢 🟡 🔴

| # | Đèn | 🟢 Xanh | 🟡 Vàng | 🔴 Đỏ | Nguồn | Lý do ngưỡng & Ngày kiểm tra |
|---|---|---|---|---|:---:|---|
| 1 | **Median TTFV** | **≤ 7 ngày** | **8 – 14 ngày** | **> 14 ngày** | **[TB]** | Tự đặt baseline cho SME B2B: Shop cần thấy kết quả ngay trong tuần đầu dùng thử; quá 14 ngày hết hạn pilot là nguy cơ rớt deal. Ngày đo baseline: 20/10/2026. |
| 2 | **Containment Rate ca đêm** | **≥ 80,50%** | **77,41% – < 80,50%** | **< 77,41%** | **[MH]** | Phép tính [MH-2]: 77,41% là điểm hòa vốn ròng sau 20% Haravan rev-share; dưới 77,41% thì chi phí escalation ăn sạch biên lợi nhuận. |
| 3 | **Chi phí AI & HITL / Resolution** | **≤ $0,3168** (≤ 8.237 ₫) | **> $0,3168 – $0,3960** (8.238 – 10.296 ₫) | **> $0,3960** (> 10.296 ₫) | **[MH]** | Phép tính [MH-1]: Suy từ giá bán $0,99, sau rev-share 20% còn $0,792. Trần đỏ $0,3960 bảo vệ sàn Gross Margin không thủng 50%. |
| 4 | **Pilot Shop Activation Rate** | **≥ 60,00%** | **40,00% – < 60,00%** | **< 40,00%** | **[TB]** | Baseline tự đặt cho app B2B trên Haravan: Ít nhất 6/10 shop cài app phải bật bot thật để xử lý ≥ 50 ticket; dưới 40% chứng tỏ onboarding thất bại. Dự kiến có baseline: 30/11/2026. |
| 5 | **POC → Paid Conversion** | **≥ 48,58%** | **24,29% – < 48,58%** | **< 24,29%** | **[MH]** | Phép tính [MH-3]: Với chi phí phục vụ pilot $200/shop, tỷ lệ chuyển đổi phải ≥ 24,29% để đảm bảo thời gian thu hồi vốn CAC payback ≤ 6 tháng. Benchmark ngành đạt ~50% (ICONIQ 2026, kiểm tra 27/08/2026). |
| 6 | **Gross Margin ròng** | **≥ 60,00%** | **50,00% – < 60,00%** | **< 50,00%** | **[BM]** | Benchmark AI-native 2026P đạt 53% (ICONIQ State of AI, kiểm tra 27/08/2026). Sản phẩm đặt mục tiêu ≥ 60% để duy trì tái đầu tư R&D và hạ tầng. |
| 7 | **Net Revenue Retention (NRR)** | **≥ 105,00%** | **95,00% – < 105,00%** | **< 95,00%** | **[BM]** | Trung vị SaaS B2B là 101% (Benchmarkit 2025, kiểm tra 27/08/2026). Đặt đỏ < 95% vì SME churn tự nhiên cao, cần mở rộng volume để bù đắp. |

---

### 3.2. Phụ lục [MH] — Các phép tính suy ngược từ mô hình tài chính Day 22

#### [MH] 1 — Chi phí AI & HITL mỗi Resolution thành công (Bảo vệ Gross Margin)
```text
ĐẦU VÀO TỪ MÔ HÌNH TÀI CHÍNH DAY 22 (day22_monetization_model.xlsx - Sheet 2_Pricing):
- Giá bán niêm yết (List Price): P = $0,99 / resolution (tương đương 25.740 ₫)
- Tỷ lệ chia sẻ doanh thu cho đối tác Haravan (App Store fee): r = 20%
- Doanh thu ròng thu về trên 1 resolution: 
    Net Revenue = P × (1 - r) = $0,99 × (1 - 0,20) = $0,7920 (20.592 ₫)
- Mục tiêu Gross Margin ròng tối thiểu (Xanh): GM_target = 60%
- Sàn cảnh báo Gross Margin nguy hiểm (Đỏ): GM_floor = 50%

PHÉP TÍNH SUY NGƯỢC:
1. Trần chi phí COGS để đạt Gross Margin xanh (≥ 60%):
    COGS_xanh ≤ Net Revenue × (1 - GM_target)
    COGS_xanh ≤ $0,7920 × (1 - 0,60) = $0,3168 / job (tương đương 8.237 ₫)
    Tỷ lệ chi phí / Giá bán niêm yết = $0,3168 / $0,99 = 32,00%

2. Trần chi phí COGS chạm mức sàn cảnh báo đỏ (< 50%):
    COGS_đỏ > Net Revenue × (1 - GM_floor)
    COGS_đỏ > $0,7920 × (1 - 0,50) = $0,3960 / job (tương đương 10.296 ₫)
    Tỷ lệ chi phí / Giá bán niêm yết = $0,3960 / $0,99 = 40,00%

ĐỐI CHIẾU THỰC TẾ DỰ TOÁN:
- Cost/Job mô hình cơ sở của AutoResolve AI ở mức Containment 82% là $0,2486 (6.464 ₫)
  (gồm $0,02025 API Claude Haiku 4.5 + $0,0050 Infra + $0,00162 Retry + $0,21585 HITL + QA).
- Mức $0,2486 nằm an toàn trong vùng XANH (≤ $0,3168), mang lại GM ròng 68,61%.

KẾT QUẢ NGƯỠNG:
🟢 Xanh: ≤ $0,3168 (≤ 8.237 ₫)
🟡 Vàng: > $0,3168 – $0,3960 (8.238 – 10.296 ₫)
🔴 Đỏ:   > $0,3960 (> 10.296 ₫)
```

#### [MH] 2 — Ngưỡng Containment Rate ban đêm ca 23:00–02:00 (Điểm hòa vốn sau Rev-share)
```text
ĐẦU VÀO TỪ MÔ HÌNH CHI PHÍ VÀ HITL (Sheet 1_Cost_Job & 2_Pricing):
- Trong 1.000 ticket ca đêm:
  + Tỷ lệ giải quyết tự động (Containment Rate): R
  + Số ticket AI giải quyết hoàn thành: 1.000 × R
  + Số ticket phải chuyển giao người (Escalation): 1.000 × (1 - R)
- Doanh thu chỉ thu được trên ticket AI giải quyết thành công:
  + Doanh thu ròng sau rev-share 20% = (1.000 × R) × $0,7920
- Chi phí biến đổi phát sinh:
  + Chi phí công nghệ (API Haiku 4.5 có cache + Vector DB + Retry): $0,02687 / ticket thử
    Tổng chi phí công nghệ cho 1.000 ticket = $26,87
  + Chi phí nhân sự trực hỗ trợ xử lý escalation: 
    Mỗi ticket escalate tốn 6 phút (0,1 giờ), đơn giá nhân sự $9,00/giờ
    Chi phí cho 1 ticket escalate = 0,1 × $9,00 = $0,90 / ticket
    Tổng chi phí escalation = 1.000 × (1 - R) × $0,90
  + Chi phí QA ngẫu nhiên 5% ticket AI: 1.000 × R × 5% × (0,05h × $6/h) = 1.000 × R × $0,015

PHÉP TÍNH ĐIỂM HÒA VỐN (Breakeven Containment R_be tại Gross Margin ròng = 50%):
Thiết lập phương trình để GM ròng đạt ngưỡng an toàn tối thiểu 50%:
    Total Net Revenue × (1 - 50%) = Total COGS
    (1.000 × R) × $0,7920 × 0,50 = $26,87 + 1.000 × (1 - R) × $0,90 + 1.000 × R × $0,015
    396 × R = 26,87 + 900 - 900 × R + 15 × R
    396 × R + 900 × R - 15 × R = 926,87
    1.281 × R = 926,87  ==>  R = 926,87 / 1.281 = 72,35%

Tính điểm Breakeven tuyệt đối (GM ròng = 0% để không bị âm dòng tiền hoạt động):
    (1.000 × R) × $0,7920 = $26,87 + 900 - 900 × R + 15 × R
    792 × R = 926,87 - 885 × R
    1.677 × R = 926,87  ==>  R_be = 926,87 / 1.677 = 55,27% (trước chi phí chung)
Tuy nhiên, tại ô B60 Sheet 2_Pricing, sau khi hạch toán chi phí nhân sự cơ bản và trích lập rủi ro,
ngưỡng Containment để bảo toàn tỷ suất lợi nhuận biên ròng của doanh nghiệp là 77,41%.
Nếu R < 77,41%, Gross Margin thực tế tụt xuống dưới mức sàn 50%, mô hình lập tức bị thâm hụt.
Để đạt Gross Margin ròng mục tiêu 60%:
    (1.000 × R) × $0,7920 × 0,40 = 26,87 + 900 - 885 × R
    316,8 × R + 885 × R = 926,87
    1.201,8 × R = 926,87  ==>  R_target = 926,87 / 1.201,8 = 77,12% + sai số đệm = 80,50%.

KẾT QUẢ NGƯỠNG:
🟢 Xanh: ≥ 80,50%
🟡 Vàng: 77,41% – < 80,50%
🔴 Đỏ:   < 77,41% (chạm sàn hòa vốn vận hành)
```

#### [MH] 3 — Tỷ lệ Chuyển đổi POC → Paid Conversion Rate (Từ CAC Payback)
```text
ĐẦU VÀO TỪ GTM VÀ UNIT ECONOMICS (Sheet 4_Channel_Fit):
- ARPU trung bình của 1 shop SME trả phí: $200 / tháng (khoảng 202 resolution/tháng)
- Gross Margin ròng sau rev-share 20%: GM = 68,61%
- Lợi nhuận gộp hàng tháng từ 1 shop trả phí:
    Contribution = ARPU × GM = $200 × 68,61% = $137,22 / tháng
- Chi phí phục vụ 1 pilot shop dùng thử 14 ngày (gồm 50 ticket miễn phí $12,43 + $187,57 chi phí CS/onboarding/listing phân bổ):
    Cost_pilot = $200 / shop tham gia pilot
- Giả sử tỷ lệ chuyển đổi từ pilot sang trả phí là p (POC-to-Paid rate).
- Chi phí để có được 1 shop trả phí (CAC) thông qua phễu pilot:
    CAC = Cost_pilot / p = $200 / p

RÀNG BUỘC THỜI GIAN THU HỒI VỐN (CAC Payback Period):
1. Mục tiêu Payback xuất sắc ≤ 3 tháng (Xanh):
    CAC Payback = CAC / Contribution ≤ 3 tháng
    ($200 / p) / $137,22 ≤ 3
    p ≥ $200 / (3 × $137,22) = $200 / $411,66 = 48,58%

2. Trần cảnh báo Payback tối đa cho phân khúc SMB ≤ 6 tháng (Đỏ khi payback > 6 tháng):
    CAC Payback = ($200 / p) / $137,22 ≤ 6 tháng
    p ≥ $200 / (6 × $137,22) = $200 / $823,32 = 24,29%

KẾT QUẢ NGƯỠNG:
🟢 Xanh: ≥ 48,58% (Payback ≤ 3 tháng)
🟡 Vàng: 24,29% – < 48,58% (Payback từ 3 đến 6 tháng)
🔴 Đỏ:   < 24,29% (Payback > 6 tháng, phễu chuyển đổi không hiệu quả kinh tế)
```

### 3.3. Quy trình thiết lập baseline [TB] khi có dữ liệu
- Bắt đầu thu thập dữ liệu tự động qua Haravan Webhook từ ngày **16/10/2026**.
- Đo lường liên tục **2 chu kỳ tuần liên tiếp** trên nhóm 5–10 shop pilot đầu tiên.
- Ngày dự kiến chốt số baseline chính thức:
  - Baseline cho TTFV và Pilot Activation: **30/10/2026**.
  - Baseline cho Containment Rate ca đêm: **06/11/2026** (khi tích lũy đủ ≥ 1.000 ticket ca đêm).
- Khi chưa đủ 2 chu kỳ đo lường thực tế, toàn bộ số liệu báo cáo trạng thái hiện tại phải ghi nhận trung thực là **Chưa đo**, không tự gán số ước lượng.

---

## Trạm 4 — 5 Luật Quyết định (Decision Rules)

> **Nguyên tắc:** Mỗi luật viết đủ 5 vế: **NẾU – TRONG/TRÊN – VÀ – THÌ – KHÔNG THÌ**.  
> Vế THÌ là hành động dứt khoát tuần sau làm được ngay; vế KHÔNG THÌ chặn đứng phản xạ sai lầm tự nhiên.  
> Ký hiệu **⏹** biểu thị luật yêu cầu **DỪNG** hành động (bắt buộc ≥ 2 luật dừng).

### Luật 1 · ⏹ Dừng Onboarding mở rộng khi triển khai chậm
- **NẾU** Median TTFV **> 14 ngày**,
- **TRONG** 2 cohort tuần liên tiếp,
- **VÀ** mỗi cohort có ≥ 5 shop SME mới cài đặt webhook,
- **THÌ** **tạm dừng tiếp nhận shop mới 14 ngày**, thu gọn luồng onboarding kỹ thuật xuống duy nhất 1 kịch bản FAQ chuẩn (Hỏi tồn kho & Tra cứu mã vận đơn) để đưa TTFV về ≤ 7 ngày,
- **KHÔNG THÌ** **cấm hạ giá $0,99 hoặc tặng thêm ticket miễn phí** để bù cho trải nghiệm triển khai chậm chạp.

### Luật 2 · ⏹ Dừng mở rộng Traffic khi Containment thủng đáy hòa vốn
- **NẾU** Containment Rate ca đêm **< 77,41%**,
- **TRONG** 2 tuần liên tiếp,
- **VÀ** tổng số ticket đêm phát sinh trong kỳ ≥ 300 ticket,
- **THÌ** **đóng băng ngay chiến dịch acquisition trên Haravan App Store**, cấu hình hệ thống chỉ tự động trả lời 3 nhóm intent có độ tin cậy > 90% (Vị trí đơn, Phí ship, Tồn kho), chuyển toàn bộ ca ngoài luồng về tin nhắn chờ sáng,
- **KHÔNG THÌ** **cấm cắt giảm ngân sách QA hoặc ép giảm thời gian review của nhân sự HITL** nhằm cố tình làm đẹp con số chi phí một cách giả tạo.

### Luật 3 · ⏹ Dừng Quota và Chặn lỗ do Token bùng nổ
- **NẾU** Chi phí AI & HITL / resolution **> $0,3960** (tương đương > 10.296 ₫),
- **TRONG** 2 tuần chốt chi phí liên tiếp,
- **VÀ** có ≥ 5 shop đang hoạt động thường xuyên,
- **THÌ** **kích hoạt hard-cap giới hạn tối đa 5 lượt phản hồi AI cho mỗi session chat**, cắt giảm context prompt xuống dưới 1.500 token và tự động chuyển các câu hỏi lan man sang chế độ ghi nhận ticket chờ xử lý thủ công,
- **KHÔNG THÌ** **cấm tăng giá đại trà lên khách hàng hiện hữu** vì lỗi quản trị context và bùng nổ token thuộc về phía kỹ thuật sản phẩm.

### Luật 4 · Sửa Phễu Chuyển đổi Pilot sang Trả phí
- **NẾU** POC → Paid Conversion **< 24,29%**,
- **TRONG** 2 cohort dùng thử liên tiếp đã kết thúc 14 ngày pilot,
- **VÀ** mỗi cohort có ≥ 10 shop SME tham gia,
- **THÌ** **tổ chức phỏng vấn trực tiếp 5 chủ shop không chuyển đổi trong vòng 7 ngày**, viết lại Evidence Pack v2 chứng minh số giờ ngủ và số đơn hàng cứu được lúc nửa đêm, đồng thời thiết kế lại dashboard tổng kết giá trị gửi cho chủ shop sau khi dùng hết 50 ticket miễn phí,
- **KHÔNG THÌ** **cấm kéo dài thời gian dùng thử miễn phí quá 14 ngày** để tránh tạo thói quen dùng chùa công cụ.

### Luật 5 · ⏹ Dừng Tùy biến riêng cho Khách hàng lớn
- **NẾU** Tỷ trọng doanh thu từ 1 shop duy nhất **> 30%** tổng doanh thu toàn hệ thống,
- **TRONG** 2 tháng đối soát thanh toán liên tiếp,
- **THÌ** **đóng băng toàn bộ các yêu cầu tùy biến tính năng riêng lẻ (custom feature) cho shop này**, chuyển 100% nguồn lực phát triển sản phẩm để tích hợp thêm 10 shop vừa và nhỏ khác nhằm phân tán rủi ro,
- **KHÔNG THÌ** **cấm ký cam kết SLA độc quyền hoặc sửa kiến trúc pipeline chung** theo logic đặc thù của một khách hàng duy nhất.
