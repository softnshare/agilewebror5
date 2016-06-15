# Task H: Sending Mail

## Setting config

### config.action_mailer.delivery_method

+ :smtp
+ :sendmail
+ :test

### example:

`/config/environments/development.rb`

```ruby
config.action_mailer.delivery_method = :smtp

config.action_mailer.smtp_settings = {
  address: 'smtp.gmail.com',
  port: 587,
  enable_starttls_auto: true,
  authentication: "plain",
  user_name: ENV['SMTP_USERNAME'],
  password: ENV['SMTP_PASSWD']
}
```

## generate mailer command takes

### command

```
bin/rails generate mailer HelloWorld hi
```

### result

```
Running via Spring preloader in process 52911
      create  app/mailers/hello_world_mailer.rb
      create  app/mailers/application_mailer.rb
      invoke  erb
      create    app/views/hello_world_mailer
      create    app/views/hello_world_mailer/hi.text.erb
      create    app/views/hello_world_mailer/hi.html.erb
      invoke  test_unit
      create    test/mailers/hello_world_mailer_test.rb
      create    test/mailers/previews/hello_world_mailer_preview.rb
```

`/app/mailers/application_mailer.rb`

```ruby
class ApplicationMailer < ActionMailer::Base
  default from: 'from@example.com'
  layout 'mailer'
end
```

PS:要手動建立 mailer layout（`/app/views/layouts/mailer.html.erb`）

```ruby
class HelloWorldMailer < ApplicationMailer
  def hi(name)
    @greeting = "Hi #{name}"

    mail to: "wnagjoe1105@gmail.com", subject: "Rails book club", cc: "joe@dinngo.co"
  end
end
```

`/app/views/layouts/mailer.html.erb`

```ruby
<!DOCTYPE html>
<html lang="zh-Hant">

<head>
  <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
</head>

<body style="font-family:Arial, Helvetica, sans-serif; margin:0px; padding:0px">
  <%= yield %>
</body>
```

`/app/views/hello_world_mailer/hi.html.erb`

```ruby
<h1>HelloWorld</h1>
<p><%= @greeting %></p>
```

## Visual email testing

[mail_view](https://github.com/basecamp/mail_view)

## supplement

### interception email

`/config/initializers/action_mailer.rb`

```ruby
ActionMailer::Base.register_interceptor(BanDisposableInterceptor)
```

`/lib/ban_disposable_interceptor.rb`

```ruby
class BanDisposableInterceptor
  def self.delivering_email(mail)
    mail.perform_deliveries = false unless BanDisposableValidator.valid?(mail.to[0])
  end
end
```

