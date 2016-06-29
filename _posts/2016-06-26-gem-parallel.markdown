---
layout: post
title:  "Giới thiệu gem parallel"
date:   2016-06-26
summary: Bài viết giới thiệu gem parallel được sử dụng để chạy multi-threading và multi-processing
categories: [ruby on rails]
tags: ["ruby", "rails"]
images: /images/ruby-threading.png
author: Son Dang
---

Ở bài viết kỳ trước mình đã giới thiệu về process, thread và điểm khác biệt giữa hai khái niệm này. Bài viết này sẽ giới thiệu cơ bản gem parallel, một gem sử dụng library của ruby để chạy bất kì đoạn code nào theo thread hoặc process một cách đơn giản.

### Ruby và single thread
Ruby developer đều biết là code MRI chỉ cho phép chạy single thread. Tại sao lại có điều này? MRI có một thành phần gọi là GIL (Global Interpreter Lock). Thành phần này sẽ khóa code bất kì đoạn code ruby đang được thực thi. Điều này có nghĩa là trong trường hợp chạy multi-thread, chỉ có một thread trên một process có thể thực thi ruby code trong một thời điểm.

![ruby-threading](/images/ruby-threading.png)

Vì thế nếu chúng ta có 8 thread hoạt động trên một máy tính có 8 core thì chỉ có 1 thread và 1 core là được sử dụng tại một thời điểm bất kỳ. GIL tồn tại để bảo vệ ruby khỏi trường hợp tương tranh (race condition) có thể làm hỏng dữ liệu. Đây là điểm mạnh cũng như điểm yếu của ruby.

Bài viết này sẽ không đề cập chi tiết tới GIL. Các bạn có thể đọc thêm về GIL ở loạt bài viết [Nobody understands the GIL](http://www.jstorimer.com/blogs/workingwithcode/8085491-nobody-understands-the-gil) của Jesse Storimer.

### Gem parallel
Ruby có 2 thư viện cơ bản để thực hiện multi-threading và multi-processing là Thread và Process. Tuy nhiên có rất nhiều vấn đề khi sử dụng 2 thư viện này mà developer cần phải giải quyết như race condition. Có một vài thư viện mở có cách sử dụng khá đơn giản như [celluloid](https://github.com/celluloid/celluloid), [concurrent-ruby](https://github.com/ruby-concurrency/concurrent-ruby) hay [parallel](https://github.com/grosser/parallel). Tuy nhiên khi muốn thực thi những tác vụ nhỏ như đọc và ghi dữ liệu thì gem parallel lại được ưa chuộng do tính đơn giản và "light-weight" của nó.

Gem parallel có hai chức năng cơ bản là phân chia tác vụ theo thread và theo process.

{% highlight ruby %}
# 2 CPUs -> work in 2 processes (a,b + c)
results = Parallel.map(['a','b','c']) do |one_letter|
  expensive_calculation(one_letter)
end

# 3 Processes -> finished after 1 run
results = Parallel.map(['a','b','c'], in_processes: 3) { |one_letter| ... }

# 3 Threads -> finished after 1 run
results = Parallel.map(['a','b','c'], in_threads: 3) { |one_letter| ... }
{% endhighlight %}

Tương tự có thể sử dụng với `each`
{% highlight ruby %}
Parallel.each(['a','b','c']) { |one_letter| ... }
{% endhighlight %}

#### ActiveRecord

Các bạn có thể áp dụng parallel cho ActiveRecord
{% highlight ruby %}
# reproducibly fixes things (spec/cases/map_with_ar.rb)
Parallel.each(User.all, in_processes: 8) do |user|
  user.update_attribute(:some_attribute, some_value)
end
User.connection.reconnect!

# maybe helps: explicitly use connection pool
Parallel.each(User.all, in_threads: 8) do |user|
  ActiveRecord::Base.connection_pool.with_connection do
    user.update_attribute(:some_attribute, some_value)
  end
end

# maybe helps: reconnect once inside every fork
Parallel.each(User.all, in_processes: 8) do |user|
  @reconnected ||= User.connection.reconnect! || true
  user.update_attribute(:some_attribute, some_value)
end
{% endhighlight %}

#### Break
Sử dụng để dùng những tác vụ đã được thực hiện
{% highlight ruby %}
Parallel.map(User.all) do |user|
  raise Parallel::Break # -> stops after all current items are finished
end
{% endhighlight %}

#### Kill
Chỉ nên sử dụng cách này nếu sub-process an toàn để kill process ở bất kỳ thời điểm nào
{% highlight ruby %}
Parallel.map([1,2,3]) do |x|
  raise Parallel::Kill if x == 1# -> stop all sub-processes, killing them instantly
  sleep 100
end
{% endhighlight %}

#### Worker number
Sử dụng `Parallel.worker_number` để tìm ra vị trí của worker mà tác vụ đang được thực thi.

{% highlight ruby %}
Parallel.each(1..5, :in_processes => 2) do |i|
  puts "Item: #{i}, Worker: #{Parallel.worker_number}"
end

# Item: 1, Worker: 1
# Item: 2, Worker: 0
# Item: 3, Worker: 1
# Item: 4, Worker: 0
# Item: 5, Worker: 1
{% endhighlight %}


Trên đây là giới thiệu sơ bộ về gem parallel. Source code của gem khá đơn giản, các bạn có thể đọc thêm tại [lib/parallel.rb](https://github.com/grosser/parallel/blob/master/lib/parallel.rb) và [lib/parallel/processor_count.rb](https://github.com/grosser/parallel/blob/master/lib/parallel/processor_count.rb).

Một vài tài liệu về process và thread trong ruby:

* [Parallelism is a Myth in Ruby](https://www.igvita.com/2008/11/13/concurrency-is-a-myth-in-ruby/)
* [Threads in Ruby](https://www.sitepoint.com/threads-ruby/)
* [Ruby - Multithreading
](http://www.tutorialspoint.com/ruby/ruby_multithreading.htm)
* [Nobody understands the GIL
](http://www.jstorimer.com/blogs/workingwithcode/8085491-nobody-understands-the-gil)
* [Nobody understands the GIL - Part 2: Implementation
](http://www.jstorimer.com/blogs/workingwithcode/8100871-nobody-understands-the-gil-part-2-implementation)
* [Does the GIL Make Your Ruby Code Thread-Safe?](http://www.rubyinside.com/does-the-gil-make-your-ruby-code-thread-safe-6051.html)

