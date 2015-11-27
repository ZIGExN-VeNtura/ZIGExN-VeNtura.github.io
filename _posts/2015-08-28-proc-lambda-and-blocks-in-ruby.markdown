---
layout: post
title:  "Proc, Lambda và Block in Ruby"
date:   2015-08-27
summary: Hôm nay tôi xin giới thiệu về Anonymous Functions như là Proc, Lambda, Blocks để thấy rõ về tính Callable và Runable của Objects trong Ruby.
description: Hôm nay tôi xin giới thiệu về Anonymous Functions như là Proc, Lambda, Blocks để thấy rõ về tính Callable và Runable của Objects trong Ruby.
categories: [ruby on rails]
tags: [""]
images: /images/block-proc-lambda.jpg
author: Kiet Nguyen
---

Hôm nay, tôi xin cùng các bạn bàn về Tính Callable và Runable của Objects trong Ruby. Theo tôi được biết thì trong Ruby bạn có thể gọi, thực thi hoặc là chạy: Threads, Anonymous functions, Stríng, thậm chí là methods đã được chuyển thành objects (gọi bằng objects).

Hôm nay tôi xin giới thiệu về Anonymous Functions như là Proc, Lambda, Blocks để thấy rõ về tính Callable và Runable của Objects trong Ruby.

__1. Anonymous Functions: The Proc Class__

Một trong những Callable Object quan trọng của Ruby là Proc objects and Lambdas. Proc objects thì tự bản thân nó chứa trình tự các dòng code và bạn có thể khởi tạo, lưu trữ, hoặc truyền 'arguments' và khi cần thì thực thi nó bằng method 'call'.

- Hiểu về Proc objects thì chúng ta cần nắm những thứ sau đây:

  + Tạo và sử dụng procs.
  + Cách mà procs sử dụng 'arguments' và truyền các biến.
  + Tính bao đóng của procs.
  + Sự liên quan giống và khác giữa procs và code blocks.
  + Sự khác nhau giữa Proc, Lambda.

- Hi all, I'm Proc

Let's start, để tạo 1 instance của Proc object bằng câu lệnh: ```Proc.new```
{% highlight bash %}
proc_object = Proc.new { puts "Inside a Proc's block" }
 => #<Proc:0x00000002839f58@(irb):7>
proc_object.call
Inside a Proc's block
{% endhighlight %}

- The proc method

```proc``` là một method tương đồng với ```Proc.new``` cả hai cùng tạo ra Proc object và trả về cùng kết quả.

Ví dụ:

{% highlight bash %}
proc_object_1 = proc { puts "Hi!" }
 => #<Proc:0x00000002839f58@(irb):7>
proc_object_1.call
Hi!
proc_object_1.class
Proc
>
proc_object_2 = Proc.new { puts "Hi!" }
 => #<Proc:0x00000002853408@(irb):10>
proc_object_2.call
Hi!
proc_object_2.class
Proc
{% endhighlight %}

__2. Proc và Block khác nhau như thế nào?__

Khi tạo 1 ```Proc``` object thì chúng ta luôn luôn cần cung cấp 1 code block, nhưng không phải mọi code block đều là ```Proc``` hoặc là phục vụ cho 1 ```Proc``` cụ thể nào.

Ví dụ: ```[1, 2, 3].each { |x| puts x * 10 }```

Code block phía trên không tạo ra 1 Proc object.

Đào sâu hơn, 1 method có thể bắt 1 block, và chuyển block đó thành Proc object bằng 1 syntax đặc biệt là ```&```

Chúng ta định nghĩa 1 method và đối số truyền vào là 1 code block.
Trong Ruby gọi là 'Capture the block - USING PROCS FOR BLOCKS'

Ví dụ:

{% highlight bash %}
def capture_block(&block)
 puts "convert block into proc and call"
 block.call
 puts "block class"
 block.class
end
 => :capture_block

capture_block { puts "I'm a Proc or Block" }
convert block into proc and call
I'm a Proc or Block
block class
 => Proc
{% endhighlight %}

Điều quan trọng là chúng ta phải hiểu được ```&``` dùng để làm gì?
```&``` trong capture_block(&block) mang 2 ý nghĩa:

  1. Nó sẽ chuyển block thành proc object bằng cách trigger gọi method to_proc trên block.
  2. Và để đánh dấu block đó là proc object, và có thể thực thi bằng method call trong method capture_block.

Chúng ta không thể gọi capture_block theo kiểu ```capture_block(p)``` hay là ```capture_block(p.to_proc)``` với p là 1 Proc vì với các kiểu truyền này thì Ruby hiểu là ```regular parameter``` và bạn không thể trigger call proc trong method.

Ngoài ra ```Proc``` còn có thể được gọi trong code block cũng bằng ```&```

Ví dụ:

{% highlight bash %}
proc = Proc.new {|x| puts x.upcase }
%w{ ruby and coffee }.each(&proc)
RUBY
AND
COFFEE
 => ["ruby", "on", "coffee"]
{% endhighlight %}

__3. Giới thiệu về lambda và Sự khác biệt giữa lambda và proc__

Giống như ```Proc.new```, ```lambda``` method cũng tạo 1 Proc object

{% highlight bash %}

lam = lambda { puts "A lambda!" }
=> #<Proc:0x441bdc@(irb):13 (lambda)>
lam.call
A lambda!

{% endhighlight %}

Giữa lambda và proc có sự khác biệt ở ba điểm:

- lambda được khởi tạo một cách rõ rãng, tường minh. Nơi mà Ruby khởi tạo ngầm các Proc object thì các proc object được khởi tạo đó đểu là các proc thông thường, không phải là lambda. Ví dụ như trường hợp đã nêu ở trên đối với ```def capture_block(&block)``` block trong hàm capture_block được chuyển ngầm thành Proc object thì block này không phải là lambda vì  lambda phải được khởi tạo 1 cách rõ rãng, không ngầm tạo.

- lambda và proc khác biệt ở chỗ thứ 2 đó là việc xử lý method ```return``` của chúng. Nếu câu lệnh return được thực hiện bên trong lambda thì sẽ return ra khỏi lambda đó trở về context body của nơi gọi lambda. Còn đối với return bên trong proc thì nó sẽ return ra khỏi method. Ví dụ sau đây sẽ minh họa cụ thể: output chỉ in ra "Still here!" vì proc return ra khỏi method.

{% highlight bash %}

def return_test
  l = lambda { return }
  l.call
  puts "Still here!"
  p = Proc.new { return }
  p.call
  puts "You won't see this message!"
end

return_test

Still here!
 => nil
{% endhighlight %}


- Điều cuối cùng là: lambda sẽ không được thực thi nếu truyền sai arguments.

{% highlight bash %}

lam = lambda {|x| p x }
=> #<Proc:0x42ee9c@(irb):21 (lambda)>

lam.call(1)
1
=> 1

lam.call
ArgumentError: wrong number of arguments (0 for 1)

lam.call(1,2,3)
ArgumentError: wrong number of arguments (3 for 1)

proc = proc {|x| p x }
=> #<Proc:0x42ee9c@(irb):21 (lambda)>

proc.call(1)
1
=> 1

proc.call
nil
=> nil

proc.call(1,2,3)
1
=> 1

{% endhighlight %}

Tóm lại, lambda sẽ thực hiện theo phương pháp lập luận chặt chẽ. Còn proc thì được 'thả lỏng hơn'.