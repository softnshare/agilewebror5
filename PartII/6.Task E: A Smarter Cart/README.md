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
find_by()會根據Key回傳指定的紀錄，如果沒有符合的Key，則是回傳nil
where()根據條件回傳array

## 合併新舊版本資料(up)
舊版的line_items沒有quantity欄位，所以會出現多筆product_id
合併資料
<code>rails generate migration combine_items_in_cart</code>
確認合併指令碼
原始碼/db/migrate/日期＋流水號_combine_items_in_cart.rb
1. 先檢查所有的cart
2. 用product_id匯總cart內的物品
3. 如果有sums>1，先清除這個product_id的所有紀錄，然後新增一筆有quantity的紀錄
<code>
def up
Cart.all.each do |cart|
sums = cart.line_items.group(:product_id).sum(:quantity)
sums.each do |product_id, quantity|
if quantity > 1
cart.line_items.where(product_id: product_id).delete_all
item = cart.line_items.build(product_id: product_id)
item.quantity = quantity
item.save!
end
end
end
end
</code>
升級
<code>rails db:migrate</code>

為了向下相容，實做down()，步驟正好與up()相反
1.找出所有quantity>1的product_id
2.新增line_items，讓quantity=1，直到筆數與原本的quantity一致
3.移除quantity>1
<code>
def down
LineItem.where("quantity>1").each do |line_item|
line_item.quantity.times do
LineItem.create cart_id: line_item.cart_id,
product_id: line_item.product_id, quantity: 1
end
line_item.destroy
end
end
</code>
降級
<code>rails db:rollback</code>

檢查migrate狀態
<code>rails db:migrate:status</code>

