---
layout: post
title:  "Khái niệm Duck typing trong Ruby"
date:   2016-11-23
summary: Bài viết giới thiệu cơ bản về cách sử dụng kỹ thuật duck typing cho ruby
categories: [Ruby]
tags: ["Ruby"]
images: /images/duck-typing.jpg
author: Huy Binh
---

### Giới thiệu duck typing:

Ruby vẫn luôn được coi là ngôn ngữ rất hướng đối tượng, vì trong Ruby mọi thứ đều là object. Chúng ta có thể viết chương trình Ruby để thể hiện mọi tính chất của hướng đôí tượng. Tuy nhiên, nếu đã có căn bản Ruby, thì chắc bạn đã để ý là không như những ngôn ngữ ví dụ như Java, Ruby không có từ khóa để thể hiện một cách tường minh tính trừu tượng (từ khóa trong Java: interface) và đa hình (từ khóa trong Java: override)...

Nhưng Ruby có tính năng gọi là Duck Typing để thay thế. Duck Typing là các public interface không được gắn với một lớp cụ thể nào, cung cấp khả năng xử lý mềm dẻo với các sự phụ thuộc trong hệ thống. Một đối tượng được thể hiện thông qua các hành vi của nó hay qua các public interface, tuy nhiên, nó không chỉ đơn thuần thể hiện chỉ một interface mà ứng với mỗi một kiểu thông điệp gửi đến thì nó sẽ đáp ứng với một interface phù hợp. Người dùng đối tượng sẽ không cần quan tâm, không cần biết về lớp của nó. Các lớp chỉ đơn thuần là nơi để đối tượng thực hiện một public interface nào đó.

### Ta có ví dụ sau:

{% highlight ruby %}
class Animal
  attr_reader :name

  def initialize(name)
    @name = name
  end

  def say_hello
    raise 'Subclasses should implement this method'
  end
end

class Cat < Animal
  def say_hello
    puts "Meow, my name is #{name}"
  end
end

class Dog < Animal
  def say_hello
    puts "Woof woof, my name is #{name}"
  end
end

class Duck
  def initialize(name)
    @name = name
  end

  def say_hello
    puts "Trump, my name is #{name}"
  end
end

class Zoo
  def initialize
    @animals = []
  end

  def add(animal)
    @animals << animal
  end

  def say_hello_all
    @animals.each { |a| a.say_hello }
  end
end

# Some test
z = Zoo.new
z.add(Cat.new('Kitty'))
z.add(Dog.new('Pluto'))
z.add(Duck.new('Donald'))
z.say_hello_all
{% endhighlight %}

### Chạy thử sẽ thấy kết qủa sau:

{% highlight ruby %}
Meow, my name is Kitty

Woof woof, my name is Pluto

Trump, my name is Donald
{% endhighlight %}

### Phân tích code trên ta sẽ thấy:

- Duck không thừa kế từ Animal. Trong các ngôn ngữ như Java, ta phải định nghĩa interface có phương thức say_hello, để các con vật trong Zoo có chung interface. Còn trong Ruby thì miễn là Duck có phương thức có cùng tên là ta có thể nhét chung với các Animal khác. Ý tưởng của Duck Typing là: Vịt trắng vịt đen đều là vịt, thậm chí bất kể con gì đi như vịt nói như vịt cũng đều có thể coi là vịt.
- Cat và Dog thừa kế phương thức say_hello, và override lại phương thức say_hello của Animal. Nhưng Ruby không có từ khóa override như các ngôn ngữ như Java để thể hiện tường minh chỗ này. Việc này có lẽ nằm trong ý đồ thiết kế của Ruby, là trung thành với ý tưởng Duck Typing. Bạn cũng có thể bỏ say_hello trong Animal đi mà chương trình vẫn chạy tốt.

Hai phân tích ở trên cho thấy mặc dù Duck Typing làm cho Ruby rất uyển chuyển tăng tính mềm dẻo, làm cho ứng dụng giảm bớt chi phí bảo trì và dễ dàng thay đổi khi có yêu cầu, nhưng vì Ruby không bắt buộc phải định rõ interface và override, để chương trình rõ ràng để những lập trình viên cùng tham gia viết chương trình có thể làm việc suôn sẻ với nhau, trong chương trình cần phải cẩn thận viết comment những chỗ mang tính trừu tượng và đa hình. Nếu không, chỉ cần chương trình to to một chút và vài người cùng viết, thì mã nguồn sẽ thành đống bùi nhùi lúc nào không hay.
