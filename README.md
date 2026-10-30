# HỆ THỐNG DỰ BÁO THỜI TIẾT CLIENT - SERVER (SOCKET TCP)
## BÀI TẬP LỚN LẬP TRÌNH MẠNG - ĐỀ SỐ 6

---

## 1. YÊU CẦU ĐỀ BÀI
- **Chương trình Client**: Biểu diễn thông tin dự báo thời tiết, cho phép người dùng chọn vùng/khu vực để xem.
- **Chương trình Server**: Phân tích và lấy thông tin dự báo thời tiết ở một số trang Web để trả về cho Client.
- **Môi trường hiện thực**: Ngôn ngữ lập trình **Java** (Mô hình Client - Server sử dụng **Socket TCP**, đa luồng **Multi-threading**).

---

## 2. ĐIỂM NỔI BẬT & TÍNH NĂNG CỦA DỰ ÁN

### 🌐 Phân tích dữ liệu từ nhiều trang Web (Multi-source Web Scraping)
Chương trình Server đáp ứng chuẩn xác và vượt trội yêu cầu *"ở một số trang Web"* bằng cách tích hợp đồng thời 3 nguồn dữ liệu:
1. **`thoitiet.vn`**: Cào và phân tích trực tiếp mã nguồn HTML tiếng Việt bằng thư viện **Jsoup** (`Document`, `Elements`, `select`). Lấy thông tin thời tiết hiện tại, cảm giác như, độ ẩm, gió, UV, điểm ngưng, dự báo chi tiết theo từng giờ và chuỗi dự báo 7 ngày tới.
2. **`Open-Meteo`**: Dịch vụ khí tượng toàn cầu mã nguồn mở, độ chính xác cao theo tọa độ địa lý, giải mã mã chuẩn WMO sang tiếng Việt và cung cấp dự báo 7 ngày.
3. **`wttr.in`**: Dịch vụ thời tiết web hỗ trợ tiếng Việt, dùng để đối chiếu độc lập và làm nguồn dự phòng.
4. **Cơ chế Caching & Fallback tự động**: Tự động lưu cache trong 10 phút để tối ưu tốc độ và tránh bị chặn IP. Nếu một trang web bị sự cố mạng, Server tự động chuyển đổi sang nguồn dữ liệu kế tiếp đảm bảo không bao giờ gián đoạn.
5. **Tính năng So sánh Đa Nguồn (Multi-source Comparison)**: Đối chiếu cùng lúc kết quả dự báo giữa 3 trang web khác nhau trên cùng một bảng dữ liệu trực quan.

### 💻 Giao diện Client hiện đại & Thân thiện
- **Giao diện Đồ họa (GUI Swing + FlatLaf)**: Thiết kế thẻ Card UI chuẩn hiện đại, hỗ trợ đổi màu Giao diện Sáng (Light Mode) / Tối (Dark Mode).
- **Hỗ trợ chọn vùng toàn diện**:
  - Tích hợp sẵn toàn bộ **63 tỉnh thành phố Việt Nam** (phân loại Miền Bắc, Miền Trung, Miền Nam) và các đô thị lớn trên thế giới (Tokyo, Seoul, London, Paris, New York,...).
  - Hỗ trợ ô tìm kiếm thông minh: Tự động loại bỏ dấu tiếng Việt (ví dụ gõ `"da lat"` hoặc `"Đà Lạt"` đều tìm chính xác).
- **Trực quan hóa đa dạng**:
  - **Tab 1 - Thời tiết hiện tại**: Thẻ Hero Card (nhiệt độ cỡ lớn, biểu tượng emoji, tình trạng bầu trời, cảm giác như, nguồn web) + Lưới 6 chỉ số (Độ ẩm, Gió, UV, Tầm nhìn, Điểm ngưng, Thấp/Cao).
  - **Tab 2 - Dự báo 7 ngày tới**: Danh sách thẻ từng ngày kèm biên độ nhiệt độ, độ ẩm, tốc độ gió.
  - **Tab 3 - Dự báo theo giờ**: Dòng thời gian cuộn ngang hiển thị diễn biến thời tiết trong ngày.
  - **Tab 4 - Bảng so sánh các trang Web**: Bảng so sánh chỉ số giữa `thoitiet.vn` vs `Open-Meteo` vs `wttr.in`.
  - **Tab 5 - Nhật ký Gói tin TCP Socket (Network Packet Inspector)**: Theo dõi trực tiếp các gói tin JSON gửi và nhận qua Socket TCP, hiển thị thời gian phản hồi (latency ms) phục vụ cho việc chấm điểm môn Lập trình mạng.
- **Giao diện Dòng lệnh (CLI)**: Cung cấp menu tương tác console cho người dùng muốn chạy trong terminal.

---

## 3. CẤU TRÚC THƯ MỤC DỰ ÁN

```
D:\hk1n3\LTM\DE6\
├── pom.xml                               # Cấu hình Maven (Jsoup, Gson, FlatLaf, Shade plugin)
├── README.md                             # Tài liệu hướng dẫn chi tiết
├── CHAY-TAT-CA.bat                       # Script 1-click mở CẢ Server GUI & Client GUI cùng lúc
├── 1-CHAY-SERVER.bat                     # Script 1-click khởi động Server GUI (Port 8888)
├── 2-CHAY-CLIENT.bat                     # Script 1-click khởi động Client GUI (Dự báo Thời tiết)
├── build.bat                             # Script biên dịch lại project nếu cần
├── lib/                                  # Thư viện JAR (jsoup, gson, flatlaf)
└── src\com\ltm\weather\
    ├── Main.java                         # Điểm vào chính của ứng dụng
    ├── AppLauncherGUI.java               # Giao diện Launcher trực quan
    ├── model\
    │   ├── RegionInfo.java               # Thực thể thông tin vùng/khu vực
    │   ├── CurrentWeather.java           # Dữ liệu thời tiết hiện tại
    │   ├── DailyForecast.java            # Dữ liệu dự báo theo ngày (7 ngày)
    │   ├── HourlyForecast.java           # Dữ liệu dự báo theo giờ
    │   ├── WeatherData.java              # Gói dữ liệu thời tiết đầy đủ
    │   ├── WeatherRequest.java           # Định dạng yêu cầu gửi qua Socket TCP
    │   └── WeatherResponse.java          # Định dạng phản hồi trả về qua Socket TCP
    ├── server\
    │   ├── WeatherServer.java            # Server Socket TCP đa luồng (ServerSocket)
    │   ├── ClientHandler.java            # Xử lý phiên kết nối của từng Client (Thread)
    │   ├── gui\
    │   │   └── WeatherServerGUI.java     # Cửa sổ Quản trị Server bằng giao diện GUI
    │   └── service\
    │       ├── IWeatherScraper.java      # Interface chuẩn cho các module cào web
    │       ├── ThoiTietVnScraper.java    # Cào và phân tích HTML từ thoitiet.vn bằng Jsoup
    │       ├── OpenMeteoScraper.java     # Lấy và xử lý dữ liệu từ Open-Meteo
    │       ├── WttrInScraper.java        # Lấy và xử lý dữ liệu từ wttr.in
    │       └── WeatherService.java       # Quản lý cào web, Cache và So sánh đa nguồn
    ├── client\
    │   ├── WeatherClient.java            # Client Socket TCP kết nối tới Server
    │   ├── gui\
    │   │   ├── WeatherClientGUI.java     # Cửa sổ chính giao diện đồ họa Swing
    │   │   ├── CurrentWeatherPanel.java  # Panel thông tin thời tiết hiện tại
    │   │   ├── DailyForecastPanel.java   # Panel danh sách thẻ dự báo 7 ngày
    │   │   ├── HourlyForecastPanel.java  # Panel dòng thời gian theo giờ
    │   │   ├── SourceComparisonPanel.java# Panel bảng so sánh các trang web
    │   │   └── NetworkLogPanel.java      # Panel giám sát gói tin TCP Socket
    │   └── cli\
    │       └── WeatherClientCLI.java     # Giao diện dòng lệnh console tương tác
    └── util\
        ├── JsonUtil.java                 # Tiện ích chuyển đổi JSON với Gson
        └── RegionDirectory.java          # Danh mục 63 tỉnh thành & thuật toán tìm kiếm
```

---

## 4. GIAO THỨC TRUYỀN THÔNG (TCP SOCKET PROTOCOL)

Hệ thống sử dụng luồng ký tự mã hóa UTF-8 truyền các chuỗi JSON kết thúc bằng ký tự xuống dòng (`\n`).

### 4.1. Định dạng Yêu cầu (Client -> Server)
```json
{
  "action": "GET_WEATHER",
  "region": "Hà Nội",
  "source": "AUTO"
}
```
Các tham số:
- `action`: `GET_WEATHER` (lấy thời tiết), `COMPARE_SOURCES` (so sánh đa nguồn), `GET_REGIONS` (lấy danh mục tỉnh thành), `PING` (kiểm tra trạng thái kết nối).
- `region`: Tên tỉnh/thành phố (ví dụ `"Hà Nội"`, `"TP. Hồ Chí Minh"`, `"Đà Nẵng"`).
- `source`: `AUTO` (tự động chọn nguồn tốt nhất), `THOITIET_VN` (chỉ lấy thoitiet.vn), `OPEN_METEO` (chỉ lấy Open-Meteo), `WTTR_IN` (chỉ lấy wttr.in), `ALL` (lấy tất cả để so sánh).

### 4.2. Định dạng Phản hồi (Server -> Client)
```json
{
  "status": "SUCCESS",
  "message": null,
  "serverProcessingTimeMs": 420,
  "data": {
    "regionName": "Hà Nội",
    "sourceName": "thoitiet.vn (HTML Web Scraper)",
    "fetchedAt": "30/09/2026 20:30:15",
    "current": {
      "temperature": "31.7°C",
      "feelsLike": "38.1°C",
      "description": "Bầu trời quang đãng",
      "humidity": "70%",
      "windSpeed": "5.14 km/h",
      "uvIndex": "0 - Thấp",
      "visibility": "10 km",
      "dewPoint": "26.9°C",
      "tempLowHigh": "27.9° / 35°",
      "iconEmoji": "☀️"
    },
    "dailyForecasts": [ ... ],
    "hourlyForecasts": [ ... ],
    "comparisons": { ... }
  }
}
```

---

## 5. HƯỚNG DẪN KHỞI CHẠY CHƯƠNG TRÌNH

### Cách 1: Khởi chạy nhanh bằng file `.bat` (Khuyên dùng trên Windows)
- **Lựa chọn 1 (Tiện nhất - 1 Click)**: Nhấp đúp chuột vào file **`CHAY-TAT-CA.bat`**: Sẽ tự động khởi động Server GUI và mở Client GUI cùng lúc!
- **Lựa chọn 2 (Mở từng bước)**:
  1. Nhấp đúp **`1-CHAY-SERVER.bat`** để khởi động Máy chủ (Server GUI tại cổng 8888).
  2. Nhấp đúp **`2-CHAY-CLIENT.bat`** để mở Giao diện người dùng xem thời tiết (Client GUI).
*(Có thể nhấp mở `2-CHAY-CLIENT.bat` nhiều lần để mở nhiều Client cùng một lúc kiểm thử khả năng phục vụ đa luồng của Server!)*

---

### Cách 2: Khởi chạy bằng dòng lệnh Terminal / PowerShell

#### 1. Biên dịch dự án:
```powershell
cd D:\hk1n3\LTM\DE6
mvn clean package -DskipTests
```

#### 2. Chạy Server:
```powershell
java -jar target\weather-forecast-app-1.0.0.jar server 8888
```

#### 3. Chạy Client Giao diện đồ họa (GUI):
```powershell
java -jar target\weather-forecast-app-1.0.0.jar client localhost 8888
```

#### 4. Chạy Client Giao diện dòng lệnh (CLI):
```powershell
java -jar target\weather-forecast-app-1.0.0.jar cli localhost 8888
```

---

## 6. KẾT QUẢ KIỂM THỬ THỰC TẾ

1. **Kiểm thử kết nối TCP Socket**:
   - Client gửi lệnh `PING` -> Server phản hồi `PONG` tức thời (< 5ms).
   - Server đa luồng tiếp nhận đồng thời nhiều Client mà không gây nghẽn.
2. **Kiểm thử cào HTML từ `thoitiet.vn` bằng Jsoup**:
   - Phân tích chính xác thẻ HTML `span.current-temperature` lấy nhiệt độ thực.
   - Trích xuất bảng dự báo 7 ngày từ các thẻ `<details class="weather-day">`.
   - Lấy chuỗi dự báo theo giờ từ `<div id="carousel-hourly">`.
3. **Kiểm thử so sánh đa nguồn**:
   - Lấy đồng thời từ `thoitiet.vn`, `Open-Meteo` và `wttr.in`, điền vào bảng so sánh tại Tab 4.
4. **Kiểm thử Fallback & Caching**:
   - Khi cào một khu vực thành công, truy vấn lần 2 cùng khu vực trả về từ Cache trong < 2ms.
   - Khi một nguồn gặp sự cố, hệ thống tự động fallback sang nguồn tiếp theo mà không làm crash ứng dụng.

---
*Tác giả: Sinh viên môn Lập trình Mạng - Đề tài: Hiển thị Dự báo Thời tiết (Đề 6)*
# DE6
