#Task D: Cart Creation
###Interation D1: Finding a Cart
- 建立購物車	
- Concern in Rails
	- 存在Model跟Controller中 
	- 用來放置可以重複使用的程式
	- 以Module形式出現


###Interation D2:Connecting Products to Carts

![](/home/kennychiu/rails_prj/agilewebror5/PartII/5.Task D: Cart Creation/sceenshot/ERD.png) 

<table class="table table-bordered table-striped table-condensed">
<tr>
  <th>Model A</th>
  <th>Relation</th>
  <th>Model B</th>
</tr>
<tr>
   <td>has_many</td>
  <td>1:n</td>
  <td>belongs_to</td>
</tr>
<tr>
   <td>has_one</td>
  <td>1:1</td>
  <td>belongs_to</td>
</tr>
<tr>
   <td>has_and_belongs_to_many</td>
  <td>n:n</td>
  <td>has_and_belongs_to_many</td>
</tr>
</table>

- <code>has_many: :line_items, dependent: [action] </code>，常用action種類如下
    - destroy：會把相關連的資料也刪除，會執行line_items的destroy callback
    - delete：會把相關連的資料也刪除，但不會執行line_items的destroy callback
    - nullify：不會把關聯資料刪除，只會清除關聯ID值，可能會產生很多垃圾資料，該action也是預設值


#### Reference
- [ActiveRecord Association](http://guides.rubyonrails.org/association_basics.html) 
- [實戰聖經|ActiveRecord - 資料表關聯](https://ihower.tw/rails4/activerecord-relationships.html) 

###Interation D3:Adding a Button
-  <code><%=button_to("Add to Cart", line_items_path(product_id: product))%></code> 轉譯為
    - `	<`form class="button_to" method="post" action="/line_items?product_id=3">
`	<`input type="submit" value="Add to Cart">
`	<`/form>
-  使用include model_name加入共用程式
- params 包含透過browser傳遞的參數

#### Reference
- [URI Helper](http://api.rubyonrails.org/classes/ActionView/Helpers/UrlHelper.html) 
- [實戰聖經|RESTful應用程式](https://ihower.tw/rails4/restful.html) 
