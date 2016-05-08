#Task E: A Smarter Cart

延續前一個task，強化原本購物車的功能。

1. 分辨使用者新增多次同樣的商品

2. 處理例外或錯誤狀況

---
比較聰明的購物車
## Migration
* step1:產生更新資料結構用的ruby檔
rails generate migration add_quantity_to_line_items quantity:integer
在line_items資料表中新增一個quantity欄位，型別為integer
除了add_欄位_to_資料表，也有remove_欄位_from_資料表

* step2:微調
新增欄位的時候，預設值會是null，也可以進指令碼自己調整
指令碼位置
原始碼/db/migrate/日期＋流水號_add_quantity_to_line_items.rb
default: 預設值

* step3:執行
rails db:migrate

## Models
<code>current_item=line_items.find_by(product_id:product_id)</code>

用product_id檢查line_items資料表內是否已經新增過相同的物品

如果有，直接將目前的數量＋1

如果沒有，才是新增一個current_item

<code>find_by()</code>會根據Key回傳指定的紀錄，如果沒有符合的Key，則是回傳nil

<code>where()</code>根據條件回傳array

## 合併新舊版本資料(up)
舊版的line_items沒有quantity欄位，所以會出現多筆product_id

合併資料

<code>rails generate migration combine_items_in_cart</code>

確認合併指令碼

原始碼/db/migrate/日期＋流水號_combine_items_in_cart.rb

1. 先檢查所有的cart

2. 用product_id匯總cart內的物品

3. 如果有sums>1，先清除這個product_id的所有紀錄，然後新增一筆有quantity的紀錄

升級
<code>rails db:migrate</code>

為了向下相容，實做down()，步驟正好與up()相反

1. 找出所有quantity>1的product_id

2. 新增line_items，讓quantity=1，直到筆數與原本的quantity一致

3. 移除quantity>1

降級
<code>rails db:rollback</code>

檢查migrate狀態
<code>rails db:migrate:status</code>

---
## 處理錯誤

<code>@cart=Cart.find(params[:id])</code>

如果沒有資料，Active Recorde會引發RecordNotFound例外

要如何處理例外？

* 忽略-比較安全，潛在攻擊者比較不容易得到系統資訊。但相對的，系統沒有任何回應，可能會讓使用者疑惑。

* 處理-分為兩步驟

1. log-將目前狀況寫到內部紀錄檔

2. redisplay-重新顯示比較user friendly的訊息

### 攔截例外
<code>rescue_from ActiveRecord::RecordNotFound, with: :invalid_cart</code> 

rescue_from-攔截例外

ActiveRecord::RecordNotFound-要攔截的例外類型

with: :invalid_cart-轉交給invalid_cart處理


<code>logger.error "invalid cart" </code>

用rails內建的logger機制紀錄錯誤訊息

<code>redirect_to store_url, notice: 'invalid cart'</code>

重新導向到指定的url

### rails的錯誤處理機制
flash:rails定義的結構，類似hash。

flash的內容會保留到下一個request（相同session），然後就會被自動清除。

### 只接受必要的action參數
<code>params.require(:line_item).permit(:product_id)</code>

只允許product_id作為參數
（只允許必要的參數傳進action，因為url後面的參數有可能被偽造）

### 清除log
<code>rake log:clear LOGS=test</code>

### 跑測試
<code>rails test:controllers</code>



---
本節內容

1. 在目前的table內新增一個欄位，預設值為1

2. 用新的資料結構合併現有的資料

3. 透過flash提供錯誤訊息回應

4. 透過rails內建的logger機制紀錄系統資訊

5. 用permitted list限制可以使用的參數

6. 刪除紀錄

7. 調整頁面，用table顯示購物車內容，配合css打造漂亮的ui


