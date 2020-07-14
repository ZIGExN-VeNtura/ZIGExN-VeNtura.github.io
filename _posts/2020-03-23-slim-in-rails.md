---
layout: post
title: Slim template cho rails
date: 2020-03-23
summary: Slim là một ngôn ngữ giao diện mẫu, sử dụng nhanh, gọn nhẹ, mục đích là nhằm giảm cú pháp cho phần view mà không làm cho code trở nên khó hiểu.
categories: [ruby on rails]
tags: [“Views in Rails”]
image_hidden: true
author: Anh Dep Trai
---

### Slim là gì?

Slim là một ngôn ngữ giao diện mẫu, sử dụng nhanh, gọn nhẹ, mục đích  là nhằm giảm cú pháp cho phần view mà không làm cho code trở nên khó hiểu.

### Tính năng của Slim
- Cú pháp ngắn gọn, không có các thẻ đóng tag (Thay thế bằng lùi đầu dòng).
- Có thể cấu hình các shortcut tags (ví dụ `#` thay cho `<div id="...">` và `.` thay cho `<div class="...">`) trong cấu hình mặc định.
- An toàn: hỗ trợ `html_safe` cho Rails.
- Tự động escape HTML mặc định.
- Khả năng tùy biến cao, linh hoạt.
- Khả năng mở rộng thông qua các plugins.
- Logic less mode tương tự như Mustache.
- Includes.
- Translator/I18n.
- Hiệu năng cao tốc độ tương đương với ERB/Erubis.
- Hỗ trợ Streaming trong Rails.
- Hỗ trợ đầy đủ Unicode cho các thẻ tag và các thuộc tính.
- Hỗ trợ bởi hầu hết các framework lớn (như Rails, Sinatra, ...).

### Cài đặt và sử dụng
Để sử dụng, đầu tiên bạn phải thêm dòng sau vào trong Gemfile:
{% highlight ruby %}
# Gemfile
gem 'slim-rails'
{% endhighlight %}

Sau đó chạy lệnh `bundle install` để cài đặt gem này.

Để chuyển đổi file từ đuôi `*.html.erb` thành đuôi `*.html.slim`, bạn cần thêm dòng sau vào Gemfile:
{% highlight ruby %}
# Gemfile
gem 'html2slim'
{% endhighlight %}

Sau đó chạy `bundle install` để cài đặt gem này.

Sử dụng lệnh `erb2slim app/views --delete` sẽ chuyển đổi toàn bộ file nằm trong đường dẫn `app/views` sang Slim, `--delete` có tác dụng xóa đi file cũ, nếu không thiết lập `--delete` thì sau khi chuyển đổi sẽ tồn tại song song 2 file đuôi `.erb` và `.slim`.

### Cấu trúc của Slim template
Dưới đây là một ví dụ thể hiện cấu trúc của một Slim template:
{% highlight ruby %}
# app/views/posts/index.html.slim
doctype html
html
  head
    title Slim Examples
    link type="text/css" rel="stylesheet" href="post.css"
    script src="http://ajax.googleapis.com/ajax/libs/jquery/1.11.2/jquery.min.js"
  body
    h1 Markup examples
    #content
      p This example shows you how a basic Slim file looks.
    == yield
    - if @posts.any?
      table#posts
        - @posts.each do |post|
          tr
            td.title = posts.title
            td.contents = posts.contents
            td.comments = posts.comments
    - else
      p No post found
        Thank you!
    #footer
      == render 'footer'
      | Copyright © #{@year} #{@author}
{% endhighlight %}

### Một vài cú pháp hay dùng trong Slim
#### Verbatim text `|`
`|` sẽ sao chép tất cả những dòng có lề thụt vào nhiều hơn nó để hiển thị như một string. Ví dụ:
{% highlight ruby %}
p
  | This line is the first line.
    This line is the second line.
{% endhighlight ruby %}

Đoạn trên sẽ được dịch ra HTML như sau:
{% highlight HTML_ruby %}
<p>
  This line is the first line.
  This line is the second line.
</p>
{% endhighlight HTML_ruby %}

#### Control code `-`
Sử dụng `-` thay thế cho `<% %>` trong view của Rails, ví dụ như đối với vòng lặp hay điều kiện. Nếu code ruby của bạn cần viết trên nhiều dòng, hãy thêm `\` vào cuối của dòng đó. Nếu như dòng đó kết thúc với dấu `,` (ví dụ các tham số của một phương thức) khi đó sẽ không cần thêm dấu `\` nữa.
{% highlight ruby %}
body
  - if users.empty?
    | No inventory
{% endhighlight ruby %}

#### Output  =
Sử dụng `=` thay thế cho `<%= %>` trong view của Rails. Tương tự như với `-`, nếu code ruby của bạn cần viết trên nhiều dòng, hãy thêm `\` vào cuối dòng. Và nếu như kết thúc dòng là `,` thì không cần thêm `\` nữa.
{% highlight HTML_ruby %}
= javascript_include_tag \
   "jquery",
   "application"
{% endhighlight HTML_ruby %}

#### Output without HTML escaping `==`
Cũng giống như `=`, nhưng mà nó không qua phương thức `escape_html`.

#### Code comment `/` and HTML comment `/!`
Sử dụng `/` cho code comments và `/!` cho html comments. Ví dụ:
{% highlight HTML_ruby %}
p
  / This line won't get displayed.
    Neither does this line.
  /! This will get displayed as html comments.
{% endhighlight HTML_ruby %}

Đoạn Slim trên dịch sang HTML sẽ là:
{% highlight HTML_ruby %}
<p><!--This will get displayed as html comments.--></p>
{% endhighlight HTML_ruby %}

#### IE conditional comment  /[...]
{% highlight HTML_ruby %}
/[if IE]
     p Get a better browser
{% endhighlight HTML_ruby %}

Đoạn Slim trên dịch sang HTML sẽ là:
{% highlight HTML_ruby %}
<!--[if IE]><p>Get a better browser.</p><![endif]-->
{% endhighlight HTML_ruby %}

### Một số HTML Tags trong Slim
#### Closed tags
Bạn có thể đóng thẻ một cách rõ ràng bằng cách thêm dấu `/` đằng sau. Tuy nhiên, điều này là không cần thiết vì những thẻ tag như (img, br ...) đều được đóng tự động.
{% highlight HTML_ruby %}
img src="image.png" /
{% endhighlight HTML_ruby %}

#### Inline tags
Để sử dụng các thẻ tag trên cùng một dòng, sử dụng `:` sau mỗi một thẻ, ví dụ:
{% highlight HTML_ruby %}
ul
   li: a href="http://google.com.vn" Google
{% endhighlight HTML_ruby %}

Tuy nhiên, để cho dễ đọc, các thuộc tính nên được bao lại bằng các kí tự {...}, (...), [...]. Ví dụ:
{% highlight ruby %}
ul
   li: a[href="http://google.com.vn"] Google
{% endhighlight ruby %}

#### Ruby attributes
Code Ruby được viết sau dấu `=`. Nếu code có chứa các khoảng trắng, bạn cần phải viết các thuộc tính ở trong `(...)` hoặc `{...}` và `[...]`.

{% highlight ruby %}
table
 - @users.each do |user|
    td id="user_#{user.id}" class= user.role
       a href=user_path(user) = user.name
{% endhighlight ruby %}

#### Attribute merging
Bạn có thể sử dụng một `Array` để nhóm nhiều giá trị của một thuộc tính lại với nhau, ví dụ:
{% highlight HTML_ruby %}
a class=["green","blue"]
a class=[:green, :blue]
{% endhighlight HTML_ruby %}

#### Splat attributes `*`
Splat shortcut cho phép bạn trả về một hash với các cặp thuộc tính/giá trị.
{% highlight HTML_ruby %}
.card*{'data-url'=>place_path(place), 'data-id'=>place.id} = place.name
{% endhighlight HTML_ruby %}

khi đó, mã HTML sinh ra là:
{% highlight HTML_ruby %}
<div class="card" data-id="1234" data-url="/place/1234">Slim's house</div>
{% endhighlight HTML_ruby %}

Bạn cũng có thể sử dụng các methods hoặc instance variables trả về một hash như sau:
{% highlight HTML_ruby %}
.card *method_which_returns_hash = place.name
.card *@hash_instance_variable = place.name
{% endhighlight HTML_ruby %}

### Shortcut
#### ID shorcut `#` và class shortcut `.`
Nếu không có thẻ tag nào phía trước thì :
- `#` sẽ tương đương với mã HTML: `<div id="..."></div>`
- `.` sẽ tương đương với mã HTML: `<div class="..."></div>`

Đối với trường hợp có thẻ tag phía trước, khi đó:

`h1#content` <-> `h1 id="content"`

`h1.content_css` <-> `h1 class="content_css"`

#### Tag shortcuts
Bạn có thể tự định nghĩa các tag shortcuts ở trong `config/initializers/slim.rb` bằng cách cài đặt option `:shorcut`. Ví dụ:
{% highlight ruby %}
# config/initializers/slim.rb
Slim::Engine.set_options shortcut: {'c' => {tag: 'container'}, '#' => {attr: 'id'}, '.' => {attr: 'class'} }
{% endhighlight ruby %}

Khi đó, đoạn code Slim:
{% highlight HTML_ruby %}
c.content Some text
{% endhighlight HTML_ruby %}

sẽ được dịch sang mã HTML như sau:
{% highlight HTML_ruby %}
<container class="content">Some text</container>
{% endhighlight HTML_ruby %}

#### Attribute shortcuts
Chúng ta cũng có thể tự định nghĩa các attribute shortcuts ở trong `config/initializers/slim.rb`, ví dụ như sau:
{% highlight ruby %}
# config/initializers/slim.rb
Slim::Engine.set_options shortcut: {'&' => {tag: 'input', attr: 'type'}, '#' => {attr: 'id'}, '.' => {attr: 'class'}}
{% endhighlight ruby %}

Khi đó, đoạn code Slim:
{% highlight HTML_ruby %}
&text name="user"
&password name="password"
{% endhighlight HTML_ruby %}

sẽ được dịch sang mã HTML như sau:
{% highlight HTML_ruby %}
<input type="text" name="user" />
<input type="password" name="password" />
{% endhighlight HTML_ruby %}

Chúc các bạn code zui zẻ.

### Nguồn tham khảo
1.  [https://github.com/slim-template/slim](https://github.com/slim-template/slim)
2.  [https://github.com/slim-template/slim-rails](https://github.com/slim-template/slim-rails)
3.  [http://www.rubydoc.info/gems/slim/frames](http://www.rubydoc.info/gems/slim/frames)
