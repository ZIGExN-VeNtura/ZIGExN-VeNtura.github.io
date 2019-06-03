---
layout: post
title: "Rails development với Docker Compose"
date:   2016-12-26
summary: Giới thiệu cách setup môi trường development cho một dự án Rails cơ bản.
categories: [Docker]
tags: ["docker", "docker-compose", "rails", "devops"]
image: assets/images/docker-funny.png
author: Son Dang
---

![docker](/assets/images/docker-compose.png)

Docker là một công cụ hữu ích khi bạn muốn xây dựng một môi trường development tương đồng với production. Với Docker Compose, Docker càng giúp ích hơn cho developer với nhu cầu tách các dependencies như MySQL, Solr hay Redis ra thành các components riêng biệt. Trong bài viết này, mình sẽ hướng dẫn các bạn các thiết lập môi trường development cho Rails app đơn giản với Docker và Docker Compose.

## Cài đặt Docker và Docker Compose

### Cài đặt Docker cho Ubuntu 16.04

* Update apt sources
{% highlight bash %}
$ sudo apt-get update
$ sudo apt-get install apt-transport-https ca-certificates
{% endhighlight %}

* Thêm GPG key
{% highlight bash %}
$ sudo apt-key adv \
              --keyserver hkp://ha.pool.sks-keyservers.net:80 \
              --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
{% endhighlight %}

* Thêm Docker repo vào source list
{% highlight bash %}
$ echo "deb https://apt.dockerproject.org/repo ubuntu-xenial main" | sudo tee /etc/apt/sources.list.d/docker.list
{% endhighlight %}

* Update apt package
{% highlight bash %}
$ sudo apt-get update
{% endhighlight %}

* Cài đặt `docker-engine`
{% highlight bash %}
$ sudo apt-get install docker-engine
{% endhighlight %}

* Khởi động `service` cho docker
{% highlight bash %}
$ sudo service docker start
{% endhighlight %}

### Cài đặt Compose cho Ubuntu 16.04
* Cài đặt từ source
{% highlight bash %}
$ curl -L "https://github.com/docker/compose/releases/download/1.9.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
{% endhighlight %}

* Thêm quyền `executable` cho `docker-compose`
{% highlight bash %}
$ chmod +x /usr/local/bin/docker-compose
{% endhighlight %}

## Phát triển Rails với Docker và Docker Compose

### Setup Docker

#### Dockerfile
Trong thư mục Rails project, tạo `Dockerfile` với nội dung:
{% highlight ruby %}
# Dockerfile
FROM ruby:2.3.3

RUN apt-get update -qq && apt-get install -y build-essential

# postgres
RUN apt-get install -y libpq-dev

# nokogiri
RUN apt-get install -y libxml2-dev libxslt1-dev

# capybara-webkit
RUN apt-get install -y libqt4-webkit libqt4-dev xvfb

# JS runtime
RUN apt-get install -y nodejs

ENV APP_HOME /myapp
RUN mkdir $APP_HOME
WORKDIR $APP_HOME

ADD Gemfile* $APP_HOME/
RUN bundle install

ADD . $APP_HOME
{% endhighlight %}

Mình sẽ giải thích một vài command ở `Dockerfile` trên:

1. `FROM`: Chỉ định image mà sử dụng để làm base cho container. Trong trường hợp này, chúng ta sử dụng ruby 2.3.3
2. `RUN`: Chỉ định command được sử dụng khi build
3. `ENV`: Set biến môi trường
4. `WORKDIR`: Chỉ định thư mục base. Tất cả command được gọi sau đó sẽ được thực thi bên trong thư mục này.
5.  `ADD`: Copy files từ máy host vào container
Mỗi command này sẽ tạo ra một cached image mà chúng ta có thể xây dựng tiếp lên trên.
Để build docker image, trong thư mục chứa `Dockerfile`, chạy lệnh:
`docker build .`

####  Compose file
Chúng ta đã xây dựng thành công một Docker image để chứa Rails, tuy nhiên chúng ta vẫn còn cần phải thiết định cho các dependencies như database hay search engine. Lúc này thì chúng ta sẽ sử dụng Docker Compose.
Trong cùng thư mục chứa Rails, tạo file `docker-compose.yml` có nội dung:
{% highlight ruby %}
# config/database.yml
version: '2'
services:
  db:
    image: postgres:9.6.1
    ports:
      - "5432:5432"
    volumes:
      - /data/postgresql-9.6.1:/var/lib/postgresql/data

  web:
    build: .
    command: bin/rails server --port 3000 --binding 0.0.0.0
    ports:
      - "3000:3000"
    depends_on:
      - db
    volumes:
      - .:/myapp
{% endhighlight %}

Trong file Docker Compose này chúng ta đã chỉ định 2 `services` sẽ chạy song song là `web` và `db`.

* `db` được chỉ định sẽ sử dụng image `postgres:9.6.1` qua`image: postgres:9.6.1` và có thể truy xuất qua cổng `5432`. Thiết định mặc định của `ports` là `<HOST_PORT>:<DOCKER_PORT>`. `volumes` sẽ kết nối thư mục ở host vào thư mục được chỉ định trên container; ở đây chúng ta kết nối thư mục `/data/postgres-9.6.1` trong Rails root vào thư mục `/var/lib/postgresql/data` trong container; nhờ vậy khi có bất kì thay đổi gì trong database, data sẽ được lưu xuống host thay vì chỉ nằm trên container tránh tình trạng mất dữ liệu khi "lỡ tay" xóa container.
* `web` sử dụng `Dockerfile` thông qua `build: .` và truy xuất qua cổng `3000`. `depends_on` sẽ chỉ định services mà `web` sẽ cần và từ đó chạy services đó trước khi chạy `web`; ở đây là `db`. Chúng ta kết nối thư mục Rails vào thư mục `/myapp` trong container thông qua `volumes`

Lúc này trong thư mục gốc của Rails, để bắt đầu build và khởi động container, bạn chạy lệnh:

{% highlight bash %}
$ docker-compose up
{% endhighlight %}

### Thiết lập cho database
Do chúng ta kết nối tới database trên container, nên sẽ cần chỉnh sửa file `config/database.yml`:
{% highlight ruby %}
# docker-compose.yml
development: &default
  adapter: postgresql
  database: myapp_development
  pool: 5
  username: postgres
  host: db

test:
  <<: *default
  database: myapp_test
{% endhighlight %}

Sau khi thiết lập kết nối với database từ Rails app, ta có thể chạy lệnh tạo và migrate database qua:
`docker-compose run web rake db:create db:setup`

### Tương tác với container

#### Tương tác với console
Bạn có thể mở Rails console thông qua command: `docker-compose run web rails console`

Cấu trúc của command để tương tác với container là `docker-compose run {CONTAINER_NAME} {COMMAND}`

#### Một vài command hữu ích khác
* Truy cập shell của `web`: `docker-compose exec -it web bash`
* Kết nối với server console: `docker attach <CONTAINER_FULL_NAME>`
* Khởi động lại container `web`: `docker-compose restart web`
* List tất cả các container đang hoạt động: `docker ps`
* Ngoài ra còn nhiều command khác, bạn có thể tham khảo tại [đây](https://docs.docker.com/compose/reference/)

## Kết luận
Trên đây là một ví dụ nhỏ về việc xây dựng môi trường cho Rails app với Docker. Việc thêm vào các services khác trong cùng một application như Redis hay Solr là việc vô cùng đơn giản với Docker Compose.

## Tham khảo
* [Docker documents](https://docs.docker.com/)
* [Docker compose documents](https://docs.docker.com/compose/reference/)
