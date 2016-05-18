Hướng dẫn sử dụng elasticsearch với Ruby On Rails.

## 1) Cài đặt elasticsearch
Làm theo hướng dẫn tại trang chủ của [elasticsearch](https://www.elastic.co/downloads/elasticsearch)

## 2) Cài đặt gem hỗ trợ tương tác với elasticsearch
Bổ sung vào Gemfile:
```
gem 'elasticsearch', git: 'git://github.com/elasticsearch/elasticsearch-ruby.git'
gem 'elasticsearch-model', git: 'git://github.com/elasticsearch/elasticsearch-rails.git'
gem 'elasticsearch-rails', git: 'git://github.com/elasticsearch/elasticsearch-rails.git'
```
Chú ý:
- Có 1 gem khác dễ sử dụng hơn là [searchkick](https://github.com/ankane/searchkick).
- Bài viết này chỉ hướng dẫn cách sử dụng bộ 3 gem trên, không áp dụng cho searchkick.

## 3) Tương tác với elasticsearch
Gỉa sử bạn có các model sau:

Book
- id: integer
- title: text (search field)
- content: text (search field)
- author_id: integer
- note: text
- created_at: datetime
- update_at: datetime

Author
- id: integer
- name: text (search field)
- created_at: datetime
- update_at: datetime

### Step 1: tạo mapping
```
book.rb
...

# enable elasticsearch on this model
include Elasticsearch::Model

# set name of index
index_name [Rails.application.engine_name, Rails.env].join('_')

# we use 1 shard for 1 node
settings index: { number_of_shards: 1 } do
  # create mapping
  mappings  do
    indexes :title
    indexes :content
    indexes :authors, type: 'nested' do
      indexes :name
    end
  end
end

# adust data format to index
  def as_indexed_json(options={})
    as_json(
      only: [:title, :content],
      include: { authors: {only: :name} }
    )
  end
```

### Step 2: index data
Chạy lệnh sau trong rails console

Tạo index
```
Book.__elasticsearch__.create_index!
```

Index data
```
Book.import
```

Để rails tự động add, update, delete index của model Book, bạn add dòng lệnh sau vào book.rb
```
book.rb
...

include Elasticsearch::Model::Callbacks
```

### Step 3: tìm kiếm
Thử tìm kiếm trong rails console
```
books = Book.search('search text')
books.class
=> Elasticsearch::Model::Response::Response
```

Sử dụng kết quả tìm kiếm không thông qua db (nhanh, chỉ có thông tin đã được index trong elasticsearch)
```
books = Book.search('search text').results
book = books[0]._source
book.content
```

Sử dụng kết quả tìm kiếm thông qua db (chậm hơn do phải truy xuất từ db, đầy đủ thông tin)
```
books = Book.search('search text').records
book = books.where('created_at > ?', 3.day.ago).first
book.note
```

## 4) Bonus
### Bạn có thể overwrite method search mặc định của gem elasticsearch
Ví dụ
```
book.rb
...

def self.search(_query)
  # create query in json format
  query = {
    "from": 0,
    # only get 5 results
    "size": 5,
    "query": {
      "bool": {
        "should": [
          {
            "multi_match": {
              "query": _query,
              # title is weighted 5 times more than normal when calculating matching score
              "fields": ["title^5","content"],
              # enable fuzzy match with 1 edit to the original string
              "fuzziness": 1
            }
          }
        ]
      }
    }
  }
  __elasticsearch__.search query
end
```
