#Task C: Catalog Display

## Rails Generate Controller

`$ bin/rails generate controller Store index`

## Route

* 辨識 HTTP Request 的 URL 網址，對應到設定的Controller Action

* 處理網址內的參數字串，例如：/products/show/1 送到 Products controller 的 show action 時，

  會將 params[:id] 設定為 1

### 設定網站首頁
(p.102)

`root :to => ‘store#index’`

### Named Routes

幫助我們產生URL helper，而不需要用 {:controller => ‘store’, :action => 'index'} 的方式：

`get ‘store#index’, :as => "store"`

as 的部份會產生 Helpers：

* store_path: 相對路徑（較常用）

* store_url: 絕對路徑（在 Email 信件等情境，網址需包含 domain 時使用）
 
### Resources

`resources :products, as: 'items'`

| HTTP Verb	| Path	| Controller#Action |	Named Helper |
|---|---|---|---|
| GET | /products	| products#index	| items_path |
| GET	| /products/new	| products#new	| new_item_path |
| POST | /products	| products#create	| items_path |
| GET	| /products/:id |	products#show |	item_path(:id) |
| GET	| /products/:id/edit |	products#edit	| edit_item_path(:id) |
| PATCH/PUT |	/products/:id |	products#update	| item_path(:id) |
| DELETE	| /products/:id	| products#destroy |	item_path(:id) |


### More About Routing 

http://rails.ruby.tw/routing.html 

(en: http://guides.rubyonrails.org/routing.html / zh-cn: http://guides.ruby-china.org/routing.html)

https://ihower.tw/rails4/routing.html

http://guides.ruby.tw/ruby-rails-style-guides/#rails.routing

## Application Layout

Rails 內建的靜態檔案 (Assets) 輔助方法：

* **合併** Stylesheet 和 JavasSript 檔案，加速瀏覽器下載

* **編譯** Sass 和 CoffeeScript 等透過 Assets template engine 產生的靜態檔案

* 在靜態檔案網址中加上 **時間序號**，如果內容有修改則會重新產生

* 變更Assets host主機位址時，可以一次搞定（ex. 搬移至 CDN）

### Stylesheets

* app/assets

* lib/assets

* vendor/assets (3rd party assets: jQuery, bootstrap ......)

可以指定相對於根目錄的完整路徑，或是 URL 也可以。舉例：

`<%= stylesheet_link_tag "main" %>`

### More About Assets

http://rails.ruby.tw/asset_pipeline.html

http://rails-practice.com/content/Chapter_6/6.1.html

## Caching

開發＆測試環境預設為不啟用快取機制，需調整設定：

`config.action_controller.perform_caching = true`

或是在終端機下指令：

`$ rails dev:cache`

### Fragment Caching

`<% cache @product do %>`

產生的 cache key：

`Write fragment views/products/3-20150620164035711340000/b0699b1b8be94ebd1bfcfe74a21571f8 (21.5ms)`

```
p = Product.last
p.cache_key
 => "products/3-20150620164035711340000" 
```
給 cache 增加一些參數：

`<% cache(action: 'new', action_suffix: 'all_products') do %>`

產生的 cache key：

`Write fragment views/localhost:3000/products/new?action_suffix=all_products/02c540e3ab26f72d5e9273d5824c204e`

### Cache Data

```
Rails.cache.read("city")   # => nil
Rails.cache.write("city", "Duckburgh")
Rails.cache.read("city")   # => "Duckburgh"
```

write 支援 expires_in 參數可以設定時效

### More About Caching

http://rails.ruby.tw/caching_with_rails.html

http://blog.xdite.net/posts/2012/09/03/cache-digest-new-strategy

http://www.nateberkopec.com/2015/07/15/the-complete-guide-to-rails-caching.html
