---
layout: post
title:  "Giới thiệu chức năng Recommendation của Solr"
date:   2016-10-05
summary: Bài viết giới thiệu cơ bản về chức năng recommendation của Solr
categories: [Solr]
tags: ["solr"]
image: images/solr-logo.png
author: Son Dang
---

![solr](/images/solr-logo.png)

Khi mà công nghệ tìm kiếm được phổ biến bởi những dự án mở như Lucene và Solr thì các tổ chức bắt đầu giải quyết những vấn đề phức tạp hơn trong ứng dụng tìm kiếm của mình. Một trong những vấn đề này là làm sao để đưa nhiều thông tin có liên quan đến người dùng hơn với mục tiêu là giữ cho người dùng tương tác với ứng dụng. Để giải quyết được vấn đề này yêu cầu chương trình được thiết kế đủ thông minh để giới thiệu thêm thông tin tới người dùng thay vì họ phải tự mình tìm kiếm những thông tin đó. Solr được trang bị đầy đủ để có thể dễ dàng thực hiện chức năng này.


### Tìm kiếm (search) vs Giới thiệu (recommendation)

Khi nói đến một công cụ tìm kiếm, người ta sẽ nghĩ tới một khung search để điền thông tin. Tương tự khi nghĩ tới một công cụ recommendation, người ta sẽ nghĩ tới một thuật toán tự động đề xuất những thông tin dựa trên nhu cầu và hành vi của người dùng. Trong thực tế, cả search và recommendation nhìn chung đều là so khớp giữa input và thông tin có sẵn.

Về bản chất, cả search và recommendation hoạt động tương tự nhau: xây dựng kết nối giữa các văn bản và tìm kiếm thông tin có liên quan nhất dựa trên những chỉ số được thiết lập sẵn. Search và recommendation khác nhau cơ bản ở chỗ search yêu cầu input từ người dùng một cách cụ thể trong khi recommendation tự động cung cấp những thông tin dựa trên thông tin từ người dùng, có thể là trong profile hay lịch sử search.

### So khớp dựa trên attributes

Để xây dựng một cơ chế recommendation dựa trên attributes, chúng ta cần phải chắc chắn rằng cả người dùng và văn bản đều được đánh dấu bởi cùng loại attributes. Nếu chúng ta đang xây dựng một ứng dụng để tìm kiếm công việc, thì những công việc này phải có attributes như địa điểm, lương và tên vị trí. Nếu ta có người dùng ứng tuyển vào những công việc này, ta cần phải thu tập được những thông tin tương tự thông qua CV, profiles hoặc lịch sử search của họ.

Giả sử Solr server của chúng ta đang lưu trữ thông tin của những công việc đang được mở. Làm cách nào chúng ta có thể sử dụng profile của một người dùng để tự động thực hiện search và cung cấp recommendation về những công việc có liên quan cho người dùng này? Lấy ví dụ người dùng của chúng ta là anh Yamada:

{% highlight ruby %}
Profile: {
  Name: "Yamada Matsumoto",
  Industry: "Information Technology",
  Locations: "Osaka, Kansai",
  JobTitle: "Web Developer",
  Salary:{
    min:80000,
    max:95000
  },
}
{% endhighlight %}

Với thông tin từ profile của Yamada, cho dù Yamada không tìm kiếm bất cứ thông tin gì, chúng ta cũng có thể cung cấp recommendation cho anh này bằng cách tự động tìm kiếm những công việc có trên index. Dưới đây là ví dụ của một recommendation query:


#### Query
{% highlight ruby %}
http://localhost:8983/solr/jobs/select?
  fl=jobtitle,city,state,salary&
  q=(jobtitle:"web developer"^25 OR jobtitle:(web developer)^10)
    AND ((prefecture:"Osaka" AND area:"Kansai")^15
    OR area:"Kansai")
    AND _val_:"map(salary, 80000,95000,10,0)"
{% endhighlight %}

#### Kết quả
{% highlight ruby %}
{
  ...
  "response":{"numFound":22,"start":0,"docs":[
    {
      "jobtitle":"Web Developer (Osaka / Kansai)",
      "prefecture":"Osaka",
      "area":"Kansai",
      "salary":81503},
    {
      "jobtitle":"Ruby on Rails Web Developer",
      "prefecture":"Nara",
      "area":"Kansai",
      "salary":86183},
    {
      "jobtitle":"PHP Developer",
      "prefecture":"Hyogo",
      "area":"Kansai",
      "salary":91359},
  ...
]}}
{% endhighlight %}

Với một câu query đơn giản như bên trên, chúng ta đã có thể cung cấp cho người dùng một số recommendation cơ bản mà họ có thể quan tâm thông qua những trường thông tin của người dùng cung cấp.

### Hierarchical matching

In discussing attribute-based matching in the last section, we made a point to show that not all attributes are created the same, and sometimes you need to make specific attributes optional, although weighting them higher if they do match. The idea behind hierarchical matching is the same. Assuming that you have the ability to classify your users and content into some kind of hierarchy, from more general to more spe- cific categories, you can then query this hierarchy and apply a stronger relevancy weight to more specific matches, although still matching at all levels of the hierarchy.

Trong ví dụ bên trên ta có thể nhận thấy là không phải tất cả các attribute đều được tạo ra với độ ưu tiên là như nhau. Đôi khi ta cần thiết lập vài attributes cụ thể là không bắt buộc, mặc dù chúng có trọng số cao hơn nếu khớp với index. Ý tưởng ở mục này cũng tương tự. Giả sử là chúng ta có thể phân chia người dùng và nội dung vào một thứ bậc (hierarchy) nào đó và sau đó query trong hierarchy này và áp dụng một trọng số tốt hơn cho mỗi kết quả khớp.

Quay lại ví dụ bên trên, ta thiết lập một vài mục mà có thể liên quan tới anh Yamada hơn:
{% highlight ruby %}
Yamada_Profile: {
  MostLikelyCategory:"technology.developer.rubyonrails",
  2ndMostLikelyCategory:"technology.developer.php",
  3rdMostLikelyCategory:"technology.web.administrator",
...
}
{% endhighlight %}

Trong profile của anh Yamada và search index, ta có thể query như sau:

{% highlight ruby %}
http://localhost:8983/solr/jobs/select?
  df=classification&
  q=(
    (
    "technology.developer.rubyonrails"^40
    OR
    "technology.developer"^20
    OR
    "technology"^10
    )
  OR
    (
    "technology.developer.php"^20
    OR
    "technology.developer"^10
    OR
    "technology"^5
    )
  OR
    (
    "technology.web.administrator"^10
    OR
    "technology.web"^5
    OR
    "technology"
    )
{% endhighlight %}
Ta có thể nhận thấy một vài điểm trong cấu trúc câu query bên trên. Đầu tiên, mỗi mục của profile của Yamada được chia làm 3 thành phần trong query, với mỗi thành phần liên quan tới độ chi tiết của từng cấp bậc (`technology.developer.rubyonrails`, `technology.developer` và `technology`). Điểm thứ hai là mỗi thành phần khi được thiết lập một trọng số query thì càng chi tiết thì càng có trọng số cao. Điều này phục vụ cho mục đích nâng cao độ chi tiết (dẫn tới recommend tốt hơn) cho mỗi kết quả search. Thứ ba là có ba tập hợp query khác nhau tương ứng với ba mục chia trong profile của Yamada `technology.developer.rubyonrails`, `technology.developer.php` và `technology.web.administrator`.

Do có khả năng là mục `MostLikely` của chúng ta là không chính xác hoặc tất cả các mục đều mang lại ý nghĩa, những mục thay thế cũng được thêm vào query với trọng số thấp hơn. Kết quả cuối cùng mà phương pháp này mang lại là trả về dữ liệu có thể khớp với bất kì yêu cầu nào trong khi vẫn có độ chính xác trong tầm kiểm soát của hierarchy.

### Kết luận

Bên trên giới thiệu hai phương pháp cơ bản về recommendation trong Apache Solr mà bạn có thể dễ dàng thực hiện. Ngoài ra còn có những phương pháp và công cụ khác ví dụ như MoreLikeThis - một request handler của Solr cung cấp công cụ nhằm thực hiện chức năng "More Like This" mà bạn thường thấy ở những web như IMDB hay Amazon.

Nếu bạn muốn tìm hiểu sâu hơn về chức năng recommendation, dưới đây là một vài nguồn thông tin hữu ích:

- [Building a real time, solr-powered recommendation engine](http://www.slideshare.net/treygrainger/building-a-real-time-solrpowered-recommendation-engine)
- [Search-Aware Product Recommendation in Solr](http://opensourceconnections.com/blog/2013/10/05/search-aware-product-recommendation-in-solr/)
- [Building a real time, solr-powered recommendation engine (Video)](https://www.youtube.com/watch?v=13yQbaW2V4Y)
