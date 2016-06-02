# Task F: Add a Dash of Ajax
       <div id="cart">
         <%= render @cart %> 
       </div>
   include CurrentCart
   before_action :set_cart
       format.js { @current_item = @line_item }
 <% if line_item == @current_item %>
   <tr id="current_item">
 <% else %>
   <tr>
 <% end %>
 $('#current_item').css({'background-color':'#88ff88'}).
 animate({'background-color':'#114411'}, 1000);
 if ($('#cart tr').length == 1) { $('#cart').show('blind', 1000); }
   def hidden_div_if(condition, attributes = {}, &block)
     if condition
       attributes["style"] = "display: none"
     end
     content_tag("div", attributes, &block) 
   end
 config.action_cable.disable_request_forgery_protection = true
       @products = Product.all 
       ActionCable.server.broadcast 'products',
         html: render_to_string('store/index', layout: false) 
  App.productsChannel =
    App.cable.subscriptions.create { channel: "ProductsChannel" },  
      received: (data) -> $(".store #main").html(data.html)
 <% if @cart %>
 <% end %>
   assert_select 'h2', 'Your Cart'
   assert_select 'td', "Programming Ruby 1.9"