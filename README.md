# Quản lý bệnh nhân can thiệp hô hấp — Khoa Nội Hô Hấp, BVQY 175

Ứng dụng web quản lý quy trình bệnh nhân can thiệp hô hấp (EBUS-TBNA, EBUS-TBLB, Nội soi phế quản + sinh thiết, Sinh thiết xuyên thành, Sinh thiết màng phổi) từ lúc hẹn nhập viện đến khi trả kết quả.

## ⚠️ Tình trạng hiện tại — đọc trước khi dùng cho bệnh nhân thật

Đây là **1 file HTML chạy hoàn toàn phía trình duyệt (client-side)**, không có backend/database.

- **Không có nơi lưu trữ dùng chung.** Toàn bộ dữ liệu chỉ tồn tại trong bộ nhớ của tab trình duyệt đang mở — tải lại trang hoặc đóng tab là **mất hết dữ liệu**.
- **Không đồng bộ nhiều người dùng.** Nếu 2 người mở file này trên 2 máy khác nhau, họ sẽ thấy 2 danh sách bệnh nhân độc lập, không ai thấy dữ liệu của người kia.
- Dữ liệu mẫu/demo trước đây đã được **xóa hoàn toàn** — ứng dụng khởi động với danh sách rỗng, sẵn sàng để bác sĩ/điều dưỡng nhập bệnh nhân thật qua nút "Thêm bệnh nhân".

Vì khoa cần **nhiều bác sĩ/điều dưỡng cùng dùng chung 1 danh sách bệnh nhân**, file này **chưa thể dùng ngay cho thực tế** — cần gắn thêm một backend/database thật để lưu và đồng bộ dữ liệu. Xem mục "Bước tiếp theo" bên dưới.

## Bước tiếp theo: chọn giải pháp lưu trữ dùng chung

Vì đây là dữ liệu bệnh nhân thật của bệnh viện quân y, việc chọn nơi lưu trữ dữ liệu **cần được đơn vị CNTT / An ninh của bệnh viện xem xét và phê duyệt** trước, không nên tự ý đẩy dữ liệu bệnh nhân lên một dịch vụ cloud bên ngoài khi chưa được phép. Một số hướng phổ biến, xếp theo mức độ phù hợp:

1. **Server/database nội bộ của bệnh viện (khuyến nghị nếu có sẵn)** — nhờ phòng CNTT bệnh viện dựng một backend nhỏ (ví dụ Node.js/Python + PostgreSQL hoặc MySQL) đặt trong mạng nội bộ hoặc trên hạ tầng do bệnh viện quản lý. Đây là lựa chọn an toàn nhất cho dữ liệu bệnh nhân quân y.
2. **Dịch vụ cloud (Firebase, Supabase, Google Sheets API...)** — triển khai nhanh, có gói miễn phí, nhưng dữ liệu sẽ nằm trên hạ tầng của bên thứ ba ở nước ngoài. **Chỉ nên dùng nếu đã được đơn vị CNTT/an ninh bệnh viện đồng ý bằng văn bản**, và cần bật xác thực (authentication) + quy tắc phân quyền chặt chẽ trước khi nhập bất kỳ dữ liệu bệnh nhân thật nào.
3. **Tạm thời dùng thử 1 người/1 máy** — nếu chỉ cần dùng thử trước khi có backend, có thể thêm lưu trữ cục bộ (localStorage) để không mất dữ liệu khi tải lại trang, nhưng vẫn chỉ giới hạn 1 máy, không dùng cho vận hành thật với nhiều người.

Khi đã chọn được giải pháp, mã nguồn (`index.html`) đã sẵn sàng để nối vào backend: toàn bộ thao tác thêm/sửa bệnh nhân, tick checklist, chuyển giai đoạn đều đi qua các hàm JavaScript riêng biệt (`addPatient()`, `advance()`, `toggleChk()`, `removeP()`...) — chỉ cần thay phần đọc/ghi mảng `patients` trong bộ nhớ bằng lời gọi API tới backend thật.

## Đưa lên GitHub

- **Dùng repository RIÊNG TƯ (private)**, không public — kể cả khi chưa có dữ liệu bệnh nhân thật, đây là công cụ nội bộ của khoa.
- **Không commit dữ liệu bệnh nhân thật vào git** dưới bất kỳ hình thức nào (không paste vào README, không lưu file .csv/.json export vào repo). Vì lịch sử git lưu lại vĩnh viễn, xóa file sau này không xóa được dữ liệu khỏi lịch sử.
- Nếu sau này thêm backend, tuyệt đối không commit API key / mật khẩu / chuỗi kết nối database vào repo — dùng biến môi trường (environment variables) hoặc file cấu hình nằm ngoài git (`.gitignore`).
- Có thể host tĩnh bằng **GitHub Pages** (Settings → Pages → chọn nhánh) để có link truy cập nhanh trong lúc chưa có backend — nhưng nhắc lại: ở trạng thái hiện tại (chưa có backend) chỉ nên dùng để xem giao diện/demo quy trình, không nhập dữ liệu bệnh nhân thật vào đó vì sẽ mất khi đóng trình duyệt.

## Trước khi dùng chính thức, khoa nên tự rà lại thêm

- Danh sách bác sĩ trong file (`DOCTORS` ở đầu phần `<script>`) hiện là **tên giữ chỗ (placeholder)** — cần thay bằng danh sách bác sĩ thật của khoa.
- Cân nhắc thêm chức năng đăng nhập/phân quyền nếu nhiều người cùng dùng, để biết ai thao tác gì trên hồ sơ bệnh nhân nào.
- Cân nhắc chính sách sao lưu (backup) định kỳ khi đã có backend/database thật.

## Đã sửa so với bản trước

- Xóa toàn bộ danh sách bệnh nhân mẫu/demo — khởi động với danh sách rỗng.
- Ngày "hôm nay" trong ứng dụng nay lấy theo ngày thực tế của máy khi mở trang (trước đây hardcode cố định 18/07/2026).
- Checklist của bệnh nhân mới luôn khởi tạo ở trạng thái **chưa hoàn thành toàn bộ** (trước đây tự động tick sẵn ngẫu nhiên một số mục — dễ gây hiểu lầm là đã làm xong việc chưa làm).
- Form "Thêm bệnh nhân" có thêm các trường Địa chỉ, Đối tượng BHYT, Số thẻ BHYT (trước đây các trường này bị tự sinh số liệu giả).
- Panel "Thống kê" và "Bệnh nhân nổi bật/cần ưu tiên" nay tính từ dữ liệu bệnh nhân thật đang có trong hệ thống, không còn số liệu/ngày tháng cố định.
