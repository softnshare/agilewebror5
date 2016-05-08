#Task B: Validation and Unit Testing

## Iteration B1: Validating!

Validation：ActiveRecord 的驗證功能，依照設定規則檢查資料的正確性，如果驗證失敗就無法存入資料庫。

**Validation 是應用層級的驗證方法，若要確保資料絕對正確必須設計資料庫層級的驗證。**

例如：不同應用程式存取同一個資料庫、並發性(concurrency)存取時，就無法保證資料庫資料正確性。

### 驗證範例

ex. 檢查標題、敘述、圖片網址是否存在。
```ruby
validates :title, :description, :image_url, presence: true
```

ex. 檢查價錢大於等於 0.01。
```ruby
validates :price, numericality: {greater_than_or_equal_to: 0.01}
```

ex. 檢查圖片網址格式，允許空白，並且自訂錯誤訊息。
```ruby
validates :image_url, allow_blank: true, format: { with: %r{\.(gif|jpg|png)\Z}i,
message: 'must be a URL for GIF, JPG or PNG image.'
}
```

[更多 validation helper](http://guides.rubyonrails.org/active_record_validations.html#validation-helpers)

### 運作原理

p.87 錯誤訊息怎麼顯示在網頁上的？

validation 驗證失敗時，rails 會將錯誤原因寫入 model instance `errors`，資料儲存時會拋出例外 `ActiveRecord::RecordInvalid`，產生 rollback 拒絕變更資料，view 也可利用此錯誤訊息提示使用者錯誤的原因。

```ruby
# app/views/products/_form.html.erb

<% product.errors.full_messages.each do |message| %>
  <li><%= message %></li>
<% end %>
```

從 console 下指令試驗看看

```ruby
product = Product.new
product.save # rollback transaction
product.errors
```

### [Coding style](https://github.com/bbatsov/rails-style-guide#activerecord)

```ruby
# bad
validates_presence_of :email
validates_length_of :email, maximum: 100

# good
validates :email, presence: true, length: { maximum: 100 }
```

### 自訂驗證器

除了 rails 提供的 helper 以外，也可以自行定義驗證方法(method)，若驗證方法在多處被使用，可以進一步包成驗證器(validator)。

#### Custom method

```ruby
validate :some_method

private

def some_method
  # do_something
  errors.add(:field, 'error message')
end
```

#### Custom validator

http://guides.rubyonrails.org/active_record_validations.html#custom-validators

http://www.rails-dev.com/custom-validators-in-ruby-on-rails-4


### 參考資料

http://guides.rubyonrails.org/active_record_validations.html

https://ihower.tw/rails4/activerecord-lifecycle.html

https://robots.thoughtbot.com/the-perils-of-uniqueness-validations
