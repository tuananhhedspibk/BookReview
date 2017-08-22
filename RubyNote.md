# Ruby Note :green_book: :memo:

### 1. Block
  * Mọi thứ nằm giữa {} hoặc `do` và `end` là 1 `Block`
  * VD:
    ```ruby
      [1, 2, 3].each do |x|
        puts x * x
      end
    ```
  * 1 `block` truyền vào 1 `method` sẽ được coi như là 1 `Proc - có thể hiểu như 1 con trỏ tới block`
  * `Block` không phải là 1 `object`
  * Khi ```ruby function() do block end```
    + Thì khi đó block sẽ được ngầm định truyền vào trong function()
    + Khi `block` truyền ngầm định - không qua tham số vào `function`, ta sẽ gọi thực thi `block` bằng lời gọi `yield`
    + Khi `block` được truyền tường minh vào trong hàm, block sẽ được gọi thông qua lời gọi `call`
      - ```ruby block.call [param]```
### 2. Procs
  * Là 1 `object` biểu diễn 1 `Block`
  * Syntax: ```ruby procs_name = Proc.new block```
  * Gọi thực thi 1 `Procs`: ```ruby procs.call [param]```

### 3. Lambda
  * Là 1 dạng `Anonymous function`
  * Vận hành giống như `global function`
  * VD:
    ```ruby
      succ = lambda {|x| x + 1}
      succ = -> (x) {x + 1}
      # succ: có thể coi như con trỏ hàm

      f = -> (x, y ; z, t)
      # x,y: tham số vào, z, t: biến cục bộ của hàm
  * ```ruby Lambda.class = Proc```

### 4. Chú ý giữa Block, Procs, Lambda
  * Chỉ truyền được nhiều nhất là 1 tham số `block` vào `method`
  * Có thể truyền nhiều `Procs` vào `method`
  * `Lambda`, `Procs` đều là `Procs object`
  * `Lambda` kiểm tra số lượng tham số truyền vào, còn `Procs` thì không
    - Nếu `params` truyền vào `lambda` không đủ thì sẽ báo lỗi `argument method`
    - Nếu `params` truyền vào `procs` không đủ thì mặc định các cái thiếu là `null`
  * `return` trong `Lambda` thì đoạn `code` bên ngoài `Lambda` vẫn được thực thi 1 cách trọn vẹn (gần như độc lập với `Lambda`)
    - Chạy hết `Block` cha rồi mới ngắt
  * `return` trong `Procs` sẽ khiến cho đoạn `code` bên ngoài `Procs` không được thực thi
  * `Lambda` có thể được sử dụng ở bên ngoài phạm vi mà nó được định nghĩa

### 5. Sự khác biệt giữa count, length, size
  * `count` sẽ đếm bằng lệnh `select count` trong SQL
  * `length` sẽ kéo dữ liệu về 1 mảng, rồi bóc ra độ dài của mảng đó
  * `size` lúc đầu hoạt động như `count` nhưng khi đã có `cache` thì `size` sẽ lấy gía trị từ `cache` ra

### 6. So sánh giữa update_attribute và update_attributes
  * `update_attribute` chỉ update 1 trường đơn
    - Không có `validate`
    - Gọi `callback`, chỉ `update` dữ liệu khi có sự thay đổi
    - Chỉ update khi dữ liệu thay đổi
    - Nếu trường chỉ đọc => báo lỗi `ActiveRecord::ActiveRecordError`
  * `update_attributes` có thể update nhiều trường
    - Có validates dữ liệu
    - Nếu đối tượng không hợp lệ thì `return false`

### 7. Cách form_for phân biệt edit, new
  * Khi 1 biến instance truyền vào `form_for`
  * Thông qua gía trị trả về của `@variable.persisted?`
    - `true`: `edit form`
    - `false`: `new form`

### 8. Sự khác biệt của hidden_field, hidden_field tag
  * `hidden_field :email => :email` sẽ được truyền vào `params[:email]`
  * `hidden_field_tag :email => :email` sẽ được truyền vào `params[:user][:email]`

### 9. Sự khác biệt giữa scope và class method
  #### 9.1. Định nghĩa 1 `scope`
  * VD: 2 cách định nghĩa `scope`
    ```ruby
      class Post < ActiveRecord::Base
        scope :published, where status: "published"
        scope :draft, ->{where status: "draft"}
      end
    ```
  * Sự khác biệt cơ bản giữa 2 `scope` này là cách dùng
    - Biểu thức điều kiện của `published` sẽ được gọi 1 lần duy nhất khi class được gọi lần đầu tiên => Gặp lỗi khi tham số trong điều kiện là `Time`
    - Biểu thức của `draft` sẽ được gọi lại mỗi khi scope được thực thi
  #### 9.2 Scope cũng là class method
  * Bản thân `ActiveRecord` cũng đã chuyển đổi `scope` thành `class method`
  * VD:
    ```ruby
      def self.scope name, body
        singleton_class.send :define_method, name, &body
      end

      def self.published
        where status: "published"
      end
    ```
  #### 9.3 Scope gọi liên tiếp được
  * Khi người dùng có thể lọc các bài viết theo trạng thái, và sắp xếp chúng
  ```ruby
    # Sử dụng scope
    class Post < ActiveRecord::Base
      scope :by_status, -> status {where status: status}
      scope :recent, -> {order "posts,updated_at DESC"}
    end

    # Gọi: 
    Post.by_status("published").recent
    Post.by_status(params[:status]).recent

    # Sử dụng class method
    class Post < ActiveRecord::Base
      class << self
        def by_status status
          where status: status
        end

        def recent
          order "posts.updated_at DESC"
        end
      end
    end
  ```
  * Khi `status` là `nil`, `blank`, ta sẽ chuyển như sau
    ```ruby
      Post.by_status(nil).recent
      Post.by_status('').recent

      scope :by_status, -> status {where status: status if status.present?}
    ```
  * Sau đó ta thay đổi như sau:
    ```ruby
      class Post < ActiveRecord::Base
        class << self
          def by_status status
            where status: status if status.present?
          end
        end
      end

      # Kết quả
      Post.by_status("").recent
      NoMethodError: undefined method `recent' for nil:NilClass
    ```
  * Không nên trả về `nil` với `class method`
  * Scope luôn trả về 1 `ActiveRecord Relation`
  #### 9.4. Scope mở rộng được
  * Điều quan trọng khi phân trang là biết được số lượng trang chia ra cho dữ liệu, sau đó là số bản ghi mỗi trang
    - ``` Post.page(2).per(15)```
  * Ta có thể mở rộng `scope` bằng cách thêm các thành phần bên trong nó, các thành phần mở rộng này chỉ có tác dụng với `object` nếu `scope` được gọi
  * VD:
    ```ruby
      scope :page, -> num { # some limit + offset logic here for pagination } do
        def per(num)
          # more logic here
        end

        def total_pages
          # some more here
        end

        def first_page?
          # and a bit more
        end

        def last_page?
          # and so on
        end
      end

      # Các thành phần mở rộng này sẽ được gọi khi page được gọi
  * 1 cách viết khác với `class method`
    ```ruby
      def self.page(num)
        scope = # some limit + offset logic here for pagination
        scope.extend PaginationExtensions
        scope
      end

      module PaginationExtensions
        def per(num)
          # more logic here
        end

        def total_pages
          # some more here
        end

        def first_page?
          # and a bit more
        end

        def last_page?
          # and so on
        end
      end
    ```