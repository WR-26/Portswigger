Bộ nhớ đệm web


Bộ nhớ đệm web là một hệ thống nằm giữa máy chủ gốc và người dùng. Khi máy khách yêu cầu một tài nguyên tĩnh, yêu cầu trước tiên được chuyển đến bộ nhớ đệm. Nếu bộ nhớ đệm không chứa bản sao của tài nguyên (được gọi là lỗi bộ nhớ đệm), yêu cầu sẽ được chuyển tiếp đến máy chủ gốc, nơi xử lý và phản hồi yêu cầu. Phản hồi sau đó được gửi đến bộ nhớ đệm trước khi được gửi đến người dùng. Bộ nhớ đệm sử dụng một tập hợp các quy tắc được cấu hình sẵn để xác định xem có nên lưu trữ phản hồi hay không.

Khi có yêu cầu truy cập cùng một tài nguyên tĩnh trong tương lai, bộ nhớ đệm sẽ cung cấp bản sao đã lưu trữ của phản hồi trực tiếp cho người dùng (được gọi là truy cập thành công vào bộ nhớ đệm).

Khóa bộ nhớ đệm


Khi bộ nhớ đệm nhận được yêu cầu HTTP, nó phải quyết định xem có phản hồi được lưu trong bộ nhớ đệm mà nó có thể phục vụ trực tiếp hay không, hoặc liệu nó có phải chuyển tiếp yêu cầu đến máy chủ gốc hay không. Bộ nhớ đệm đưa ra quyết định này bằng cách tạo ra một 'khóa bộ nhớ đệm' từ các thành phần của yêu cầu HTTP. Thông thường, điều này bao gồm đường dẫn URL và các tham số truy vấn, nhưng nó cũng có thể bao gồm nhiều thành phần khác như tiêu đề và loại nội dung.

Nếu khóa bộ nhớ đệm của yêu cầu đến trùng khớp với khóa của yêu cầu trước đó, bộ nhớ đệm sẽ coi chúng là tương đương và trả về bản sao của phản hồi đã được lưu trong bộ nhớ đệm.

Ghi chú:
Để tìm hiểu cách thao tác các khóa bộ nhớ cache nhằm chèn nội dung độc hại vào bộ nhớ cache, hãy xem chủ đề "Đầu độc bộ nhớ cache web" trong Học viện của chúng tôi.

Quy tắc bộ nhớ đệm
Các quy tắc bộ nhớ đệm xác định những gì có thể được lưu vào bộ nhớ đệm và thời gian lưu trữ. Các quy tắc bộ nhớ đệm thường được thiết lập để lưu trữ các tài nguyên tĩnh, thường không thay đổi thường xuyên và được sử dụng lại trên nhiều trang. Nội dung động không được lưu vào bộ nhớ đệm vì nó có nhiều khả năng chứa thông tin nhạy cảm, đảm bảo người dùng nhận được dữ liệu mới nhất trực tiếp từ máy chủ.

Các cuộc tấn công lừa đảo bộ nhớ cache web khai thác cách thức áp dụng các quy tắc bộ nhớ cache, vì vậy điều quan trọng là phải biết về một số loại quy tắc khác nhau, đặc biệt là những quy tắc dựa trên các chuỗi được xác định trong đường dẫn URL của yêu cầu. Ví dụ:

Các quy tắc về phần mở rộng tệp tĩnh - Những quy tắc này khớp với phần mở rộng tệp của tài nguyên được yêu cầu, ví dụ như .cssđối với các tệp định kiểu CSS hoặc .jstệp JavaScript.
Quy tắc thư mục tĩnh - Các quy tắc này khớp với tất cả các đường dẫn URL bắt đầu bằng một tiền tố cụ thể. Chúng thường được sử dụng để nhắm mục tiêu vào các thư mục cụ thể chỉ chứa các tài nguyên tĩnh, ví dụ như /statichoặc /assets.
Quy tắc đặt tên tập tin - Các quy tắc này khớp các tên tập tin cụ thể với các tập tin đích cần thiết cho các hoạt động trên web và hiếm khi thay đổi, chẳng hạn như robots.txtvà favicon.ico.
Bộ nhớ đệm cũng có thể triển khai các quy tắc tùy chỉnh dựa trên các tiêu chí khác, chẳng hạn như tham số URL hoặc phân tích động.

Xây dựng một cuộc tấn công đánh lừa bộ nhớ cache web
Nhìn chung, việc xây dựng một cuộc tấn công đánh lừa bộ nhớ cache web cơ bản bao gồm các bước sau:

Xác định điểm cuối mục tiêu trả về phản hồi động chứa thông tin nhạy cảm. Xem xét các phản hồi trong Burp, vì một số thông tin nhạy cảm có thể không hiển thị trên trang được hiển thị. Tập trung vào các điểm cuối hỗ trợ các phương thức GET, HEAD, hoặc OPTIONSvì các yêu cầu thay đổi trạng thái của máy chủ gốc thường không được lưu vào bộ nhớ cache.
Xác định sự khác biệt trong cách máy chủ bộ nhớ đệm và máy chủ gốc phân tích đường dẫn URL. Sự khác biệt này có thể nằm ở cách chúng:

Ánh xạ URL đến các tài nguyên.
Xử lý các ký tự phân cách.
Chuẩn hóa đường dẫn.
Hãy tạo một URL độc hại sử dụng sự khác biệt này để đánh lừa bộ nhớ cache, khiến nó lưu trữ một phản hồi động. Khi nạn nhân truy cập URL đó, phản hồi của họ sẽ được lưu trữ trong bộ nhớ cache. Sau đó, sử dụng Burp, bạn có thể gửi yêu cầu đến cùng URL đó để lấy phản hồi đã được lưu trong bộ nhớ cache, chứa dữ liệu của nạn nhân. Tránh thực hiện việc này trực tiếp trên trình duyệt vì một số ứng dụng chuyển hướng người dùng mà không có phiên hoặc vô hiệu hóa dữ liệu cục bộ, điều này có thể che giấu lỗ hổng.
Chúng ta sẽ cùng tìm hiểu một số phương pháp khác nhau để xây dựng một cuộc tấn công đánh lừa bộ nhớ cache web.

Sử dụng bộ lọc bộ nhớ đệm
Trong quá trình kiểm tra sự khác biệt và tạo ra một công cụ khai thác đánh lừa bộ nhớ cache web, hãy đảm bảo rằng mỗi yêu cầu bạn gửi đều có một khóa cache khác nhau. Nếu không, bạn có thể nhận được các phản hồi đã được lưu trong cache, điều này sẽ ảnh hưởng đến kết quả kiểm tra của bạn.

Vì cả đường dẫn URL và bất kỳ tham số truy vấn nào thường được bao gồm trong khóa bộ nhớ cache, bạn có thể thay đổi khóa bằng cách thêm chuỗi truy vấn vào đường dẫn và thay đổi nó mỗi khi bạn gửi yêu cầu. Tự động hóa quy trình này bằng cách sử dụng tiện ích mở rộng Param Miner. Để làm điều này, sau khi bạn đã cài đặt tiện ích mở rộng, hãy nhấp vào menu Param miner > Cài đặt ở cấp cao nhất , sau đó chọn Thêm bộ xóa bộ nhớ cache động . Burp giờ đây sẽ thêm một chuỗi truy vấn duy nhất vào mỗi yêu cầu mà bạn thực hiện. Bạn có thể xem các chuỗi truy vấn đã thêm trong tab Nhật ký .

Phát hiện các phản hồi được lưu trong bộ nhớ cache
Trong quá trình kiểm thử, việc xác định được các phản hồi được lưu trong bộ nhớ cache là rất quan trọng. Để làm được điều đó, hãy xem xét các tiêu đề phản hồi và thời gian phản hồi.

Một số tiêu đề phản hồi có thể cho biết dữ liệu đã được lưu vào bộ nhớ đệm. Ví dụ:

Phần X-Cachetiêu đề cung cấp thông tin về việc phản hồi có được trả về từ bộ nhớ đệm hay không. Các giá trị điển hình bao gồm:
X-Cache: hit- Phản hồi được lấy từ bộ nhớ đệm.
X-Cache: miss- Bộ nhớ đệm không chứa phản hồi cho khóa của yêu cầu, vì vậy phản hồi đã được lấy từ máy chủ gốc. Trong hầu hết các trường hợp, phản hồi sau đó sẽ được lưu vào bộ nhớ đệm. Để xác nhận điều này, hãy gửi lại yêu cầu để xem giá trị có được cập nhật thành công hay không.
X-Cache: dynamic- Máy chủ gốc đã tạo nội dung một cách động. Điều này thường có nghĩa là phản hồi không phù hợp để lưu vào bộ nhớ đệm.
X-Cache: refresh- Nội dung được lưu trong bộ nhớ cache đã lỗi thời và cần được làm mới hoặc xác thực lại.
Phần Cache-Controltiêu đề có thể bao gồm một chỉ thị cho biết việc lưu vào bộ nhớ đệm, ví dụ như publicvới giá max-agetrị cao hơn 0. Lưu ý rằng điều này chỉ cho thấy tài nguyên có thể được lưu vào bộ nhớ đệm. Nó không phải lúc nào cũng cho thấy việc lưu vào bộ nhớ đệm, vì bộ nhớ đệm đôi khi có thể ghi đè lên tiêu đề này.
Nếu bạn nhận thấy sự khác biệt lớn về thời gian phản hồi cho cùng một yêu cầu, điều này cũng có thể cho thấy rằng phản hồi nhanh hơn được lấy từ bộ nhớ đệm.


Khai thác các quy tắc bộ nhớ đệm mở rộng tĩnh
Các quy tắc bộ nhớ đệm thường nhắm mục tiêu vào các tài nguyên tĩnh bằng cách khớp với các phần mở rộng tệp phổ biến như .csshoặc .js. Đây là hành vi mặc định trong hầu hết các CDN.

Nếu có sự khác biệt trong cách máy chủ bộ nhớ đệm và máy chủ gốc ánh xạ đường dẫn URL đến tài nguyên hoặc sử dụng dấu phân cách, kẻ tấn công có thể tạo ra yêu cầu đối với tài nguyên động với phần mở rộng tĩnh mà máy chủ gốc bỏ qua nhưng bộ nhớ đệm lại xem xét được.

Sự khác biệt trong việc lập bản đồ đường đi
Ánh xạ đường dẫn URL là quá trình liên kết các đường dẫn URL với các tài nguyên trên máy chủ, chẳng hạn như tệp, tập lệnh hoặc lệnh thực thi. Có nhiều kiểu ánh xạ khác nhau được sử dụng bởi các framework và công nghệ khác nhau. Hai kiểu phổ biến là ánh xạ URL truyền thống và ánh xạ URL RESTful.

Ánh xạ URL truyền thống thể hiện đường dẫn trực tiếp đến tài nguyên nằm trên hệ thống tập tin. Dưới đây là một ví dụ điển hình:

http://example.com/path/in/filesystem/resource.html

http://example.comtrỏ đến máy chủ.
/path/in/filesystem/Biểu thị đường dẫn thư mục trong hệ thống tệp của máy chủ.
resource.htmlĐây là tệp tin cụ thể đang được truy cập.
Ngược lại, URL kiểu REST không khớp trực tiếp với cấu trúc tệp vật lý. Chúng trừu tượng hóa đường dẫn tệp thành các phần logic của API:

http://example.com/path/resource/param1/param2

http://example.comtrỏ đến máy chủ.
/path/resource/là một điểm cuối đại diện cho một tài nguyên.
param1và param2là các tham số đường dẫn được máy chủ sử dụng để xử lý yêu cầu.

Những sai lệch trong việc lập bản đồ đường đi - Tiếp tục
Sự khác biệt trong cách bộ nhớ đệm và máy chủ gốc ánh xạ đường dẫn URL đến các tài nguyên có thể dẫn đến các lỗ hổng đánh lừa bộ nhớ đệm web. Hãy xem xét ví dụ sau:

http://example.com/user/123/profile/wcd.css

Máy chủ gốc sử dụng ánh xạ URL kiểu REST có thể hiểu đây là yêu cầu đến /user/123/profileđiểm cuối và trả về thông tin hồ sơ cho người dùng 123, bỏ qua wcd.csstham số không quan trọng.
Bộ nhớ đệm sử dụng phương pháp ánh xạ URL truyền thống có thể xem đây là yêu cầu đối với một tệp có tên wcd.cssnằm trong /profilethư mục con /user/123. Nó diễn giải đường dẫn URL là /user/123/profile/wcd.css. Nếu bộ nhớ đệm được cấu hình để lưu trữ phản hồi cho các yêu cầu có đường dẫn kết thúc bằng .css, nó sẽ lưu vào bộ nhớ đệm và phục vụ thông tin hồ sơ như thể đó là một tệp CSS.

Khai thác sự sai lệch trong việc lập bản đồ đường dẫn
Để kiểm tra cách máy chủ gốc ánh xạ đường dẫn URL đến các tài nguyên, hãy thêm một đoạn đường dẫn tùy ý vào URL của điểm cuối mục tiêu. Nếu phản hồi vẫn chứa cùng dữ liệu nhạy cảm như phản hồi cơ bản, điều đó cho thấy máy chủ gốc đã trừu tượng hóa đường dẫn URL và bỏ qua đoạn được thêm vào. Ví dụ, đây là trường hợp nếu việc sửa đổi vẫn trả /api/orders/123về /api/orders/123/foothông tin đơn hàng.

Để kiểm tra cách bộ nhớ đệm ánh xạ đường dẫn URL đến các tài nguyên, bạn cần sửa đổi đường dẫn để cố gắng khớp với quy tắc bộ nhớ đệm bằng cách thêm phần mở rộng tĩnh. Ví dụ: cập nhật /api/orders/123/foothành /api/orders/123/foo.js. Nếu phản hồi được lưu vào bộ nhớ đệm, điều này cho thấy:

Điều đó có nghĩa là bộ nhớ đệm diễn giải toàn bộ đường dẫn URL bao gồm cả phần mở rộng tĩnh.
Có một quy tắc bộ nhớ đệm để lưu trữ phản hồi cho các yêu cầu kết thúc bằng .js.
Bộ nhớ đệm có thể có các quy tắc dựa trên các phần mở rộng tĩnh cụ thể. Hãy thử một loạt các phần mở rộng, bao gồm .css, .ico, và .exe.

Sau đó, bạn có thể tạo một URL trả về phản hồi động được lưu trữ trong bộ nhớ cache. Lưu ý rằng cuộc tấn công này chỉ giới hạn ở điểm cuối cụ thể mà bạn đã thử nghiệm, vì máy chủ gốc thường có các quy tắc trừu tượng khác nhau cho các điểm cuối khác nhau.

Ghi chú
Burp Scanner tự động phát hiện các lỗ hổng đánh lừa bộ nhớ cache web do sự sai lệch trong việc ánh xạ đường dẫn trong quá trình kiểm tra. Bạn cũng có thể sử dụng ứng dụng Web Cache Deception Scanner BApp để phát hiện các bộ nhớ cache web bị cấu hình sai.

Bài thực hành: Khai thác kỹ thuật ánh xạ đường dẫn để đánh lừa bộ nhớ đệm web.
người học việc

### (PHÒNG THÍ NGHIỆM
Chưa được giải quyết
Để hoàn thành bài thực hành, hãy tìm khóa API cho người dùng carlos. Bạn có thể đăng nhập vào tài khoản của mình bằng thông tin đăng nhập sau: wiener:peter.

Kiến thức cần thiết
Để hoàn thành bài thực hành này, bạn cần biết:

Cách các điểm cuối regex ánh xạ đường dẫn URL đến tài nguyên.
Làm thế nào để phát hiện và khai thác sự khác biệt trong cách bộ nhớ đệm và máy chủ gốc ánh xạ đường dẫn URL.
Các điểm này đã được đề cập trong chủ đề "Lừa đảo bộ nhớ cache web" của chúng tôi.) ( in đậm lại để sau tôi cho 1 url writeup vào đây )

