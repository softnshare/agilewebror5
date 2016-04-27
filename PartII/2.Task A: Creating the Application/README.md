##The Depot Application 

### 1. Setup Rails 5
- 因為目前Rails5仍在beta，所以我們要手動設置rails5版本號。
<code>gem 'rails', '>= 5.0.0.beta3', '< 5.1'</code>
- 更新＆安裝套件(gem)
<code>bundle update</code>
<code>bundle install</code>

###2. Creating the Database
- 不想建資料庫
	- SQLite
- 想練習連database的可以考慮用docker
	- GUI tool: [Kitematic](https://docs.docker.com/kitematic/userguide "Title") 
	
### 3. Generating the Scaffold
- Scaffold
	<code>bin/rails generate scaffold Product title:string description:text image_url:string price:decimal</code>
- 快速建立CRUD應用
	- database scheme
	- model
	- view
	- controller
	- routes
	- test unit

### 4. Applying the Migration
- 將我們新增的migration應用到資料庫中
<code>rails db:migrate</code>
- 舊版本是(rails 5將rake跟rails指令整合在一起)
<code>rake db:migrate</code>
- 將資料庫回復到上次migration的狀態
<code>rake db:rollback</code>
	- 不要隨便rollback! 開發用資料庫跟正式資料庫上版控時可能會不同步!
	- 移除欄位之類的指令 -> 開新的migration
- 查看相關rake指令
<code>rake -T</code>

### 5. Seeing the List of Products
- 開啟Server(預設127.0.0.1(localhost)、3000 port)
<code>bin/rails server</code>
- Browser
<code>localhost:3000</code> or <code>127.0.0.1:3000</code>
- config/routes.rb
<code>resources :products</code>
<code>rake routes</code>
- 建立種子資料
<code>rake db:seed</code>

### 6. Iteration A2: Making Prettier Listings
 1. Assets Pipeline
	- CoffeeScript
	- SCSS
 2. ActionView 
	- 內建Helper
	- 自己定義Helper
 3. 前端框架
	- react, anguler...etc
	- bootstrap

### What We Just Did
 1. 建立開發用database
 2. 利用migration更改database schema
 3. 利用scaffold generator產生product的頁面
 4. 美化頁面

### 開發工具
- git
	- 帳號密碼、Access Key相關的東西盡量不要放上git! (除非你是自己架的git)
	- 請將敏感資料加入 .gitignore
	- 部署機器在本地上傳就好
- 版控工具
	- sourcetree
