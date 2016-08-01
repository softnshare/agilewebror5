# Ch.21 Action View

## Using Template
- Where the templates go
	- render()默認會有一個對應的template in `app/view`
	- render(action: 'action_name') 
	- render(template: 'controller/name')
	- render(file: 'dir/template')
		
```ruby
render :edit
render action: :edit
render "edit"
render "edit.html.erb"
render action: "edit"
render action: "edit.html.erb"
render "books/edit"
render "books/edit.html.erb"
render template: "books/edit"
render template: "books/edit.html.erb"
render "/path/to/rails/app/views/books/edit"
render "/path/to/rails/app/views/books/edit.html.erb"
render file: "/path/to/rails/app/views/books/edit"
render file: "/path/to/rails/app/views/books/edit.html.erb"
```
- The environment they run in 
	- instance variable可以在template內被使用
		- `<div> <%= @instace_variable %> </div>`
	- 可以在template內被access的object
		- flash, headers, logger, params, request, response, and session
- What goes in a Template
	- html、xhtml => erb
	- js => erb
	- json => jbuilder
	- xml、rss、atom => builder
## Using Helper
- 寫在對應controller的Helper Module
- 全域的Helper寫在`application_helper.rb`
- [參照ihower](https://ihower.tw/rails4/actionview-helpers.html)
- [rails awesome gem](https://github.com/hothero/awesome-rails-gem#view-helper)

- Processing Forms
	- ![Proccssing Forms](https://i.imgur.com/2jt6AWR.png)

## Uploading Files
- 上傳
	- 存binary在database(見書上範例)
	- 圖片
		- [MiniMagick](https://github.com/minimagick/minimagick) 
	- [carrierwave](https://github.com/carrierwaveuploader/carrierwave)
	- [fog](https://github.com/fog/fog)
- 下載
	- [send_data](http://apidock.com/rails/ActionController/DataStreaming/send_data)
	- [send_file](http://apidock.com/rails/v4.2.1/ActionController/DataStreaming/send_file)
	
## Reducing Maintenance with Layouts and Partials
### Layouts
- The layout sets out a standard HTML page, with the head and body sections.
#### Locating Layout Files
- controller-specific
- ex. store_controller => store.html.erb
	- `app/views/layouts`
- override
	```ruby
	class StoreController < ApplicationController  
	  layout "standard", except: [ :rss, :atom ]
	end
	```
- dynamic layout
	```ruby
	class StoreController < ApplicationController
	  layout :determine_layout # ...
	  private
	  def determine_layout 
	    if Store.is_closed?
	      "store_down"
	    else
	      "standard"
	    end
	  end
	end
	```
- Passing Data to Layouts
	- instance variable(@)
- yield
	- default
	- assign: `<%= yield :layout %>`
	- content_for
	- ex. 
		- template
		```html
		<html>
		  <head>
			<%= yield :head %>
		  </head>
		  <body>
			<%= yield %>
		  </body>
		</html>
		````
		- layout
		```html
		<% content_for :head do %>
		  <title>A simple page</title>
		<% end %>

		<p>Hello, Rails!</p>
		```
### Partials
- DRY
	- ex. `_article.html.erb`
	```html
	<div class="article">
	  <div class="articleheader">
	    <h3><%= article.title %></h3>
	  </div>
	  <div class="articlebody"> 
	    <%= article.body %>
	  </div>
	</div>
	```
	- other template
	```html
	<%= render(partial: "article", object: @article) %>
	```
- Partials and Collections
	- ex. index.html.erb
	```html
	<h1>Products</h1>
	<%= render partial: "product", collection: @products %>
	```
	- ex. _product.html.erb
	```html
	<p>Product Name: <%= product.name %></p>
	```
- Shared Templates
	- Rails assumes that the target template is in the current controller’s view directory.
	- 如果有'/'，則預設在app/views底下。
	```ruby
	<%= render("shared/header", locals: {title: @article.title}) %>
	<%= render(partial: "shared/post", object: @article) %>
	```

# CH25 plugins
## Payment
- activemerchant
## Templating Engine
- haml
## Pagination
- Kaminari