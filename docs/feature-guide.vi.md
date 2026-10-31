# Tổng quan tính năng Launchpad

Hướng dẫn này là bản tóm tắt ngắn gọn về bộ công cụ khởi động Launchpad dành cho các nhà mua hàng tiềm năng. Nó tóm tắt những gì được bao gồm, những gì yêu cầu dịch vụ bên ngoài hoặc cấu hình, và những gì bộ công cụ không cung cấp một cách tự động. Hướng dẫn tính năng đầy đủ và mã nguồn của dự án đã mua vẫn là các tài liệu tham chiếu triển khai có thẩm quyền.

## Những gì Launchpad bao gồm

Launchpad là một ứng dụng SaaS mô-đun được xây dựng trên một mã nguồn chung duy nhất. Các mô-đun tùy chọn của nó bao gồm thanh toán, trò chuyện AI, tổ chức, xuất bản, hỗ trợ, cộng đồng, thông báo, tiếp thị qua email, phân tích, giới thiệu, quảng cáo, SEO, tích hợp và lưu trữ tệp riêng tư. Trình hướng dẫn cài đặt cho phép bạn chọn các mô-đun tùy chọn cần giữ lại và mục tiêu triển khai. Các thành phần tài khoản, bảo mật, cài đặt và cơ sở dữ liệu dùng chung cốt lõi vẫn được giữ nguyên.

Trong một bản sao Git, quá trình cài đặt sử dụng sparse-checkout để các tệp được theo dõi nhưng bị loại trừ vẫn còn trong lịch sử Git. Trong một tệp ZIP đã mua, các tệp mô-đun bị loại trừ sẽ được xóa khỏi dự án đã giải nén; chúng không thể được khôi phục từ lịch sử Git vì tệp ZIP không chứa siêu dữ liệu kho lưu trữ. Đây là một lần cài đặt duy nhất, không phải là công cụ chuyển đổi mô-đun tại chỗ. Hãy sao lưu tệp ZIP đã giải nén trước khi cấu hình nếu bạn có thể muốn khôi phục mã nguồn bị loại trừ sau này.

Trình hướng dẫn ghi các mô-đun đã chọn và sở thích triển khai vào
`launchpad.config.json`, và các bí mật nhà cung cấp vào tệp `.env` bị Git bỏ qua.
Nó có thể kiểm tra thông tin xác thực của nhà cung cấp đã chọn khi bạn chủ động chọn **Kiểm tra kết nối**. Nó không tạo tài khoản nhà cung cấp, triển khai hạ tầng,
hoặc cấu hình các bảng điều khiển bên ngoài. Bạn vẫn cần hoàn thành và duy trì
các dịch vụ đó một cách độc lập.

Khu vực plugin hiển thị một bộ các thành phần được đóng gói theo danh sách cho phép; nó không
phải là một chợ gói tùy ý và không tải xuống hoặc thực thi mã plugin được tải lên. Dịch vụ chuyển tiếp sự kiện tùy chọn có thể được triển khai riêng cho sự kiện cập nhật hồ sơ tài khoản hiện được hỗ trợ. Nó không phải là một
bộ đăng ký dịch vụ chung, bus tin nhắn, hoặc hệ thống xác nhận cho từng người tiêu dùng.

## Tài khoản và bảo mật

Xác thực hỗ trợ đăng nhập bằng email và đăng nhập tùy chọn bằng Google và GitHub.
Trong môi trường phát triển, Mailpit có thể nhận email đăng nhập khi chưa cấu hình nhà cung cấp email. Môi trường triển khai cần một nhà cung cấp email cho đăng nhập bằng email và email giao dịch. Các trang và API quản trị thực thi vai trò trên máy chủ.

Xác thực đa yếu tố TOTP tùy chọn hoạt động với các ứng dụng xác thực tương thích và bao gồm các mã khôi phục dùng một lần. Cài đặt bảo mật bao gồm quản lý phiên và hoạt động kiểm toán. Bộ công cụ không bao gồm
đăng nhập dựa trên mật khẩu hoặc luồng khôi phục mật khẩu.

Danh sách kiểm tra onboarding tài khoản bao gồm cài đặt hồ sơ, bảo mật tài khoản,
và bảng điều khiển; nó là tùy chọn và có thể được xem xét lại.

## Thanh toán và quyền truy cập trả phí

Mô-đun thanh toán hỗ trợ Stripe Checkout cho các gói đăng ký đã cấu hình và
mua một lần trọn đời, cùng với Stripe Customer Portal. Xác minh mua hàng tùy chọn từ Apple
App Store và Google Play có thể cung cấp một nguồn thanh toán khác khi các sản phẩm, thông tin xác thực, tài khoản cửa hàng và tích hợp client bản địa yêu cầu được cấu hình. Các tuyến đường cửa hàng không khả dụng cho đến khi cài đặt của chúng được cung cấp; Stripe có thể tiếp tục hoạt động độc lập.

Bộ công cụ xử lý các sự kiện thanh toán và đăng ký đã xác minh, ghi lại lịch sử thanh toán, và xử lý hoàn tiền và tranh chấp. Nó **không** tự động
bảo vệ mọi tính năng trong sản phẩm của bạn chỉ vì thanh toán được bật.
`paymentBlocked` là một tín hiệu thanh toán, không phải là một hàng rào thanh toán phổ quát. Mỗi tính năng
trả phí phải thực thi quyền truy cập trên máy chủ; ẩn một nút trong trình duyệt là
không đủ. Hãy quyết định những tính năng nào là trả phí và những gì khách hàng bị hạn chế
vẫn có thể làm, sau đó triển khai và kiểm tra các quy tắc đó. Bộ công cụ cũng không
bao gồm một luồng công việc quản trị viên sẵn sàng để xác minh các thanh toán được thực hiện
bên ngoài Stripe.

Email thanh toán sử dụng nhà cung cấp email đã cấu hình, nhưng thanh toán không
yêu cầu email để hoạt động. Việc gửi email có thể bị trì hoãn, thất bại, hoặc bị
nhân đôi sau một phản hồi mạng không chắc chắn; nó không phải là một hệ thống
gửi đúng một lần.

## Email, danh sách chờ, và tiếp thị

Vận chuyển email dùng chung hỗ trợ Resend, SendGrid và SMTP. Nó được sử
dụng cho xác thực, thông báo thanh toán, thông báo, bản tin và
tin nhắn tiếp thị. Danh tính người gửi và thông tin xác thực nhà cung cấp phải được
cấu hình. Kiểm tra SMTP của trình hướng dẫn cài đặt xác thực mà không gửi
tin nhắn.

Danh sách chờ truy cập sớm được địa phương hóa yêu cầu sự đồng ý rõ ràng. Trạng thái của nó
được quản lý thủ công: mời hoặc đánh dấu một người đã tham gia không tạo
tài khoản, gửi lời mời, hoặc cấp quyền truy cập. Sự đồng ý danh sách chờ là riêng biệt
khỏi việc đăng ký bản tin.

Bản tin hỗ trợ đăng ký dựa trên sự đồng ý, liên kết hủy đăng ký và
các chiến dịch văn bản thuần túy do quản trị viên tạo. Tự động hóa tiếp thị hỗ trợ
các chuỗi email ngắn cho người đăng ký bản tin mới. Các công cụ này không phải là
trình soạn thảo email trực quan, hệ thống phân khúc đối tượng, hoặc dịch vụ tuân thủ/pháp lý. Chúng không tự động đăng ký các mục danh sách chờ hoặc gửi tin nhắn
qua thông báo đẩy và webhooks. Các chiến dịch lên lịch và tự động hóa
cũng yêu cầu một trình lên lịch bên ngoài cũng như một nhà cung cấp email đã cấu hình.

## Trợ lý AI và tổ chức

Trợ lý bảng điều khiển tùy chọn kết nối với một API văn bản tương thích OpenAI đã cấu hình. Nó lưu trữ các cuộc trò chuyện trong cơ sở dữ liệu ứng dụng, thực thi
giới hạn yêu cầu tài khoản và hỗ trợ các cuộc trò chuyện cá nhân và tổ chức.
Nó không bao gồm một mô hình, thực thi công cụ, hoặc thực hiện các hành động tự chủ.
Các lời nhắc và phản hồi được gửi đến nhà cung cấp đã cấu hình và được lưu trữ bởi
ứng dụng; hãy cấu hình cài đặt quyền riêng tư và lưu trữ của nhà cung cấp và
công bố quá trình xử lý này cho người dùng.

Các tổ chức cung cấp không gian làm việc dùng chung, vai trò thành viên và lời mời.
Quyền ủy quyền được kiểm tra dựa trên thành viên tổ chức, nhưng đây là
sở hữu chung cơ sở dữ liệu, không phải là một cơ sở dữ liệu hoặc lược đồ riêng biệt cho mỗi khách hàng.
Thanh toán, cài đặt tài khoản, hỗ trợ, thông báo, phân tích, bản tin
và nội dung xuất bản toàn cục không được tự động giới hạn phạm vi theo tổ chức.

## Xuất bản, hỗ trợ và cộng đồng

Các mô-đun riêng biệt cung cấp một blog Markdown được địa phương hóa, nguồn cập nhật phát hành và
trang CMS. Quản trị viên quản lý bản nháp và nội dung đã xuất bản. Các trang công khai
bao gồm siêu dữ liệu được địa phương hóa và hỗ trợ sitemap. Các công cụ xuất bản không
bao gồm xuất bản lên lịch, dịch thuật tự động, tải lên hình ảnh, hoặc một
hệ thống kiểm duyệt bình luận.

Mô-đun hỗ trợ cung cấp vé khách hàng và phản hồi từ quản trị viên.
Mô-đun cộng đồng cung cấp một diễn đàn thảo luận. Danh mục được tuyển chọn cho phép
quản trị viên quản lý các mục công khai. Đây là các tính năng khởi đầu chứ không phải là
tích hợp với một bàn hỗ trợ được lưu trữ hoặc một mạng xã hội đầy đủ: tệp đính kèm,
tự động hóa gán/SLA, phản hồi diễn đàn, câu trả lời được chấp nhận và hồ sơ người dùng
công khai không được bao gồm.

## Thông báo và dữ liệu khách hàng

Bảng điều khiển có một hộp thư thông báo vĩnh viễn và các kênh giao hàng tùy chọn qua email, Web
Push và webhook đã ký. Thông báo đẩy yêu cầu các khóa VAPID và đăng ký trình duyệt; người tiêu dùng webhook phải xác minh chữ ký và
loại bỏ trùng lặp sự kiện. Việc thử lại tự động cho thông báo, tiếp thị và
dọn dẹp lưu trữ cần các công việc được lên lịch bên ngoài.

Cài đặt tài khoản cung cấp một bản xuất JSON của dữ liệu tài khoản được hỗ trợ và một
liên kết riêng biệt đến quản lý thanh toán. Bản xuất loại trừ thông tin xác thực,
phiên, bí mật ký và các token nhạy cảm khác. Nó là một
tải xuống dạng luồng, không phải là một bản sao lưu cơ sở dữ liệu đầy đủ hoặc một dịch vụ lưu trữ lưu trữ nền.

## Phân tích, địa phương hóa và tùy chỉnh

Bảng điều khiển bao gồm các chế độ xem tài khoản và thanh toán. Phân tích sản phẩm tùy chọn
thu thập các sự kiện được kiểm soát bởi sự đồng ý và cung cấp các báo cáo quản trị; chúng
không phải là sự thay thế cho một nền tảng phân tích bên thứ ba. Thuộc tính,
theo dõi tiếp thị và cookie giới thiệu có thể có nghĩa vụ đồng ý trong
môi trường triển khai của bạn. Launchpad không bao gồm một nền tảng quản lý sự đồng ý.

Giao diện hỗ trợ 18 ngôn ngữ: tiếng Anh, tiếng Ả Rập, tiếng Bengal, tiếng Đức,
tiếng Tây Ban Nha, tiếng Pháp, tiếng Hindi, tiếng Indonesia, tiếng Ý, tiếng Nhật, tiếng Hàn, tiếng Hà Lan, tiếng Ba Lan,
tiếng Bồ Đào Nha, tiếng Nga, tiếng Thổ Nhĩ Kỳ, tiếng Việt và tiếng Trung. Tiếng Anh là
tài mục dự phòng và ngôn ngữ dự phòng của trình hướng dẫn cài đặt. Hãy xem xét các bản dịch
và văn bản pháp lý cho đối tượng của bạn trước khi ra mắt.

Trình xây dựng sản phẩm chỉnh sửa một bộ các phần ứng dụng theo danh sách cho phép,
điều hướng, văn bản được địa phương hóa và cài đặt thương hiệu. Nó không tạo ra
mã tùy ý, tạo các tuyến đường ứng dụng, hoặc thay đổi logic kinh doanh. Chủ đề,
chuyển động và các sở thích giao diện cá nhân khác được lưu trữ trong trình duyệt,
không được đồng bộ hóa giữa các thiết bị.

## Lưu trữ, giới thiệu, quảng cáo và SEO

Lưu trữ tệp riêng tư tùy chọn sử dụng lưu trữ đối tượng tương thích S3.
Ứng dụng xác minh loại và kích thước tệp, sử dụng các URL tải lên/tải xuống có thời hạn ngắn và kiểm tra quyền sở hữu tệp. Hãy cấu hình một bucket riêng tư và các
thông tin xác thực yêu cầu. Mô-đun không quét tệp để tìm mã độc, và việc xóa tài khoản
không tự động xóa các đối tượng từ xa.

Mô-đun giới thiệu theo dõi các liên kết đối tác và hoa hồng phái sinh từ Stripe.
Việc chi trả được thực hiện bên ngoài Launchpad. Nó không tính hoa hồng
cho các mua hàng từ App Store hoặc Google Play. Thuộc tính giới thiệu sử dụng một
cookie 30 ngày; Launchpad không có trình quản lý sự đồng ý tích hợp, vì vậy hãy đánh giá và
triển khai các yêu cầu đồng ý áp dụng.

Các công cụ quảng cáo tổ chức thông tin chiến dịch và tài khoản và nhập
báo cáo từ CSV. Chúng không kết nối với các mạng quảng cáo để tạo hoặc quản lý các
chiến dịch trực tiếp. Các tính năng SEO cung cấp siêu dữ liệu được địa phương hóa, sitemap, quy tắc robots,
dữ liệu có cấu trúc và một bản kiểm toán cơ sở dữ liệu ứng dụng; chúng không phải là một
trình thu thập web, trình theo dõi xếp hạng, công cụ nghiên cứu từ khóa, hoặc trình quản lý chuyển hướng.

Danh mục tích hợp báo cáo xem cấu hình mong đợi có hiện diện hay không.
Nó không chứng minh rằng thông tin xác thực hoạt động, tạo tài khoản nhà cung cấp, hoặc
đảm bảo việc giao hàng.

## Triển khai và trách nhiệm vận hành

Các bản triển khai yêu cầu PostgreSQL và các biến môi trường phù hợp,
bao gồm kết nối cơ sở dữ liệu và `AUTH_SECRET`. Hãy cấu hình email, Stripe,
thanh toán cửa hàng, lưu trữ đối tượng, AI, đẩy và các công việc lên lịch chỉ cho các
tính năng bạn bật. Các di trú sản xuất phải được áp dụng bằng cách sử dụng
quy trình triển khai được tài liệu hóa; giữ các môi trường xem trước trên một
cơ sở dữ liệu cách ly.

Ứng dụng bao gồm xác minh phía máy chủ, kiểm tra vai trò, các biện pháp
kiểm soát bảo mật và ghi nhật ký kiểm toán, nhưng nó không phải là một chương trình
vận hành bảo mật hoàn chỉnh. Nó không cung cấp WAF, quét mã độc, giám sát bảo mật
bên ngoài, hoặc một Chính sách Bảo mật Nội dung nghiêm ngặt. Hãy cấu hình một ingress đáng tin cậy
và các biện pháp bảo vệ lưu trữ, xem xét mô hình mối đe dọa và kiểm tra ứng dụng đã triển khai
trước khi ra mắt.

Để biết thông tin chi tiết về cấu hình, quyền sở hữu dữ liệu, giới hạn, di trú và
thủ tục vận hành, hãy xem hướng dẫn tính năng đầy đủ được cung cấp cùng với dự án.
