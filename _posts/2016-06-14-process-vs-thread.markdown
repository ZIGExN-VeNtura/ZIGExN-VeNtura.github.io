---
layout: post
title:  "Process vs Thread"
date:   2016-06-14
summary: Bài viết giới thiệu process và thread và sự khác nhau giữa hai khái niệm này.
categories: [Infrastructure]
tags: ["ruby"]
image: assets/images/process-threads.png
author: Son Dang
---

Trong lập trình bạn thường hay gặp thuật ngữ multithreading và multiprocessing. Vậy process và thread là gì? Chúng khác nhau như thế nào? Chúng ta sẽ cùng tìm hiểu hai khái niệm này trong bài viết ngày hôm nay.

### Tổng quát
Đầu tiên, để hiểu được process và thread, chúng ta cần hiểu một chương trình được xử lý như thế nào.

Giả sử chúng ta có một chương trình đơn giản là lấy 3 cộng 4 và lưu kết quả vào bộ nhớ. Chương trình này bao gồm các bước như sau:

1. Lấy số 3
2. Lấy số 4
3. Thực hiện phép tính cộng 2 số trên
4. Lưu kết quả vào bộ nhớ

Mỗi bước này sẽ được chuyển thành ngôn ngữ máy hay là mã nhị phân (binary code) để máy có thể hiểu được và xử lý. Mỗi bước đã được chuyển thành binary code được gọi là một chỉ thị (instruction). Tập hợp những instruction này được gọi là một instruction set. Khi bạn bắt đầu chạy chương trình trên, việc đầu tiên là những dòng instruction này sẽ được load lên RAM. Các dòng instruction này được chuyển qua CPU thông qua một hệ thống phụ chuyển dữ liệu (bus). Một khi chương trình được chuyển vào CPU, các instruction này sẽ được đưa vào một hàng (queue) để CPU xử lý tuần tự.

### Process
Giờ chúng ta có thể tưởng tượng instruction set này tạo thành 1 chương trình (application) và tập hợp của instruction kết hợp với nhau tạo thành một tác vụ (task) cụ thể. Khi được thực thi thì tác vụ này là một process.

![single-thread](/assets/images/single-thread.png)

Vậy process là gì? Process là quá trình hoạt động của một ứng dụng. Điều này có nghĩa là process bao gồm các dòng lệnh và trạng thái thực thi của những dòng lệnh đó. Ví dụ khi bạn click đúp vào icon Microsoft Word trên màn hình desktop để khởi động chương trình chính là bạn đã bắt đầu một process của chương trình Word. Khi process bắt đầu thì một vùng nhớ sẽ được phân phối để sử dụng trong quá trình thực thi instruction.

Hệ điều hành (OS) ngày nay cho phép phân phối nhiều process cùng một lúc lên bộ nhớ để thực hiện đồng thời nhiều tác vụ (multitasking). Thực tế thì những process này không được thực thi đồng thời mà sẽ được phân bổ qua một bộ phận của OS gọi là scheduler. Bạn có thể tưởng tượng bộ phận này như một công tắc chuyển đổi để lần lượt đưa các chương trình này vào CPU. Bộ phận này sẽ thay đổi qua lại giữa các chương trình liên tục trong khoảng thời gian rất ngắn, giả sử là 1 mili giây. Giả sử mỗi mili giây sẽ có khoảng 1000 instruction được thực thi thì đối với con người là quá nhanh và làm chúng ta tưởng như là 2 chương trình đang chạy đồng thời (concurrent).

### Thread
Đa số chương trình ngày nay hỗ trợ đa luồng (multithread). Ví dụ như một process của chương trình Word có thể có 1 luồng (thread) để nhận chỉ thị từ người dùng qua bàn phím và hiển thị lên màn hình, 1 thread để thực hiện chức năng in tài liệu ở background, 1 thread được dùng để lưu xuống ổ cứng.

Có thể tưởng tượng thread là một phiên bản nhẹ hơn của process. Một thread có thể thực hiện bất cứ tác vụ gì mà một process có thể thực hiện. Tuy vậy, do thread nằm trong process, những tác vụ của thread thực hiện thường nhỏ hơn. Điểm khác biệt cơ bản giữa thread và process là nhiều thread nằm trong cùng một process dùng một không gian bộ nhớ giống nhau, trong khi process thì được phân bổ riêng biệt. Điều này cho phép các thread đọc và viết cùng một kiểu cấu trúc và dữ liệu, giao tiếp dễ dàng giữa các thread với nhau. Việc có nhiều thead chạy song song trong cùng một process giúp multitasking nhẹ và dễ dàng hơn.

![multithreaded-process](/assets/images/multithreaded-process.png)

### Process vs. Thread
Dưới đây là bảng so sánh giữa process và thread:

|Processes|Threads|
|---------|-------|
|Sử dụng nhiều bộ nhớ |Sử dụng ít bộ nhớ hơn|
|Nếu process cha ngưng trước khi process con thoát thì process con sẽ vẫn tồn tại nhưng không hoạt động|Tất cả threads sẽ ngừng hoạt động khi process ngừng hoạt động|
|Sử dụng nhiều tài nguyên hơn vì phải OS phải lưu và reload mỗi khi chuyển đổi giữa process| Sử dụng ít tài nguyên hơn vì dùng chung không gian và bộ nhớ|
|Process được tạo mới được cung cấp cho một không gian bộ nhớ ảo (process isolation)| Thread chia sẻ bộ nhớ nên cần phải giải quyết vấn đề chia sẻ tài nguyên cho hợp lý|
|Đòi hỏi giao tiếp giữa process với nhau|Có thể giao tiếp qua queue và bộ nhớ chung|
|Tạo và xóa chậm hơn|Tạo và xóa nhanh hơn|
|Dễ dàng code và debug hơn|Có thể vô cùng phức tạp để code và debug|
|Mỗi process có code, data và cấu trúc kernel riêng biệt|Thread chia sẻ code, data và cấu trúc kernel trong cùng  1 process|


### Multithreading trong ruby

Để bắt đầu một thread, chỉ cần truyền một block khi gọi `Thread.new`. Một thread mới sẽ được tạo ra để thực thi code trong block, và thread gốc ban đầu sẽ được trả lại trạng thái hoạt động để thực thi statement tiếp theo.

{% highlight ruby %}
# Thread #1 is running here
Thread.new {
  # Thread #2 runs this code
}
# Thread #1 runs this code
{% endhighlight %}

Ví dụ về một chương trình multithreading:

{% highlight ruby %}
#!/usr/bin/ruby

def func1
   i = 0
   while i <= 2
      puts "func1 at: #{Time.now}"
      sleep(2)
      i = i+1
   end
end

def func2
   j = 0
   while j <= 2
      puts "func2 at: #{Time.now}"
      sleep(1)
      j = j + 1
   end
end

puts "Started At #{Time.now}"
t1 = Thread.new{func1()}
t2 = Thread.new{func2()}
t1.join
t2.join
puts "End at #{Time.now}"
{% endhighlight %}

Code bên trên sẽ trả về kết quả sau:

{% highlight ruby %}
Started At Wed May 16 08:22:54 -0700 2008
func1 at: Wed May 16 08:22:54 -0700 2008
func2 at: Wed May 16 08:22:54 -0700 2008
func2 at: Wed May 16 08:22:55 -0700 2008
func1 at: Wed May 16 08:22:56 -0700 2008
func2 at: Wed May 16 08:22:56 -0700 2008
func1 at: Wed May 16 08:22:58 -0700 2008
End at Wed May 16 08:23:00 -0700 2008
{% endhighlight %}

Để tìm hiểu thêm về multithreading trong ruby, bạn có thể tham khảo bài viết [Ruby - Multithreading](http://www.tutorialspoint.com/ruby/ruby_multithreading.htm)

Nguồn tham khảo về process và thread:

* [Process and Thread](http://web.stanford.edu/class/cs140/cgi-bin/lecture.php?topic=process)
* [Threads: Basic Theory and Libraries](https://www.cs.cf.ac.uk/Dave/C/node29.html#SECTION002910000000000000000)
* [Difference Between Process and Thread](https://www.youtube.com/watch?v=O3EyzlZxx3g)
