# Basic User Administration System

## I1. Adding Users

### Create User Table

`$ bin/rails generate scaffold User name:string password:digest`

has_secure_password() => `gem 'bcrypt', '~> 3.1.7'`

The following validations are added automatically:
- Password must be present on creation
- Password length should be less than or equal to 72 characters
- Confirmation of password (using a password_confirmation attribute)

```
user = User.new(name: 'yoyoyo', password: '', password_confirmation: 'nomatch')
user.save                                     # => false, password required
user.password = 'mUc3m00RsqyRe'
user.save                                     # => false, confirmation doesn't match
user.password_confirmation = 'mUc3m00RsqyRe'
user.save                                     # => true
user.authenticate('notright')                 # => false
user.authenticate('mUc3m00RsqyRe')            # => user
```
> source code: https://github.com/rails/rails/blob/3-2-stable/activemodel/lib/active_model/secure_password.rb

> api doc: http://api.rubyonrails.org/classes/ActiveModel/SecurePassword/ClassMethods.html

### Add User Form

`The <fieldset> tag:`
- used to group related elements in a form
- draws a box around the related elements.

`The <legend> tag:`
- defines a caption for the <fieldset> element.

Group related elements in a form:
```
<form>
  <fieldset>
    <legend>Health Info</legend>
    Height: <input type="text" />
    Weight: <input type="text" />
  </fieldset>
</form>
```

## I2. Authenticating Users

### Sessions
- store small amounts of data that will be persisted between requests

#### Session Storage
- ActionDispatch::Session::CookieStore (Rails uses this by default)
  - store around 4kB of data (usually enough)

- ActionDispatch::Session::CacheStore - Stores the data in the Rails cache.

- ActionDispatch::Session::ActiveRecordStore
  - Stores the data in a database using Active Record. (require activerecord-session_store gem)
  ```
  $ rails g active_record:session_migration
  $ rake db:migrate
  ```

#### More About Session

http://guides.rubyonrails.org/action_controller_overview.html#session

https://rocodev.gitbooks.io/rails-102/content/chapter2-rails/cookies-and-session.html

## I3.

### Controller Filters

- Filters are methods that are run before, after or "around" a controller action
  - before action
    - may halt the request cycle
    - common case: user authentication

    `skip_before_action :require_login, only: [:new, :create]`

  - after action

  - around action

NOTE: Filters are inherited, so if you set a filter on ApplicationController, it will be run on every controller in your application

#### More About Filters

http://guides.rubyonrails.org/action_controller_overview.html#filters

https://ihower.tw/rails4/actioncontroller.html

#### Discussion
##### Loging and Authetication tools(gem)
- rorify
- cancancan
- devise
- request_store
- administrate
- activeadmin
- pundit 

