---
layout: post
title:  Sử dụng gem tablesmith trong ruby on rails
date:   2020-03-20
summary: Hiển thị dữ liệu của Array, Hash, ActiveRecord trong console dạng bảng.
categories: [Gems]
tags: [“Ruby”, “Ruby on Rails”]
image_hidden: true
author: Anh Dep Trai

---

### Giới thiệu:
Trong quá trình làm việc với Rails thì đa phần chúng ta sử dụng và làm việc với màn hình console rất nhiều. Để làm cho công việc trở nên thú vị và sinh động hơn, thì mình phát hiện 1 gem khá thú dị. Đó là tablesmith.

### Cài đặt gem
Đơn giản là mình thêm vào Gemfile trong ứng dụng :
{% highlight ruby %}
gem tablesmith
{% endhighlight %}
Sau đó chạy:
{% highlight ruby %}
bundle install
{% endhighlight %}

###  Sử Dụng
- Trong irb hoặc pry :
{% highlight ruby %}
require 'tablesmith'
> [[:foo, :bar], [1,3]].to_table
=> +-----+-----+
| foo | bar |
+-----+-----+
| 1   | 3   |
+-----+-----+
> {a: 1, b: 2}.to_table
=> +---+---+
| a | b |
+---+---+
| 1 | 2 |
+---+---+
> [{date: '1/1/2020', amt: 35}, {date: '1/2/2020', amt: 80}].to_table
=> +----------+-----+
|   date   | amt |
+----------+-----+
| 1/1/2020 | 35  |
| 1/2/2020 | 80  |
+----------+-----+
> p1 = Person.create(first_name: 'chrismo', custom_attributes: { instrument: 'piano', style: 'jazz' })
=> #<Person:0x00007fac3eb406a8 id: 1, first_name: 'chrismo', last_name: nil, age: nil, custom_attributes: {:instrument=>'piano', :style=>'jazz'}>
> p2 = Person.create(first_name: 'romer', custom_attributes: { hobby: 'games' })
=> #<Person:0x00007fac3ebcbb68 id: 2, first_name: 'romer', last_name: nil, age: nil, custom_attributes: {:hobby=>'games'}>
> batch = [p1, p2].to_table
=> +----+------------+-----------+-----+--------+------------+--------+
|              person               |      custom_attributes       |
+----+------------+-----------+-----+--------+------------+--------+
| id | first_name | last_name | age | hobby  | instrument | style  |
+----+------------+-----------+-----+--------+------------+--------+
| 1  | chrismo    |           |     |        | piano      | jazz   |
| 2  | romer      |           |     | games  |            |        |
+----+------------+-----------+-----+--------+------------+--------+
{% endhighlight %}

Với cách làm này, mình có thể xem dữ liệu một cách trực quan, đỡ phải căng mắt ra nhìn màn hình console một cách đơn điệu.
Chúc các bạn code zui zẻ.
