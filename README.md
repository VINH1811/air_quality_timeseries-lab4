# 🌏 Dự Án: "Bắt Mạch" Nhịp Thở Đô Thị - Dự Báo Bụi Mịn PM2.5 Tại Bắc Kinh (SARIMA)

![Project Banner](https://img.shields.io/badge/Project-Time_Series_Forecasting-blue?style=for-the-badge&logo=python)
![Status](https://img.shields.io/badge/Status-Completed-success?style=for-the-badge)
![Model](https://img.shields.io/badge/Model-SARIMA-orange?style=for-the-badge)

> **"Không chỉ dự báo con số, chúng tôi dự báo nhịp điệu của thành phố."**

---
## 👥 Thông tin Nhóm
- **Nhóm:** WL
- **Thành viên:** - [Nguyễn Văn Vinh]
  - [Đỗ Văn Vinh]
  - [Lại Thành Đoạn]
  - [Bạch Ngọc Lương]
## 📖 1. Giới thiệu & Đặt vấn đề
Trong bối cảnh đô thị hóa nhanh chóng, ô nhiễm không khí—đặc biệt là **bụi mịn PM2.5**—đã trở thành mối đe dọa thầm lặng nhưng nghiêm trọng. 

Chúng ta thường quen với các dự báo thời tiết chung chung như "Ngày mai trời nắng". Tuy nhiên, với chất lượng không khí, biết mức trung bình của ngày mai là **chưa đủ**. Nồng độ bụi có thể ở mức an toàn vào buổi trưa nhưng tăng vọt lên mức nguy hại vào giờ tan tầm hoặc đêm khuya do hiện tượng nghịch nhiệt.Nếu chỉ nhìn vào số liệu hiện tại hoặc áp dụng ngưỡng cảnh báo tĩnh, nhà quản lý sẽ bỏ lỡ các đợt ô nhiễm tăng vọt theo giờ.

**Mục tiêu dự án:**
Dự án này tập trung giải quyết bài toán dự báo ngắn hạn (short-term forecasting) nồng độ PM2.5 theo từng giờ. Thay vì sử dụng mô hình ARIMA cơ bản (vốn hạn chế trong việc bắt các quy luật lặp lại), chúng tôi triển khai mô hình **SARIMA (Seasonal ARIMA)** để mô hình hóa tính chu kỳ (mùa vụ) 24 giờ của ô nhiễm, từ đó hỗ trợ ra quyết định cảnh báo sớm chính xác hơn.

---

## 📊 2. Dữ liệu & Công cụ

* **Bộ dữ liệu:** Beijing Multi-Site Air Quality Data (PRSA).
* **Thời gian:** 01/03/2013 - 28/02/2017.
* **Trạm quan trắc trọng tâm:** **Aotizhongxin**.
* **Tần suất:** Hàng giờ (Hourly).
* **Công cụ kỹ thuật:**
    * Ngôn ngữ: Python.
    * Thư viện: `pandas`, `statsmodels`, `matplotlib`, `sklearn`.
    * Pipeline: Preprocessing -> Stationarity Test -> ACF/PACF Analysis -> Grid Search -> Forecasting.

---

## 🔍 3. Khám phá dữ liệu (EDA): Bằng chứng của "Nhịp thở" 24h

Trước khi đi vào mô hình hóa, chúng tôi đã thực hiện Phân tích khám phá dữ liệu (EDA) để trả lời câu hỏi cốt lõi: **PM2.5 thay đổi ngẫu nhiên hay có quy luật?**

### 3.1. Toàn cảnh sự biến động (Overview)
Dữ liệu PM2.5 tại Bắc Kinh cho thấy sự biến động cực mạnh.Các đỉnh nhọn (spikes) thường xuyên vượt ngưỡng 300-400 $\mu g/m^3$, thậm chí chạm mốc 999 $\mu g/m^3$. Chuỗi dữ liệu mang tính **không dừng (non-stationary)** rõ rệt về phương sai và trung bình, đòi hỏi phải xử lý sai phân (differencing) trước khi huấn luyện mô hình.

### 3.2. Soi chi tiết (Zoom-in Analysis)
Khi phóng to vào khung thời gian ngắn (1 tháng), quy luật vận động bắt đầu lộ diện. Các đường biểu đồ không đi ngẫu nhiên mà có dạng sóng lên xuống nhịp nhàng:
* **Ban đêm/Sáng sớm:** Bụi thường tích tụ cao.
* **Buổi chiều:** Nồng độ giảm xuống (do nhiệt độ tăng, không khí đối lưu tốt hơn).
Đây là dấu hiệu sơ khởi của **Mùa vụ trong ngày (Daily Seasonality)**.

### 3.3. Bằng chứng thép từ biểu đồ Tự tương quan (ACF)
Để khẳng định khoa học, chúng tôi sử dụng biểu đồ ACF (Auto-Correlation Function). Kết quả cho thấy:
* Các cột tương quan **không tắt dần đều**.
* Xuất hiện các đỉnh nhọn (peaks) lặp lại đều đặn ở các độ trễ (lags): **24, 48, 72, 96...**.

👉 **Kết luận:** Giá trị PM2.5 tại thời điểm $t$ có mối liên hệ mật thiết với chính nó tại $t-24$, $t-48$.Do đó, tham số chu kỳ mùa vụ **$s=24$** là bắt buộc.

---

## 🛠 4. Phương pháp luận: Từ ARIMA đến SARIMA

### Tại sao ARIMA là chưa đủ?
Mô hình ARIMA truyền thống $(p,d,q)$ hoạt động tốt với xu hướng ngắn hạn nhưng "mù" trước các quy luật lặp lại dài hạn. Nếu dùng ARIMA, đường dự báo thường có xu hướng đi phẳng về giá trị trung bình sau vài bước thời gian, làm mất đi thông tin về các đỉnh ô nhiễm trong ngày.

### Giải pháp: SARIMA $(p,d,q) \times (P,D,Q,s)$
Chúng tôi thiết lập cấu hình mô hình như sau:

| Thành phần | Tham số | Giải thích chi tiết |
| :--- | :---: | :--- |
| **Trend Order** | `(1, 0, 1)` | **p=1, q=1**: Nắm bắt mối quan hệ tức thời giữa các giờ liền kề. **d=0**: Chuỗi đã được xử lý để đạt tính dừng tương đối. |
| **Seasonal Order** | `(0, 1, 1, 24)` | **s=24**: Chu kỳ mùa vụ 24 giờ (Daily cycle).<br>**D=1**: Thực hiện sai phân mùa vụ (Seasonal Differencing: $Y_t - Y_{t-24}$) để loại bỏ sự phụ thuộc chu kỳ, giúp chuỗi trở nên dừng.<br>**Q=1**: Xử lý sai số trung bình trượt ở cấp độ mùa vụ. |

---

## 📈 5. Kết quả & Đánh giá hiệu suất

Mô hình được huấn luyện trên dữ liệu 2013-2016 và kiểm thử (test) trên dữ liệu từ **01/01/2017** với đường chân trời dự báo (horizon) là **48 giờ**.

### Trực quan hóa: Forecast vs Actual
Đường dự báo của SARIMA (màu đỏ) đã mô phỏng lại khá tốt "nhịp điệu" lên xuống của đường thực tế (màu đen). Khác với đường trung bình đi ngang, SARIMA đã "học" được cách uốn lượn: **tăng vào đêm, giảm vào ngày**.

### Bảng chỉ số đánh giá (Metrics)

| Metric | Giá trị | Ý nghĩa thực tiễn |
| :--- | :--- | :--- |
| **RMSE** | **~35.5** | (Root Mean Squared Error) Chỉ số này khá cao, phản ánh việc mô hình bị "phạt nặng" khi dự báo sai các điểm đỉnh (spikes) đột biến. |
| **MAE** | **~22.1** | (Mean Absolute Error) Sai số tuyệt đối trung bình. Trung bình mỗi giờ, dự báo lệch khoảng 22 $\mu g/m^3$ so với thực tế. |

---

## 💡 6. Năm (5) Insight Quản trị & Khuyến nghị Hành động

Từ kết quả kỹ thuật, chúng tôi rút ra 5 đề xuất chiến lược dành cho **Cơ quan Quản lý Môi trường Đô thị**:

1.  **Quy luật 24h là bất biến:**
    * *Insight:* Dù mùa đông hay hè, ô nhiễm luôn tuân theo chu kỳ ngày đêm.
    * *Hành động:* Thay vì phát bản tin 1 lần/ngày, cần triển khai hệ thống **biển báo điện tử thời gian thực**: Cảnh báo Đỏ (7h-9h), chuyển sang Vàng (14h-16h) dựa trên dự báo giờ.

2.  **Thách thức từ các "Đỉnh ô nhiễm" (Spikes):**
    * *Insight:* RMSE cao hơn MAE chứng tỏ mô hình đôi khi vẫn bị "giật mình" bởi các đợt tăng cực đại bất thường.
    * *Hành động:* Thiết lập quy trình **Phản ứng nhanh**: Khi đường dự báo SARIMA có xu hướng dốc đứng, kích hoạt ngay kịch bản hạn chế giao thông cục bộ mà không cần đợi chỉ số đạt đỉnh thực tế.

3.  **Ưu thế "Nhịp điệu" của SARIMA:**
    * *Insight:* SARIMA giữ được biên độ dao động tốt hơn ARIMA (vốn hay bị mean reversion - kéo về trung bình).
    * *Hành động:* Sử dụng SARIMA làm mô hình nòng cốt cho các ứng dụng điều tiết giao thông ngắn hạn (trong vòng 48h).

4.  **Giới hạn của dữ liệu quá khứ:**
    * *Insight:* SARIMA chỉ nhìn vào lịch sử PM2.5. Nó sẽ thất bại nếu có một cơn mưa rào bất chợt (yếu tố ngoại sinh) làm sạch không khí ngay lập tức.
    * *Hành động:* Nâng cấp hệ thống lên **SARIMAX**, tích hợp thêm biến đầu vào: *Tốc độ gió (WSPM)* và *Lượng mưa (RAIN)* để tăng độ chính xác khi thời tiết biến động.

5.  **Tối ưu hóa nguồn lực nhân sự:**
    * *Insight:* Chúng ta biết trước các khung giờ "nóng" nhờ chu kỳ dự báo.
    * *Hành động:* Phân bổ Cảnh sát giao thông và Đội kiểm tra khí thải tập trung vào các giờ cao điểm dự báo, giảm bớt nhân sự vào giờ thấp điểm để tiết kiệm ngân sách.

---

## 🏁 7. Kết luận

Dự án đã chứng minh rằng việc tích hợp yếu tố mùa vụ ($s=24$) thông qua mô hình **SARIMA** là bước tiến quan trọng so với các phương pháp thống kê cơ bản.

[cite_start]SARIMA không chỉ là những con số toán học khô khan, mà là công cụ giúp chúng ta **"lắng nghe nhịp thở của thành phố"**, từ đó chuyển đổi dữ liệu thô thành các hành động bảo vệ sức khỏe cộng đồng kịp thời và hiệu quả hơn[cite: 524].

---
