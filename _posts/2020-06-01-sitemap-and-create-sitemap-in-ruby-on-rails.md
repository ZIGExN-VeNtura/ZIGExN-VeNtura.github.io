---
layout: post
title:  Sitemap là gì? Tạo Sitemap trong Rails với gem sitemap_generator
date:   2020-06-01
summary: Giới thiệu Sitemap là gì. Hướng dẫn tạo Sitemap trong Rails với gem sitemap_generator.
categories: [Gems]
tags: [“SEO”, “Ruby on Rails”]
image_hidden: true
author: Anh Dep Trai

---

### Giới thiệu
Trong quá trình phát triển một website thì SEO là một công việc không thể thiếu. Định nghĩa đơn giản nhất SEO là quá trình tối ưu hóa trang web của bạn cho công cụ Tìm kiếm. SEO có rất nhiều kỹ thuật như:
- Thêm menu Breadcrumbs cho bài đăng và trang của bạn
- Kiểm tra URL chuẩn của bạn
- Tối ưu hóa 404 Trang của bạn
- Xem xét việc áp dụng AMP
- Tối ưu hóa và gửi Sơ đồ trang web XML của bạn tới Google và Bing
- ..vv..

Hôm nay mình giới thiệu Sơ đồ trang web XML hay còn gọi là Sitemap. XML Sitemap là một bản đồ của website, là một đường dẫn trên trang web của bạn có đuôi .xml. Khi người dùng nhấn vào đường dẫn này sẽ thấy được toàn bộ các trang có thể truy cập trên trang web của bạn.

Tạo ra sitemap không có nghĩa là gia tăng thứ hạng, nhưng nó giúp cho các Search Engines hiểu rõ về website của bạn hơn, từ đó xếp hạng từ khóa tốt cho đối với trang web của bạn.

Format của file này khá đơn giản, ta hoàn toàn có thể làm được bằng tay. Tuy nhiên, đối với những website lớn, cấu trúc phức tạp, việc làm thủ công sẽ tốn rất nhiều thời gian. Vì vậy, hôm nay mình sẽ giới thiệu với các bạn 1 gem giúp generate tự động file sitemaps trong rails - [sitemap_generators](https://github.com/kjvarga/sitemap_generator)


### Cài đặt gem
Đơn giản là mình thêm vào Gemfile trong ứng dụng :
{% highlight ruby %}
gem sitemap_generator
{% endhighlight %}
Sau đó chạy:
{% highlight ruby %}
bundle install
{% endhighlight %}

###  Sử Dụng
Sau đó chạy lệnh
{% highlight ruby %}
bundle exec rake sitemap:install
{% endhighlight %}

Lệnh trên sẽ tự động tạo 1 file trong thư mục `config/sitemap.rb` để config sitemap.

Trong phần `default_host` ta trỏ tới domain của website
{% highlight ruby %}
SitemapGenerator::Sitemap.default_host = "https://anh_dep_trai.com"
{% endhighlight %}

Ngoài ra còn những mục config như sau:

{% highlight ruby %}
SitemapGenerator::Sitemap.sitemaps_path = "search/result/"
SitemapGenerator::Sitemap.public_path = "public/sitemaps_list/"
SitemapGenerator::Sitemap.include_root = false
{% endhighlight %}

Những mục config chính, ta sẽ đặt nó trong block `SitemapGenerator::Sitemap.create {}`

{% highlight ruby %}
SitemapGenerator::Sitemap.create do
 add root_path, changefreq: "weekly"
end
{% endhighlight %}

Trong đó, method add cung cấp cho chúng ta thêm các options như:
- changefreq: Tần suất thay đổi của path này là - always, hourly, daily, weekly, monthly, yearly, never
- lastmod: Thời gian của lần thay đổi cuối cùng, mặc định là Time.now.
- host: Host tương ứng với url đấy, mặc định là default_host mình đã set ở trên.
- priority: Độ ưu tiên của URL so với những cái còn lại. Mặc định là 0.5, giá trị chạy từ 0 đến 1.
- expires: Thời gian hết hạn.

###  Generate và kiểm tra Sitemap files
Sau khi add path xong trong file config, ta chạy lệnh:
{% highlight ruby %}
rails sitemap:refresh
{% endhighlight %}

Khi script đó được chạy, nó sẽ tự động ping tới Google và Bing search engines để thông báo phiên bản mới của sitemap đã sẵn sàng. Trên terminal của ta sẽ hiển thị kết quả ping như sau:
{% highlight ruby %}
+ sitemap.xml.gz                                          32 links /  533 Bytes
Sitemap stats: 32 links / 1 sitemaps / 0m00s

Pinging with URL 'http://www.anh_dep_trai.com/sitemap.xml.gz':
  Successful ping of Google
  Successful ping of Bing

{% endhighlight %}

Nếu bạn muốn thay đổi Search engines khác, hay sửa lại hash của `SitemapGenerator::Sitemap.search_engines` trong file `config/sitemap.rb`. Giá trị mặc định bên trong nó sẽ là:
{% highlight ruby %}
{:google=>"http://www.google.com/webmasters/tools/ping?sitemap=%s",
 :bing=>"http://www.bing.com/ping?sitemap=%s"}
{% endhighlight %}

File sitemap generate ra được đặt trong folder `/public`, add thêm nó vào trong file `robot.txt` nữa:

public/robots.txt
{% highlight ruby %}
Sitemap: http://www.anh_dep_trai.com/sitemap.xml.gz
{% endhighlight %}

Đây là những kiến thức cơ bản. Còn nhiều cái phức tạp hơn mình sẽ có một bài viết chia sẽ ở bài viết sau.
Chúc các bạn code zui zẻ.

### Nguồn tham khảo
1.  [https://support.google.com/webmasters/answer/183668?hl=en](https://support.google.com/webmasters/answer/183668?hl=en)
2.  [https://github.com/kjvarga/sitemap_generator](https://github.com/kjvarga/sitemap_generator)
3.  [https://www.sitepoint.com/start-your-seo-right-with-sitemaps-on-rails/](https://www.sitepoint.com/start-your-seo-right-with-sitemaps-on-rails/)
