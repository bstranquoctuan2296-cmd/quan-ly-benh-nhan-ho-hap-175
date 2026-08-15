# Quản lý bệnh nhân can thiệp hô hấp — Khoa Nội Hô Hấp, BVQY 175

Ứng dụng web quản lý quy trình bệnh nhân can thiệp hô hấp (EBUS-TBNA, EBUS-TBLB, Nội soi phế quản + sinh thiết, Sinh thiết xuyên thành, Sinh thiết màng phổi) từ lúc hẹn nhập viện đến khi trả kết quả.

## ✅ Đã có backend dùng chung — đọc trước khi dùng cho bệnh nhân thật

Ứng dụng hiện đã kết nối **backend thật (Firebase — Firestore + Authentication)**, thay cho bản trước chỉ chạy trong bộ nhớ trình duyệt.

- **Dữ liệu dùng chung, đồng bộ theo thời gian thực.** Mọi bác sĩ/điều dưỡng đăng nhập vào cùng 1 danh sách bệnh nhân. Thêm/sửa bệnh nhân, tick checklist, chuyển giai đoạn ở máy này sẽ hiện ngay ở máy khác đang mở ứng dụng — không cần tải lại trang.
- **Cần đăng nhập để dùng.** Mở file `index.html` sẽ hiện màn hình đăng nhập/đăng ký trước khi thấy được danh sách bệnh nhân.
- **Đăng ký tự do (self-service).** Bất kỳ ai có link tới file này đều có thể tự tạo tài khoản bằng email + mật khẩu (không cần người quản trị duyệt trước). Đây là lựa chọn ưu tiên tốc độ triển khai nhanh cho khoa; xem mục "Lưu ý về bảo mật & phân quyền" bên dưới nếu khoa muốn siết lại.
- **Dữ liệu không còn nằm trong máy/trình duyệt của từng người** mà nằm trên hạ tầng Google Cloud (Firebase, khu vực Singapore). Khoa đã xác nhận nội dung bệnh nhân trong ứng dụng này không thuộc diện cần bảo mật nghiêm ngặt, nên đã chọn giải pháp cloud để triển khai nhanh cho nhiều người dùng chung.

### Kiến trúc tóm tắt

- **Firestore** (`patients` collection): mỗi bệnh nhân là 1 document, đồng bộ real-time tới mọi tab đang mở qua `onSnapshot()`.
- **Firebase Authentication** (Email/Password): gác cổng toàn bộ ứng dụng qua `onAuthStateChanged` — chưa đăng nhập thì không thấy giao diện chính.
- **Mã bệnh nhân (id)** được cấp phát tuần tự, an toàn khi nhiều người thêm bệnh nhân cùng lúc (dùng Firestore transaction trên document đếm `meta/counter`), tránh trùng ID.
- Toàn bộ Firebase SDK được nhúng qua CDN (`gstatic.com`), không cần cài đặt hay build gì thêm — vẫn là 1 file `index.html` mở trực tiếp bằng trình duyệt (hoặc host tĩnh, xem mục GitHub bên dưới).

### Lưu ý về bảo mật & phân quyền — khoa nên biết rõ trước khi dùng rộng rãi

- **Ai đăng nhập cũng có toàn quyền đọc/ghi mọi bệnh nhân.** Firestore rules hiện tại chỉ kiểm tra "đã đăng nhập hay chưa" (`request.auth != null`), không phân biệt vai trò (bác sĩ điều trị / điều dưỡng / khoa khác...). Nghĩa là bất kỳ tài khoản nào cũng sửa/xoá được hồ sơ của bất kỳ bệnh nhân nào.
- **Không có xác minh danh tính khi đăng ký** — ai có link cũng tự tạo được tài khoản. Nếu khoa muốn giới hạn chỉ người trong khoa dùng được, có 2 hướng đơn giản có thể nhờ làm thêm sau:
  1. Tắt đăng ký tự do, chuyển sang mô hình admin tạo tài khoản sẵn cho từng người (Firebase Console → Authentication → Users → "Add user").
  2. Giới hạn theo domain email (ví dụ chỉ cho phép `@benhvien175.vn`) ngay trong lúc đăng ký.
- **Quản lý/xoá tài khoản:** vào [Firebase Console](https://console.firebase.google.com/) → chọn project `quan-ly-benh-nhan-ho-hap` → **Authentication → Users**. Có thể xoá tài khoản (ví dụ người đã chuyển khoa/nghỉ việc) trực tiếp tại đây, không cần sửa code.
- **Xem/xoá dữ liệu bệnh nhân trực tiếp:** Firebase Console → **Firestore Database** → collection `patients`. Có thể sửa/xoá từng document tại đây nếu cần, tách biệt hoàn toàn với việc sửa qua giao diện ứng dụng.
- **`firebaseConfig` (chứa `apiKey`) hiển thị công khai trong mã nguồn `index.html` — đây KHÔNG phải là rò rỉ bí mật.** Theo thiết kế của Firebase, các key này chỉ dùng để xác định đúng project, không phải thông tin xác thực bí mật; an ninh thật sự nằm ở Firestore Rules + Authentication (đã bật). Vì vậy không cần và không nên tìm cách "giấu" `firebaseConfig` khỏi mã nguồn.
- Nếu sau này bổ sung thêm dịch vụ khác cần khoá bí mật thật (ví dụ server riêng, API key của bên thứ ba khác), **tuyệt đối không commit các khoá đó vào git** — dùng biến môi trường hoặc file cấu hình nằm ngoài git (`.gitignore`), khác với trường hợp `firebaseConfig` ở trên.

## Đưa lên GitHub

- Dùng repository **RIÊNG TƯ (private)**, không public — đây là công cụ nội bộ của khoa. (Đã đặt private.)
- **Không commit dữ liệu bệnh nhân thật vào git** dưới bất kỳ hình thức nào (không paste vào README, không lưu file `.csv`/`.json` export vào repo). Lịch sử git lưu lại vĩnh viễn — xoá file sau này không xoá được dữ liệu khỏi lịch sử. (Lưu ý: dữ liệu bệnh nhân vận hành thật hiện nằm trong Firestore, không nằm trong file `index.html` hay trong git — nên rủi ro này chủ yếu áp dụng nếu sau này có ai export CSV rồi lỡ tay commit vào repo.)
- Có thể host tĩnh bằng GitHub Pages (Settings → Pages) nếu muốn có link truy cập nhanh — với backend Firebase đã bật, host ở bất kỳ đâu (kể cả mở file `index.html` trực tiếp từ máy) đều cho cùng 1 dữ liệu dùng chung.

## Trước khi dùng chính thức, khoa nên tự rà lại thêm

- Cân nhắc siết lại đăng ký tài khoản theo hướng ở mục "Lưu ý về bảo mật & phân quyền" nếu thấy cần, thay vì để đăng ký tự do mãi mãi.
- Cân nhắc thêm phân quyền theo vai trò (ví dụ chỉ bác sĩ phụ trách mới xoá được bệnh nhân của mình) nếu quy mô dùng lớn hơn — hiện tại chưa có, mọi tài khoản ngang quyền nhau.
- Cân nhắc chính sách sao lưu (backup) định kỳ dữ liệu Firestore (Firebase có tính năng export tự động trên gói trả phí, hoặc export thủ công định kỳ qua Console ở gói miễn phí).
- Theo dõi hạn mức sử dụng miễn phí (Spark plan) của Firebase nếu số lượng bệnh nhân/thao tác tăng nhiều — xem tại Firebase Console → Usage.

## Đã sửa so với bản trước

- **Tích hợp backend Firebase (Firestore + Authentication)** — thay cho lưu trữ chỉ trong bộ nhớ trình duyệt. Dữ liệu nay dùng chung và đồng bộ real-time giữa nhiều người dùng.
- Thêm màn hình đăng nhập/đăng ký bằng email + mật khẩu, gác cổng toàn bộ ứng dụng.
- Mọi thao tác thêm/sửa bệnh nhân, tick checklist, chuyển giai đoạn, xoá bệnh nhân đều ghi trực tiếp vào Firestore và đồng bộ tới các tab/máy khác đang mở.
- Cấp mã bệnh nhân (ID) tuần tự an toàn khi nhiều người thêm bệnh nhân cùng lúc (transaction Firestore), tránh trùng ID giữa các bác sĩ.
