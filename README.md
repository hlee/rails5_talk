# new feature in rails 5


### ActionCable

[ActionCable](https://github.com/rails/actioncable) is the new thing in Rails 5, it is a framework for real time communication over web sockets.

An ActionCable server will expose one or more channels into where a consumer will be able to subscribe via web socket connection, each channel broadcast messages to all subscribers in realtime.

Current ActionCable dependencies include the need of [Redis](http://redis.io) and its feature [PubSub](http://redis.io/topics/pubsub). It also requires on the Ruby side [faye-websocket](https://github.com/faye/faye-websocket-ruby) and celluloid although these may change in the future.

An application with ActionCable support requires of 3 basic components:

* Connection: Is a class derived from ActionCable::Connection::Base, this is where authorization and connection establishments happen.
* Channel: This is what we will expose via web sockets and it is derived from ActionCable::Channel::Base
* Client: Is a javascript library to become an ActionCable client.

ActionCable will require from us to start an additional server to respond to web sockets connections.

If you want to explore ActionCable, the Rails team has put together a Github repository with a [sample application](https://github.com/rails/actioncable-examples).



### Rails API

Rails API was the opinionated way to build APIs with Ruby on Rails. Rails API was a separate gem with its own generator to build the skeleton of API applications.

Its goal was to help to create fast Rails API application by removing unnecessary middleware while providing sensitive defaults for these kinds of applications.

Starting with Rails 5, Rails API is integrated into the framework, there is no need to include additional gems. To create a new API only Rails app we just pass the --api option to rails command.

```ruby
$ rails new michelada-api --api
```

The new application will include ActiveModelSerializers and will remove JQuery and Turbolinks gems from our Gemfile. Also, config/application.rb and aplication_controller.rb files to have config.api_only = true option and to not check for CSRF protection, also it will be inherited from ActionController::API.

```ruby
# application.rb file
module MicheladaApi
  class Application < Rails::Application
     config.api_only = true
  end
end

# application_controller.rb file
class ApplicationController < ActionController::API
end
```

If you want to learn more about creating Rails API application, here is a starter post from WyeWorks: [”HOW TO BUILD A RAILS 5 API ONLY AND EMBER APPLICATION”](http://wyeworks.com/blog/2015/6/30/how-to-build-a-rails-5-api-only-and-ember-application/)




### Render a template outside controllers

Rails 5 allows you to render templates or inline code outside controllers. This feature is important and useful for ActiveJob and the new ActionCable (we will discuss this one later).

ActionController::Renderer is what makes this happen and it’s available in our ApplicationController class.

Let’s dig into few examples on how this works.

```ruby
# render inline code
ApplicationController.render inline: '<%= "Hello Rails" %>' # => "Hello Rails"

# render a template
ApplicationController.render 'sample/index' # => Rendered sample/index.html.erb within layouts/application (0.0ms)

# render an action
SampleController.render :index # => Rendered sample/index.html.erb within layouts/application (0.0ms)

# render a file
ApplicationController.render file: ::Rails.root.join('app', 'views', 'sample', 'index.html.erb') # =>   Rendered sample/index.html.erb within layouts/application (0.8ms)
```

This is really nice but what if we need to pass assigns or locals to our template?


```ruby
# Pass assigns
ApplicationController.render assigns: { rails: 'Rails' }, inline: '<%= "Hello #{@rails}" %>' # => "Hello Rails"

# Pass locals
ApplicationController.render locals: { hello: 'Hello' }, assigns: { rails: 'Rails' }, inline: '<%= "#{hello} #{@rails}" %>' # => "Hello Rails"
Now if we have to use route helper’s functions like root_url or any other helper that needs access to the environment we can do it, but what ActionController::Renderer will receive is not a real environment. Take in account that we might use this functionality outside HTTP request.

ApplicationController.render inline: '<%= root_url %>' # => "http://example.org/"

# Inspect ActionController::Renderer environment
ApplicationController.renderer.defaults # => {:http_host=>"example.org", :https=>false, :method=>"get", :script_name=>"", "rack.input"=>""}

# To modify this environment we have to explicitly create a renderer
renderer = ApplicationController.renderer.new(
    http_host: 'michelada.io'
  ) # => #<#<Class:0x007fdf9985a338>:0x007fdf947981c0 @env={"HTTP_HOST"=>"michelada.io", "HTTPS"=>"off", "SCRIPT_NAME"=>"", "rack.input"=>"", "REQUEST_METHOD"=>"GET", "action_dispatch.routes"=>#<ActionDispatch::Routing::RouteSet:0x007fdf93d29450>}>
  
renderer.render inline: '<%= root_url %>' # => "http://michelada.io/"

```



### ActiveRecord’s attribute API

This new API adds functionality on top of ActiveRecord models, it made possible to override an attribute type to be a different type.

Consider the case where we have a field in the database defined as decimal but in our app we only care for the integer part of the number. We can in our app just ignore the decimal part and format our number everywhere we need to use it to only display the integer part.

With attribute API we can do this in an easy way:

```ruby
class Book < ActiveRecord::Base
end

book.quantity # => 12.0

class Book < ActiveRecord::Base
  attribute :quantity, :integer
end

book.quantity # => 12
```

Here we are overriding the automatically generated attribute from the database schema to be cast in our model as an integer instead the original decimal. For every interaction of our model with the database, the attribute will be treated as a decimal as it should be.

We can even define our own custom types just by creating a class derived from `ActiveRecord::Type::Value` and implementing its contract to `#cast`, `#serialize`, and `#deserialize` values.

Custom attributes will honor ActiveModel::Dirty to track changes in our models. Also, these new attributes can be virtual, so there is no need to be backed by a table column.


* [Attributes API documentation](https://github.com/rails/rails/blob/master/activerecord/lib/active_record/attributes.rb)
* [Introduction to Rails 5 Attributes](http://jakeyesbeck.com/2015/12/20/rails-5-attributes/)
* [Using the Rails 5 Attributes API Today, in Rails 4.2](https://nvisium.com/blog/2015/06/22/using-rails-5-attributes-api-today-in/)


### ApplicationRecord

And speaking of ActiveRecord, you might have noticed that the Product model above inherits from ApplicationRecord, not ActiveRecord::Base. This is because Rails now generates that class so that we can add all our customizations there instead of mokeypatching ActiveRecord::Base. Read more about it here:

```ruby
class Post < ApplicationRecord
end
```

* [ApplicationRecord in Rails 5](http://blog.bigbinary.com/2015/12/28/application-record-in-rails-5.html)



### or method in ActiveRecord::Relation

Finally ActiveRecord::Relation is getting #or method, this will allow us to write queries with ActiveRecord DSL as follows:

```ruby
Book.where('status = 1').or(Book.where('status = 3'))
\# => SELECT * FROM books WHERE (status = 1) OR (status = 3)
#or method accepts a second relation as a parameter that is combined with an or. #or can also accept a relation in a form of model scope.

class Book < ActiveRecord::Base
  scope :new_coming, -> { where(status: 3) }
end

Book.where('status = 1').or(Book.new_coming)
\# => SELECT * FROM books WHERE (status = 1) OR (status = 3)
```

### ActiveRecord::Relation#in_batches

The new #in_batches method yields a relation, unlike #find_in_batches which yields an array. We can use this method for things like this:

```ruby
Person.where('age >= 18').in_batches(of: 1000) do |people|
  people.update_all(can_vote: true)
end

```

* [Documentation](https://github.com/rails/rails/blob/v5.0.0.beta1/activerecord/lib/active_record/relation/batches.rb#L132)
* [New in ActiveRecord: #in_batches](http://crypt.codemancers.com/posts/2015-12-23-new-in-batches-method-in-active-record/)




### MySQL ActiveRecord adapter gets JSON support

If you happen to run your Rails application on top of MySQL 5.7.8 then your database have a new native JSON data type.

From Rails 5 you should be able to use this new data type within your ActiveRecord models.

```ruby
create_table :json_data_type do |t|
  t.json :settings
end
```

### New command router

Ever accidentally typed rails db:migrate or rake generate model? Figuring out which command to use is often confusing to beginners. Now all such commands are routed through rails, so we will be able to use commands like rails db:migrate. The rake commands are still available if you want to use them.


### Supports only Ruby 2.2.2+

Rails has always pushed the community towards the latest Ruby versions, and 5.0 is no different. This version will only work on Ruby 2.2.2 and above. This lets Rails to improve performance by making use of the latest improvements in the Ruby garbage collector (like incremental GC and symbol GC).

If you’re upgrading Ruby anyway, you might want to look at Ruby 2.3 which was released recently with a bunch of new features.



### Upgrading

With the release of Rails 5, all versions of Rails from 4.1.x and below will no longer be supported. Bug fixes will only be released for Rails 5.0.x and 4.2.x. Now would be a good time to upgrade, at least to 4.2.x if you haven’t already done so. If you’re planning to upgrade to Rails 5.0, these resources could be useful:

* [Upgrading to Ruby on Rails 5.0 from Rails 4.2 – application use case](http://dev.mensfeld.pl/2015/12/upgrading-to-ruby-on-rails-5-0-from-rails-4-2-application-use-case/)
* [A Guide for Upgrading Ruby on Rails](http://edgeguides.rubyonrails.org/upgrading_ruby_on_rails.html)



### others

* [Better Minitest test runner](#)
* [Turbolinks](#) 

### resource and references

* [official log](http://weblog.rubyonrails.org)
* [thoughtbot upcase](https://thoughtbot.com/upcase/videos/rails-5-whats-in-it-for-you)
* [rails 5: what's new](https://medium.com/evil-martians/the-rails-5-post-9c76dbac8fc#.fl246pnz0)
* [rails 5 series](http://blog.bigbinary.com/categories/Rails-5)






