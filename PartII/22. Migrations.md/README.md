為了達到敏捷開發、快速迭代的目的，Rails 設計了 Migration 機制對 database schema 做更動。


# Creating and Running Migrations

- Migration 命名方式一般是 `14個數字_migration名稱`，數字是 migration 創造時的 UTC timestamp, 也是 version number, 由年(4位數)、月、日、時、分、秒(各2位數)組成，它所代表的時序意義也是這個機制的重點。
- Timestamp 可能重複，但機率太低，可以忽略。
- 用產生器產生 migration 有兩種方法：
`$ rails generate model discount -T`
```
invoke active_record
create db/migrate/20121113133549_create_discounts.rb 
create app/models/discount.rb
```
`$ rails generate migration add_price_column`
```
invoke active_record
create db/migrate/20121113133814_add_price_column.rb
```

延伸閱讀：到底是 GMT+8 還是 UTC+8 ? http://pansci.asia/archives/84978


Running Migrations
---
- Migration 用 `$ rails db:migrate` 這個 rake task 來執行。
- Rails 有個 `schema_migrations` table, 只有一個欄位 `version`, 紀錄所有 migrate 過的 timestamps.
- 跑 `$ rails db:migrate` 時，會先去找 schema_migrations table, 沒有的話就建一個。
- 接著去 db/migrate 檢查所有的 migration 檔。如果 migration 的 timestamp 已經紀錄在 schema_migrations table 的 version 欄位，就跳過；反之，就依照 migration 內容變更資料庫，並將 timestamps 個別紀錄到 schema_migrations table 中。
- 多人使用一個版本控制系統時，可能發生 migrations 時序亂掉的情形，這時會建議將 database 還原，處理好時序問題；或者讓每個 migration 是獨立的。
- 可以強制 database 到指定的 version:
`$ rails db:migrate VERSION=20160704010305`
- 如果指定的 version 是最晚(數字最大)的，就可以順利執行 task; 反之，如果指定的 version 不是最晚的，所有紀錄在 schema_migrations table 並且比這 version 晚的 migrations 都會被退回(undo)到還沒 migrate 的狀態。
- `$ rails db:migrate:redo` 可以回退(rollback)一個 migration 再重新跑一次 task; 
`$ rails db:migrate:redo STEP=N` 則可以退 N 個 migrations 然後重跑一次 task.


# Anatomy of a Migration


- 這是一個 migration 檔裡面的內容：

```ruby
class SomeMeaningfulName < ActiveRecord::Migration 
  def up
    # ...
  end
  def down 
    # ...
  end 
end
```
 - migration 檔名在 `timestamp_` 後的部分，用 snake_case 表示，並且轉換成 CamelCase 後要跟 migration 檔中 class name 一樣。以這個範例來說，migration 檔名會長得像 `20160704121212_some_meaningful_name.rb`
 - 各個 migration 不能有跟其它 migration 一樣的 class name.
 - `up()` method 對 schema 做變動，`down()` method 退回那些變動，都是單向的；`change()` method 是雙向的，可以取代兩者：

`add_column(table_name, column_name, type, options = {})`

```rb
class AddEmailToOrders < ActiveRecord::Migration 
  def change
    add_column :orders, :e_mail, :string
  end
end
```



Column Types
---
Rails 幫使用者避開很多麻煩。使用者不需要去管底層的東西，比如說我們可以在 development 階段使用 sqlite3, 在 production 階段使用 pg. 同樣地，databases 一般沒有 :string 這種欄位屬性，如果我們透過 migration 給予某個欄位 :string 的屬性，Rails 會自動去找目前的 database 是哪一個，而自動給予相對應的欄位屬性，比如 :string 在 sqlite3 上就是 varchar(255), 而在 pg 上則是 char varying(255). 

Migration 支援的欄位屬性有：
- :binary
- :boolean
- :date
- :datetime
- :decimal
- :float
- :integer
- :string
- :text
- :time
- :timestamp


除了前述的定義方式外，大多數的欄位還能透過 migration 外加三種 options:
- null: true or false
允不允許欄位值是 null. 它獨立於 presence: true validation 之外，是在 model layer 運作的。
- limit: size
限制欄位能儲存的長度
- default: value
起始值

如果是 `:decimal` 欄位則還能再加上兩種 options:
- :precision
有多少位數字能被儲存
- :scale
小數點後有多少位數

由於不同的 database 對 `:decimal` 欄位有不同的支援度，所以強烈建議每個 `:decimal` 欄位都要定義 `:presision` 及 `:scale`.


Renaming Columns - 改欄位名稱
---

`rename_column(table_name, column_name, new_column_name)`

1. 建立一個 migration 
2. 用 `rename_column()` method 來改欄位名稱。 
3. `rename_column()` 是雙向的，可以取代 `up()` 以及 `down()`.
4. `rename` 並不會刪掉修改的欄位內現有的任何資料
5. 不是所有的 database adapter 都支援 `rename` method.

```rb
class RenameEmailColumn < ActiveRecord::Migration 
  def change
    rename_column :orders, :e_mail, :customer_email 
  end
end
```


Changing Columns - 改欄位參數
---
`change_column(table_name, column_name, type, options = {})`

假設我們有個 `order_type` 欄位屬性是 integer, 有個值是 123, 現在要將其屬性改成 string, 並且要保留現有的資料，也就是將 123 改成 "123"。這很簡單，步驟同 `add_column`, 只是不是新增欄位，而是從現有的欄位挑出要改的。

```rb
def up
  change_column :orders, :order_type, :string
end
```

接著，我們在屬性已經改成 string 的 `order_type` 欄位存入 "new" 這樣一個非數字的值。這時如果要把欄位屬性從 string 改成 integer, "new" 無法轉換成 integer, 這筆資料就會消失。

```rb
def down
  change_column :orders, :order_type, :integer
end
```
如果能接受，那就這樣做；不能接受的話，Rails 提供一個特例來避免這種情況：

```rb
class ChangeOrderTypeToString < ActiveRecord::Migration 
  def up
    change_column :orders, :order_type, :string, null: false 
  end

  def down
    raise ActiveRecord::IrreversibleMigration
  end 
end
```

用 `raise ActiveRecord::IrreversibleMigration` 避免執行 down migration


# Managing Tables

建表單用 `create_table()`
- 表單名稱用複數
- block 內定義這個表單物件的欄位
- 不用定義 id 欄位，這是 primary key, Rails 會對所有新的表單去建立這個欄位。
- `t.timestamps` 會一口氣建立好 `created_at` 以及 `updated_at` 兩個欄位，並定義它們的 timestamp 資料屬性。

```rb
class CreateOrderHistories < ActiveRecord::Migration 
  def change
    create_table :order_histories do |t| 
      t.integer :order_id, null: false 
      t.text :notes
      t.timestamps
    end 
  end
end
```

拿掉表單用 `drop_table()` `drop_table(table_name, options = {})`
- 但其實不需要 `drop_table()`, 因為 `add_table()` 本身是雙向的。
- 如果要用，只需要註明一個參數，就是要拿掉的表單名稱。


Options for Creating Tables
---

- `force: true` 強制建立新表單，已經存在的同名表單會被取代，資料也會消失。
- `temporary: true` 建立一個臨時表單，程式一和資料庫中斷連線，表單就消失。
- `options: "xxxx"`



Renaming Tables
---

`rename_table(table_name, new_name)`

```rb
class RenameOrderHistories < ActiveRecord::Migration 
  def change
    rename_table :order_histories, :order_notes 
  end
end
```

當我們 rename table 時會產生一個不容易發現的問題，舉例說明：

假設我們在第四個 migration 建了 `order_histories` 這個表單，並建立了一些資料


```rb
def up
  create_table :order_histories do |t|
    t.integer :order_id, null: false 
    t.text :notes
    t.timestamps
  end
  
  order = Order.find :first
  OrderHistory.create(order_id: order, notes: "test") 
end
```

接著在第七個 migration 將 `order_histories` 表單 rename 為 `order_notes`, 這麼做同時也就將 `OrderHistory` 這個 model rename 為 `OrderNote`.

我們現在決定 drop 掉 development 下的 database 並重新執行這些 migrations. 一旦我們這麽做，會在第四個 migration 出現錯誤導致 migration 失敗，原因是我們已經沒有 `OrderHistory` class 了。

有一個土炮的解法，就是看 migration 需要什麼 model classes, 就直接在 migration 檔裡面建這些 model classes. 比如說以下這個做法，即使沒有 `OrderHistory` class, 第四個 migration 仍然能執行。要注意的是，如果要讓這方法能夠持續有效果，我們就只需要一個空殼子假裝有這些 model classes， 所以不要在這些 model classes 裡增加可能會在這個 migration 用到的功能。

```rb
class CreateOrderHistories < ActiveRecord::Migration 

+ class Order < ApplicationRecord::Base; end
+ class OrderHistory < ApplicationRecord::Base; end
  
  def change
    create_table :order_histories do |t|
      t.integer :order_id, null: false 
      t.text :notes
      t.timestamps
    end

    order = Order.find :first
    OrderHistory.create(order: order_id, notes: "test") 
  end
end
```


Defining Indices
---

為了提升搜索效率，我們用 `add_index()` 來建立 index:

`add_index(table_name, column_name, options = {})`

```rb
class AddCustomerNameIndexToOrders < ActiveRecord::Migration 
  def change
    add_index :orders, :name
  end
end
```

如果 option 是 `unique: true`, 就會產生一個 unique index, 在 indexed column 裡的值都會是 unique 的。

index 的預設名稱是 `index_table_on_column`, 如果我們要自創 index 的名稱，就用 `name: "somename"` 當 option, 不過要記得如果將來用 `remove_index()` 移除這個 index, 也要再用 `name: "somename"` 去註明。

我們還能給複數個欄位建一個 index, 這種叫做 composite index. 方法是給 `add_index()` 一組 column names 組成的 array, array 中的第一個 column 會被用來命名這個 composite index.


Primary Keys
---
Rails 預期每個表單都有一個 primary key, 通常是 id 欄位，並且在這個表單新增的每一筆資料都會得到一組新數字。如果有表單少了 primary key, Rails 會秀逗，所以強烈建議照著教材的流程讓 Rails 來建立 primary key.

我們也會用到沒有 primary key 的表單的時候，在 Rails 中最常見的就是 join table. Join table 沒有 primary key, 只有兩個 foreign key 欄位，用來連結另外兩個有 primary key 的表單。

```rb
create_table :authors_books, id: false do |t| 
  t.integer :author_id, null: false
  t.integer :book_id, null: false
end
```


# Advanced Migrations

Using Native SQL
---
Migration 不直接用資料庫語法來維護 schema, 當 migration 沒有你需要的方法時，就要用資料庫原生的語法來操作。Rails 提供兩種方法：

- method 後面加 `options`
- `execute()`


常見的例子是"給子表單加上 foreign key constraint"
延伸閱讀：What are database constraints?
http://stackoverflow.com/questions/2570756/what-are-database-constraints

首先，我們寫一個像下面這樣的 method 到 migration source file:
```rb
def foreign_key(from_table, from_column, to_table) 
  constraint_name = "fk_#{from_table}_#{to_table}" 
  execute %{
    CREATE TRIGGER #{constraint_name}_insert
    BEFORE INSERT ON #{from_table}
    FOR EACH ROW BEGIN
      SELECT
        RAISE(ABORT, "constraint violation: #{constraint_name}")
      WHERE
        (SELECT id FROM #{to_table} WHERE
          id = NEW.#{from_column}) IS NULL;
    END;
  }
  execute %{
    CREATE TRIGGER #{constraint_name}_update
    BEFORE UPDATE ON #{from_table}
    FOR EACH ROW BEGIN
      SELECT
        RAISE(ABORT, "constraint violation: #{constraint_name}")
      WHERE
        (SELECT id FROM #{to_table} WHERE
          id = NEW.#{from_column}) IS NULL;
    END;
  }
  execute %{
    CREATE TRIGGER #{constraint_name}_delete
    BEFORE DELETE ON #{to_table}
    FOR EACH ROW BEGIN
      SELECT
        RAISE(ABORT, "constraint violation: #{constraint_name}")
      WHERE
        (SELECT id FROM #{from_table} WHERE
          #{from_column} = OLD.id) IS NOT NULL;
    END;
  }
end
```

接著，我們在 `up()` migration 中使用上面這個 method:

```rb
def up
  create_table ... do
  end
  foreign_key(:line_items, :product_id, :products) 
  foreign_key(:line_items, :order_id, :orders)
end
```

如果我們要進一步讓所有的 migrations 都能用到 `foreign_key()` method, 我們要在程式的 `lib` directory 中建一個 module, 然後把 `foreign_key()` 加進去：

```rb
module MigrationHelpers
  def foreign_key(from_table, from_column, to_table)
    constraint_name = "fk_#{from_table}_#{to_table}"
    execute %{
      CREATE TRIGGER #{constraint_name}_insert 
      BEFORE INSERT ON #{from_table}
      FOR EACH ROW BEGIN
        SELECT
          RAISE(ABORT, "constraint violation: #{constraint_name}")
        WHERE
          (SELECT id FROM #{to_table} WHERE id = NEW.#{from_column}) IS NULL;
      END;
    }
    execute %{
      CREATE TRIGGER #{constraint_name}_update 
      BEFORE UPDATE ON #{from_table}
      FOR EACH ROW BEGIN
        SELECT
          RAISE(ABORT, "constraint violation: #{constraint_name}")
        WHERE
          (SELECT id FROM #{to_table} WHERE id = NEW.#{from_column}) IS NULL;
      END; 
    }
    execute %{
      CREATE TRIGGER #{constraint_name}_delete 
      BEFORE DELETE ON #{to_table}
      FOR EACH ROW BEGIN
        SELECT
          RAISE(ABORT, "constraint violation: #{constraint_name}")
        WHERE
          (SELECT id FROM #{from_table} WHERE #{from_column} = OLD.id) IS NOT NULL;
      END; 
    }
  end 
end
```

最後，只要在任何一個 migration 中像下面加上那兩行 code, 就可以用 `foreign_key()` 了：

```rb
+ require "migration_helpers"
  class CreateLineItems < ActiveRecord::Migration
+   extend MigrationHelpers
```

`require "migration_helpers"` 載入這個 module 的定義

`extend MigrationHelpers` 則是把 `MigrationHelpers` module 內的 methods 加進 migration 中成為 class methods.

透過這種技巧，我們可以創建及分享許許多多的 migration helpers. 
甚至還有人寫了個 plugin 來進一步簡化 "foreign key constraint" 的流程：
https://github.com/matthuhiggins/foreigner



Custom Messages and Benchmarks
---
在 advanced migrations 中顯示客製化的訊息及 benchmark 其實不是 advanced migrations 的一部分，但這麼做有好處。

`say_with_time(message)` 跑 code block 前先印出 message, 跑完 code block 再印出 benchmark 顯示整個過程花了多少時間。

```rb
def up
  say_with_time "Updating prices..." do
    Person.all.each do |p|
      p.update_attribute :price, p.lookup_master_price
    end 
  end
end
```


# When Migrations Go Bad

使用 migration 會遇到的一個嚴重問題是如果 database 改爛掉就掰了。舉例來說：

```rb
class ExampleMigration < ActiveRecord::Migration 
  def change
    create_table :one do ... 
    end
    create_table :two do ... 
    end
  end 
end
```

這個 migration 要一次加兩個 tables, 假如我們因為 migration 沒寫好，而導致成功建 table `one`, 但在建 table `two` 時遇到問題，就算我們修改好 migration, 也會因為 table `one` 已經存在而無法成功跑 migration.

我們可以試著回退 migration, 但因為前一次 migration 並沒有成功，schema version 並沒有被更新到，所以根本無從回退起。我們也可以手動修改 schema 相關訊息，然後 drop 掉 table `one`, 但這並不值得。

在這個情境下，建議的做法是：
- drop 掉整個 database 
- re-create database
- 跑 migration

這樣做，你不會損失任何東西，並且你會知道 schema 恢復正常了。

所有這些討論都是在說明一件事：用 migration 對 production 階段的資料庫進行操作是很危險的，如果要做這些事：
- 先備份資料庫
- 先備份資料庫
- 先備份資料庫
- 很重要所以說三遍
- 接著導入 production 伺服器上的資料庫到 application directory
- 執行 `RAILS_ENV=production bin/rails db:migrate`


# Schema Manipulation Outside Migrations
這章提到的 migration methods 都可以使用在 models, views, controllers 中。

假設有個已經存在一段時間的表單 `orders`, 我們給裡面的 `city` 欄位加上 index, 產出時間跨度長的報告會快得多；但我們平常用不到這個 index, 留著它反而嚴重拖慢程式速度。因此，我們寫一個 method 放在 private scope 或是 library 來解決問題，這 method 做的事情是：
- Creates the index
- Runs a block of code
- Drops the index

```rb
def run_with_index(*columns) 
  connection.add_index(:orders, *columns)
  begin
    yield 
  ensure
    connection.remove_index(:orders, *columns) 
  end
end
```

下面是一個統計數據的方法，就可以用到上面那個 method.

```rb
def get_city_statistics 
  run_with_index(:city) do
    # .. calculate stats
  end 
end
```