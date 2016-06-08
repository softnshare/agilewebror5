# Task J: Internationalization
- i18n
### 設定多語種類
```ruby
# ~/config/initializers/i18n.rb

I18n.default_locale = :en
LANGUAGES = [
  ['English', 'en'],
  ["Espa&ntilde;ol".html_safe, 'es']
]

```
- `html.safe` 不要用在用戶Input產生html的地方
- [Ruby China: html.safe & raw](https://ruby-china.org/topics/16633)
- [iHower: Action View - Helpers 方法](https://ihower.tw/rails4/actionview-helpers.html)

### Routes
- 在想要加入多語的地方加入optional的route

```ruby=
# ~/config/routes.rb

scope '(:locale)' do
  resources :orders
  resources :line_items
  resources :carts
  get 'store/index'
  root 'store#index', as: 'store', via: :all
end
```

### Setup
- Load前先確認當前語系是否需要多語化
```ruby=
# app/controllers/application_controller.rb

class ApplicationController < ActionController::Base
  before_action :set_i18n_locale_from_params
# ...
protected
  def set_i18n_locale_from_params if params[:locale]
    if I18n.available_locales.map(&:to_s).include?(params[:locale]) 
      I18n.locale = params[:locale]
    else
      flash.now[:notice] = "#{params[:locale]} translation not available"
      logger.error flash.now[:notice] 
    end
  end
end
```

### I18n API
- `I18n.translate`
	- Alias: `I18n.t`
	- Helper: `t (unique dot-qualified name)`
  ```htmlembedded=
  # app/views/layouts/application.html.erb
  # ...
  <ul>
    <li><a href="http://www...."><%= t('.home') %>  </a></li>
    <li><a href="http://www..../faq"><%=   t('.questions') %></a></li>
    <li><a href="http://www..../news"><%= t('.news') %></a></li>
    <li><a href="http://www..../contact"><%= t('.contact') %></a></li>
  </ul>
  # ...
  ```

### Mapping text
- YAML
	- `config/locales/<language>.yml`
	- 貨幣符號/格式都可以translate
		- 綁定helper
	- en
	```ruby=
	layouts:
	  application:
	    title: "Pragmatic Bookshelf" 
	    home: "Home"
	    questions: "Questions"
	    news: "News"
	    contact: "Contact"
	```
	- es
	```ruby=
	layouts:
      application:
        title:       "Publicaciones de Pragmatic"
        home:        "Inicio"
        questions:   "Preguntas"
        news:        "Noticias"
        contact:     "Contacto"
	```
- HTML_safe
	- en
	```ruby=
	store:
      index:
        title_html:  "Your Pragmatic Catalog"
        add_html:    "Add to Cart"
	```
	- es
	```ruby=
	store:
      index:
        title_html:  "Su Cat&aacute;logo de Pragmatic"
        add_html:    "A&ntilde;adir al Carrito"
	```
## Tips
- 不要把locale放到session/cookie內
	- restful: stateless
- 多語設置方式 -> 用爬的
	- domain name
		- Top
		```ruby= 
		# ApplicationController
		before_action :set_locale

		def set_locale
		  I18n.locale = extract_locale_from_tld || I18n.default_locale
		end
		# Get locale from top-level domain or return nil if such locale is not available
		# You have to put something like:
		#   127.0.0.1 application.com
		#   127.0.0.1 application.it
		#   127.0.0.1 application.pl
		# in your /etc/hosts file to try this out locally
		def extract_locale_from_tld
		  parsed_locale = request.host.split('.').last
		  I18n.available_locales.map(&:to_s).include?(parsed_locale) ? parsed_locale : nil
		end
		```
		- subdomain
		```ruby= 
		# Get locale code from request subdomain (like http://it.application.local:3000)
		# You have to put something like:
		#   127.0.0.1 gr.application.local
		# in your /etc/hosts file to try this out locally
		def extract_locale_from_subdomain
		  parsed_locale = request.subdomains.first
		  I18n.available_locales.map(&:to_s).include?(parsed_locale) ? parsed_locale : nil
		end
		```
	- URL 
		- route parameter
			- 見書本範例
		- query string
			- 有設定localize route好像有辦法自動抓query string
				- 原理不明
	- 其他方式
		- GeoIP
		- Browser setting
- Keywords
	- [gettext](https://zh.wikipedia.org/wiki/Gettext)


# Reference
1. [RailsGuides: Rails Internationalization (I18n) API
](http://guides.rubyonrails.org/i18n.html)
2. [Juanito Fatas: Rails I18n 介紹 (RailsGuides的中文心得)](http://juanitofatas.com/2014/06/25/rails-i18n-intro/)
3. [Phraseapp 翻譯平台](https://phraseapp.com/pricing)


