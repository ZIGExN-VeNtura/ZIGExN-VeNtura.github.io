---
layout: post
title:  Xử lý magic number trong Rails
date:   2019-07-25
summary: Magic number là những đoạn hardcode, giá trị tĩnh và không rõ ràng khiến code chúng ta bốc mùi...
categories: [ruby on rails]
tags: ["Ruby on Rails"]
image: ""
image_hidden: true
author: sondh
---

### Magic number là gì?

Magic number là những đoạn hardcode, giá trị tĩnh và không rõ ràng. Chúng có thể là số, chữ...  
*Ví dụ* 
```ruby
if client_id == 21
  # your code
end
```
```ruby
validates :name, length: {maximum: 100}
```
Những đoạn code như trên sẽ dẫn đến các vấn đề như:
- Khó khăn trong việc review và đọc code  
  (Người đọc code sẽ cần phải tìm xem 21 là client nào)
- Dễ sai sót trong việc maintaince  
  (Thay đổi giá trị ở class này nhưng quên ở class khác)

### Cách xử lý magic number
##### 1. Các hàm có sẵn
```ruby
numbers = [1, 2, 3]
language = 'Ruby'
```
```ruby
# bad
numbers[0] # => 1
numbers.size > 0 # => true
language.length == 0 # => false
```
Ruby - A PROGRAMMER'S BEST FRIEND cung cấp cho chúng ta những hàm rất dễ hiểu như
```ruby
# good
numbers.first # => 1
numbers.any? # => true
language.zero? # => false
```

##### 2. Hằng số
```ruby
# bad
if client_id == 21
  # your code
end
```
Đặt tên hằng số có ý nghĩa làm cho code trông dễ đọc hơn như thế này
```ruby
# good
ZIGEXN_CLIENT_ID = 21

if client_id == ZIGEXN_CLIENT_ID
  # your code
end
```

##### 3. Gem Config
Gem config giúp chúng ta dễ dàng quản lý và sử dụng các giá trị setting của hệ thống.  
Cài đặt gem `config`
```ruby
# Gemfile
gem 'config'
```
```ruby
bundle install
rails g config:install
```

Thêm các giá trị cần dùng vào file `settings.yml`
```ruby
# config/settings.yml
client:
  zigexn:
    id: 21
    name: Zigexn
```
Sử dụng trong code
```ruby
# good  
if client_id == Settings.client.zigexn.id
  # your code
end
```
Đọc thêm về Gem config: [railsconfig/config](https://github.com/railsconfig/config)


### Tổng kết
Vậy là mình đã hoàn thành bài giải thích khái niệm magic number và cách xử lý chúng trong Rails.  
Cùng quay lại project của mình và xem code của mình có đang bốc mùi không nhé ;)

***

**Thảm khảo**:
- [http://www.chrisrolle.com/en/blog/replacing-magic-numbers](http://www.chrisrolle.com/en/blog/replacing-magic-numbers)
- [https://www.eliotsykes.com/magic-numbers](https://www.eliotsykes.com/magic-numbers)
- [https://github.com/railsconfig/config](https://github.com/railsconfig/config)
