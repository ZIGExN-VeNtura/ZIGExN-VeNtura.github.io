---
layout: post
title:  "Micro-optimize Ruby"
date:   2015-10-07
summary: Bài viết giới thiệu một số kỹ thuật optimize Ruby code. Gọi là micro-optimize vì nó chỉ tăng performance lên từng chút một, nhưng nhiều cái micro đó góp lại sẽ tăng performance của cả hệ thống lên đáng kể.
description: Bài viết giới thiệu một số kỹ thuật optimize Ruby code. Gọi là micro-optimize vì nó chỉ tăng performance lên từng chút một, nhưng nhiều cái micro đó góp lại sẽ tăng performance của cả hệ thống lên đáng kể...
categories: [ruby on rails]
tags: [""]
images: /images/micro_optimize_ruby.jpg
author: Vinh Nguyen
---

Có nhiều mức độ optimize Ruby code:

- Design: kiến trúc và thuật toán(ví dụ: n + 1 query)
- Source: viết code tối ưu nhất
- Build: build phiên bản ruby phù hợp với yêu cầu của source code.
- Compile: dùng các compiler khác nhau của Ruby. Ví dụ: mrbc, jrubyc, rbx
- Runtime: tối ưu ở bước runtime(ví dụ như dùng các biến môi trường khi execute ruby: `RUBY_GC_MALLOC_LIMIT`)

Trong bài viết này, ta sẽ tập trung vào các kỹ thuật optimize trên source code.

Một vấn đề quan trọng khi optimize chính là cần phải benchmark, nghĩa là không dựa vào cảm giác để quyết định một đoạn code này sẽ chạy nhanh hơn đoạn code kia, mà phải dựa vào kết quả benchmark khách quan. Vì vậy trước tiên, ta sẽ làm quen với các công cụ benchmark của Ruby.

## Các công cụ benchmark

### Benchmark IPS

Rất hữu dụng khi muốn so sánh tốc độ thực thi giữa 2 đoạn code. Cách sử dụng rất đơn giản:

{% highlight ruby %}
require benchmark/ips

Benchmark.ips do |x|
  x.report( fast ) { fast_code }
  x.report( slow ) { slow_code }
end
{% endhighlight %}

Khi execute đoạn code trên ta sẽ được 1 report như sau:

    Calculating -------------------------------------
                slow     71.254k i/100ms
                fast     68.658k i/100ms
    -------------------------------------------------
                slow     4.955M (± 8.7%) i/s -     24.155M
                fast     24.011M (± 9.5%) i/s -    114.246M

    Comparison:
               slow 23958619.8 i/s - 1.00x slower
               fast: 24011974.8 i/s


Tham khảo: [https://github.com/evanphx/benchmark-ips](https://github.com/evanphx/benchmark-ips)

### memory_profiler

Dùng gem này khi muốn kiểm tra số lượng object được tạo ra.

{% highlight ruby %}
require 'memory_profiler'
report = MemoryProfiler.report do
  # run your code here
end

report.pretty_print
{% endhighlight %}

Tham khảo: [https://github.com/SamSaffron/memory_profiler](https://github.com/SamSaffron/memory_profiler)

### allocation_tracer

Có chức năng khá tương tự với `memory_profiler`, gem này tương thích với Ruby 2 trở lên.

{% highlight ruby %}
require 'allocation_tracer'
require 'pp'

pp ObjectSpace::AllocationTracer.trace{
  50_000.times{|i|
    i.to_s
    i.to_s
    i.to_s
  }
}
{% endhighlight %}

Tham khảo: [https://github.com/ko1/allocation_tracer](https://github.com/ko1/allocation_tracer)

## Một số phương pháp optimize với Ruby

### 1\. Giảm số lượng object tạo ra

Giảm số lượng object tạo ra sẽ giúp bộ nhớ ít chiếm dụng hơn, Garbage Collector ít bị overhead hơn dẫn đến code sẽ được execute nhanh hơn.
Dưới đây là một cách để hạn chế phát chiếm dụng bộ nhớ.

#### Sử dụng String#freeze

Mỗi khi một string được gọi, Ruby sẽ tạo một vùng nhớ mới để lưu string này, do đó nếu sử dụng cùng một chuỗi nhiều lần, nên dùng method String#freeze.
Việc này sẽ giúp tránh allocate nhiều vùng nhớ không cần thiết.

Ví dụ:

{% highlight ruby %}
10.times do
  puts "Hello world".object_id
end

10.times do
  puts "Hello world".freeze.object_id
end
{% endhighlight %}

#### Sử dụng ! method(a.k.a bang method)

Trong Ruby có 2 loại method: method trả về object và method thay đổi object. Ví dụ ta có `map` và `map!`
Dùng các hàm ! nếu chỉ cần thay đổi object sẽ giúp giảm việc allocate bộ nhớ.
Vì khi gọi method thường sẽ allocate một vùng nhớ mới và trả về vùng nhớ đó cho câu gọi hàm.

{% highlight ruby %}
array = ['a', 'b', 'c']
array.map(&:upcase)
array.map!(&:upcase)
{% endhighlight %}

Ngoài các ! method, Ruby còn có một số cặp method trả về object và thay đổi một object. Chẳng hạn:

#### select vs. keep_if

{% highlight ruby %}
a = %w{ a b c d e f }
a.keep_if { |v| v =~ /[aeiou]/ }
puts a
=> ["a", "e"]
{% endhighlight %}

{% highlight ruby %}
a = %w{ a b c d e f }
a.select { |v| v =~ /[aeiou]/ }
puts a
=> ["a", "b", "c", "d", "e", "f"]
{% endhighlight %}

#### reject vs. delete_if

{% highlight ruby %}
a = [1,2,3,4,5]
a.delete_if(&:even?)
puts a
=> [1,3,5]
{% endhighlight %}

{% highlight ruby %}
a = [1,2,3,4,5]
a.reject(&:even?)
puts a
=> [1,2,3,4,5]
{% endhighlight %}

### 2\. Dùng các method đã được optimize sẵn của Ruby

#### map.flatten(1) vs. flat_map

{% highlight ruby %}
require 'benchmark/ips'

ARRAY = 100.times.map { (0..9).to_a }

def slow
  ARRAY.map { |x| x }.flatten(1)
end

def fast
  ARRAY.flat_map { |x| x }
end

Benchmark.ips do |x|
  x.report('slow') { slow }
  x.report('fast') { fast }
end
{% endhighlight %}

Kết quả benchmark:

    slow 12348.2 (±9.0%) i/s -  61450 in 5.017159s
    fast 56647.8 (±7.2%) i/s - 282152 in 5.006973s


#### reverse.each vs. reverse_each

{% highlight ruby %}
require 'benchmark/ips'

ARRAY = (1..100).to_a

def slow
  ARRAY.reverse.each{|x| x}
end

def fast
  ARRAY.reverse_each{|x| x}
end

Benchmark.ips do |x|
  x.report('slow') { slow }
  x.report('fast') { fast }
end
{% endhighlight %}

Kết quả benchmark:

    slow 12348.2 (±9.0%) i/s -  61450 in 5.017159s
    fast 56647.8 (±7.2%) i/s - 282152 in 5.006973s


#### Hash#keys và Enumerable#each vs. Hash#each_key

Hash#keys.each sẽ tạo một mảng mới, trong khỉ `each_key` không sinh một mảng mới, do đó dùng `each_key` sẽ hiệu quả hơn.

{% highlight ruby %} require 'benchmark/ips'

HASH = Hash[*('aa'..'zz')]

def slow
  HASH.keys.each { |k| k }
end

def fast
  HASH.each_key { |k| k }
end

Benchmark.ips do |x|
  x.report('slow') { slow }
  x.report('fast') { fast }
end
{% endhighlight %}

Kết quả benchmark:

    slow 12348.2 (±9.0%) i/s -  61450 in 5.017159s
    fast 56647.8 (±7.2%) i/s - 282152 in 5.006973s


#### shuffle.first vs. sample

`Array#shuffle` sẽ allocate một array mới bộ nhớ, trong khi `Array#sample` chỉ thực hiện trên index của array, do đó không cần allocate thêm bộ nhớ.
Tốc độ thực thi sẽ nhanh hơn 15X.

{% highlight ruby %}
require 'benchmark/ips'

ARRAY = (1..100).to_a

def slow
  ARRAY.shuffle.first
end

def fast
  ARRAY.sample
end

Benchmark.ips do |x|
  x.report('slow') { slow }
  x.report('fast') { fast }
end
{% endhighlight %}

Kết quả benchmark:

    slow  324806.7 (±8.1%) i/s - 1620738 in 5.028042s
    fast 5069719.9 (±9.5%) i/s - 24872400 in 5.011482s

[https://github.com/rails/rails/pull/14240](https://github.com/rails/rails/pull/14240)

[https://github.com/rails/rails/pull/17244](https://github.com/rails/rails/pull/17244)

[https://github.com/rails/rails/pull/17099](https://github.com/rails/rails/pull/17099)

[https://github.com/rails/rails/pull/17245](https://github.com/rails/rails/pull/17245)
