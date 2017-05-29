---
layout: post
title:  "Áp dụng Google AMP cho ứng dụng Rails"
date:   2017-05-27
summary: Bài viết tổng hợp các lưu ý đẻ tạo AMP view tiêu chuẩn cho ứng dụng Rails
categories: [AMP]
tags: ["amp"]
images: /images/amp.jpg
author: Vy Dao
---

## 1. Giới thiệu
Google AMP là tiêu chuẩn mới để xây dựng các trang web cho mobile với tốc độ tải nhanh hơn. Cho phép tải trang gần như ngay lập tức.

Tổng quan [Google AMP](https://www.ampproject.org/)

## 2. Các Validation quan trọng của AMP cần lưu ý

[https://www.ampproject.org/docs/reference/validation_errors](https://www.ampproject.org/docs/reference/validation_errors)

**Sử dụng tool [AMP test](https://search.google.com/search-console/amp) để check AMP page pass tất cả validation.**

### TL;DR

**CSS**
- Không đưọc sử dụng inline CSS và external CSS.
- Add internal CSS bằng thẻ `<style amp-custom></style>`.
- Internal CSS không vượt quá 50KB.
- Internal CSS không đưọc chứa `!important` `charset`.

**JS**
- Không được include bất cứ file hoặc đoạn js nào, nếu cần thì phải sử dụng các component của AMP
- Chỉ đưọc include thẻ `<script>` với type là `application/ld+json`
- Không cho phép chứa function `javascript` ví dụ: `href=javascript:;`
- Không cho phép sử dụng `onclick`, có thể dùng `on` để thay thế. Cú pháp `eventName:targetId[.methodName[(arg1=value, arg2=value)]`

  Ví dụ:
   ```html
    <a on="tap:targetForm.hide">Click</a>
    <form id="targetForm" on="submit.modal-id.show"></form>
  ```
- Sử dụng `amp-lightbox` thay thế modal bootstrap.
- Khi muốn toggle show/hide một đối tượng thì sử dụng `toggleVisibility`

  Ví dụ:
  ```html
    <button on="tap:targetForm.toggleVisibility">Toggle</button>
  ```

  Lưu ý, để sử dụng đưọc `toggleVisibility` thì không được ẩn object bằng `display: none` nó sẽ ngăn cản AMP thay đổi trạng thái của đối tượng, sử dụng attribute `hidden` để thay thế.

**Google Analytics**
- Sử dụng `amp-analytic` để thực hiện tracking của GA

**Images and Iframe**
- Sử dụng `amp-img` thay cho thẻ `img`. Lưu ý, cần phải cung cấp `width` và `height`, nếu sử dụng responsive có thể thêm `layout="responsive"`
- Sử dụng attribute `fallback` để load hình ảnh thay thế khi ảnh gốc bị lỗi hoặc load chậm
  Ví dụ:
  ```html
    <amp-img alt="Mountains"
    width="550"
    height="368"
    src="images/mountains.webp">
    <amp-img alt="Mountains"
      fallback
      width="550"
      height="368"
      src="images/mountains.jpg"></amp-img>
    </amp-img>
  ```
- Sử dụng `amp-iframe` thay thế cho `iframe`, `src` của `amp-iframe` phải là `https`.

**Display dynamic data**
- Có thể display dữ liệu động bằng `amp-binding`

  Ví dụ: submit form bằng ajax và hiển thị kết quả trên modal

  ```html
    <form on="submit-success:targetLightbox, AMP.setState({ formResponse: event.response })">
      <button type="submit">Submit</button>
    </form>

    <amp-lightbox id="targetLightbox" layout="nodisplay">
      <div class="lightbox">
        <amp-img src="my-full-image.jpg" width=300 height=800 on="tap:my-lightbox.close">
        <p [text]="formResponse.mesage"></p>
        <p [text]="formResponse.status"></p>
      </div>
    </amp-lightbox>
  ```

  Lưu ý: `amp-binding` là feature đang trong giai đoạn thử nghiệm [Experimental features](https://ư.ampproject.org/dóc/refernce/experimental) vì vậy cần phải enable feature này trên môi trường development bằng cách thực hiện lệnh
    ```js
      AMP.toggleExperiment('experiment')
    ```
  trên `Chrome devtools console`.

  Nếu muốn sử dụng `amp-binding` trên môi trường Production thì cần phải đăng kí origin trials [Tại đây](https://dóc.google.com/a/google.com/forms/d/e/1FAIpQLSfGCAjUU4pDu84Sclw6wjGVDiFJhVr61pYTMehIt6ẽ4ửm1Q/viewform) sau khi đăng kí thành công chúng ta sẽ nhận đưọc 1 token, add token vào thẻ `<head>`

  ```html
    <meta name="amp-experiment-token" content="HfmyLgNLmblRg3Alqy164Vywr">
  ```

**Form**
- Bắt buộc phải chứa attribute `target` bằng `_top` hoặc `_blank`
- Sử dụng `amp-form` thay cho `form_tag`, lưu ý cần add thêm authenticate token của rails vào form

  ```ruby
    <%= hidden_field_tag :authenticity_token, form_authenticity_token -%>
  ```
- Các events quan trọng `submit`, `submit-success`, `submit-error`

**Others**
- Trên môi trường Development nếu sử dụng gem `rack-mini-profiler` hoặc `xray-rails` thì cần phải disable trước khi test vì những gem này sẽ add những đoạn script vào page của chúng ta dẫn đến AMP validate sẽ bị fail.
- Đối với Production, script tracking của `NewRelic` sẽ gây fail, tham khảo [NewRelic Disable monitoring](https://docs.newrelic.com/docs/browser/new-relic-browser/installation-configuration/disable-browser-monitoring-specific-pages) để fix.

**Security**
- CORS Requests in AMP: Khi user tìm thấy page AMP của chúng ta trên kết quả tìm kiếm của của Google và click vào xem thì họ sẽ access vào một cached page từ Google AMP Cache, có nghĩa là user không access trực tiếp vào trang AMP của chúng ta mà thông qua một platform khác, ở đây, Google Search sử dụng Google AMP Cache để render AMP page một cách nhanh chóng, cached page đưọc cung cấp bởi Google AMP Cache với một domain khác, do đó chúng ta cần phải handle CORS request.

Tham khảo [CORS Security in AMP](https://github.com/ampproject/amphtml/blob/master/spec/amp-cors-requests.md)

## 3. Apply AMP cho ứng dụng Rails

#### 3.1 Định nghĩa một MIME type mới

Để tạo view mới cho những pages đã có sẵn mà không cần thay đổi nhiều ở controller thì cách dễ dàng nhất là định nghĩa một mime type mới.
Ví dụ: Ta muốn `/products/abc` sẽ load view HTML chuẩn, còn `/products/abc.amp` sẽ load view của AMP
Đầu tiên, tạo một mime type mới trong `config/initializers/mime_types.rb`

```ruby
  Mime::Type.register 'text/html', :amp
```

Tạo một layout mới cho AMP version tuân thủ theo các tiêu chuẩn của AMP

- Bắt đầu bằng doctype `<!doctype html>`.
- top-level tag `<html ⚡>` hoặc `<html amp>`.
- Chứa thẻ `<head>` và `<body>`.
- Chứa thẻ `<meta charset="utf-8">` là `first child` của thẻ `<head>`.
- Chứa thẻ `<script async src="https://cdn.ampproject.org/v0.js"></script>` là `second child` của thẻ `<head>`.
- Chứa thẻ `<link rel="canonical" href="$SOME_URL" />` bên trong thẻ `<head>` để chỉ đển version HTML của page.
- Chứa thẻ `<meta name="viewport" content="width=device-width,minimum-scale=1">` bên trong thẻ `<head>`.
- Chứa `amp-boilerplate` bên trong thẻ `<head>`
  ```html
    <style amp-boilerplate>body{-webkit-animation:-amp-start 8s steps(1,end) 0s 1 normal both;-moz-animation:-amp-start 8s steps(1,end) 0s 1 normal both;-ms-animation:-amp-start 8s steps(1,end) 0s 1 normal both;animation:-amp-start 8s steps(1,end) 0s 1 normal both}@-webkit-keyframes -amp-start{from{visibility:hidden}to{visibility:visible}}@-moz-keyframes -amp-start{from{visibility:hidden}to{visibility:visible}}@-ms-keyframes -amp-start{from{visibility:hidden}to{visibility:visible}}@-o-keyframes -amp-start{from{visibility:hidden}to{visibility:visible}}@keyframes -amp-start{from{visibility:hidden}to{visibility:visible}}</style><noscript><style amp-boilerplate>body{-webkit-animation:none;-moz-animation:none;-ms-animation:none;animation:none}</style></noscript>
  ```
- Ngoài ra, bên trong thẻ `<head>` có thẻ chứa `metadata` định nghĩa structure của page. Phần này sẽ đưọc đề cập ở phần sau.

File `app/views/layouts/application.amp.erb` sẽ có nội dung như sau

```html
  <!doctype html>
  <html ⚡>
    <head>
      <meta charset="utf-8">
      <script async src="https://cdn.ampproject.org/v0.js"></script>
      <link rel="canonical" href="<%= url_for(format: :html, only_path: false) %>" >
      <meta name="viewport" content="width=device-width,minimum-scale=1,initial-scale=1">
      <style amp-boilerplate>body{-webkit-animation:-amp-start 8s steps(1,end) 0s 1 normal both;-moz-animation:-amp-start 8s steps(1,end) 0s 1 normal both;-ms-animation:-amp-start 8s steps(1,end) 0s 1 normal both;animation:-amp-start 8s steps(1,end) 0s 1 normal both}@-webkit-keyframes -amp-start{from{visibility:hidden}to{visibility:visible}}@-moz-keyframes -amp-start{from{visibility:hidden}to{visibility:visible}}@-ms-keyframes -amp-start{from{visibility:hidden}to{visibility:visible}}@-o-keyframes -amp-start{from{visibility:hidden}to{visibility:visible}}@keyframes -amp-start{from{visibility:hidden}to{visibility:visible}}</style><noscript><style amp-boilerplate>body{-webkit-animation:none;-moz-animation:none;-ms-animation:none;animation:none}</style></noscript>
    </head>
    <body>
      <div class="amp">
        <%= yield %>
      </div>
    </body>
  </html>
```

#### 3.2 Add custom CSS cho AMP

- Tạo một file sass mới cho AMP `app/assets/stylesheets/amp/application.scss`

```
  body {
    ...some styles here...
  }
```
- Register file này vào precompilation bằng cách add vào `config/application.rb`

```ruby
  config.assets.precompile << 'amp/application.scss'
```

- Do AMP chỉ chấp nhận internal CSS nên ta sẽ phải copy tất cả sass đã compile vào view bằng cách add code sau vào thẻ `<head>`

```ruby
  <% if Rails.application.assets && Rails.application.assets['amp/application'] %>
    <style amp-custom><%= Rails.application.assets['amp/application'].to_s.html_safe %></style>
  <% else %>
    <style amp-custom><%= File.read "#{Rails.root}/public#{stylesheet_path('amp/application', host: nil)}" %></style>
  <% end %>
```

- ở môi trường development ta sử dụng `Rails.application.assets` helper để get tất cả sass compiled của AMP.
- ở môi trường production do variable này là `nil` nên chúng ta phải đọc compiled file trong thư mục `public/assets`.

#### 3.3 Chỉ định AMP page để Google có thể index
- Sau khi hoàn thành page AMP chúng ta cần phải giúp Google bot tìm thấy và index chúng bằng cách gắn các thẻ `<link rel="canonical">`
- Gắn vào layout của page AMP link đến page format html

```ruby
  <link rel="canonical" href="<%= url_for(format: :html, only_path: false) %>" >
```

- ở layout non-AMP link đến page AMP

```ruby
  <link rel="amphtml" href="<%= url_for(format: :amp, only_path: false) %>" >
```
