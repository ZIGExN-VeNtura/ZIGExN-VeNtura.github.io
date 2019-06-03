---
layout: post
title:  "Sử dụng gem cells trong ruby on rails"
date:   2017-07-14
summary: Cells là 1 gem được sử dụng để gộp cả view lẫn controller lại thành 1 package.
categories: [Ruby]
tags: ["Ruby"]
image: images/cells.jpg
author: Trong Huu
---

### Giới thiệu:

Trong mô hình MVC của Rails, mối quan hệ View - Models là rất quan trọng và có thể xây dựng nó theo OOP để việc tái sử dụng các hàm logic phức tạp trở lên dễ dàng, đồng thời giảm nhẹ và tối ưu code cho Models trong các dự án lớn.

Method “partial” có lẽ đã trở lên rất quen thuộc đối với mỗi lập trình viên Ruby on Rails, ưu điểm của nó là có thể gộp và tái sử dụng code View nhiều lần, tuy nhiên khi cần phải thao tác với nhiều hàm logic và muốn gộp chúng lại thành một Package để sử dụng thì vấn đề trở lên rất phức tạp và không khả thi.

Cells là một lựa chọn tuyệt vời cho bạn để giải quyết vấn đề này.

### Cells là gì?

Cells là 1 gem được sử dụng để gộp cả view lẫn controller lại thành 1 package, phát huy tối đa tính đóng gói, kế thừa, đặc biệt với những view có logic phức tạp, thì cells sẽ giúp code trở nên dễ đọc và dễ dàng tái sử dụng hơn.

### Cài đặt gem Cells trong Rails

Thêm vào Gemfile trong ứng dụng :

{% highlight ruby %}
gem 'cells'
{% endhighlight %}

Để sử dụng bạn cần generate :

{% highlight ruby %}
rails generate cell popup_login show -e haml
{% endhighlight %}

Gọi partial show trong gem cells như sau:

{% highlight ruby %}
= render_cell :popup_login, :show
{% endhighlight %}

### Ví dụ với Popup Login

- Khi sử dụng partial :
header_sign_in.html.haml

{% highlight ruby %}
- unless user_signed_in?
  .header
    .header__login-button
      = link_to 'login', 'javascript:void(0);'
= render 'popup_login'
{% endhighlight %}

_popup_login.html.haml

{% highlight ruby %}
- unless user_signed_in?
  .popup-login
    = form_for(@user, url:user_session_path) do |f|
      %dl
        %dt= f.label :email
        %dd= f.email_field :email
        %dt= f.label :password
        %dd= f.password_field :password
      = f.submit 'Login'
{% endhighlight %}

Sẽ có lỗi xảy ra nếu object @user dùng trong file _popup_login.html.haml chưa được định nghĩa trong controller.

Ta có thể định nghĩa @user trong tất cả các action có gọi _popup_login, tuy nhiên một giải pháp tốt hơn là định nghĩa trong application_controller.

application_controller.rb

{% highlight ruby %}
class ApplicationController < ActionController::Base
  before_action: :pre_load
  def pre_load
    @user = User.new unless current_user.present?
  end
end
{% endhighlight %}

Cấu trúc thư mục Cells sẽ như sau :

{% highlight ruby %}
/app
/assets
/cells
/popup_login
show.html.haml
popup_login_cell.rb
{% endhighlight %}

Cụ thể, chức năng popup_login sẽ được viết như sau :

app/cells/popup_login_cells.rb

{% highlight ruby %}
class PopupLoginCell < Cell::Rails
  def show
    @user = User.new unless current_user.present?
    render
  end
end
{% endhighlight %}

app/cells/popup_login/show.html.haml

{% highlight ruby %}
- unless user_signed_in?
  .popup-login
    = form_for(@user, url:user_session_path) do |f|
      %dl
        %dt= f.label :email
        %dd= f.email_field :email
        %dt= f.label :password
        %dd= f.password_field :password
      = f.submit 'Login'
   :javascript
     $('.header__login-button a').click(function(){
       $('.popup-login').css({ 'display' : 'block'});
     });
{% endhighlight %}

Khi partial show được gọi, cells sẽ tự động gọi controller show tương ứng và thực hiện các login cần thiết.

Với cách làm này, những hàm logic sẽ chỉ được thực hiện khi gọi cells (tối ưu hơn hẳn khi dùng application_controller), và ta chỉ cần quan tâm đến việc gọi partial trong view là đủ.
