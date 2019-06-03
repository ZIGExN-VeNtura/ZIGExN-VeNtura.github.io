---
layout: post
title:  "Web server: NGINX and APACHE"
date:   2016-06-29
summary: NGINX và APACHE là 2 open source web server phổ biến nhất hiện nay. Chúng phục vụ hơn 50% lượng traffic trên mạng internet.
categories: [Infrastructure]
tags: ["web server"]
image: assets/images/nginx-vs-apache.png
featured: true

author: Kiet Nguyen
---

### I. Tổng quan

`NGINX` và `APACHE` là 2 open source web server phổ biến nhất hiện nay. Chúng phục vụ hơn 50% lượng traffic trên mạng internet.

`APACHE` HTTP server được tạo bởi Robert McCool vào năm 1995 và được tiếp tục phát triển bởi `APACHE Software Foundation` từ năm 1999. `APACHE HTTP Server` là web server phổ biến nhất từ năm 1996. Tài liệu hướng dẫn về `APACHE` rất nhiều và phổ biến. `APACHE` được nhiều người chọn là nó rất linh hoạt, phổ biến, sức mạnh và sự hỗ trợ support rộng rãi. `APACHE` có thể được mở rộng bởi các dynamic module và có thể làm việc tốt với một số lượng lớn ngôn ngữ biên dịch mà không phải kết nối với phần mềm riêng biệt.

`NGINX` được tạo bới Igor Sysoev vào năm 2002, để giải quyết vấn đề C10K. Vấn đề C10K là vấn đề nói về việc một web server có thể handle 10 ngàn kết nối đồng thời tới hệ thống, là một yêu cầu của một trang web hiện đại. `NGINX` có lợi điểm là gọn nhẹ, tốn tài nguyên ít, có thể dễ mở rộng trên nền phần cứng hạn chế. `NGINX` trội ở việc phục vụ những file tĩnh nhanh chóng, và nó được thiết kế cho việc điều hướng request động đến những phần mềm (server khác) để phục vụ đúng nhu cầu tốt hơn.
`NGINX` thường được lựa chọn bởi người admin với mục đích sử dụng hiệu quả nguồn tài nguyên và đáp ứng theo tải trọng. Những người ủng hộ `NGINX` thì họ tập trung vào tính năng core feature của web server và tính năng proxy.

### II. Kiến trúc xử lý kết nối

#### 1. `APACHE`: cung cấp nhiều module xử lý được gọi là MPMs, cơ bản có 3 loại module:

- mpm_prefork: là 1 module xử lý, nó sinh ra nhiều process con, với mỗi process có 1 thread để handle request từ client. Tại một thời điểm, mỗi process con này chỉ xử lý 1 connection. Miễn là khi số lượng request vẫn còn nhỏ hơn số lượng process con này thì module MPM này xử lý rất nhanh. Tuy nhiên về performance trong nhiều trường hợp thì không phải là sự lựa chọn tốt. Vì mỗi process chiếm một tài nguyên RAM nhất định, nên mpm_prefork rất khó scale hiệu quả.

- mpm_worker: là 1 module xử lý, sinh ra nhiều process con có thể quản lý nhiều thread. Và mỗi thread đó thì handle 1 kết nối đơn. Thread thì sẽ hiệu quả hơn process, module này sẽ scale tốt hơn mpm_prefork. Với module này, số lượng thread nhiều hơn, cũng có nghĩa là lượng connection có thể phục vụ là nhiều hơn, hiệu quả hơn.

- mpm_event: là 1 module giống với mpm_worker trong nhiều trường hợp, nhưng nó được tối ưu để xử lý connection keep-alive.
`APACHE` cung cấp nhiều kiến trúc linh hoạt, dựa trên sự khác biệt về kết nối, giải thuật xử lý request.

#### 2. `NGINX`: có ý thức hơn về những vấn đề kết nối đồng thời mà một hệ thống lớn có thể gặp phải.

`NGINX` được thiết kế để sử dụng bất đồng bộ, non-blocking, event-driven connection handling algorithm. `NGINX` sinh ra worker processes, mỗi worker sẽ handle nhiều connections (lên đến hàng ngàn). Những con Worker thực hiện được điều này bằng cách hiện thực một kiến trúc fast looping vừa kiểm tra và thực hiện những sự kiện trên.

Kiến trúc `NGINX`:

- Master process: thực thi những tác vụ như đọc config, binding ports, tạo một số lượng các process con.
Cache loader process: process này chạy lúc khởi động để nạp bộ nhớ disk cache vào memory sau đó nó sẽ exit. Process này được lên kế hoạch trước, sử dụng ít tài nguyên hệ thống.

- Cache manager process: process chạy định kỳ, để giữ cho bộ nhớ disk cache luôn đúng kích thước như trong config.

- Worker process: là  những process làm việc với connections, nó đọc và ghi nội dung vào disk, và giao tiếp với app server, handle request từ clients.

![nginx-worker](/assets/images/nginx-worker.png)

### III. So sánh giữa `APACHE` và `NGINX`:

`APACHE` và `NGINX` cả 2 đều là open source web server, được sử dụng để phục vụ cho clients các file tĩnh và nội dung động. Cả 2 đều cần phải cấu hình để tiếp nhận request từ client và routing đến đúng nơi. Điều khác biệt lớn nhất là ở cách 2 web server xử lý connection từ client. Đối với  `APACHE` thì `APACHE` sử dụng multi-processing modules (MPMs) để xử lý request từ client. Trong khi đó, đối với `NGINX`, Những kết nối và clients được xử lý bằng cách đặt chúng vào trong vòng looping, nơi chúng được xử lý bất đồng bộ.

`APACHE` sử dụng phương thức dựa trên file thông thường để xử lý các request lấy nội dung tĩnh và nội dung động bằng cách nhúng vào bộ xử lý của ngôn ngữ. `APACHE` cho phép thêm file config trên từng directory thông qua việc đánh giá và chỉ thị trong file ẩn có tên là `.htcaccess`. Trong `APACHE` các modules được load và unload tự động.

`NGINX` thì được thiết kế là web server cũng như là một proxy server.

#### 1. Serve static and dynamic content.

`APACHE` server xử lý yêu cầu static content từ client bằng phương pháp dựa trên file thông thường.

`APACHE` cũng có thể xử lý nội dung động bằng cách nhúng một bộ xử lý của ngôn ngữ trong request vào mỗi worker của mình. Điều này cho phép web server có thể tự xử lý các yêu cầu dynamic mà không cần phụ thuộc vào một ứng dụng thứ 3 nào. Bộ xử lý dynamic này được bật lên nhờ việc tự động nạp module. `APACHE` có khả năng xử lý các nội dung dynamic => việc config thiết lập cho việc này trở nên đơn giản hơn. Sự giao tiếp sẽ không cần có sự phối hợp với một thành phần khác trong phần mềm, các modules có thể dễ dàng được tháo ra nếu như nội dung request thay đổi.

`NGINX` thì không có khả năng xử lý các yêu cầu nội dung dynamic một cách tự nhiên, mà phải gửi cho một bộ phận xử lý bên ngoài xử lý hộ rồi chờ việc xử lý nội dung xong rồi mới trả về cho client. Đối với người administratos thì điều này có nghĩa là anh ta phải config giữa `NGINX` và  bộ xử lý thông qua một trong những protocol như `FastCGI`, `SCGI`, `uWSGI`, `Memcache`. Điều này có thể gây phức tạp. Tuy nhiên, phương thức này mang lại thuận lợi. Bộ xử lý dynamic không tích hợp trong worker process, chỉ khi nào xử lý yêu cầu dynamic. Còn nội dung tĩnh thì được phục vụ 1 cách trực tiếp đến client, và bộ xử lý interpreter sẽ chỉ được liên lạc khi cần thiết.

#### 2. Distributed VS Centralized Configuration.

`APACHE` cho phép một kiểu config thêm bằng file `.htaccess` . File này được nằm trong cùng thư mục source code application. Và một khi tiếp nhận 1 request, `APACHE` sẽ kiểm tra request đó trong file `.htaccess` để đưa ra việc xử lý tiếp theo. Điều này hiệu quả cho việc phân quyền trên web server, thường được sử dụng cho những mục đích như URL rewrite, restrict access, authenticate, authorization, caching policies.

File `.htaccess` mang lại những lợi ích quan trọng như sau:

- Có thể xử lý logic đối với request bất kỳ mà không cần phải reload hay restart lại web server.

- Cho phép những người không phải administrator cũng có thể tinh chỉnh (restrict access, authen, authorization) mà không phải đụng tới toàn bộ config của `APACHE`.

- Cho phép những người làm dịch vụ Hosting, có thể dễ dàng phân quyền access dễ dàng cho người sử dụng, chỉ access trong những folder của riêng mình.

- Phù hợp cho những hệ thống quản lý nội dung.

`NGINX` không có thêm config `.htaccess` như `APACHE`, cũng như là không cung cấp kiểu config dựa trên sub-directory bên ngoài config hệ thống. Điều này dẫn tới sự không linh hoạt bằng `APACHE` nhưng `NGINX` có những lợi thế riêng của mình.

`NGINX` xử lý request nhanh hơn `APACHE` do không phải xử lý file `.htaccess`.

Về vấn đề security, đối với `APACHE` việc phân mảnh config dựa trên sub-directory tỏ ra nguy hiểm, có nhiểu risk hơn vì trách nhiệm về bảo mật phụ thuộc vào từng cá nhân riêng biệt, những người có thể thay đổi `.htaccess`. Trách nhiệm về security bị phân mảnh, còn đối với `NGINX` thì nó được tập trung trong tay người administration.

#### 3. File vs URI-based interpretation. Cách mà một web server diễn giải những request và map chúng đến tài nguyên thật sự trên hệ thống server như thế nào?

`APACHE`:

`APACHE` cung cấp khả năng diễn giải một request như là một tài nguyên vật lý trên file hệ thống, hay là một đường dẫn URI mà có thể được lượng giá trừu tượng hơn. Ví dụ trong nhiều trường hợp, `APACHE` sử dụng những blocks như là <Directory> hay là <File>, hoặc <Location> cho tài nguyên trừu tượng.

Bởi vì `APACHE` được thiết kế từ sự phát triển đi lên của một Web server, mặc định nó sẽ diễn giải một request như là yêu cầu một tài nguyên hệ thống file. Nó bắt đầu bằng việc lấy đường dẫn document root và thêm vào host và port number để cố tìm file thật sự trên server. Về cơ bản, Cấu trúc hệ thống file trên server được diễn tả trên web giống như một cây thư mục.

`NGINX`:

`NGINX` thì nó vừa là web server vừa có chức năng là một proxy server. `NGINX` không có cơ chế xác định cấu hình cho thư mục filesystem, mà thay bằng việc là nó sẽ tự parse URI.

`NGINX` sử dụng khối cấu hình `server` và `location`. Khối `server` sẽ phân tích `host` của request gửi tới, còn khối `location` sẽ phân tích những thành phần đi sau `host` và `port` của URI request. Điều khác biệt với `APACHE` là `NGINX` đã giao tiếp và phân tích request dựa trên URI chứ không phải dựa trên filesystem như là `APACHE`.

Đối với những static file thì các request này được map đến cái file trên filesystem. Đầu tiên thì `NGINX` sẽ chọn ra khối `server` và `location` sẽ handle request này rồi dựa vào document root với URI rồi serve đúng file.

Điều này có vẻ giống với `APACHE`, nhưng mà `NGINX` phân tích các request chủ yếu là dựa vào URI thay thế cho vị trí filesystem. Điều này cho phép `NGINX` sẽ dễ dàng hoạt động hơn cho cả Web, Mail và Proxy server roles. `NGINX` được thiết kế sao cho dễ dàng đáp ứng được cho các mô hình request khác nhau. `NGINX` sẽ không kiểm tra sự tồn tại của filesystem cho đến khi nó sẵn sàng để phục vụ cho yêu cầu nào đó, điều này giải thích cho việc tại sao `NGINX` không hiện thực file `.htaccess` như `APACHE`.

#### 4. Modules
`APACHE` và `NGINX` đều hỗ trợ mở rộng bằng các modules.

`APACHE`: cho phép bạn có thể tự động load hoặc un-load modules trong quá trình server running. Lõi của `APACHE` luôn hiện diện, còn các modules có thể được bật tắt, thêm bớt tính năng gắn vào main server. `APACHE` sử dụng modules trong rất nhiều tác vụ của nó. Các modules có thể được sử dụng để customize chức năng core của server, ví dụ như là `mod_php` module này được tích hợp vào trong trình biên dịch PHP của mỗi worker running. Các modules này không bị giới hạn khi xử lý các nội dung động. Ngoài ra, các modules này có thể được sử dụng được sử dụng để viết lại URL, chứng thực, đăng nhập, bộ nhớ caching, compressor, proxy, rate limiting, và mã hóa. Module động có thể mở rộng các chức năng lõi của web server.

`NGINX` cũng hiện thực hóa hệ thống modules nhưng khác với `APACHE`, Modules trong `NGINX` không tự động load mà nó phải được chọn và được biên dịch vào hệ thống lõi của `NGINX`. Module của `NGINX` cũng có các chức năng như `APACHE` như caching, proxy, access limit, deny IP... Với `NGINX` thì với cách thức lựa chọn module như trên, thì bạn có thể thêm module đúng theo những gì mình cần, nâng cao tính bảo mật, sự thuận tiện, dễ bảo trì hơn so với `APACHE`. Nhưng bạn cần biết chính xác những module mà bạn cần để tích hợp vào `NGINX`.

#### 5. Sử dụng `NGINX` chung với `APACHE`

Sử dụng `NGINX` làm reverse proxy đứng trước `APACHE`. `NGINX` sẽ handle tất cả requests từ clients. Với tốc độ xử lý nhanh của `NGINX`, có thể handle được nhiều connection đồng thời.
Đối với static content, `NGINX` sẽ handle nhanh hơn và trả kết quả trực tiếp đến cho client. Còn đối với những dynamic content thì `NGINX` sẽ proxy pass đến `APACHE`, `APACHE` sẽ xử lý kết quả, trả về tương ứng. Sau đó `NGINX` sẽ trả kết quả từ `APACHE` về phía Clients.
Với cách thiết kế này, `NGINX` đóng vai trò như một máy phân loại request, giảm bớt request đến phía `APACHE`, handle nhanh hơn, giảm số lượng process và thread của `APACHE`.
Thêm nữa là, việc scale out thêm server backend khá dễ.

### IV. Kết luận

Cả `NGINX` và `APACHE` đều mạnh mẽ và linh hoạt. Việc chọn lựa web server nào tốt nhất phụ thuộc vào requirements và việc testing với kết quả mà bạn mong đợi. Việc chọn lựa phụ thuộc vào những mục tiêu mà yêu cầu đặt ra.







