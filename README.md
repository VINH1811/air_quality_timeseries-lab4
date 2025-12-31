# 🌏 Dự Án: "Bắt Mạch" Nhịp Thở Đô Thị - Dự Báo Bụi Mịn PM2.5 (Beijing Air Quality)

![Project Banner](https://img.shields.io/badge/Project-Time_Series_Forecasting-blue?style=for-the-badge&logo=python)
![Status](https://img.shields.io/badge/Status-Completed-success?style=for-the-badge)
![Model](https://img.shields.io/badge/Model-SARIMA_&_Regression-orange?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)

> **"Chúng tôi không chỉ dự báo những con số vô hồn, chúng tôi dự báo nhịp điệu sinh học của cả một thành phố."**

---

## 👥 Đội Ngũ Thực Hiện (Team WL)

| Thành viên | Vai trò |
| :--- | :--- |
| **Nguyễn Văn Vinh** | Data Engineer & Pipeline |
| **Đỗ Văn Vinh** | Model Developer (SARIMA) |
| **Lại Thành Đoạn** | Data Analyst (EDA) & Visualization |
| **Bạch Ngọc Lương** | Research & Documentation |

---

## 📑 Mục Lục
1. [Giới thiệu & Đặt vấn đề](#-1-giới-thiệu--đặt-vấn-đề)
2. [Dữ liệu & Công nghệ](#-2-dữ-liệu--công-nghệ)
3. [Phân tích Dữ liệu (EDA)](#-3-khám-phá-dữ-liệu-eda-bằng-chứng-của-nhịp-thở-24h)
4. [Phương pháp luận & Mô hình hóa](#-4-phương-pháp-luận--mô-hình-hóa)
    * [Baseline: Hồi quy (Regression)](#41-chiến-lược-1-hồi-quy-tuyến-tính-baseline)
    * [Advanced: SARIMA](#42-chiến-lược-2-sarima---mô-hình-hóa-mùa-vụ)
5. [Kết quả & Đánh giá](#-5-kết-quả--đánh-giá-hiệu-suất)
6. [Insight Quản trị & Khuyến nghị](#-6-năm-5-insight-quản-trị--khuyến-nghị-hành-động)
7. [Hướng dẫn Cài đặt & Chạy](#-7-hướng-dẫn-cài-đặt--chạy-dự-án)

---

## 📖 1. Giới thiệu & Đặt vấn đề

Trong bối cảnh đô thị hóa nhanh chóng, ô nhiễm không khí—đặc biệt là **bụi mịn PM2.5**—đã trở thành mối đe dọa thầm lặng nhưng nghiêm trọng đối với sức khỏe cộng đồng.

Chúng ta thường quen với các dự báo thời tiết chung chung như "Ngày mai trời nắng". Tuy nhiên, với chất lượng không khí, biết mức trung bình của ngày mai là **chưa đủ**. Nồng độ bụi có thể ở mức an toàn vào buổi trưa nhưng tăng vọt lên mức nguy hại vào giờ tan tầm hoặc đêm khuya do hiện tượng nghịch nhiệt. Nếu chỉ nhìn vào số liệu hiện tại hoặc áp dụng ngưỡng cảnh báo tĩnh, nhà quản lý sẽ bỏ lỡ các đợt ô nhiễm tăng vọt theo giờ.

**🎯 Mục tiêu dự án:**
Dự án tập trung giải quyết bài toán dự báo ngắn hạn (short-term forecasting) nồng độ PM2.5 theo từng giờ. Thay vì chỉ sử dụng các mô hình ARIMA cơ bản, chúng tôi triển khai mô hình **SARIMA (Seasonal ARIMA)** để mô hình hóa tính chu kỳ (mùa vụ) 24 giờ của ô nhiễm, từ đó hỗ trợ ra quyết định cảnh báo sớm chính xác hơn.

---

## 📊 2. Dữ liệu & Công nghệ

### 2.1. Bộ dữ liệu (Dataset)
* **Nguồn:** Beijing Multi-Site Air Quality Data (PRSA).
* **Phạm vi thời gian:** 01/03/2013 - 28/02/2017.
* **Trạm quan trắc trọng tâm:** **Aotizhongxin**.
* **Tần suất:** Hàng giờ (Hourly).
* **Đặc điểm:** Dữ liệu chứa các biến khí tượng (TEMP, PRES, DEWP, RAIN, WSPM) và các chất gây ô nhiễm (PM2.5, PM10, SO2, NO2, CO, O3).

### 2.2. Tech Stack
* **Ngôn ngữ:** Python 3.9+
* **Core Libraries:** `pandas`, `numpy`, `statsmodels`, `scikit-learn`, `matplotlib`.
* **Architecture:** Modular Design (OOP) với thư mục `src/` chứa các class xử lý riêng biệt (Clean, Regression, TimeSeries).

---

## 🔍 3. Khám phá dữ liệu (EDA): Bằng chứng của "Nhịp thở" 24h

Trước khi mô hình hóa, chúng tôi thực hiện EDA để trả lời câu hỏi cốt lõi: **PM2.5 thay đổi ngẫu nhiên hay có quy luật?**

### 3.1. Toàn cảnh sự biến động (Overview)
Dữ liệu PM2.5 tại Bắc Kinh biến động cực mạnh. Các đỉnh nhọn (spikes) thường xuyên vượt ngưỡng 300-400 $\mu g/m^3$, thậm chí chạm mốc 999 $\mu g/m^3$. Chuỗi dữ liệu mang tính **không dừng (non-stationary)** rõ rệt, đòi hỏi xử lý sai phân.

![Overview Plot](images/hinh1_overview.png)
*(Hình 1: Biến động PM2.5 toàn giai đoạn 2013-2017)*

### 3.2. Soi chi tiết (Zoom-in Analysis)
Khi phóng to vào khung thời gian ngắn (1 tháng), quy luật vận động lộ diện:
* **Ban đêm/Sáng sớm:** Bụi tích tụ cao.
* **Buổi chiều:** Nồng độ giảm (do nhiệt độ tăng, đối lưu không khí tốt).
Đây là dấu hiệu của **Mùa vụ trong ngày (Daily Seasonality)**.

![Zoom Plot](images/hinh2_zoom.png)
*(Hình 2: Chi tiết biến động trong 1 tháng)*

### 3.3. Bằng chứng thép từ ACF (Autocorrelation)
Biểu đồ ACF cho thấy các cột tương quan **không tắt dần đều** mà xuất hiện các đỉnh nhọn lặp lại ở các độ trễ: **24, 48, 72, 96...**.

👉 **Kết luận:** Giá trị PM2.5 tại thời điểm $t$ có mối liên hệ mật thiết với chính nó tại $t-24$. Do đó, tham số chu kỳ mùa vụ **$s=24$** là bắt buộc.

![ACF Plot](images/hinh3_acf.png)
*(Hình 3: ACF Plot khẳng định chu kỳ 24h)*

---

## 🛠 4. Phương pháp luận & Mô hình hóa

Chúng tôi tiếp cận theo hai chiến lược để so sánh và tối ưu hóa.

### 4.1. Chiến lược 1: Hồi quy tuyến tính (Baseline)
Biến bài toán chuỗi thời gian thành bài toán **Supervised Learning** có giám sát.

* **Feature Engineering:** Tạo các biến trễ (Lag features). Đặc trưng quan trọng nhất là `lag_24` (giá trị của giờ này ngày hôm qua).
* **Time Splitting:** Sử dụng `CUTOFF = '2017-01-01'` để chia Train/Test. Tuyệt đối không dùng Shuffle để tránh **Data Leakage** (nhìn trộm tương lai).

### 4.2. Chiến lược 2: SARIMA - Mô hình hóa Mùa vụ (Main Approach)

**Tại sao ARIMA là chưa đủ?**
ARIMA $(p,d,q)$ chỉ bắt được xu hướng ngắn hạn. Nếu dùng ARIMA, đường dự báo thường đi phẳng về giá trị trung bình, mất đi thông tin về các đỉnh ô nhiễm trong ngày.

**Giải pháp: SARIMA $(p,d,q) \times (P,D,Q,s)$**
Chúng tôi thiết lập cấu hình mô hình dựa trên quy trình Grid Search và kiểm định AIC:

| Loại tham số | Giá trị | Giải thích kỹ thuật |
| :--- | :---: | :--- |
| **Trend (p, d, q)** | `(1, 0, 1)` | **p=1, q=1**: Nắm bắt quan hệ tức thời. **d=0**: Chuỗi đã xử lý nên khá dừng (hoặc dừng yếu). |
| **Seasonal (P, D, Q, s)** | `(0, 1, 1, 24)` | **s=24**: Chu kỳ 24h.<br>**D=1**: Sai phân mùa vụ ($Y_t - Y_{t-24}$) để loại bỏ tính chu kỳ. |

---

## 📈 5. Kết quả & Đánh giá hiệu suất

Mô hình được kiểm thử trên tập Test (từ 01/01/2017) với Horizon dự báo là **48 giờ**.

### Trực quan hóa: Forecast vs Actual
Đường dự báo của SARIMA (màu đỏ) đã mô phỏng lại khá tốt "nhịp điệu" lên xuống của đường thực tế (màu đen). Khác với đường trung bình đi ngang, SARIMA đã "học" được cách uốn lượn: **tăng vào đêm, giảm vào ngày**.

![Forecast Plot](images/hinh4_forecast.png)
*(Hình 4: Kết quả dự báo SARIMA so với thực tế)*

### Bảng chỉ số đánh giá (Metrics)

| Metric | Giá trị | Ý nghĩa thực tiễn |
| :--- | :--- | :--- |
| **RMSE** | **~35.5** | (Root Mean Squared Error) Chỉ số này khá cao, phản ánh việc mô hình bị "phạt nặng" khi dự báo sai các điểm đỉnh (spikes) đột biến. |
| **MAE** | **~22.1** | (Mean Absolute Error) Sai số tuyệt đối trung bình. Trung bình mỗi giờ, dự báo lệch khoảng 22 $\mu g/m^3$ so với thực tế. |

---

## 💡 6. Năm (5) Insight Quản trị & Khuyến nghị Hành động

Từ kết quả phân tích dữ liệu và mô hình, chúng tôi đề xuất 5 chiến lược hành động cho cơ quan quản lý:

1.  **Quy luật 24h là bất biến:**
    * *Insight:* Ô nhiễm luôn tuân theo chu kỳ ngày đêm.
    * *Hành động:* Thay vì bản tin ngày, triển khai **biển báo điện tử thời gian thực**: Cảnh báo Đỏ (7h-9h), Vàng (14h-16h).

2.  **Thách thức từ "Đỉnh ô nhiễm" (Spikes):**
    * *Insight:* RMSE cao chứng tỏ mô hình đôi khi bị "giật mình" bởi các đợt tăng cực đại.
    * *Hành động:* Thiết lập quy trình **Phản ứng nhanh**: Kích hoạt hạn chế giao thông cục bộ ngay khi đường dự báo SARIMA dốc đứng, không đợi đạt đỉnh thực tế.

3.  **Ưu thế "Nhịp điệu" của SARIMA:**
    * *Insight:* SARIMA giữ biên độ dao động tốt hơn ARIMA (tránh hiện tượng mean reversion).
    * *Hành động:* Dùng SARIMA làm mô hình nòng cốt cho điều tiết giao thông ngắn hạn (48h).

4.  **Giới hạn của quá khứ:**
    * *Insight:* SARIMA sẽ thất bại nếu có mưa rào bất chợt (yếu tố ngoại sinh) làm sạch không khí.
    * *Hành động:* Nâng cấp lên **SARIMAX**, tích hợp biến *Gió (WSPM)* và *Mưa (RAIN)* để tăng độ chính xác.

5.  **Tối ưu hóa nguồn lực nhân sự:**
    * *Insight:* Biết trước khung giờ "nóng".
    * *Hành động:* Tập trung Cảnh sát giao thông và Kiểm tra khí thải vào giờ cao điểm dự báo để tối ưu ngân sách.

---

## 🛠 7. Hướng dẫn Cài đặt & Chạy dự án

Dự án được cấu trúc dạng module (OOP) thay vì notebook rời rạc.

### Cấu trúc thư mục
```bash
├── data/
│   ├── raw/                # Chứa file PRSA2017_Data_....zip
│   └── processed/          # Chứa file parquet đã xử lý
├── notebooks/              # Chứa các file jupyter chạy thử nghiệm
├── src/                    # Source code chính
│   ├── classification_library.py
│   ├── regression_library.py
│   └── timeseries_library.py
└── requirements.txt
