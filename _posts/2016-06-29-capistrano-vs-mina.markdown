---
layout: post
title:  "Capistrano VS Mina"
date:   2016-06-29
summary: Sự khác biệt giữa Capistrano và Mina trong việc Deploy Rails Application
categories: [Rails]
tags: ["deployment"]
image: images/mina.jpg
author: Kiet Nguyen
---

### I. Tổng quát

Switching from Capistrano to Mina for Rails Deployment

Tất cả dịch vụ web đều cần phải được deploy lên một server hay vps nào đó. Nếu bạn sử dụng những nền tảng PaaS như Heroku, AppFog thì bạn không cần quan tâm đến vấn đề deployment vì PaaS đã làm giúp bạn, còn nếu một ruby on rails app đơn thuần, để auto deploy thì nhiều developer trên khắp thế giới đã và đang sử dụng Capistrano nó là một tool giúp chúng ta auto-deployment rất thuận tiện và mạnh mẽ. Tôi cũng đã sử dụng Capistrano trong nhiều dự án.

Việc deploy bằng Capistrano thì dễ dàng, thuận tiện nhưng về mặt thời gian thì khá lâu, đặc biệt đối với những tác vụ như `rake assets:precompile`. Và với việc tool Mina ra đời đã không những giải quyết vấn đề về mặt thời gian của Capistrano mà việc config deploy càng dễ dàng hơn Capistrano.

Capistrano gặp bất lợi khá lớn nếu như dự án hay ứng dụng có tốc độ phát triển nhanh, cần phải được deploy hằng ngày thì Capistrano mất rất nhiều thời gian. Very Slow.

So sánh giữa Capistrano và Mina đê thấy rõ hơn tại sao cần phải từ bỏ Capistrano và Sử dụng Mina cho việc auto-deployment

### II. Capistrano Deploy

Để sử dụng capistrano cho Rails thì cần phải:

- Add gem 'capistrano' and gem 'capistrano-rails' to Gemfile.

- Run `cap install`

- edit Capfile, config/deploy.rb and config/deploy/production.rb

- Run `cap production deploy`

Example with Capistrano:

{% highlight ruby %}
INFO [07036a87] Running /usr/bin/env mkdir -p /tmp/capistrano/ on capistrano.gabrijelskoro.com
DEBUG [07036a87] Command: /usr/bin/env mkdir -p /tmp/capistrano/
INFO [07036a87] Finished in 1.236 seconds with exit status 0 (successful).
DEBUG Uploading /tmp/capistrano/git-ssh.sh 0.0%
INFO Uploading /tmp/capistrano/git-ssh.sh 100.0%
INFO [b1b91149] Running /usr/bin/env chmod +x /tmp/capistrano/git-ssh.sh on capistrano.gabrijelskoro.com
DEBUG [b1b91149] Command: /usr/bin/env chmod +x /tmp/capistrano/git-ssh.sh
INFO [b1b91149] Finished in 0.172 seconds with exit status 0 (successful).
...
INFO [9a6ef5d6] Running /usr/bin/env echo "Branch master deployed as release 20131206155724 by gabrijelskoro; " >> /var/www/c/capistrano.gabrijelskoro.com/revisions.log on capistrano.gabrijelskoro.com
DEBUG [9a6ef5d6] Command: echo "Branch master deployed as release 20131206155724 by gabrijelskoro; " >> /var/www/c/capistrano.gabrijelskoro.com/revisions.log
INFO [9a6ef5d6] Finished in 0.228 seconds with exit status 0 (successful).
{% endhighlight %}

### III. Mina Deploy

Tương tự:

- Add gem 'mina' to Gemfile.

- Run `mina init`

- Edit config/deploy.rb file.

- Run `mina setup`

- Run `mina deploy`

Example with Mina:

{% highlight ruby %}
-----> Creating a temporary build path
-----> Cloning the Git repository
       Cloning into bare repository '/var/www/m/mina.gabrijelskoro.com/scm'...
-----> Using git branch 'master'
       Cloning into '.'...
       done.
-----> Using this git commit

       Gabrijel Skoro (f78408d):
       > initial commit

-----> Symlinking shared paths
-----> Installing gem dependencies using Bundler
       Fetching gem metadata from https://rubygems.org/..........
       Fetching gem metadata from https://rubygems.org/..
       Installing rake (10.1.0)
       Installing i18n (0.6.9)
       ...
-----> Migrating database
-----> Precompiling asset files
-----> Build finished
-----> Moving build to releases/1
-----> Updating the current symlink
-----> Launching
-----> Done. Deployed v1
       Elapsed time: 85.00 seconds
{% endhighlight %}

Có một điều khác biệt giữa việc deploy lần đầu và deploy những lần tiếp theo cho việc auto-deployment cho Rails Application nói chung.
Với lần đầu tiên deploy, thì thời gian sẽ lâu hơn những lần tiếp sau, vì lần đầu server cần phải thực hiện khá nhiều nhiệm vụ của Rails:

- Khởi tạo database

- Chạy migration

- Chạy bundle để download những thư viện và gem cần thiết phụ thuộc Gemfile

- Chạy tác vụ assets precompiles

Những tác vụ này mất khá nhiều thời gian, nó phụ thuộc vào đường truyền mạng và độ lớn của ứng dụng.

Nhưng từ lần deploy thứ 2 trở đi việc deploy sẽ nhanh hơn nhiều.

Một ví dụ về tốc độ giữa Capistrano và Mina trong việc deploy:

Đối với một project nhỏ: Deploy lần đầu tiên Capistrano mâts khoảng 55s thì Mina chỉ mất có 9s đê deploy xong. Rất nhanh. Mina nhanh hơn Capistrano gần 8 lần

![mina-faster-7xtime](/images/mina.jpg)

Còn đối với project lớn: Mina nhanh hơn Capistrano gần 28 lần. Great!

Deploy lần đầu tiên Capistrano mất 15 phút thì Mina mất 11 phút. Deploy lần 2 trở đi thì Capistrano mất 140s trong khi Mina mất chỉ 5s

![mina-faster-7xtime](/images/mina2.jpg)

### IV. Tại sao Mina lại nhanh hơn Capistrano ?

Capistrano thực hiện từng dòng lệnh trong mỗi ssh session của chúng, nghĩa là từng dòng lệnh chạy ở máy local, muốn thực thi được thì trước hết phải tạo kết nối thông qua SSH, rồi mỗi dòng lệnh đó sẽ thực thi trên server thông qua kết nối SSH này. Điều này giải thích tại sao Capistrano mất khá nhiều thời gian để thực thi những dòng lệnh của mình.

Còn đối với Mina, Mina sẽ tạo ra file Bash Script từ tất cả các tác vụ mà nó cần thực thi, gom chung lại, dịch ra Bash Script, rồi sau đó upload file Bash Script này lên server deploy, rồi chạy file này trên server đó. Chính sự khác biệt này mang đến cho Mina tốc độ deploy cực nhanh.
Which to use?

Nếu như bạn không quan tâm lắm đến tốc độ, thời gian tiêu tốn, và  cũng như là bạn không cảm thấy phiền khi phải chỉnh sửa 3 file để deploy thành công thì bạn có thể dùng Capistrano, giống như đại đa số những người khác.

Mina là công cụ deploy cực kỳ nhanh, và đơn giản hơn rất nhiều. Nhưng có một câu hỏi, nếu Mina chạy Bash Script trên server thì lỡ deploy failed thì sao? có rollback như capistrano được không? => Tất nhiên là có, với phiên bản release update sau này của Mina, thì Mina có hỗ trợ Safe Deploy, Mina sẽ deploy vào một thư mục tạm trên server. Nếu deploy thành công thì thư mục tạm này được move vào thư mục releases và được link cho thư mục current như Capistrano, còn nếu deploy thất bại thì Mina chỉ cần xóa thư mục tạm và không làm gỉ ảnh hưởng đến lần release thành công trước đó. Safe Deploy!

### V. Chuyển đổi từ Capistrano sang Mina

Nếu dự án đang dùng Capistrano mà bạn muốn chuyển đồi qua Mina thì cực kỳ đơn giản, do Mina chỉ cần edit file config/deploy.rb nên là ta chỉ cần backup file config/deploy.rb hiện tại (đang dùng Capistrano) ví dụ: `mv config/deploy.rb config/deploy_capis_bk.rb`
Rồi sau đó thực hiện trình tự set up với Mina. Quá đơn giản!
