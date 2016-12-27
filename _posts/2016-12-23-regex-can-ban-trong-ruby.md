---
layout: post
title:  "Một số Regex thường dùng trong ruby"
date:   2016-12-23
summary: Bài viết giới thiệu cơ bản về cách sử dụng kỹ thuật regex dùng để xử lý chuỗi trong ruby.
categories: [Ruby]
tags: ["Ruby"]
images: /images/regex.jpg
author: Trong Huu
---

### Giới thiệu:

Biểu thức (regexps) là mô hình trong đó mô tả nội dung của một chuỗi. Chúng sử dụng để kiểm tra xem một chuỗi có chứa một mẫu nhất định, hoặc xuất ra các phần được tìm thấy. Chúng được tạo ra với / pat / và% r {} pat hoặc Regexp.new.

### Chúng ta xem một vài ví dụ sau:

Tìm vị trí ký tự hoặc nhiều ký tự trong một chuỗi.

{% highlight ruby %}
s = "/learn/regex"
idx = s =~ /regex/

#Kết quả trả về là:
# => 7

#Nếu không tìm thấy sẽ trả về nil.
{% endhighlight %}

Kiểm tra ký tự hoặc nhiều ký tự có nằm trong chuỗi hay không?

{% highlight ruby %}
s = "/learn/regex"
if s =~ /regex/
  puts "found the chars"
else
  puts "not found"
end

#Kết quả trả về là:
# =>  "found the chars"
{% endhighlight %}

Chúng ta thấy đoạn mã trên cũng giống đoạn mã tìm vị trí ký tự. Nhưng nó khác ở chổ chúng ta có thể sử dụng điều kiện if trong trường hợp này.

Xuất chuỗi nâng cao

{% highlight ruby %}
s = "learn regex extra"
s =~ /^(.+) .+? (.+)/
puts $1
puts $2

#Kết quả trả về là:
#  => "learn"
#  => "extra"
{% endhighlight %}

Phương thức thay thế

{% highlight ruby %}
phone = "2004-959-559 #This is Phone Number"
phone = phone.sub!(/#.*$/, "")
puts "Phone Num : #{phone}"

phone = phone.gsub!(/\D/, "")
puts "Phone Num : #{phone}"

#Kết quả trả về là:

#  => "Phone Num : 2004-959-559"
#  => "Phone Num : 2004959559"
{% endhighlight %}

Tiếp theo, Chúng ta hãy xem đoạn code sau

{% highlight ruby %}
text = "rails are rails, really good Ruby on Rails"

# Đổi từ chữ rails thành chữ Rails
text.gsub!("rails", "Rails")

# In hoa chữ R trong chữ rails
text.gsub!(/\brails\b/, "Rails")
puts "#{text}"

#Kết quả trả về là:

#  => "Rails are Rails, really good Ruby on Rails"
{% endhighlight %}

Chúng ta lưu ý rằng. hàm ```sub``` và ```gsub``` sẽ trả về một chuỗi mới. Nó không sửa chuỗi hiện tại nên nó sẽ làm cho code chúng ta chậm hơn. Vì thế, chúng ta nên dùng ```sub!``` và ```gsub!```.