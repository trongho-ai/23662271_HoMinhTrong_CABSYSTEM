# Bước 1: Đọc và phân tích yêu cầu — Business Context & Business Problem

## Câu hỏi 1: Khách hàng muốn giải quyết vấn đề gì?

Công ty ABC muốn thay thế hệ thống đặt xe thủ công (tổng đài + app đơn giản) bằng một **nền tảng CAB tự động hóa**, giải quyết 4 vấn đề cụ thể:

1. Phân công tài xế đang làm **thủ công** → cần tự động matching
2. Khách hàng **không theo dõi được** trạng thái chuyến đi → cần real-time tracking
3. Thông tin thanh toán **chưa quản lý tập trung** → cần hệ thống thanh toán tập trung, tích hợp bên thứ 3
4. Hệ thống hiện tại **khó mở rộng** → cần kiến trúc linh hoạt, scale được

## Câu hỏi 2: Vì sao hệ thống hiện tại không thể đáp ứng?

- Hệ thống ban đầu chỉ thiết kế cho quy mô nhỏ (tổng đài + app đơn giản), **không có thuật toán/cơ chế tự động** để ghép tài xế với khách hàng.
- Không có module quản lý thanh toán tập trung → dữ liệu giao dịch phân tán, khó đối soát, khó báo cáo.
- Kiến trúc hiện tại (ngầm hiểu là monolithic/rời rạc) **không tách rời được từng thành phần**, nên khi một chức năng (VD thanh toán) lỗi thì ảnh hưởng cả hệ thống, và khó mở rộng độc lập từng phần khi tải tăng.
- Không có cơ chế realtime (thông báo, cập nhật trạng thái) để khách hàng/tài xế nắm được tình trạng chuyến đi.

## Câu hỏi 3: Ai sử dụng hệ thống này?

| Nhóm | Vai trò chính |
|---|---|
| **Khách hàng** | Đặt xe, theo dõi chuyến, thanh toán, đánh giá tài xế |
| **Tài xế** | Nhận/từ chối chuyến, cập nhật trạng thái & vị trí |
| **Nhân viên vận hành** | Quản lý khách hàng/tài xế/phương tiện/chuyến đi, xử lý sự cố |
| **Ban lãnh đạo** *(gián tiếp, qua báo cáo)* | Xem báo cáo doanh thu, tỷ lệ hoàn thành/hủy, hiệu quả tài xế để ra quyết định |

## Câu hỏi 4: Giá trị sau khi tạo ra hệ thống là gì?

- **Khách hàng:** trải nghiệm tốt hơn — đặt xe nhanh, biết chính xác trạng thái chuyến đi, thanh toán linh hoạt (tiền mặt/điện tử)
- **Tài xế:** nhận chuyến minh bạch, công bằng, thao tác đơn giản trên app
- **Vận hành/Doanh nghiệp:** giảm công sức xử lý thủ công, có dữ liệu tập trung để giám sát & ra quyết định (báo cáo số chuyến, doanh thu, tỷ lệ hủy...)
- **Về mặt kỹ thuật/dài hạn:** hệ thống chịu tải tốt hơn giờ cao điểm, các thành phần lỗi không kéo sập toàn hệ thống, dễ dàng bổ sung dịch vụ/phương thức thanh toán mới mà không phải xây lại từ đầu

# Bước 2: Ma trận Stakeholder

## Bảng Stakeholder

| Stakeholder | Vai trò / Được làm gì |
|---|---|
| **Ban lãnh đạo** | Kỳ vọng hệ thống phục vụ số lượng lớn khách hàng và tài xế, phát triển thêm tính năng tương lai; nhận báo cáo số lượng chuyến, doanh thu, tỷ lệ hoàn thành, tỷ lệ hủy, hiệu quả hoạt động tài xế. |
| **Khách hàng** | Đăng ký, đăng nhập, cập nhật thông tin cá nhân, đặt xe, theo dõi chuyến đi, xem lịch sử, thanh toán, đánh giá tài xế, nhận thông báo về chuyến. |
| **Tài xế** | Đăng ký/được tạo tài khoản, cập nhật hồ sơ và phương tiện, bật trạng thái sẵn sàng, nhận thông báo chuyến, chấp nhận/từ chối chuyến, cập nhật trạng thái và vị trí trong chuyến. |
| **Nhân viên vận hành** | Quản lý khách hàng, tài xế, phương tiện, chuyến đi; xem chuyến đang diễn ra, xử lý sự cố, tra cứu lịch sử giao dịch; một số chức năng được phân quyền. |
| **Nhà cung cấp thanh toán bên ngoài** | Được tích hợp để xử lý thanh toán điện tử; đảm nhiệm lưu trữ thông tin thẻ/tài khoản thanh toán thay vì lưu trong hệ thống CAB. |

## Ma trận mức độ ảnh hưởng (Power) và mức độ quan tâm (Interest)

| Stakeholder | Power | Interest | Căn cứ |
|---|---|---|---|
| **Ban lãnh đạo** | Cao | Cao | "Ban lãnh đạo kỳ vọng hệ thống mới hỗ trợ ít nhất ba nhóm người dùng chính..."; là bên nhận báo cáo doanh thu, tỷ lệ hoàn thành/hủy, hiệu quả tài xế. |
| **Nhân viên vận hành** | Cao | Cao | Trực tiếp "quản lý khách hàng, tài xế, phương tiện và chuyến đi", "hỗ trợ xử lý các trường hợp chuyến bị lỗi". |
| **Khách hàng** | Thấp | Cao | Là người trực tiếp "gửi yêu cầu đặt xe và theo dõi chuyến đi", chịu ảnh hưởng trực tiếp bởi chất lượng hệ thống nhưng không có vai trò quyết định trong đề bài. |
| **Tài xế** | Thấp | Cao | Trực tiếp "nhận được thông báo và có thể chấp nhận hoặc từ chối chuyến", "cập nhật trạng thái" trong suốt chuyến đi. |
| **Nhà cung cấp thanh toán bên ngoài** | Trung bình | Trung bình | "Doanh nghiệp muốn tích hợp với một nhà cung cấp thanh toán bên ngoài" nhưng không tham gia vận hành nghiệp vụ hàng ngày. |

## Ma trận Power/Interest

|  | **Interest Thấp** | **Interest Cao** |
|---|---|---|
| **Power Cao** | *(không có)* | Ban lãnh đạo, Nhân viên vận hành |
| **Power Thấp/Trung bình** | Nhà cung cấp thanh toán | Khách hàng, Tài xế |
