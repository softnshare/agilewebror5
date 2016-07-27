# CH24 Rails’ Dependencies

softnshare 讀書會 - Agile Web Development with Rails 5

第二十四章 Rails’ Dependencies

講者：puff.tw

# Introduce

這章節會介紹

1. templating engines ( 樣板引擎 ) Rails 預設內建以下兩種。
    - Builder
    - ERb
2. Bundler 安裝 gem 套件的管理工具，會處理套件的相依性。
3. Rack 一種 Middleware 介面
4. Rake 自動化任務

# Generating XML with Builder

- Builder 是獨立的 library
- 可以用 Ruby code 產生 XML
- 用 `.xml.builder` 當作副檔名。
- 範例 `rails50/depot_u/app/views/products/index.xml.builder`

```ruby=
xml.div(class: "productlist") do

  xml.timestamp(Time.now)

  products.each do |product|
	xml.product do
	  xml.productname(product.title)
	  xml.price(product.price, currency: "USD")
	end
  end
end
```

會產生下面這樣的XML

```html=
<div class="productlist">
  <timestamp>2016-01-29 09:42:07 -0500</timestamp>
  <product>
    <productname>CoffeeScript</productname>
    <price currency="USD">36.0</price>
  </product>
  <product>
    <productname>Programming Ruby 1.9</productname>
    <price currency="USD">49.5</price>
  </product>
  <product>
    <productname>Rails Test Prescriptions</productname>
    <price currency="USD">43.75</price>
  </product>
</div>
```

method name 衝突的解決方式，使用 `tag!()`

```ruby
xml.tag!("id", product.id)

```

# Generating HTML with ERB

本質上 ERB 是一個
- 就像在寫 HTML 檔一樣
- 能產生動態的內容
- 範例 純HTML檔

```html=
<h1>Hello, Dave!</h1>
<p>
How are you, today?
</p>
```

- 範例 產生動態的內容

```htmlmixed=
<h1>Hello, Dave!</h1>
<p>
It's <%= Time.now %>
</p>
```

ERB 會把 <%= 和 %> 之間的內容當作程式執行，並把結果用 to_s 轉成 string 輸出

```htmlmixed=
<h1>Hello, Dave!</h1>
<p>
It's <%= require 'date'
DAY_NAMES = %w{ Sunday Monday Tuesday Wednesday Thursday Friday Saturday }
today = Date.today
DAY_NAMES[today.wday]
%>
</p>
```

- 把大量的商業邏輯放進 template ，請善用 Helpers 整理。
- 只有執行沒有輸出，請用沒有 `=` 的 `<% %>`

```htmlmixed=
<% require 'date'
DAY_NAMES = %w{ Sunday Monday Tuesday Wednesday
Thursday Friday Saturday }
today = Date.today
%>

<h1>Hello, Dave!</h1>
<p>
  It's <%= DAY_NAMES[today.wday] %>.
  Tomorrow is <%= DAY_NAMES[(today + 1).wday] %>.
</p>
```

- 下面的範例，瞭解 `<% ... %>` 怎麼響影你的輸出

```htmlmixed=
<% 3.times do %>
Ho!<br/>
<% end %>
```

- 使用 `<%=…%>` 如果 string 包含`<em>hello</em>`，則會顯示`<em>hello</em>`，而不是`hello`。
- 這是因為 HTML tags 會被 escaped，怕使用者輸入惡意 tag 進行 hack
- 使用 `raw()` 則會顯示正確的`hello` 不會 escaped，但不安全
- 使用 `sanitize()` 則會安全可靠些，屬於白名單的過濾，會 escaped `<form>`、`<script>` 危險的elements，建議使用它。

# Managing Dependencies with Bundler

如果你使用舊版的 Rails 可照著下面做

```shell=
$ cd rails50/depot_u  #本書範例原始碼
$ gem update --system
$ bundle install
$ rails -v
Rails 5.0.0
```

`rails50/depot_v/Gemfile`
```ruby
source 'https://rubygems.org'
# Bundle edge Rails instead: gem 'rails', github: 'rails/rails'
gem 'rails', '>= 5.0.0.rc2', '< 5.1'
# Use sqlite3 as the database for Active Record
gem 'sqlite3'
group :production do
  gem 'mysql2', '~> 0.4.0'
end
# Use Puma as the app server
gem 'puma', '~> 3.0'
# Use SCSS for stylesheets
gem 'sass-rails', '~> 5.0'
# Use Uglifier as compressor for JavaScript assets
gem 'uglifier', '>= 1.3.0'
# Use CoffeeScript for .coffee assets and views
gem 'coffee-rails', '~> 4.1.0'
# See https://github.com/rails/execjs#readme for more supported runtimes
# gem 'therubyracer', platforms: :ruby
# Use jquery as the JavaScript library
gem 'jquery-rails'
gem 'jquery-ui-rails'
# Turbolinks makes navigating your web application faster.
# Read more: https://github.com/turbolinks/turbolinks
gem 'turbolinks', '~> 5.x'
# Build JSON APIs with ease. Read more: https://github.com/rails/jbuilder
gem 'jbuilder', '~> 2.5'
# Use Redis adapter to run Action Cable in production
# gem 'redis', '~> 3.0'
# Use ActiveModel has_secure_password
gem 'bcrypt', '~> 3.1.7'
# Use Capistrano for deployment
gem 'rvm-capistrano', group: :development
group :development, :test do
  # Call 'byebug' anywhere in the code to stop execution and get a debugger console
  gem 'byebug', platform: :mri
end
group :development do
  # Access an IRB console on exception pages or by using <%= console %> anywhere in the code.
  # gem 'web-console'
  # Spring speeds up development by keeping your application running in the
  # background. Read more: https://github.com/rails/spring
  gem 'spring'
end
# Windows does not include zoneinfo files, so bundle the tzinfo-data gem
gem 'tzinfo-data', platforms: [:mingw, :mswin, :x64_mingw, :jruby]
gem 'activemodel-serializers-xml'
```

- 第一行代表去哪邊找 gem 或是你要指定自已私人的 gem repositories
- 其它都是指定 gem 的版本
- `groups` 裡面有 `:development`, `:test`, `:production`
  - 指的是這些 gem 只適用在該環境
- Optional 的參數 `:require` 如果檔名不同，可以加上 :require
- Gem 版本號的 `comparison operator`

兩種常見的 `comparison operator`
- `>=` 指定為最低版號，向後兼容性 ( 任何版號，有新的就用新的 )
- `~>` 指定為主要版號，但不能大於指定的 ( 除了主要版號，有新的就用新的 )


| Gem 的版本號 | 主要版號 ( Major )  | 次要版號 ( Minor ) | 修訂版號 ( Patch ) |
| ------ | ----------- | ----------- | ----------- |
| 3.1.4   | 3 | 1 | 4 |


可參考龍哥的 [Ruby 語法放大鏡之「在 Gemfile 裡看到版本寫法有好幾款，各是代表什麼意思?」](http://kaochenlong.com/2016/05/02/gemfile/)


- `Gemfile.lock`
  - 產生方式 
    - `bundle install`
    - `bundle update`

- 範例 `Gemfile.lock` 的部份

```
GEM
  remote: https://rubygems.org/
  specs:
    actionmailer (4.0.0)
      actionpack (= 4.0.0)
      mail (~> 2.5.3)
    actionpack (4.0.0)
      activesupport (= 4.0.0)
      builder (~> 3.1.0)
      erubis (~> 2.7.0)
      rack (~> 1.5.2)
      rack-test (~> 0.6.2)
    activemodel (4.0.0)
      activesupport (= 4.0.0)
      builder (~> 3.1.0)
```

- `bundle install` 會用 `Gemfile.lock` 做為起點
- 一定要把`Gemfile.lock`加到版控系統裡 ( 比如 Git ), 這樣大家才能用同個版本
- 加到版控的好處，最後一次確定能正常工作時所有的 gem 以及版本號的記錄都在裡面
- `bundle update` 會更新多個 gem 也會更新 `Gemfile.lock`
- 不指定 gem 的列表， Bundler 會嘗試更新所有的 gem ( 不建議這麼做，特別是在接近 deployment )
- Bundler 會把最近一次正確的設定寫到 `Gemfile.lock`

# Interfacing with the Web Server with Rack


| web server | 差別 |
| ------ | ----------- | 
| Puma   | 獨立運行 | 
| Phusion Passenger | 整合在其它 web server ( Apache, Nginx ...etc) |

上述的差別只是要說明 Rails 整合 Rack，就像 Passenger 整合 Apache 的概念

- 執行 `rails server` 會載入 `config.ru` 設定檔，直接在 Rack 下執行

- 範例 `rails50/depot_v/config.ru`

```ruby=
# This file is used by Rack-based servers to start the application.
require_relative 'config/environment'
run Rails.application
```

用下面的指令啟動 Rails Server 

```shell=
rackup
# 等同於執行 rails server
```

範例 Rack application ( 未整合Rails )

`rails50/depot_v/app/store.rb`

```ruby=
require 'builder'
require 'active_record'

ActiveRecord::Base.establish_connection(
adapter: 'sqlite3',
database: 'db/development.sqlite3')

class Product < ActiveRecord::Base
end

class StoreApp
  def call(env)
    x = Builder::XmlMarkup.new :indent=>2
    x.declare! :DOCTYPE, :html

    x.html do
	
      x.head do
        x.title 'Pragmatic Bookshelf'
      end
	  
      x.body do
        x.h1 'Pragmatic Bookshelf'
        Product.all.each do |product|
          x.h2 product.title
          x << " #{product.description}\n"
          x.p product.price
        end
      end
	  
    end
    response = Rack::Response.new(x.target!)
    response['Content-Type'] = 'text/html'
    response.finish
  end
end
```

- 上述程式碼在做獨立秀出網頁
  - `active_record` 連資料庫，並建立 Model class `Product`
  - `builder` 用來處理 html

- 建立 rackup file `store.ru` ，然後執行這個 stand-alone application
- 範例`rails50/depot_v/store.ru`

```ruby=
require 'rubygems'
require 'bundler/setup'
require './app/store'
use Rack::ShowExceptions

map '/store' do
  run StoreApp.new
end
```

- 初始化 Bundler ，確保所有的 gem 都是正確的版本
- 載入 `'./app/store'`
- 載入 Rack 提供的 middleware classe `Rack::ShowExceptions`
  - 除錯用的，檢查 request 和調整所產生的 response
- 把 store URI 對應到這個 application
- 最後用 `rackup` 指令啟動它

```shell=
rackup store.ru
```

- 預設 port number 為 9292 而不是 3000
- 參數 `-p`  可以指定 port


![](https://i.imgur.com/rCUiPUb.png)

- Rack 主要優點，避免掉 Rails 的開銷，所以每秒處理更多的 requests
- 缺點就是沒有 Rails 這麼全面，都要另外載入或是自已寫
- 如果想讓網站繞過 Rails Controler，可以用 route 達成一部份交由 Rake 處理
- 範例 `rails50/depot_v/config/routes.rb`

```ruby=
require './app/store' ➤
Rails.application.routes.draw do
  match 'catalog' => StoreApp.new, via: :all ➤
  get 'admin' => 'admin#index'
  controller :sessions do
    get 'login' => :new
    post 'login' => :create
    delete 'logout' => :destroy
  end
  
  resources :users
  resources :products do
    get :who_bought, on: :member
  end
  
  scope '(:locale)' do
    resources :orders
    resources :line_items
    resources :carts
    root 'store#index', as: 'store_index', via: :all
  end
end
```

# Automating Tasks with Rake

- automate tasks 是定義在 Rakefile ( application 根目錄下 )
- `db:setup` 就是一個 task
  - 查看涉及哪些子任務 請執行 Rake 加 `--trace` 和 `--dry-run` 參數

```shell=
$ rake --trace --dry-run db:setup
(in /home/rubys/work/depot)
** Invoke db:setup (first_time)
** Invoke db:create (first_time)
** Invoke db:load_config (first_time)
** Invoke rails_env (first_time)
** Execute (dry run) rails_env
** Execute (dry run) db:load_config
** Execute (dry run) db:create
** Invoke db:schema:load (first_time)
** Invoke environment (first_time)
** Execute (dry run) environment
** Execute (dry run) db:schema:load
** Invoke db:seed (first_time)
** Invoke db:abort_if_pending_migrations (first_time)
** Invoke environment
** Execute (dry run) db:abort_if_pending_migrations
** Execute (dry run) db:seed
** Execute (dry run) db:setup
```

- `rake --tasks` 可以查可用 tasks 清單
- `lib/tasks` Rails 放 task 檔的地方
- 下面是備份 production database 的 task
- 範例 `rails50/depot_v/lib/tasks/db_backup.rake`
```ruby=
namespace :db do
  desc "Backup the production database"
  task :backup => :environment do
    backup_dir = ENV['DIR'] || File.join(Rails.root, 'db', 'backup')
    source = File.join(Rails.root, 'db', "production.db")
    dest = File.join(backup_dir, "production.backup")
    makedirs backup_dir, :verbose => true
    require 'shellwords'
    sh "sqlite3 #{Shellwords.escape source} .dump > #{Shellwords.escape dest}"
  end
end
```

- 第一行為 namespace 叫 db，下面放一個 task 叫 backup
- 第二行為 description
  - 執行 `rake --tasks` ，會看到剛自定 task 的 description
- 根據環境而定 rails console 有提供的都能使用
- 這是用 standard Ruby code 寫的
- 這個範例有用到 `shellwords` ，它是拿來處理萬一目錄有 space 的問題

# Survey of Rails’ Dependencies

- `Gemfile.lock` 列出所在的 Rails 相依性
- RubyGems.org 詳細的情報可再這邊找到

