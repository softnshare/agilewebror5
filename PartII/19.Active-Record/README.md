# Active Record
Rails的ORM layer

處理程式物件與關連式資料間的對應關係

資料庫|程式
----|----
table|Class
Column|Property
Data|Instance


本章重點：

1. 定義資料結構與程式物件的對應關係

2. 透過ORM管理資料表關連，以及CRUD作業

3. Active Record物件生命週期

## 定義資料
以Table name作為Class name
預設的規則包含基本的翻譯校正（可以自訂ActiveSupport）
也可以在類別內指定自己的table name

在RoR的思維中，資料結構若需要變更，就是建立一個Migration檔（由物件結構來更新資料結構）
如此的作法讓資料結構的定義能夠與程式碼一樣敏捷（完全由PG自己控制）

但在實務上通常是公司內會有DBA負責定義資料結構，DBA依需求開完table之後，再把table交給PG（由資料結構來更新物件結構）

## Identifying individual Rows
資料必須有Primary KEY，以區分資料表中的紀錄
透過Active Record新增的table，預設是以id作為PK，此時api會負責處理新資料的PK
如果要存取已經存在的資料結構時，也可以複寫primary_key屬性，提供自訂的PK欄位，相對的我們的程式必須負責處理新資料的PK
Active Record的慣例是，用id代表物件的PK，用屬性代表對應的欄位
例：book.id="0-12345-6789"代表設定PK，book.ISBN="0-12345-6789"代表設定ISBN這個欄位

預設狀況下，物件比對（==）的時候，Rails會比對PK

## 關連

1 一對一(has_one)

2 一對多(has_many)

3 多對多(has_and_belongs_to_many)


## CRUD

1. 新增(new)(create)

2. 讀取

* Person.where(name: 'Dave')

* Person.where("name='Dave'")，類似sql，因此要小心SQL injection

* 更進一步的where，讓active record處理查詢條件

pos = Order.where(params[:order])--->整個class丟進去當條件

pos = Order.where(name: params[:name],pay_type: params[:pay_type])--->特定的欄位當條件

* 自訂sql--->find_by_sql("select * from table1")

orm框架多少都會有效能問題，因為中間墊了一層抽象層，而且在run time的時候要產生SQL，因此在需要效能的情境中，有時候仍然要用調教過的SQL指令

3. 更新，讀到特定的資料之後直接更改屬性，然後save
save & save!
create & create!
有驚嘆號的會在失敗的時候跳例外

4. 刪除
destroy vs delete
destroy會invoke所有Active Record的驗證function
delete則會忽略

## 資料的生命週期

16個callbacks，其中14個before/after成對

程式裡面可以invoke這些api來達到更細節的控制

## 交易

ACID原則

1. Atomic

2. Consistency

3. Isolation

4. Durable

確保業務流程成功完成（提款機）

確保master-detail資料完整性
