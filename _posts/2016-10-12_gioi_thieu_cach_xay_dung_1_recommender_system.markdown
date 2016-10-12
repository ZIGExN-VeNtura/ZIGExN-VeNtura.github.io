---
 layout: post
 title:  "Giới thiệu Phương Pháp xây dựng một simple Recommender System dùng Machine Learning."
 date:   2016-10-12
 summary: Bài viết giới thiệu phương pháp xây dựng Recommender System
 categories: [MachineLearning]
 tags: ["MachineLearning"]
 images: http://bigdata.ices.utexas.edu/wp-content/uploads/2014/03/pmf-banner.png
 author: Vinh Quang Nguyen 
---
   ![solr](http://bigdata.ices.utexas.edu/wp-content/uploads/2014/03/pmf-banner.png)

Là 1 tập hợp những thông tin con của hệ thống, những thông tin con này được dung để tiên đoán được sở thích, rating của user trên những sản phẩm. Recommender System đã trở nên rất phổ biến trong những năm gần đây, và được sử dụng trong một loạt các lĩnh vực:. một số ứng dụng phổ biến bao gồm phim ảnh, âm nhạc, tin tức, sách, các bài báo nghiên cứu, các truy vấn tìm kiếm, thẻ xã hội, nhà hàng, dịch vụ tài chính, bảo hiểm nhân thọ, hẹn hò trực tuyến.

### Một Vài loại recommender system
- Content-based filtering.
Phương pháp dựa trên mô tả về những những item với user profile cụ thể có khả năng sẽ thích item đó. Hầu hết các hệ thống thường dùng những keyword để mô tả những item và User’s Profile. Ví dụ chúng ta có những nhà tuyển dụng họ post những jobs về “Ruby On Rails” và với những jobs như vậy chúng ta sẽ cần tìm những User's Profile mà sẽ thích “Ruby On Rails job”.  Bởi vì dùng những keywrod để description cho những item và user’s profile cho nên giải thuật tf-idf. Giả sử tôi có 64 jobs bao gồm nhiều loại jobs java, 3 ruby vvv. Bài toán đặt ra là phải làm cách nào có thể recommend jobs ruby cho 1 ruby developer. Chúng ta sẽ thử áp dụng tf-idf trên jobs document để tìm những vector keywords cho từng documents, loại bỏ những common words. Sau đó dùng giải thuật K-NN tính khoảng cách của user và jobs, hoặc là khoảng cách giữa user và user. Bởi vì nếu 2 users có the same profile chúng ta có thể dựa vào history interaction để recommend những job cho users. Ví dụ job đó được apply bởi user có java skill, location là hcm, .v.v. Thì user có the same profile có khả năng sẽ apply for this job.
<p align="center">
  <img src="https://cloud.githubusercontent.com/assets/6763141/19277707/d999a4c0-9004-11e6-9456-18e2207003a0.png?raw=true" alt="No Picture"/>
</p>

 - **Term Frequency cho 1 documents:**

    | A        | Ruby           | .....  | Rails  |.....|The|......|
    | ------------- |:-------------:|:-------------:|:-------------:|:-------------:|:-------------:| -----:|
    | 18      | 12 |  | 12 |.....|25|   |
 - **Inverse Document Frequency trên tất cả các documents:**

    | A        | Ruby           | .....  | Rails  |.....|The|......|
    | ------------- |:-------------:|:-------------:|:-------------:|:-------------:|:-------------:| -----:|
    | log(64/(1 + 63))=0      | Log(64/(1 + 3)) = Log(18)|    | log(64/(1 + 3)) = Log(18) |.....|log(64/(1 + 63))=0|   |

 - **TF-IDF:**

    | A        | Ruby           | .....  | Rails  |.....|The|......|
    | ------------- |:-------------:|:-------------:|:-------------:|:-------------:|:-------------:| -----:|
    | 0      | 12 * Log(18) |  | 12 * Log(18) |.....|0|   |

 - Chúng ta thấy rằng trong 64 docs đều có từ "the, a" cho nên những từ này là common words tf-idf=0 và keywords "rails, ruby" tần số xuất hiện hợp lý tf-idf # 0.

 - **References:**
   https://en.wikipedia.org/wiki/K-nearest_neighbors_algorithm

 - **Tools for a content-based filtering:**

    **Apache Lucene** http://lucene.apache.org/ .

    **Apache Solr** http://lucene.apache.org/solr/.

    **ElasticSearch** https://www.elastic.co/products/elasticsearch.

    **lunr.js** http://lunrjs.com/.

    **Xapian** http://xapian.org/.

- User-User collaborative filtering
Được biết đến như k-NN collaborative filtering, core của giải thuật này chính là tìm những user có rating behavior tương tự trong quá khứ với user và user ratings hiện tại để tiên đoán những item tiếp theo mà user hiện tại có khả năng thích. Giả sử chúng ta có thể mô tả d topics cho mỗi user và movie. Tập phim đó có bao nhiêu% lãng mạng, hài hước, hành động. Và how much user thích nó với.

    |         | action           | romance | drama  |.....|
    | ------------- |:-------------:|:-------------:|:-------------:|:-------------:|:-------------:| -----:|
    | Mô tả về phim     | 0.3 | 0.01  | 1.5 |.....|
    | Sở thích của user u     | 2.5 | 0  | 0.8 |.....|
    | Sở thích của user u'     | 2.5 | 0.1  | 0.7 |.....|

    Rating<Lu, Rv> = (0.3 * 2.5 + 0.01 * 0 + 1.5 * 0.8) 

    Rating<Lu', Rv> = (0.3 * 2.5 + 0.1 * 0.01 + 1.5 * 0.7)

    Chúng ta có thể dễ dàng thấy rằng nếu trong quá khư user u thích  phim Rv thì có khả năng user Lu' cũng sẽ thích phim có 

    tính chất tương tự như Rv.

- Item-Item collaborative filtering
<p>
Amazon là một trang web online shopping nổi tiếng đã sáng tạo ra phương pháp này. Họ đưa ra công thức tính toán dựa trên những đặc điểm tương đồng hay hỗ trợ của sản phẩm sau khi người dùng rating hoặc là lịch sử puchased.
</p>
<p>
Chúng ta sẽ xét 2 loại sản phẩm Iphone7, Iphone7 Case Trainium. Có nhiều người khi mua mua Iphone7 sẽ mua 1 iphone 7 case và ngược lại. Cho nên chúng ta sẽ tạo ra một Co-Occurrence Matrix để recommend những products to user.  Do những người mua iphone 7 case sẽ mua iphone 7 cho nên Matrix là symmetric. Đầu tiên tôi sẽ nhìn vào iphone 7 row và tìm những user đã mua iphone 7 sau đó quyết định proposal cho họ iphone 7 case.
</p>

    |Iphone 7 | Iphone 7           |   | ?  |?|
    | ------------- |:-------------:|:-------------:|:-------------:|:-------------:|:-------------:| -----:|
    |....|....|......|....|......|
    | ... | ... | ...| ....|....|
    | ?| ? | | **Iphone 7 Case** |**Iphone 7 Case**|

### Hướng dẫn Build 1 simple user-user recommender system dùng Movielens data.
- Database modeling for Movie-Review app: Chúng ta thấy trong mô hình này một phim có nhiều genres và 1 users có thể reviews trên nhiều phim.

![untitled diagram 1](https://cloud.githubusercontent.com/assets/6763141/19292931/fd911b2a-9049-11e6-8232-a6d65eb117ae.png)

- Phân cụm những movies theo genres:
Đối với movilens chúng ta sẽ có 19 genres đó là lý do về mặt trực quan chúng ta chia thành 19 cụm. Nhưng thật sự nếu chúng ta chia bao nhiêu cụm đi chăng nữa phương pháp này cho ra kết quả tương tự. Có thể cụm thứ nhất bao gồm những bộ phim lãng mạng, hài hước và cụm thứ 2 bao gồm những bộ phim về action, advanture v.v.v.  
clusters = Kmeans(19, all_genres_value)
- Tính toán rating cho từng user trên những phim theo group.
Xét trên tất cả những bộ phim mà user đã review. Nếu user đã review cho những tập phim thuộc group thứ nhất vơi rating 3, 4, 2, 1 và group thứ 2 với rating 5, 4, 3,  2, 5,  1.
[(1 + 2 + 3 + 4)/4, (5 + 4 + 3 + 2 + 5 + 1)/6, …..]
- Tính khoảng cách giữa tất cả user dựa trên mảng rating.
Sau khi có được vector rating của user trên tất cả cả tập phim theo group. Lúc này chúng ta có thể dễ dàng tính được khoảng cách giữa những user và generate ra một similarity matrix giữa những user. Sau khi có được similar matrix chúng ta sẽ dùng nó để recommend những phim những user trong quá khứ đã xem cho user hiện tại.

    https://en.wikipedia.org/wiki/K-means_clustering
    https://en.wikipedia.org/wiki/Pearson_product-moment_correlation_coefficient

### Result:

    Data: https://github.com/quangvinh0513/movie_recommender_app1/tree/master/app/models/import/recommender_system/movie_data

    App demo: https://github.com/quangvinh0513/movie_recommender_app1

    Result Data: https://github.com/quangvinh0513/movie_recommender_app1/blob/master/avr_g.json

    Test Data: https://github.com/quangvinh0513/movie_recommender_app1/blob/master/test.txt

    Mean Squared Error trên test data của Movielen là 1.23 .

### Kết luận:
Recommender systems là một technology giúp trích xúc những thông tin quan trọng từ user databases. Những system này giúp người dùng tìm những items mà học muốn mua 1 cách dễ dàng từ một business model và giúp tăng doanh thu cho doanh nghiệp. Ngày nay Recommender System đang trở nên vô cùng quan trọng bởi vì lượng dữ liệu của người dùng ngày càng tăng trên website cho nên việc xây dựng những recommender system là vô cùng cần thiết.
Trong bài viết này, Tôi thực sự chỉ đề cập đến 1 vài algorithms và technology để xây dựng một Simple Recommender System. Trong tương lai gần nếu có thể tôi hy vọng có thể viết một bài tech blog với một Job Intelligent application hoặc intelligent shop mà dùng Recommender System.


