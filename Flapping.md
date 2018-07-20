## Flapping
Icinga hỗ trợ tùy chọn phát hiện các host và service "flapping". Flapping xảy ra khi 1 service hoặc host thay đổi trạng thái quá thường xuyên, gây ra vấn đề và thông báo khôi phục. Flapping có thể là dấu hiệu của các vấn đề cấu hình hoặc các vấn đề về mạng thực sự.  
### How Flap Detection Works
Khi Icinga kiểm tra trạng thái của host hoặc service, nó sẽ kiểm tra xem nó đã start hay stop flapping chưa. Nó thực hiện bằng cách:  
- Lưu trữ kết quả của 21 lần kiểm tra cuối cùng của host hoặc service.  
- Phân tích các kết quả kiểm tra lịch sử và xác định những thay đổi trạng thái xảy ra.  
- Sử dụng sự chuyển biến trạng thái để xác định % sự thay đổi.  
- So sánh % với ngưỡng thấp và ngưỡng cao.  

Một host hoặc service được start flapping khi % sự thay đổi trạng thái vượt quá ngưỡng cao và stop flapping khi % sự thay đổi trạng thái xuống dưới ngưỡng thấp.  
### Example
Vì thực hiện lưu trữ 21 kết quả kiểm tra gần nhất nên sẽ có nhiều nhất là 20 lần thay đổi trạng thái. Ví dụ có 7 sự thay đổi trạng thái trong 21 lần kiểm tra gần nhất, (7/20) * 100=35% sự thay đổi trạng thái. Lấy % này so sánh với ngưỡng cao và ngưỡng thấp.  
### Flap Detection for Services
- Icinga sẽ kiểm tra 1 services có đang flapping không bất cứ khi nào services được check.  

### Flap Detection for Hosts
- Icinga sẽ kiểm tra 1 host có đang flapping không bất cứ khi nào host được check.
- Đôi khi một dịch vụ kết hợp với máy chủ đó được kiểm tra. Cụ thể hơn, khi ít nhất x thời gian trôi qua kể từ khi flap detection gần nhất được thực hiện, trong đó x bằng với khoảng thời gian trung bình của tất cả các service nằm trong host.  

### Flap Detection Thresholds
- Icinga sử dụng Global Variables và object-Specific Variables dùng cho flap detection. Nếu không khai báo biến Object-Specific thì biến Global sẽ được sử dụng.  
- Ngưỡng cao cho cả Host và Service: 50%  
- Ngưỡng thấp cho cả Host và Service: 25%  

### States Used For Flap Detection
Icinga sẽ sử dụng 21 trạng thái gần nhất để phân tích. Ta có thể chỉ định các trạng thái được sử dụng bằng các khai báo `flap_detection_options` trong định nghĩa host và services. 
### Flap Handling
Khi một service hoặc host được phát hiện flapping:
- Log lại 1 message cho biết rằng service hoặc dịch vụ đang flap.  
- Thêm một comment non-persistent để chỉ ra rằng nó đang flap.  
- Gửi "flapping start" cho service hoặc host tới contact đặc biệt.  
- Ngăn chặn các thông báo khác cho service hoặc host.  

Khi một service hoặc host stop flapping, Icinga sẽ:  
- Log lại 1 message cho biết rằng service hoặc dịch vụ đã ngừng flap.  
- Delete comment được thêm vào khi start flapping.  
- Gửi "flapping stop" cho service hoặc host tới contact đặc biệt.  
- Loại bỏ ngăn chặn thông báo.  

### Enabling Flap Detection
Kích hoạt tính năng flap detection bằng cách:  
- Set `enable_flapping = true `  

