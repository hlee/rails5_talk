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










### resource and references

* [official log](http://weblog.rubyonrails.org)
* [thoughtbot upcase](https://thoughtbot.com/upcase/videos/rails-5-whats-in-it-for-you)








