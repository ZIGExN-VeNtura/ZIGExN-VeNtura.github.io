---
layout: post
title: Theo dõi và quản lý Service với Monit trên CentOS
date:   2020-07-23
summary: Giới thiệu Monit là gì? Hướng dẫn cài đặt và tạo một ví dụ quản lý process trên CentOS
categories: [Monitor Tools]
tags: [“Linux”, “CentOS”]
image_hidden: true
author: Anh Dep Trai

---

### Giới Thiệu

Xin chào các bạn! Hôm nay mình xin chia sẻ với các bạn một cái hay vừa mới tìm hiểu được. Bài viết này chỉ là những cái cơ bản nhất. Còn cao hơn thì mình sẽ tìm hiểu và cập nhật thêm ở những bài viết sau.

Cái mà mình giới thiệu hôm nay là gì? Đó là [Monit](https://en.wikipedia.org/wiki/Monit).
Monit là một công cụ giám sát mã nguồn mở miễn phí cho Unix và Linux. Do mình mới dùng nên về phần so sánh với các monitor khác thì mình sẽ tìm hiểu thêm. Hiện tại mình thấy Monit khá là tiện lợi, tự động, dễ sử dụng. Nó có thể dùng để quản lý và theo dõi mọi thứ như: check programs, check files, check directories, check host...

### Cài Đặt

Để cài đặt ta dùng lệnh
{% highlight ruby %}
yum -y install monit
{% endhighlight %}

Sau đó start Monit với lệnh:
{% highlight ruby %}
monit
{% endhighlight %}

Kiểm tra trạng thái của Monit:
{% highlight ruby %}
monit status
{% endhighlight %}

Chú ý: các bạn muốn cho Monit nó tự động start khi restart lại máy thì dùng lệnh sau:

(ví dụ trên CentOS 6)
{% highlight ruby %}
service monit enable
{% endhighlight %}

Kiểm tra xem đã start cùng với máy chưa
{% highlight ruby %}
chkconfig --list | grep monit
{% endhighlight %}

![Monit check config](/assets/images/monit_chkconfig.png)

###  Config Monit

Tiếp theo là config Monit, bằng cách edit file `/etc/monit.conf`
{% highlight ruby %}
vi /etc/monit.conf
{% endhighlight %}

Trong file config có nhiều setting, những setting mình đã dùng qua như:

{% highlight ruby %}
set daemon  120
{% endhighlight %}
Kiểm tra service sau mỗi 2 phút. Số ở đây mình tính bằng giây.

{% highlight ruby %}
set log /var/log/monit.log
{% endhighlight %}
Set file log của Monit để theo dõi. Mình có thể thay đổi bằng directory khác như `/var/log/monit.log`

###  Giao Diện WEB của Monit
Mình cũng setting trong file `/etc/monit.conf`
{% highlight ruby %}
vi /etc/monit.conf
{% endhighlight %}

{% highlight ruby %}
set httpd port 2812
  allow 0.0.0.0/0.0.0.0
  allow admin:monit
{% endhighlight %}

Như config trên thì Monit sẽ lắng nghe ở port 2812 với `user là admin` và `pass là monit`

Reload lại Monit với lệnh
{% highlight ruby %}
/etc/init.d/monit restart
{% endhighlight %}

![Monit web interface](/assets/images/monit_web_interface.png)

### Config để Monitor Service

Các bạn chú ý ở file config `/etc/monit.conf`, cuối file có include
{% highlight ruby %}
include /etc/monit.d/*
{% endhighlight %}

Nên những file config các bạn thêm vào thì mình để trong `/etc/monit.d/` để dễ quản lý

Ví dụ như config cho `sshd`.
Đầu tiên mình tạo 1 file config `etc/monit.d/sshd_monitor` với nội dung sau:

{% highlight ruby %}
check process sshd with pidfile /var/run/sshd.pid
  start program  "/etc/init.d/sshd start"
  stop program  "/etc/init.d/sshd stop"
  if failed port 22 protocol ssh then restart
{% endhighlight %}

Sau khi thêm config thì mình test lại với lệnh:
{% highlight ruby %}
monit -t
{% endhighlight %}

Kết quả:
{% highlight ruby %}
Control file syntax OK
{% endhighlight %}

Tiếp theo là restart lại Monit để apply config mới
{% highlight ruby %}
/etc/init.d/monit restart
{% endhighlight %}

![Monit sshd Monitor](/assets/images/monit_sshd.png)

### Test
Mình sẽ stop process lại với lệnh
{% highlight ruby %}
/etc/init.d/sshd stop
{% endhighlight %}

Sau thời gian mặc định là 120s thì monit sẽ start lại. Các bạn có thể vào log Monit đễ kiểm tra.

Bài viết đến đây là hết. Còn rất nhiều nội dung khác các bạn có thể tham khảo tại đây [Monit Document](https://mmonit.com/monit/documentation/monit.html).

Hẹn các bạn ở các bài viết sau.
Chúc các bạn code zui zẻ.

### Nguồn tham khảo
1.  [https://en.wikipedia.org/wiki/Monit](https://en.wikipedia.org/wiki/Monit)
2.  [https://www.itzgeek.com/how-tos/linux/centos-how-tos/monitor-and-manage-your-services-with-monit-on-centos-6-rhel-6.html](https://www.itzgeek.com/how-tos/linux/centos-how-tos/monitor-and-manage-your-services-with-monit-on-centos-6-rhel-6.html)
3. [https://mmonit.com/monit/documentation/monit.html](https://mmonit.com/monit/documentation/monit.html)
