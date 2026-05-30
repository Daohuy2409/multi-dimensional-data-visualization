# 📊 Báo Cáo Phân Tích Trực Quan Hóa Dữ Liệu Khí Hậu & Thủy Văn

Báo cáo này trình bày quy trình tiền xử lý dữ liệu (làm sạch, định dạng) và các nhận xét khoa học được rút ra từ 4 biểu đồ trực quan hóa dữ liệu đa chiều được tạo bởi script `src/create_visualizations.py` từ các bộ dữ liệu khí hậu và thời tiết thực tế.

---

## 🛠️ Quy Trình Tiền Xử Lý và Làm Sạch Dữ Liệu

Để đảm bảo tính chính xác khoa học và hiển thị đúng đắn trên biểu đồ, các bộ dữ liệu đã được tiền xử lý như sau:

1. **Làm sạch lượng mưa vết (Trace Precipitation) trong `weather_data.csv`**:
   - **Vấn đề**: Cột `precip` chứa ký hiệu chuỗi `"T"` đại diện cho lượng mưa vết (lượng mưa cực kỳ nhỏ không đáng kể), khiến cột này không thể vẽ dưới dạng số liệu trực tiếp.
   - **Giải pháp**: Sử dụng `pd.to_numeric(..., errors='coerce')` để ép kiểu dữ liệu sang dạng số, chuyển `"T"` thành `NaN`, sau đó dùng `.fillna(0.0)` để quy đổi tất cả các giá trị thiếu hoặc lượng mưa vết về giá trị thực tế bằng `0.0`.

2. **Xử lý giá trị khuyết thiếu trong dữ liệu dị thường toàn cầu `global_temp.csv`**:
   - **Vấn đề**: Các tháng chưa xảy ra hoặc không đo đạc được (ví dụ cuối năm 2025) chứa giá trị chuỗi `"***"`.
   - **Giải pháp**: Chuyển đổi `"***"` thành giá trị `pd.NA` và ép kiểu dữ liệu tháng sang dạng số thực (`float`). Điều này giúp thư viện Seaborn nhận diện dữ liệu khuyết thiếu và bỏ trống ô tương ứng trên bản đồ nhiệt (Heatmap) thay vì báo lỗi.

3. **Tạo khoảng trống biểu diễn dữ liệu thiếu trong `minnesota_weather.csv`**:
   - **Vấn đề**: Trạm đo Duluth bị thiếu hoàn toàn bản ghi cho tháng 12 năm 1931. Nếu vẽ trực tiếp từ dữ liệu dọc, thư viện Seaborn/Matplotlib sẽ tự động nối liền điểm tháng 11/1931 với tháng 01/1932, tạo cảm giác dữ liệu liên tục.
   - **Giải pháp**: Tạo cột mốc thời gian dạng `datetime` chuẩn hóa. Sau đó, xoay bảng dữ liệu (`pivot`) để địa điểm làm cột và ngày làm chỉ mục nhằm tự động tạo ra một dòng chứa giá trị `NaN` cho điểm dữ liệu thiếu của Duluth. Khi duỗi ngược lại (`melt`), giá trị `NaN` này được bảo toàn giúp biểu đồ vẽ xuất hiện một khoảng trống ngắt quãng trực quan chính xác.

---

## 📊 Phân Tích Biểu Đồ & Nhận Xét

### 1. Bản đồ nhiệt nhiệt độ trung bình hàng tháng theo thành phố (Average Monthly Temperature Heatmap)
- **Nhận xét**: 
  Biểu đồ thể hiện rõ nét sự khác biệt khí hậu giữa các vĩ độ và bán cầu. **Mumbai** (vùng nhiệt đới) duy trì nhiệt độ cao ổn định quanh năm (> 70°F). Ngược lại, **Bắc Kinh** và **Chicago** thể hiện khí hậu lục địa sâu sắc với biên độ nhiệt mùa đông - mùa hè rất lớn (từ gần đóng băng lên đến gần 80°F). Ngoài ra, chu kỳ mùa của **Auckland** ngược lại hoàn toàn so với các thành phố bán cầu Bắc (nhiệt độ lạnh nhất rơi vào các tháng 6-8 và ấm nhất vào các tháng 12-2), phản ánh đúng quy luật bức xạ mặt trời ở bán cầu Nam.

### 2. Biểu đồ phân tán thời tiết hàng ngày (Daily Weather Scatter Plot)
- **Nhận xét**: 
  Biểu đồ chỉ ra mối tương quan phi tuyến tính giữa nhiệt độ, độ ẩm tương đối và lượng mưa. Các trận mưa lớn (kích thước điểm lớn) chủ yếu tập trung ở những ngày có độ ẩm tương đối trung bình đến cao (> 60%), chứng minh độ ẩm là điều kiện cần cho sự ngưng tụ hơi nước. Về mặt địa lý, ta thấy sự phân cụm rõ rệt: **Mumbai** tập trung ở góc trên bên phải (nhiệt độ cao, độ ẩm rất cao - đặc trưng khí hậu nhiệt đới gió mùa), trong khi **Bắc Kinh** và **Chicago** trải rộng theo trục tung, cho thấy sự biến động nhiệt độ theo mùa mạnh mẽ hơn nhiều so với độ ẩm.

### 3. Bản đồ nhiệt dị thường nhiệt độ toàn cầu (Global Temperature Anomalies Heatmap)
- **Nhận xét**: 
  Biểu đồ cung cấp bằng chứng trực quan về xu hướng nóng lên toàn cầu (Global Warming). Giai đoạn từ năm 1880 đến những năm 1930 chủ yếu là các ô màu xanh lam đậm (dị thường âm từ -0.3°C đến -0.8°C so với mốc cơ sở 1951-1980). Từ thập niên 1980 trở đi, màu sắc chuyển hẳn sang tông đỏ ấm và đỏ đậm. Đáng chú ý nhất, thập kỷ gần đây (2015-2024) bị áp đảo hoàn toàn bởi các dị thường dương cực lớn vượt mức +1.0°C đến +1.4°C, cho thấy tốc độ ấm lên của Trái Đất đang gia tăng một cách nhanh chóng và nghiêm trọng.

### 4. Biểu đồ đường lượng mưa hàng tháng tại Minnesota (Minnesota Precipitation Line Chart)
- **Nhận xét**: 
  Lượng mưa tại các vùng của Minnesota có tính chu kỳ mùa rất mạnh, đạt đỉnh vào các tháng mùa hè (tháng 6 đến tháng 8) và giảm mạnh xuống mức thấp nhất vào mùa đông (tháng 12 đến tháng 2 năm sau). Điều này phản ánh kiểu khí hậu lục địa ẩm ẩm ướt vào mùa hè. So sánh giữa các địa điểm, **Waseca** và **St. Paul** (nằm ở phía Nam và Đông Nam tiểu bang) nhận được lượng mưa trung bình cao hơn rõ rệt so với **Morris** hay **Crookston** (nằm ở phía Tây và Tây Bắc khô hạn hơn), phản ánh ảnh hưởng của khối khí ẩm từ Vịnh Mexico di chuyển từ Nam lên Bắc bị suy giảm dần. Khoảng trống đứt gãy đường vẽ của trạm **Duluth** vào tháng 12/1931 phản ánh trung thực tình trạng mất mát dữ liệu lịch sử trong quan trắc khí tượng khí hậu.