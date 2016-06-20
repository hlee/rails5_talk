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

### MySQL ActiveRecord adapter gets JSON support

If you happen to run your Rails application on top of MySQL 5.7.8 then your database have a new native JSON data type.

From Rails 5 you should be able to use this new data type within your ActiveRecord models.



### resource and references

* [official log](http://weblog.rubyonrails.org)
* [thoughtbot upcase](https://thoughtbot.com/upcase/videos/rails-5-whats-in-it-for-you)








