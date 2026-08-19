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

# Bước 3: Business Goals (Mục tiêu kinh doanh)

| Mã số | Business Goal | Mô tả |
|---|---|---|
| **BG-01** | Tự động phân công tài xế | Hệ thống tự tìm và phân công tài xế phù hợp, không cần làm thủ công như trước. |
| **BG-02** | Theo dõi chuyến đi real-time | Khách hàng biết được tài xế nào nhận chuyến, thời gian dự kiến đến và trạng thái chuyến đi. |
| **BG-03** | Quản lý thanh toán tập trung | Hỗ trợ thanh toán bằng tiền mặt và điện tử, tích hợp với nhà cung cấp thanh toán bên ngoài. |
| **BG-04** | Mở rộng quy mô người dùng | Hệ thống phục vụ được số lượng lớn khách hàng và tài xế cùng lúc. |
| **BG-05** | Hoạt động ổn định khi tải cao | Hệ thống không bị gián đoạn khi có nhiều người dùng cùng lúc, đặc biệt giờ cao điểm. |
| **BG-06** | Triển khai tính năng mới linh hoạt | Có thể thêm tính năng mới mà không ảnh hưởng đến các chức năng đang hoạt động. |
| **BG-07** | Dễ mở rộng nghiệp vụ tương lai | Dễ dàng thêm dịch vụ mới, phương thức thanh toán mới mà không phải xây lại toàn bộ hệ thống. |
| **BG-08** | Hỗ trợ báo cáo cho ban lãnh đạo | Cung cấp số liệu về số chuyến, doanh thu, tỷ lệ hoàn thành/hủy, hiệu quả tài xế. |
| **BG-09** | Bảo mật dữ liệu và hệ thống | Xác thực người dùng, phân quyền truy cập, bảo vệ thông tin cá nhân và giao dịch. |
| **BG-10** | Mở rộng kênh thông báo | Có thể thêm các kênh thông báo mới trong tương lai mà không cần sửa lại toàn bộ hệ thống. |

# Bước 4: Xác định phạm vi (Scope)

## 1. Trong phạm vi (In-Scope)

| Nhóm chức năng | Mô tả (theo yêu cầu khách hàng) |
|---|---|
| **Quản lý tài khoản khách hàng** | Đăng ký, đăng nhập, cập nhật thông tin cá nhân |
| **Đặt xe** | Nhập điểm đón/điểm đến, chọn loại xe, gửi yêu cầu đặt xe |
| **Theo dõi chuyến đi** | Xem trạng thái tìm tài xế, tài xế nhận chuyến, thời gian dự kiến đến, trạng thái hiện tại |
| **Lịch sử & đánh giá** | Xem lịch sử chuyến đi, số tiền phải trả, đánh giá tài xế sau chuyến |
| **Quản lý tài khoản tài xế** | Đăng ký hoặc được vận hành tạo tài khoản, cập nhật hồ sơ, thông tin phương tiện, trạng thái hoạt động |
| **Nhận chuyến của tài xế** | Nhận thông báo chuyến, chấp nhận/từ chối chuyến |
| **Cập nhật trạng thái chuyến** | Đã đến điểm đón, đã đón khách, đang di chuyển, hoàn thành chuyến |
| **Vị trí tài xế** | Lưu thông tin vị trí để hỗ trợ tìm tài xế gần khách hàng, dự kiến thời gian đến |
| **Tìm kiếm & phân công tài xế** | Xác định tài xế phù hợp theo vị trí, trạng thái sẵn sàng, tiêu chí vận hành; xử lý khi tài xế không phản hồi/từ chối; thông báo khi không tìm được tài xế |
| **Tính cước** | Xác định số tiền khách hàng phải trả dựa trên loại dịch vụ và thông tin chuyến đi |
| **Thanh toán** | Hỗ trợ tiền mặt và thanh toán điện tử; tích hợp nhà cung cấp thanh toán bên ngoài; xử lý khi giao dịch thất bại |
| **Thông báo** | Gửi thông báo cho khách hàng (tiếp nhận yêu cầu, tài xế nhận chuyến, tài xế đến điểm đón, hoàn thành chuyến, kết quả thanh toán) và tài xế (chuyến mới, thay đổi liên quan chuyến đang thực hiện) |
| **Quản trị vận hành** | Quản lý khách hàng, tài xế, phương tiện, chuyến đi; xem chuyến đang diễn ra; kiểm tra trạng thái tài xế; xử lý chuyến bị lỗi; tra cứu lịch sử giao dịch; phân quyền chức năng nhạy cảm |
| **Báo cáo** | Số lượng chuyến, doanh thu, tỷ lệ chuyến hoàn thành, tỷ lệ hủy, hiệu quả hoạt động tài xế |
| **Bảo mật & xác thực** | Xác thực khách hàng/tài xế trước khi dùng chức năng cần tài khoản; kiểm soát quyền truy cập thao tác quản trị; bảo vệ thông tin cá nhân, phương tiện, vị trí, giao dịch; lưu vết thao tác quan trọng |

## 2. Ngoài phạm vi (Out-of-Scope)

| Nội dung | Lý do |
|---|---|
| Lưu trữ thông tin nhạy cảm của thẻ/tài khoản thanh toán | Doanh nghiệp yêu cầu không lưu trực tiếp trong hệ thống CAB, do nhà cung cấp thanh toán bên ngoài đảm nhiệm |
| Công thức tính cước cụ thể | Doanh nghiệp chưa chốt, cần làm rõ với các bên liên quan |
| Tiêu chí ưu tiên tài xế cụ thể | Doanh nghiệp chưa chốt |
| Thời gian tài xế phải phản hồi | Doanh nghiệp chưa chốt |
| Chính sách hủy chuyến | Doanh nghiệp chưa chốt |
| Cách xử lý khi mất kết nối mạng | Doanh nghiệp chưa chốt |
| Thời gian lưu trữ dữ liệu | Doanh nghiệp chưa chốt |

## 3. Ràng buộc phạm vi (Constraints)

| Ràng buộc | Mô tả |
|---|---|
| Thời gian | Thời gian xây dựng và triển khai sản phẩm là 7 tuần |
| Kiến trúc | Hệ thống phải có khả năng mở rộng độc lập từng thành phần, không để lỗi một chức năng làm ngừng toàn bộ hệ thống, triển khai tính năng mới từng phần |
| Mở rộng tương lai | Kiến trúc phải đủ linh hoạt để bổ sung dịch vụ mới, phương thức thanh toán mới, nhà cung cấp thông báo mới mà không cần xây dựng lại toàn bộ ứng dụng |

__GẶP KHÁCH HÀNG LẦN NỮA__


# Bước 5: Yêu cầu nghiệp vụ (Business Requirements)

## Nhóm 1: Khách hàng

| Mã BR | Tên | Yêu cầu | Diễn giải |
|---|---|---|---|
| **BR-01** | Đăng ký & đăng nhập | Hệ thống phải cho phép khách hàng đăng ký tài khoản, đăng nhập và cập nhật thông tin cá nhân | Là bước đầu tiên để khách hàng sử dụng các chức năng khác |
| **BR-02** | Đặt xe | Hệ thống phải cho phép khách hàng nhập điểm đón, điểm đến, chọn loại xe và gửi yêu cầu đặt xe | Là chức năng cốt lõi để khởi tạo một chuyến đi |
| **BR-03** | Theo dõi chuyến đi | Hệ thống phải cho khách hàng biết trạng thái tìm tài xế, tài xế đã nhận chuyến, thời gian dự kiến đến, trạng thái hiện tại của chuyến đi | Giúp khách hàng nắm tiến trình chuyến đi theo thời gian thực |
| **BR-04** | Lịch sử chuyến đi | Hệ thống phải cho phép khách hàng xem lịch sử chuyến đi và số tiền phải trả | Hỗ trợ khách hàng tra cứu lại các chuyến đã thực hiện |
| **BR-05** | Đánh giá tài xế | Hệ thống phải cho phép khách hàng đánh giá tài xế sau khi hoàn thành chuyến | Góp phần đảm bảo chất lượng dịch vụ của tài xế |

## Nhóm 2: Tài xế

| Mã BR | Tên | Yêu cầu | Diễn giải |
|---|---|---|---|
| **BR-06** | Quản lý tài khoản & hồ sơ | Hệ thống phải cho phép tài xế đăng ký hoặc được nhân viên vận hành tạo tài khoản, cập nhật hồ sơ, thông tin phương tiện, trạng thái hoạt động | Đảm bảo thông tin tài xế và phương tiện luôn được cập nhật |
| **BR-07** | Trạng thái sẵn sàng | Hệ thống phải cho phép tài xế chuyển sang trạng thái sẵn sàng nhận chuyến khi đang làm việc | Là điều kiện để hệ thống xác định tài xế có thể được phân công |
| **BR-08** | Nhận & phản hồi chuyến | Hệ thống phải gửi thông báo cho tài xế khi có yêu cầu phù hợp và cho phép chấp nhận hoặc từ chối | Là bước tài xế xác nhận tham gia thực hiện chuyến |
| **BR-09** | Cập nhật trạng thái chuyến | Hệ thống phải cho phép tài xế cập nhật trạng thái: đã đến điểm đón, đã đón khách, đang di chuyển, hoàn thành chuyến | Là cơ sở để khách hàng theo dõi chuyến đi (liên quan BR-03) |
| **BR-10** | Cập nhật vị trí | Hệ thống phải lưu thông tin vị trí của tài xế | Hỗ trợ tìm tài xế gần khách hàng và cải thiện dự kiến thời gian đến |

## Nhóm 3: Tìm & phân công tài xế

| Mã BR | Tên | Yêu cầu | Diễn giải |
|---|---|---|---|
| **BR-11** | Xác định tài xế phù hợp | Hệ thống phải xác định tài xế phù hợp dựa trên vị trí, trạng thái sẵn sàng và tiêu chí vận hành khác | Ưu tiên tài xế phù hợp và gần khách hàng |
| **BR-12** | Xử lý khi tài xế không phản hồi/từ chối | Hệ thống phải tiếp tục tìm tài xế khác khi tài xế được đề xuất không phản hồi hoặc từ chối | Không yêu cầu khách hàng phải tạo lại yêu cầu đặt xe |
| **BR-13** | Thông báo không tìm được tài xế | Hệ thống phải thông báo rõ ràng cho khách hàng khi không tìm được tài xế | Tránh để khách hàng chờ đợi mà không rõ tình trạng |

## Nhóm 4: Tính cước & thanh toán

| Mã BR | Tên | Yêu cầu | Diễn giải |
|---|---|---|---|
| **BR-14** | Tính cước | Hệ thống phải xác định số tiền khách hàng phải trả dựa trên loại dịch vụ và thông tin chuyến đi | Thực hiện sau khi chuyến đi hoàn thành |
| **BR-15** | Phương thức thanh toán | Hệ thống phải hỗ trợ thanh toán bằng tiền mặt hoặc phương thức thanh toán điện tử | Đáp ứng nhu cầu đa dạng của khách hàng |
| **BR-16** | Tích hợp thanh toán bên ngoài | Hệ thống phải tích hợp với nhà cung cấp thanh toán bên ngoài | Không lưu thông tin nhạy cảm của thẻ/tài khoản thanh toán trực tiếp trong hệ thống CAB |
| **BR-17** | Xử lý giao dịch thất bại | Hệ thống phải thông báo cho khách hàng và cho phép xử lý lại khi giao dịch điện tử thất bại | Theo chính sách xử lý của doanh nghiệp (chưa chốt chi tiết) |

## Nhóm 5: Thông báo

| Mã BR | Tên | Yêu cầu | Diễn giải |
|---|---|---|---|
| **BR-18** | Thông báo cho khách hàng | Hệ thống phải gửi thông báo khi yêu cầu được tiếp nhận, khi có tài xế nhận chuyến, khi tài xế đến điểm đón, khi chuyến hoàn thành, khi thanh toán có kết quả | Giúp khách hàng nắm đầy đủ tiến trình chuyến đi |
| **BR-19** | Thông báo cho tài xế | Hệ thống phải gửi thông báo cho tài xế về chuyến mới hoặc thay đổi liên quan chuyến đang thực hiện | Đảm bảo tài xế cập nhật kịp thời thông tin chuyến |
| **BR-20** | Mở rộng kênh thông báo | Hệ thống phải có khả năng mở rộng thêm các kênh thông báo trong tương lai | Không phải thay đổi toàn bộ hệ thống khi thêm kênh mới |

## Nhóm 6: Quản trị vận hành

| Mã BR | Tên | Yêu cầu | Diễn giải |
|---|---|---|---|
| **BR-21** | Quản lý dữ liệu vận hành | Hệ thống phải cho phép nhân viên vận hành quản lý khách hàng, tài xế, phương tiện và chuyến đi | Thông qua giao diện quản trị |
| **BR-22** | Giám sát & xử lý sự cố | Hệ thống phải cho phép nhân viên xem chuyến đang diễn ra, kiểm tra trạng thái tài xế, xử lý chuyến bị lỗi | Hỗ trợ vận hành xử lý sự cố kịp thời |
| **BR-23** | Tra cứu lịch sử giao dịch | Hệ thống phải cho phép nhân viên tra cứu lịch sử giao dịch | Phục vụ công tác đối soát, kiểm tra |
| **BR-24** | Phân quyền chức năng | Hệ thống phải phân quyền một số chức năng quản trị | Nhân viên thông thường không thể thực hiện thao tác nhạy cảm |
| **BR-25** | Báo cáo | Hệ thống phải cung cấp báo cáo về số lượng chuyến, doanh thu, tỷ lệ hoàn thành, tỷ lệ hủy, hiệu quả tài xế | Phục vụ ban lãnh đạo ra quyết định |

## Nhóm 7: Vận hành hệ thống & khả năng mở rộng

| Mã BR | Tên | Yêu cầu | Diễn giải |
|---|---|---|---|
| **BR-26** | Hoạt động ổn định khi tải cao | Hệ thống phải hoạt động ổn định vào các thời điểm nhu cầu tăng cao | Đảm bảo trải nghiệm không gián đoạn ở giờ cao điểm |
| **BR-27** | Cô lập lỗi giữa các chức năng | Lỗi ở chức năng thanh toán hoặc thông báo không được làm toàn bộ hệ thống ngừng hoạt động | Tránh một điểm lỗi ảnh hưởng toàn hệ thống |
| **BR-28** | Mở rộng độc lập | Các thành phần hệ thống phải có khả năng mở rộng độc lập khi tải tăng | Hỗ trợ khả năng chịu tải linh hoạt theo từng thành phần |
| **BR-29** | Triển khai từng phần | Chức năng mới phải có thể được triển khai từng phần | Hạn chế ảnh hưởng đến các chức năng đang hoạt động |
| **BR-30** | Mở rộng nghiệp vụ tương lai | Kiến trúc phải đủ linh hoạt để bổ sung dịch vụ mới, phương thức thanh toán mới, nhà cung cấp thông báo mới | Không phải xây dựng lại toàn bộ ứng dụng |

## Nhóm 8: Bảo mật

| Mã BR | Tên | Yêu cầu | Diễn giải |
|---|---|---|---|
| **BR-31** | Xác thực người dùng | Khách hàng và tài xế phải được xác thực trước khi sử dụng chức năng yêu cầu tài khoản | Đảm bảo chỉ người dùng hợp lệ mới truy cập được chức năng |
| **BR-32** | Kiểm soát truy cập quản trị | Các thao tác quản trị phải được kiểm soát quyền truy cập | Ngăn chặn truy cập trái phép vào chức năng quản trị |
| **BR-33** | Bảo vệ dữ liệu | Hệ thống phải bảo vệ thông tin cá nhân, thông tin phương tiện, dữ liệu vị trí và dữ liệu giao dịch | Đảm bảo an toàn cho các loại dữ liệu nhạy cảm |
| **BR-34** | Lưu vết thao tác | Hệ thống phải lưu vết các thao tác quan trọng | Phục vụ kiểm tra khi có sự cố xảy ra |

# Bước 6: Business Process (Quy trình nghiệp vụ - Dạng Swimlane)

| Bước   | Business Process    | Mô tả                                                                           |
| ------ | ------------------- | ------------------------------------------------------------------------------- |
| *1*  | *Create Booking*  | Customer nhập điểm đón, điểm đến, loại xe và gửi yêu cầu                        |
| *2*  | *Find Driver*     | Hệ thống tìm Driver phù hợp dựa trên vị trí và trạng thái sẵn sàng              |
| *3*  | *Accept/Reject*   | Driver nhận thông báo và Accept hoặc Reject chuyến                              |
| *4*  | *Assign Driver*   | Driver chấp nhận → hệ thống phân công; từ chối/không phản hồi → tìm Driver khác |
| *5*  | *Pickup Customer* | Driver đến điểm đón và cập nhật trạng thái                                      |
| *6*  | *Execute Trip*    | Driver đón khách và thực hiện chuyến đi                                         |
| *7*  | *Complete Trip*   | Driver cập nhật chuyến đã hoàn thành                                            |
| *8*  | *Calculate Fare*  | Hệ thống tính số tiền Customer phải trả                                         |
| *9*  | *Payment*         | Customer thanh toán bằng Cash hoặc Electronic Payment                           |
| *10* | *Rating*          | Customer đánh giá Driver sau chuyến                                             |
| *11* | *Record & Report* | Hệ thống lưu lịch sử và cung cấp dữ liệu phục vụ quản lý/báo cáo                |

# Bước 7: Vẽ sơ đồ bằng Mermaid


# Bước 8: Yêu cầu chức năng (Functional Requirements - FR)

## Nhóm 1: Ứng dụng Khách hàng (Customer App)

| Mã FR | Tên chức năng | Mô tả chi tiết | Map với BR |
|---|---|---|---|
| **FR-CUS-01** | Đăng ký / Đăng nhập | Cho phép khách hàng tạo tài khoản qua SĐT/Email, đăng nhập, và xác thực bằng OTP/Mật khẩu. | BR-01, BR-31 |
| **FR-CUS-02** | Quản lý hồ sơ | Cho phép xem, chỉnh sửa thông tin cá nhân (Tên, SĐT, Email). | BR-01 |
| **FR-CUS-03** | Chọn điểm đón/đến | Cung cấp giao diện bản đồ, ô tìm kiếm địa chỉ để nhập tọa độ điểm đón và điểm đến. | BR-02 |
| **FR-CUS-04** | Chọn loại xe & Xem giá | Hiển thị danh sách các loại xe (VD: 4 chỗ, 7 chỗ, xe máy) kèm giá cước dự kiến. | BR-02, BR-14 |
| **FR-CUS-05** | Tracking Real-time | Hiển thị bản đồ theo dõi vị trí thực tế của tài xế và thời gian dự kiến đến (ETA). | BR-03 |
| **FR-CUS-06** | Chọn phương thức thanh toán | Cho phép chọn thanh toán Tiền mặt hoặc Thẻ/Ví điện tử trước hoặc sau chuyến. | BR-15 |
| **FR-CUS-07** | Lịch sử chuyến đi | Hiển thị danh sách các chuyến đi đã hoàn thành/đã hủy kèm chi tiết lộ trình, cước phí. | BR-04 |
| **FR-CUS-08** | Đánh giá & Phản hồi | Cung cấp giao diện chấm điểm (1-5 sao) và ghi chú nhận xét tài xế sau chuyến. | BR-05 |

## Nhóm 2: Ứng dụng Tài xế (Driver App)

| Mã FR | Tên chức năng | Mô tả chi tiết | Map với BR |
|---|---|---|---|
| **FR-DRV-01** | Đăng nhập & Hồ sơ | Đăng nhập tài khoản, xem thông tin cá nhân và phương tiện đang điều khiển. | BR-06, BR-31 |
| **FR-DRV-02** | Bật/Tắt trạng thái hoạt động | Nút gạt chuyển đổi trạng thái (Online/Sẵn sàng - Offline/Nghỉ ngơi). | BR-07 |
| **FR-DRV-03** | Màn hình nhận chuyến | Hiển thị pop-up thông tin chuyến mới (điểm đón, điểm đến, giá) kèm nút Chấp nhận/Từ chối với đồng hồ đếm ngược. | BR-08 |
| **FR-DRV-04** | Cập nhật luồng trạng thái chuyến | Cung cấp các nút bấm tuần tự: "Đã đến điểm đón" -> "Đã đón khách" -> "Hoàn thành chuyến". | BR-09 |
| **FR-DRV-05** | Truyền tải GPS (Background) | Ứng dụng liên tục lấy tọa độ GPS của thiết bị và gửi về Server theo chu kỳ (VD: 3s/lần). | BR-10 |

## Nhóm 3: Hệ thống cốt lõi & API (Core Backend)

| Mã FR | Tên chức năng | Mô tả chi tiết | Map với BR |
|---|---|---|---|
| **FR-SYS-01** | Thuật toán Dispatching | Tìm tài xế có trạng thái Online, tính toán khoảng cách ngắn nhất đến khách hàng và phát yêu cầu. | BR-11 |
| **FR-SYS-02** | Xử lý vòng lặp Dispatching | Nếu tài xế từ chối/hết giờ đếm ngược, tự động đẩy yêu cầu cho tài xế phù hợp tiếp theo. | BR-12 |
| **FR-SYS-03** | Xử lý Timeout chuyến đi | Nếu không tìm được tài xế trong thời gian X phút, tự động hủy và gửi thông báo cho khách. | BR-13 |
| **FR-SYS-04** | Tính cước cuối cùng | Dựa vào công thức (đang chờ chốt) để chốt số tiền thực tế sau khi chuyến đi hoàn thành. | BR-14 |
| **FR-SYS-05** | Tích hợp Cổng thanh toán | Gọi API nhà cung cấp bên thứ 3 để xử lý trừ tiền. Trả về kết quả Thành công/Thất bại. | BR-16, BR-17 |
| **FR-SYS-06** | Engine Thông báo (Push Notification) | Quản lý và đẩy thông báo In-app cho khách hàng và tài xế theo từng sự kiện của chuyến đi. | BR-18, BR-19, BR-20 |

## Nhóm 4: Giao diện Quản trị vận hành (Admin/Web Portal)

| Mã FR | Tên chức năng | Mô tả chi tiết | Map với BR |
|---|---|---|---|
| **FR-ADM-01** | Quản lý User (Khách/Tài xế) | CRUD (Tạo, Xem, Sửa, Khóa) tài khoản khách hàng, tài xế và duyệt hồ sơ phương tiện. | BR-21 |
| **FR-ADM-02** | Dashboard Giám sát Real-time | Bản đồ/Bảng điều khiển hiển thị tổng quan các chuyến đi đang diễn ra và trạng thái các tài xế. | BR-22 |
| **FR-ADM-03** | Tra cứu & Đối soát | Màn hình tìm kiếm lịch sử chuyến đi và lịch sử giao dịch thanh toán. | BR-23 |
| **FR-ADM-04** | Quản lý phân quyền (RBAC) | Tạo Role (Quản trị viên, Nhân viên CSKH) và gán quyền thao tác cụ thể cho từng Role. | BR-24, BR-32 |
| **FR-ADM-05** | Trích xuất báo cáo | Xuất các biểu đồ/file Excel về: số chuyến, doanh thu, tỷ lệ hủy, hiệu suất tài xế theo khoảng thời gian. | BR-25 |
| **FR-ADM-06** | System Log | Ghi nhận tự động (Lưu vết) thao tác của Admin và các lỗi hệ thống nghiêm trọng. | BR-33, BR-34 |
