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

## Giới Thiệu Monit Là Gì?

Monit là một công cụ giám sát mã nguồn mở miễn phí cho Unix và Linux. Hiện tại mình thấy Monit khá là tiện lợi, tự động, dễ sử dụng. Nó có thể dùng để quản lý và theo dõi mọi thứ như: check programs, check files, check directories, check host, v.v...

## Đối Tượng Để Monitor

### 1. Processes

Monit theo dõi các process chạy nền như: mysql, apache, nginx, v.v.... Với Monit chúng ta có thể start service nếu nó không chạy, restart service nếu nó không phản hồi, hoặc có thể kill service khi chúng tốn quá nhiều tài nguyên như: CPU, RAM....

Cú pháp:

{% highlight ruby %}
CHECK PROCESS <unique name> <PIDFILE <path> | MATCHING <regex>>
{% endhighlight %}

### 2. Files, Directories, Filesystem

Ta cũng có thể dùng Monit để kiểm tra các thay đổi của files, thư mục như: thời gian thay đổi, dung lượng, v.v....

Cú pháp:

{% highlight ruby %}
CHECK FILE <unique name> PATH <path>
CHECK FILESYSTEM <unique name> PATH <string>
CHECK DIRECTORY <unique name> PATH <path>
{% endhighlight %}

### 3. Network connections

Bạn có thể dùng Monit để gửi dữ liệu và kiểm tra phản hồi từ Server.

Cú pháp:

{% highlight ruby %}
CHECK NETWORK <unique name> <ADDRESS <ipaddress> | INTERFACE <name>>
{% endhighlight %}

### 4. Programs

Monit có thể thực hiện 1 chương trình, hay chạy 1 script tại một số thời điểm như crontab. Ngoài ra, bạn có thể kiểm tra giá trị trả về của một chương trình hay script thực thi.

Cú pháp:

{% highlight ruby %}
CHECK PROGRAM <unique name> PATH <executable file> [TIMEOUT <number> SECONDS]
{% endhighlight %}

### 5. System

Monit cũng có thể theo dõi các tài nguyên của máy như: CPU, RAM, v.v....

Cú pháp:

{% highlight ruby %}
CHECK SYSTEM <unique name>
{% endhighlight %}

## Monit Action

Monit cung cấp ta một số action trong việc theo dõi như:

- **Alert** : gửi cảnh báo cho người dùng qua email.
- **Restart** : khởi động lại service và gửi cảnh báo.
- **Start** : khởi động service và gửi cảnh báo.
- **Stop** : dừng service và gửi cảnh báo.
- **Exec** : thực thi 1 chương trình và gửi cảnh báo.
- **Unmonitor** : ngừng theo dõi và gửi cảnh báo.

## Cài Đặt

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

##  Config Monit

Tiếp theo là config Monit, bằng cách edit file `/etc/monit.conf`
{% highlight ruby %}
vi /etc/monit.conf
{% endhighlight %}

Trong file config có nhiều setting, những setting mình đã dùng qua như:

Kiểm tra service sau mỗi 2 phút. Số ở đây mình tính bằng giây.
{% highlight ruby %}
set daemon 120            # check services at 2-minute intervals
{% endhighlight %}

Set file log của Monit để theo dõi. Mình có thể thay đổi bằng directory khác như `/var/log/monit.log`
{% highlight ruby %}
set log /var/log/monit.log
{% endhighlight %}

Set mail khi monit alert
{% highlight ruby %}
set mailserver smtp.gmail.com port 587
    username "info-monitoring@your_domain" password "your_password"
    using tlsv1
    with timeout 30 second
{% endhighlight %}

Có thể thêm email để nhận mail alert. Có thể thêm nhiều mail nhận alert
{% highlight ruby %}
set alert email_address_1
set alert email_address_2
set alert email_address_3
{% endhighlight %}

Seting email-format như sau:

{% highlight ruby %}
set mail-format {
  from: monit@your_domain
  subject: monit alert --  $EVENT
  message: $EVENT Service $SERVICE
                Date:        $DATE
                Action:      $ACTION
                Host:        $HOST
                Description: $DESCRIPTION
           Your faithful employee,
           Monit }
{% endhighlight %}

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

Tại giao diện web bạn có thể thực hiện những action như: Start service, Stop service, Restart service, Disable monitoring

![Service monitor](/assets/images/monit_web_monitor_service.png)

## Config để Monitor Service

Các bạn chú ý ở file config `/etc/monit.conf`, cuối file có include
{% highlight ruby %}
include /etc/monit.d/*
{% endhighlight %}

Nên những file config các bạn thêm vào thì mình để trong `/etc/monit.d/` để dễ quản lý

Một vài config để check như:

**Check curent disk space**

Như config dưới đây thì khi disk space sử dụng quá 80% sẽ báo mail alert

{% highlight ruby %}
check device root with path /
    if SPACE usage > 80% then alert
check device var with path /var
    if SPACE usage > 80% then alert
{% endhighlight %}

**Check server system load monitoring**

Như config dưới đây thì khi memory usage trên 95%, CPU dùng trên 95%, average load is >4 for at least 5 min
thì báo mail alert

{% highlight ruby %}
check system your-domain.com
   if memory usage > 95% for 2 cycles then alert
   if cpu usage > 95% for 2 cycles then alert
   if loadavg (5min) > 4 for 2 cycles then alert
{% endhighlight %}

**Check process**

Ví dụ như với process của nginx, ta có thể kiểm tra như: start, stop, kiểm tra nếu không tồn tại thì restart lại,
hoặc mình cũng có thể thực thi một bath script tùy ý với nội dung trong file `file_script_bath.sh`

{% highlight ruby %}
check process nginx with pidfile /run/nginx.pid
  start program = "/bin/systemctl start nginx"
  stop program = "/bin/systemctl stop nginx"
  if not exist then restart
  if not exist then exec "file_script_bath.sh" else if succeeded then exec "file_script_bath.sh"
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

Có nhiều config mẫu, các bạn có thể tham khảo thêm ở đây
[Monit Configuration Examples](https://www.mmonit.com/wiki/Monit/ConfigurationExamples){:target="_blank"}

Bài viết đến đây là hết. Còn rất nhiều nội dung khác các bạn có thể tham khảo tại đây [Monit Document](https://mmonit.com/monit/documentation/monit.html){:target="_blank"}. Hẹn các bạn ở các bài viết sau.

Chúc các bạn code zui zẻ.

### Nguồn tham khảo
1.  [https://www.itzgeek.com/how-tos/linux/centos-how-tos/monitor-and-manage-your-services-with-monit-on-centos-6-rhel-6.html](https://www.itzgeek.com/how-tos/linux/centos-how-tos/monitor-and-manage-your-services-with-monit-on-centos-6-rhel-6.html){:target="_blank"}
2. [https://mmonit.com/monit/documentation/monit.html](https://mmonit.com/monit/documentation/monit.html){:target="_blank"}
3. [https://www.mmonit.com/wiki/Monit/ConfigurationExamples](https://www.mmonit.com/wiki/Monit/ConfigurationExamples){:target="_blank"}
