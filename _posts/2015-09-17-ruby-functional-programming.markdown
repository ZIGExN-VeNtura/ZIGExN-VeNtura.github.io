---
layout: post
title:  Lập trình hàm với Ruby
date:   2015-09-17 23:55:00
summary: 
categories: [ruby]
tags: ["ruby"]
images: /images/socialite.jpg
author: Vinh Nguyễn
---

Trong bài viết nay, ta sẽ làm quen với lập trình hàm bằng cách áp dụng các khái niệm trong lập trình hàm vào các đoạn code viết bằng ruby.

Nói đến lập trình hàm, ta thường nghe nói đến những khái niệm "lạ" như: immutable data, first class functions, tail call optimisation... Những kỹ thuật lập trình như parallelization, lazy evaluation, determinism.

Một trong những đặc điểm của lập trình hàm chính là việc giải quyết vấn đề side-effect. Chẳng hạn như trong ví dụ sau, hàm increment thay đổi biến @a.

{% highlight ruby %}
@a = 10
def increament
  @a += 1
end
{% endhighlight %}

Một chương trình trong lập trình hàm không thay đổi giá trị các biến nằm ngoài hàm đó. Do đó đoạn code trên có thể được viết lại như sau:

{% highlight ruby %}
def increament(a)
  return a + 1
end
@b = increament(10)
{% endhighlight %}

Immutable chính là đặc điểm cơ bản nhất của lập trình hàm. Tất cả hàm số đều trả về giá trị, không làm thay đổi trang thái hiện tại của chương trình.
Chẳng hạn trong lập trình hàm ta có phép gán sau:

{% highlight erlang %}
> a = 10
OK
> a = a + 1 
ERROR!!!
{% endhighlight %}

Như vậy, trong lập trình hàm, một khi đã khởi tạo một biến, ta không thể thay đổi nội dung của biến đó nữa. Nếu muốn thay đổi, ta phải copy giá trị của biến đó và gán vào một biến mới khác. Chẳng hạn:

{% highlight erlang %}
> a = 10
OK
> b = a + 1 
OK
{% endhighlight %}

Đến đây, có lẽ bạn sẽ tự hỏi, vậy lập trình hàm có lợi ích gì? Việc copy mỗi khi thay đổi biến như vậy có làm giảm performance của chương trình hay không?

Mặc dù Ruby là ngôn ngữ hướng đối tượng, nhưng với cú pháp mềm dẻo, ta có thể áp dụng một số kỹ thuật lập trình hàm vào Ruby, việc này sẽ giúp code được viết ra ngắn gọn, dễ hiểu hơn

