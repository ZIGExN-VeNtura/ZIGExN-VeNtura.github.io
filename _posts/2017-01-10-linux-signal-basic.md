---
layout: post
title: "Giới thiệu về Linux Signal"
date:   2017-01-10
summary: Trong bài viết này, ta sẽ làm quen với khái niệm signal trong Linux và cách xử lý các signal này trong Ruby.
categories: [Ruby]
tags: ["ruby", "devops"]
image: assets/images/linux-signal.png
image_hidden: true
author: Vinh Nguyen
featured: true

---

### Cơ chế chia sẽ thông tin giữa process trong Linux

Trong Linux, mỗi process sẽ có một không gian địa chỉ, vùng nhớ riêng. Điều này đồng nghĩa với việc một process không thể trực tiếp access hay thay đổi một process khác.
Tuy nhiên, cũng có trường hợp ta muốn gửi thông tin qua lại giữa các process, khi đó ta phải lựa chọn dùng một số cơ chế được cung cấp sẵn bởi Linux, một số cơ chế thường được sử dụng:

  - Pipe
  - Signal
  - Socket

Trong bài viết này, ta sẽ tìm hiểu về Signal, một trong những cơ chế thường được hay sử dụng trong việc gửi thông tin đơn giản giữa các process.

### Signal là gì?

Linux Signal là cơ chế đơn giản của Linux giúp ta gửi một số message được định nghĩa trước giữa kernel với process hoặc process với process. Kernel có thể gửi signal cho process khi có lỗi xảy ra, hoặc khi có một sự kiện nào đó(ví dụ khi user bấm Ctrl + C). Ngoài ra, giữa những process với nhau cũng có thể gửi nhận signal. Việc handle gửi/nhận giữa các process sẽ được kernel đảm nhiệm thông qua các system call([kill(2)](http://man7.org/linux/man-pages/man2/kill.2.html), [sigaction(2)](http://man7.org/linux/man-pages/man2/sigaction.2.html), [signal(7)](http://man7.org/linux/man-pages/man7/signal.7.html)). Có thể hiểu kernel sẽ giữ vai trò trung gian nhận yêu cầu gửi signal từ một process và gửi signal này đến process đích.

### Gửi một signal như thế nào?

Trong Ruby, ta dùng method `Process.kill` để gửi signal đến process khác. Ví dụ:

Mở một session irb và input như sau:

{% highlight bash %}
irb(main):001:0> Process.pid
=> 23204
{% endhighlight %}

Mở thêm một session irb mới:

{% highlight bash %}
irb(main):001:0> Process.kill(:QUIT, 23204)
{% endhighlight %}

Khi đó ta sẽ thấy session irb đầu tiên bị thoát ra. Ở trên ta đã dùng method `Process.kill` để gửi signal `QUIT`(interrupt) đến một process, khi một process nhận được signal này sẽ kết thúc thực thi và exit. Tương tự với method `Process.kill` trong Linux ta có command `kill` với cùng chức năng, ví dụ:

{% highlight bash %}
kill -s QUIT 23204
{% endhighlight %}

Trong Linux số lượng và ý nghĩa các signal thường ít thay đổi, dưới đây là danh sách một số signal thông dụng:

| Signal | Value | Comment |
|---------|------------|---------|
| SIGHUP | 1 | Hangup |
| SIGINT | 2 | Interrupt from keyboard |
| SIGQUIT | 3 | Quit from keyboard |
| SIGKILL | 9 | Kill signal |
| SIGTERM | 15 | Termination signal |
| SIGUSR1 | 30, 10, 16 | Custom signal 1 |
| SIGUSR2 | 31, 12, 17 | Custom signal 2 |

Trong table trên, một signal có thể được gọi bằng tên(như SIGINT, SIGKILL), hoặc cũng có thể gọi thông qua code của signal(1, 2, 3), do đó 2 command sau là tương đương:

{% highlight bash %}
kill -s INT 23204

kill -2 23204
{% endhighlight %}


### Handle signal

Ở trên ta đã dùng method `Process.kill`(hoặc command `kill`) để gửi signal, tương tự ta sẽ có cơ chế để xử lý khi process nhận được một signal, khi đó process có thể lựa chọn một trong các phương án:
- Bỏ qua, không xử lý signal, tiếp tục luồng xử lý của chương trình như bình thường.
- Tuân theo cơ chế mặc định(ví dụ, mặc định khi nhận được signal INT chương trình sẽ exit).
- Bắt và xử lý signal.

Trong Linux, để handle một signal ta sẽ dùng cơ chế trap, trong ví dụ sau ta sẽ viết một chương trình handle signal INT:

{% highlight ruby %}
# trap_int.rb
puts Process.pid
trap(:INT){ puts "INT signal received, ignored" }
sleep
{% endhighlight %}

Ở một session thứ 2, ta gửi signal INT cho process thứ nhất băng command `kill` như sau(hoặc có thể bấm Ctrl + C):

{% highlight bash %}
kill -s INT <pid>
{% endhighlight %}

Ta thấy, khi nhận signal INT proccess đầu tiên sẽ không thoát ra mà vẫn tiếp tục thực thi và chỉ in ra dòng thông báo "INT signal received, ignored".

### SIGKILL

Phần lớn các signal ta để có thể dùng cơ chế trap để bắt signal, tuy nhiên đối với signal SIGKILL, process sẽ không trap được, và khi nhận được signal này, process sẽ bị buộc phải exit. Signal này thường được gọi thông qua command:

{% highlight bash %}
kill -9 <pid>
{% endhighlight %}

### SIGUSR1, SIGUSR2

Ngoài các signal có ý nghĩa đã được định trước, Linux cung cấp 2 signal custom SIGUSR1 và SIGUSR2, ý nghĩa của 2 signal này như thế nào hoàn toàn do process của bạn định nghĩa. Chẳng hạn, khi dùng unicorn, nếu gửi signal SIGUSR2 đến process master, unicorn sẽ reload lại server.
