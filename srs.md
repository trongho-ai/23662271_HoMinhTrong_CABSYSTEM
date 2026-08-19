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
