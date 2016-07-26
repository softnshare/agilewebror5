不是所有的 Rails app 都需要 browser 來跑，比如：
- 用 cron 定時在背景載入或同步資料庫
- 你的 stand-alone app (不一定要是 Rails app) 需要直接造訪 Rails app 上的資料
- 就是想用 command line 操作 

我們先討論你的 app 和安裝了 Rails 以及擁有資料的電腦是同一台的情況，接著會討論到如何操弄遠端資料庫。

我們希望能夠：
- 在一個 stand-alone app 中用 Active Record 在 SQLite3 中包一個 orders table
- 以 id 找到某個 order
- 更改消費者的名字、將結果存進資料庫、更新原始的欄位。

### 方法一（比較難）

```
require "active_record" 
```

```
ActiveRecord::Base.establish_connection(adapter: "sqlite3", database: "db/development.sqlite3") 
```
```
class Order < ApplicationRecord
end
```
```
order = Order.find(1) 
order.name = "Dave Thomas" 
order.save
```

不需要什麼設定，Active Record 會自動依照 database schema 判斷我們需要什麼，然後去處理所有的事，


### 方法二（比較簡單）

```
require "config/environment.rb" 
order = Order.find(1) 
order.name = "Dave Thomas" 
order.save
```

這方法要能夠運作，要先讓你的 app 找到 config/environment.rb
- require 完整的路徑，或是
- 在 RUBYLIB 環境變數中 include 路徑

還有一個要注意的環境變數是 RAILS_ENV, 我們用它來選擇 development, test, 或 production 環境。

# A Library Function Using Active Support

Active Support 是一整包 Rails 工具函式庫，它也擴充了一些 Ruby 標準函式庫。部分函式庫傾向於給 Rails 內部使用，但實際上全部的函式庫都可以讓非 Rails apps 使用。

## Core Extensions (core-ext)

Active Support 延伸了一些 Ruby 內建的 classes 當做 core-ext, 最常見的幾個是：
- Array
- CGI
- Class
- Date
- Enumberable
- File
- Float
- Hash
- Integer
- Kernel
- Module
- Numeric
- Object
- String
- Time

## Additional Active Support Classes

Active Record 也提供許多額外的功能，部分設計來方便處理特定需求，但要用在非特定需求的地方也都OK.

- Benchmarkable
- Cache::Store
- Callbacks
- Concern & Dependencies
- Configurable
- Deprecation
- Duration
- Gzip
- HashwithIndifferentAccess
- I18n
- Inflections
- JSON
- LazyLoadHooks
- MessageEncryptor
- MessageVerifier
- MultiByte
- Notifications
- OptionMerger
- OrderedHash & OrderedOptions
- Railtie
- Rescueable
- StringInquirer
- TestCase
- Time & TimeWithZone


回到這章主題，我們怎麼在 stand-alone app 使用這些 methods 呢?

```
require "active_support/time"
Time.zone = 'Eastern Time (US & Canada)' 
puts Time.zone.now
```

可以只 require 需要的部分，如上面、或者下面的方式：

```
require "active_support/core_ext"
```

如果要 require 全部：

```
require "active_support/all"
```

# Using Action View Helpers

這節不是在講 Active Support，但概念類似。

Active Support 中的 methods 在整個 Rails 都能用，但這些 methods 往往比較是應用在 web requests 方面。值得一提的是，有些 Action View helpers 是可以在沒有 view 的情況下被 stand-alone app 使用的：

```
require "action_view"
require "action_view/helpers"
include ActionView::Helpers::DateHelper
puts distance_of_time_in_words_to_now(Time.parse("December 25"))
```