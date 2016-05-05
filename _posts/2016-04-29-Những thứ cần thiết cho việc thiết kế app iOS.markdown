---
layout: post
title:  "Những thứ cần thiết cho việc thiết kế app iOS"
date:   2016-04-29 23:55:00
summary: Trong bài viết này mình sẽ tóm tắt một vài thông số cơ bản của các element iOS và những điểm cần lưu ý khi thiết kế ứng dụng.
categories: [web development]
tags: ["iOS"]
images: /images/ios_image_top.png
author: ThaoNNM
---
Trước khi bạn bắt đầu thiết kế, hãy xem qua [Human Interface Guidelines](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/MobileHIG/index.html?utm_source=twitterfeed&utm_medium=twitter) cho iOS - gọi tắt là [HIG](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/MobileHIG/index.html?utm_source=twitterfeed&utm_medium=twitter) - của Apple để chắc chắn mình sẽ không vi phạm bất kỳ nguyên tắc cơ bản áp dụng cho việc thiết kế.

Trong bài viết này mình sẽ tóm tắt một vài thông số cơ bản của các element iOS và những điểm cần lưu ý khi thiết kế ứng dụng.

### Một vài thông số cơ bản

  __1. Độ phân giải màn hình của từng loại máy__
  
  ![IOS apps](/images/ios_resolution.jpg)
  
  __2. Mật độ điểm ảnh, chế độ màu, nhiệt độ màu__
  
  ![IOS apps](/images/ios_color.jpg)
  
  __3. Kích thước của icon__
  
  ![IOS apps](/images/ios_icon.png)
  
  __4. Kiểu chữ__
  
  ![IOS apps](/images/ios_fontsize.jpg)
  
### Những điểm cần lưu ý trong thiết kế

  __1. Nội dung trên mỗi màn hình__
  
  Apps không phải là website, bạn không thể thiết kế ứng dụng giống như trang web. So với website, iPhone iPad có không gian màn hình thậm chí còn rất hạn chế. Do đó, khi thiết kế một màn hình trong ứng dụng, đừng làm cho người dùng quá tải với nhiều nội dung của bạn. Sau đây là một số cách để bạn có thể làm được điều này:
  
  **Tập trung vào chức năng chính**
  
  Bạn cần phân tích và xác định những gì là quan trọng trong từng màn hình để hiển thị nó. 
  
  Ví dụ:  Màn hình Hiển thị thời tiết chỉ tập trung vào hiển thị ngày và nhiệt độ, thời tiết khí hậu của từng ngày. 
  
  ![Weather](/images/ios_weather.jpg)
  
  **Làm nổi bật những thông tin người dùng quan tâm**
  
  Bạn có thể giúp cho người dùng tập trung vào những khung thông tin được thiết kế tinh tế và nổi bật.
  
  ![quan tâm](/images/ios_screenshot.jpg)
  
  Như hình trên, mặc dù có nhiều button khác trên cùng một màn hình nhưng chúng không cạnh tranh để thu hút sự chú ý cho hành động chính.
  
  Ngoài ra, đặt những thông tin cần thiết ở phía trên cùng của màn hình là một cách tốt. Vì nơi này là nơi dễ cho mọi người thấy nhất.
  
  __2. Thiết kế phổ biến và sử dụng dễ dàng__
  
  Người dùng đã quen với việc chạm vào một đối tượng để xem chi tiết; xem những nội dung mới thường ở phía bên phải; v.v... Hãy dựa vào những gì người dùng quen thuộc để cung cấp cho họ một cảm giác tốt hơn về sự quen thuộc và tin tưởng với ứng dụng của bạn.
  
  Bên cạnh đó, bạn cũng cần những tiêu đề chính xác để người dùng hiểu rõ những gì họ đang thao tác. Sử dụng những từ ngữ ngắn gọn dễ hiểu hoặc chắc chắn rằng người sử dụng apps của bạn có thể hiểu nó. 
  
  Ngoài ra, icon được đưa vào sử dụng cũng nên được cân nhắc. iOS đã làm sẵn những button và icon cơ bản như: Refresh, Organize, Trash, Reply and Compose mail, v.v... Do vậy, tránh làm cho người dùng khó hiểu, bạn không nên sử dụng những icon này cho chức năng khác.
  
  ![easy](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/MobileHIG/Art/icon_family_2x.png)
  ![iOS icon](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/MobileHIG/Art/balanced_icons_2x.png)
  
  Một điều bạn cần quan tâm đến là chức năng **tìm kiếm** trong ứng dụng của mình. Chỉ khi chức năng tìm kiếm là một chức năng cơ bản của ứng dụng thì hãy sử dụng tab. Các trường hợp khác, hãy để thanh tìm kiếm ở trên một màn hình với danh sách vì người dùng đang quen với điều đó.
  
  __3. Khoảng không tap đủ lớn__
  
  App khác với website, người dùng hoàn toàn không sử dụng chuột. Thay vào đó, tất cả các tương tác với ứng dụng của bạn được thực hiện với công cụ ít chính xác: một ngón tay. Do vậy, để đảm bảo rằng người dùng có thể tương tác dễ dàng với giao diện ứng dụng của bạn, hãy chắc chắn rằng khoảng không Tap có kích thước tối thiểu 44x44px

Trên đây là những thứ cơ bản cần thiết trước khi bạn bắt đầu thiết kế giao diện cho ứng dụng của mình. Một tác phẩm nghệ thuật đẹp cũng giúp xây dựng thương hiệu ứng dụng của bạn trong mắt người dùng. Hãy trưng bày tác phẩm nghệ thuật của mình để mọi người chiêm ngưỡng nào. Let's start! 

Nguồn tham khảo:

1. The official [Apple iOS Human Interface Guidelines](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/MobileHIG/index.html?utm_source=twitterfeed&utm_medium=twitter)

2. [iOS User Experience Guidelines](http://www.designprinciplesftw.com/collections/ios-user-experience-guidelines)

3. [RGB.vn](http://rgb.vn/ideas/explore/mot-so-diem-can-luu-y-khi-thiet-ke-icon-ung-dung-cho-ios-7)

*ThaoNNM*
