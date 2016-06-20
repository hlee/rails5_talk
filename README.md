# new feature in rails 5


### ActionCable

[ActionCable](https://github.com/rails/actioncable) is the new thing in Rails 5, it is a framework for real time communication over web sockets.

An ActionCable server will expose one or more channels into where a consumer will be able to subscribe via web socket connection, each channel broadcast messages to all subscribers in realtime.

Current ActionCable dependencies include the need of Redis and its feature PubSub. It also requires on the Ruby side faye-websocket and celluloid although these may change in the future.

An application with ActionCable support requires of 3 basic components:

* Connection: Is a class derived from ActionCable::Connection::Base, this is where authorization and connection establishments happen.
* Channel: This is what we will expose via web sockets and it is derived from ActionCable::Channel::Base
* Client: Is a javascript library to become an ActionCable client.

ActionCable will require from us to start an additional server to respond to web sockets connections.

If you want to explore ActionCable, the Rails team has put together a Github repository with a [sample application](https://github.com/rails/actioncable-examples).









### resource and references

* [official log](http://weblog.rubyonrails.org)
* [thoughtbot upcase](https://thoughtbot.com/upcase/videos/rails-5-whats-in-it-for-you)








