---
layout: post
title:  Rail 5 Action Cable
date:   2016-06-26
summary: Bài viết giới thiệu về tính năng action cable của Rails 5
categories: [ruby on rails]
tags: ["ruby", "rails"]
images: /images/rails5.png
author: Vy Dao
---

Bản Rails 5 rc2 vừa được release là hoạt động khá tốt, bên cạnh đó Rails 5 cũng giới thiệu hỗ trợ WebSocket qua Action Cable.
Đây được coi là một tin tốt cho cộng đồng Rails trong việc xây dựng các ứng dụng realtime sẽ dễ dàng hơn rất nhiều. Tuy nhiên, bên cạnh những
điểm mạnh thì luôn tồn tại song song những điểm yếu mà chúng ta cần phải tìm hiểu kĩ trước khi đưa vào project của mình.
Cùng tìm hiểu qua về tính năng ActionCable của Rails 5

Actioncable là một bước tiến đáng kể cho nền tảng Rails, nó cung cấp cơ chế để bạn đưa Rails app hoặc một phần nào của app có thể thực thi được tính năng realtime thông qua công nghệ WebSocket với phần hỗ trợ ở client là code Javascript và phần server là Ruby. Vì được tích hợp vào Rails nên bạn hoàn toàn có thể xử lý các truy vấn đến CSDL một cách dễ dàng thông qua ActiveRecord hoặc ORM khác.

## HTTP và Websockets
Đối với HTTP, kết nối giữa server và client thì khá là ngắn: đầu tiên client request một resource trên server thì một kết nối đến server sẽ được thiết lập và resource được yêu cầu(có thể là JSON, HTML, XML,...) sẽ được truyền về cho client như là một response.
Sau đó, kết nối sẽ bị đóng. Câu hỏi đặt ra là làm thế nào để client biết được khi data từ phía server có sự thay đổi? Thông thường, HTTP sẽ sử dụng "long polling", client sẽ gửi yêu cầu đến server để nhận những thay đổi trong một khoảng thời gian nhất định.

Không giống như HTTP, WebSockets là giao thức cho phép các client và server giữ một kết nối mở cho phép truyền dữ liệu trực tiếp. Các client đăng kí một kết nối mở với server và khi có những thông tin mới thì server sẽ broadcasts dữ liệu đó đến tất cả các client đã đăng kí. Bằng cách này, cả server và client đều biết về trạng thái của dữ liệu và có thể dễ dàng đồng bộ hóa các xuất hiện các thay đổi.

![http-websocket](/images/http-websocket.png)

Các controller của Rails được xây dựng nhằm mục đích xử lý các request HTTP và Rails đã đưa ra một giải pháp để handle việc tích hợp xử lý Websockets. Rails 5 sẽ có thêm một thư mục mới bên trong thư mục app gọi là channels. Channels hoạt động như các controller xử lý các request Websockets bằng cách đóng gói các logic thành các đơn vị đặc thù, ví dụ như là chat messages hoặc notifications, các client đăng kí các channels để truyền tải dữ liệu.

## Cài đặt ActionCable
Để sử dụng được tính năng mới ActionCable chúng ta cần cài version mới nhất của Rails 5 và một trong những yêu cầu tiên quyết của Rails 5 là bạn phải có Ruby version 2.2.4 hoặc cao hơn

{% highlight ruby %}
gem install rails --pre --no-ri --no-rdoc
{% endhighlight %}

ActionCable có thể chạy trên một stand-alone server hoặc chúng ta có thể config để nó chạy trên cùng processes với app server.

Để xây dựng ứng dụng ActionCable chúng ta cần chuẩn bị:

- ActionCable yêu cầu adapter là PostgreSQL, do đó chúng ta cần cài đặt PostgreSQL trước.

- ActionCable sử dụng "Rack socket hijacking API" để điểu khiển các kết nối từ app server, do đó cần một Rack-Server hỗ trợ ActionCable, trong ví dụ này chúng ta sẽ sử dụng Puma, tuy nhiên hiện tại Passenger 5.0 đã có hỗ trợ.

- Cuối cùng, ActionCable sử dụng Redis để lưu trữ dữ liệu tạm thời, đồng bộ các nội dung của ứng dụng.

![rails5](/images/rails-rack.png)

## Xây dựng một Real-Time Chat App với Action Cable
Đầu tiên tạo một project rails 5

{% highlight ruby %}
rails new action-cable-demo --database=postgresql
{% endhighlight %}

Sau khi tạo xong project,cần config lại `database.yml` cho chính xác với PostgreSQL và `Gemfile`

{% highlight ruby %}
# Gemfile
gem 'rails', '~> 5.0.0'
gem 'redis', '~> 3.0'
gem 'puma', '~> 3.0'
{% endhighlight %}

Và chạy `bundle install`.

Bây giờ chúng ta sẽ xem qua tổng quan của ứng dụng:
Một chat room sẽ có nhiều messages, một messages sẽ có nội dung và nó sẽ thuộc về một user trong một chat room. Một user sẽ có username và tất nhiên sẽ có nhiều messages.

{% highlight ruby %}
# app/models/chatroom.rb

class Chatroom < ApplicationRecord
  has_many :messages, dependent: :destroy
  has_many :users, through: :messages
  validates :topic, presence: true, uniqueness: true, case_sensitive: false
end
{% endhighlight %}

{% highlight ruby %}
# app/models/message.rb

class Message < ApplicationRecord
  belongs_to :chatroom
  belongs_to :user
end
{% endhighlight %}

{% highlight ruby %}
# app/models/user.rb

class User < ApplicationRecord
  has_many :messages
  has_many :chatrooms, through: :messages
  validates :username, presence: true, uniqueness: true
end
{% endhighlight %}

Mình sẽ bỏ qua các bước tạo routes và controllers, lúc này xem như user đã có thể login bằng username và join vào chat room và post messages thông qua form trong chat room show page.

{% highlight ruby %}
# app/controllers/chatrooms_controller.rb

class ChatroomsController < ApplicationController
  ...
  def show
    @chatroom = Chatroom.find_by(slug: params[:slug])
    @message = Message.new
  end
end
{% endhighlight %}

Tiếp theo, tạo view cho chatroom

{% highlight ruby %}
# app/views/chatrooms/show.html.erb

<div class="row col-md-8 col-md-offset-2">
  <h1><%= @chatroom.topic %></h1>

<div class="panel panel-default">
  <% if @chatroom.messages.any? %>
    <div class="panel-body" id="messages">
      <%= render partial: 'messages/message', collection: @chatroom.messages%>
    </div>
  <% else %>
    <div class="panel-body hidden" id="messages">
    </div>
  <% end %>
</div>

  <%= render partial: 'messages/message_form', locals: {message: @message, chatroom: @chatroom}%>
</div>
{% endhighlight %}

Và form để user post messages

{% highlight ruby %}
# app/views/messages/_message_form.html.erb

<%=form_for message, remote: true, authenticity_token: true do |f|%>
  <%= f.label :your_message%>:
  <%= f.text_area :content, class: "form-control", data: {textarea: "message"}%>

  <%= f.hidden_field :chatroom_id, value: chatroom.id %>
  <%= f.submit "send", class: "btn btn-primary", style: "display: none", data: {send: "message"}%>
<% end %>
{% endhighlight %}

Viết code cho MessagesController để xử lý yêu cầu tạo message

{% highlight ruby %}
class MessagesController < ApplicationController
  def create
    message = Message.new(message_params)
    message.user = current_user
    if message.save
      # do some stuff
    else
      redirect_to chatrooms_path
    end
  end

  private
    def message_params
      params.require(:message).permit(:content, :chatroom_id)
    end
end
{% endhighlight %}

Đến đây ứng dụng chat của chúng ta đã có thể chạy được, bây giờ chúng ta sẽ tiếp tục xử lý real-time messages với Action Cable.

Cách tổ chức code của Action Cable như sau

{% highlight ruby %}
├── app
    ├── channels
        ├── application_cable
        ├── channel.rb
        └── connection.rb
{% endhighlight %}

Module ApplicationCable định nghĩa sẵn 2 class `Channel` và `Connection`

- `Connection` class sẽ là nơi mà chúng ta thực hiện authorize với những kết nối đến.

- `Channel` class sẽ là nơi chứa các shared logic giữa các channels mà chúng ta sẽ định nghĩa.

### Thiết lập Websocket connection

__Bưóc 1: Thiết lập socket connection phía server__

Thêm vào file `routes.rb`

{% highlight ruby %}
Rails.application.routes.draw do
  mount ActionCable.server => '/cable'
  resources :chatrooms, param: :slug
  resources :messages
end
{% endhighlight %}

Bây giờ, Action Cable sẽ lắng nghe Websocket requests từ `localhost:3000/cable`. Khi ứng dụng chính của chúng ta đưọc khởi tạo thì một thực thể của Action Cable cũng được tạo ra. Action Cable sẽ thiết lập một socket connection trên `localhost:3000/cable` và bắt đầu lắng nghe socket requests.

__Bước 2: Thiết lập socket connection phía client__

Trong thư mục `app/assets/javascripts/channels` chúng ta sẽ tạo một file `chatrooms.js` để định nghĩa thực thể của websocket connection phía client

{% highlight js %}
// app/assets/javascripts/channels/chatrooms.js

//= require cable
//= require_self
//= require_tree .

this.App = {};
App.cable = ActionCable.createConsumer();
{% endhighlight %}

Thêm require thư mục channels trong `application.js` file:

{% highlight js %}
// app/assets/javascripts/application.js

//= require_tree ./channels
{% endhighlight %}

__Bước 3: Tạo channel__

Chúng ta đã thiết lập một persistent connection, lắng nghe mọi websocket requests đến `ws://localhost:3000/cable`. Bây giờ chúng ta cần tạo một channel để phát sóng và truyền tải tin nhắn.

Tạo một file `app/channels/messages_channel` và định nghĩa channel của chúng ta đưọc kế thừa từ class `ApplicationCable::Channel`

{% highlight ruby %}
# app/channels/messages_channel.rb

class MessagesChannel < ApplicationCable::Channel
end
{% endhighlight %}

Messages Channel sẽ chứa một method `subscribed`, method này có trách nhiệm đăng ký và truyền tải thông điệp được broadcast trên channel này.

{% highlight ruby %}
# app/channels/messages_channel.rb
class MessagesChannel < ApplicationCable::Channel
  def subscribed
    stream_from 'messages'
  end
end
{% endhighlight %}

__Bước 4: Broadcast đến Channel__

Khi có một tin nhắn mới nó sẽ được lưu xuống db và ngay lập tức được broadcast đến Message Channel, vì vậy chúng ta sẽ đặt phần code xử lý việc broadcast trong action `create` của Messages Channel.

{% highlight ruby %}
#  app/controllers/messages_controller.rb

class MessagesController < ApplicationController
  def create
    message = Message.new(message_params)
    message.user = current_user
    if message.save
      ActionCable.server.broadcast 'messages',
        message: message.content,
        user: message.user.username
      head :ok
    end
  end
end
{% endhighlight %}

Chúng ta gọi đến method `broadcast` của Action Cable server và truyền kèm 1 vài tham số

__`mesages`__ là tên của channel chúng ta đang thực hiện broadcast.

Kèm theo là nội dung sẽ được gửi qua channel dưới dạng JSON

- `message` là nội dung của tin nhắn chúng ta vừa tạo.

- `user` là username của user tạo ra tin nhắn.

__Bước 5: Action Cable với Redis__

Action Cable sử dụng Redis để gửi và nhận messages thông qua channel. Redis đóng vai trò lưu trữ dữ liệu và đảm bảo các messages sẽ được đồng bộ trong ứng dụng của chúng ta.

Action Cable sẽ tìm cấu hình của Redis trong file `config/cable.yml` đã đưọc tạo ra khi chúng ta tạo ứng dụng ban đầu.

{% highlight ruby %}
development:
  adapter: async

test:
  adapter: async

production:
  adapter: redis
  url: redis://localhost:6379/1
{% endhighlight %}

Bây giờ, chúng ta cần thêm một subscription để đăng ký với Messages Channel.

Tạo file `app/assets/javascripts/channels/messages.js` để định nghĩa subscription:

{% highlight ruby %}
// app/assets/javascripts/channels/messages.js

App.messages = App.cable.subscriptions.create('MessagesChannel', {
  received: function(data) {
    $("#messages").removeClass('hidden')
    return $('#messages').append(this.renderMessage(data));
  },

  renderMessage: function(data) {
    return "<p> <b>" + data.user + ": </b>" + data.message + "</p>";
  }
});
{% endhighlight %}

Chúng ta thêm subscription cho client với `App.cable.subscriptions.create` kèm theo tên của channel muốn đăng ký, ở đây là Message Channel.

Khi function `subscriptions.create` được gọi, nó sẽ gọi callback đến method `MessagesChannel#subscribed`. `MessagesChannel#subscribed` có trách nhiệm truyền tải messages được broadcast trên Messages channel dưới dạng JSON đến phía client đã đăng ký channel.

Sau đó, khi phía client nhận được message mới dạng JSON nó sẽ gọi đến một helper function `renderMessage` với chức năng đơn giản là append message mới với DOM và message đưọc hiển thị trên chat room.

__Tham khảo__

- [PluralSight Creating a chat using Rails' Action Cable](http://tutorials.pluralsight.com/ruby-ruby-on-rails/creating-a-chat-using-rails-action-cable)

- [DHH Rails 5 Action Cable Example](https://github.com/rails/actioncable-examples)

- [Action Cable – Integrated WebSockets for Rails](https://github.com/rails/rails/tree/master/actioncable)

