# Introduction
## What
Sinatra is a domain-specific language for building websites, web services, and web applications in Ruby. It emphasizes a minimalistic approach to development, offering only what is essential to handle HTTP requests and deliver responses to clients.
## Example
The first Sinatra application
```
require 'sinatra'
get '/' do 
  "Hello, world!" 
end
```
Run
```
ruby -rubygems server.rb
```
Access
[http://localhost:4567/](http://localhost:4567/.)
# Foundamentals
## Routing
The core of any Sinatra application is the ability to respond to one or more routes. Routes are the primary way that users interact with your application.
### Hypertext Transfer Protocol
HTTP Message
Start line, Headers, Message body
### Verbs
GET request is used to ask a server to return a representation of a resource.
POST is used to submit data to a web server.
PUT is used to create or update a representation of a resource on a server.
DELETE is used to destroy a resource on a server.
PATCH is used to update a portion of a resource; this is in contrast to PUT, which replaces it wholesale.
### Common Route Definition
```
require 'sinatra'

get '/' do 
  "Triggered via GET"
end
post '/' do 
  "Triggered via POST" 
end
put '/' do 
  "Triggered via PUT" 
end
delete '/' do 
  "Triggered via DELETE" 
end
patch '/' do 
  "Triggered via PATCH" 
end
options '/' do 
  "Triggered via OPTIONS" 
end
```
### Many URLs, Similar Behaviors
```
require 'sinatra'
['/one', '/two', '/three'].each do |route|
  get route do
    "Triggered #{route} via GET"
  end
end
```
### Routes with Parameters
Get
```
require 'sinatra'
get '/:name' do 
  "Hello, #{params[:name]}!" 
end
```
Post
```
require 'sinatra'
post '/login' do
  username = params[:username]
  password = params[:password]
  "#{username}:#{password}"
end
```
### Routes with Query String Parameters
```
require 'sinatra'
get '/:action' do
  # assumes a URL in the form /action?name=XYZ
  "You asked for #{params[:action]} as well as #{params[:name]}"
end
```
### Routes with Wildcards
```
require 'sinatra'
get '/*' do 
  "You passed in #{params[:splat]}" 
end
```
### Routes with Regular Expressions
```
require 'sinatra'
get %r{/(sp|gr)eedy} do 
  "You got caught in the greedy route!" 
end
```
### Halting a Request
```
require 'sinatra'
get '/halt' do
  'You will not see this output.'
  halt 500
end

```
### Passing a Request
```
require 'sinatra'
get %r{/(sp|gr)eedy} do
  pass if request.path =~ /\/speedy/
  "You got caught in the greedy route!"
end
get '/speedy' do
  "You must have passed to me!"
end
```
### Redirecting a Request
```
require 'sinatra'
get '/redirect' do
  redirect 'http://www.google.com'
end
get '/redirect2' do
  redirect 'http://www.google.com', 301
end
```
## Static Files
```
#modify static directory 'public'
set :pub lic_folder, File.dirname(__FILE__) + '/your_custom_location'
```
## Views
Inline Templates
External View Files
### Passing Data into Views
main.rb
```
require 'sinatra'
get '/index' do
  @name = 'Random User'
  erb :index
end
```

views/index.erb
```
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>External template</title>
  </head>
  <body>
    <h1><%= @name %> Worked!</h1>
  </body>
</html>
```
## Filters
Sinatra supports filters as a way to modify requests and responses, both before and after a route has been executed.
```
require 'sinatra'
before do
  @before_value = 'foo'
end
get '/' do
  "before_value has been set to #{@before_value}"
end
after do
  puts "After filter called to perform some task."
end
```
## Handling Errors
### 404 Not Found
```
require 'sinatra'
before do 
  content_type :txt 
end
not_found do
  "Whoops! You requested a route that wasn't available."
end
```
### 500 Internal Server Error
```
require 'sinatra'
configure do
  set :show_exceptions, false
end

get '/div_by_zero' do
  0 / 0
  "You won't see me."
end

error do
  "NOT WORKING?"
end
```
## Configuration
```
require 'sinatra'

configure do
  mime_type :plain, 'text/plain'
end

before '/plain-text' do
  content_type :txt
end

get '/html' do
  content_type :html
  '<h1>You should see HTML rendered.</h1>'
end

get '/plain-text' do
  '<h1>You should see plain text rendered.</h1>'
end
```
## HTTP Headers
```
require 'sinatra'

before do
  content_type :txt
end

get '/' do
  headers "X-Custom-Value" => "This is a custom HTTP header." 'Custom header set'
end

get '/multiple' do
  headers "X-Custom-Value" => "foo", "X-Custom-Value-2" => "bar" 'Multiple custom headers set'
end
```
## Exploring the request Object
```
require 'sinatra'

before do
  content_type :txt
end

get '/' do
  request.env.map { |e| e.to_s + "\n" }
end
```
### Caching
### Setting Headers Manually
```
require 'sinatra'

before do
  content_type :txt
end

get '/' do
  t = Time.at(Time.now.to_i + (60 * 60)).to_s
  headers "Cache-Control" => "public, must-revalidate, max-age=3600",
          "Expires" => t
  "This page rendered at #{Time.now}."
end
```
### Settings Headers via expires
```
require 'sinatra'

before do
  content_type :txt
end

get '/cache' do
  expires 3600, :public, :must_revalidate
  "This page rendered at #{Time.now}."
end
```
### ETags
ETags, short for entity tags, are another way to represent how fresh a resource is via HTTP. They are server-generated identifiers that are used to “fingerprint” a resource in a given state. A client can safely assume that if the ETag for a resource has changed, then the resource itself has changed and should be fetched from the server again.

**Generating ETags**
```
require 'sinatra'
require 'uuid'

before do
  content_type :txt
  @guid = UUID.new.generate
end

get '/etag' do
  etag @guid
  "This resource has an ETag value of #{@guid}."
end
```

**Weak ETags**
```
require 'sinatra'
require 'uuid'
before do
  content_type :txt
  @guid = UUID.new.generate
end
get '/etag' do
  etag @guid, :weak
  "This resource has an ETag value of #{@guid}."
end
```
## Sessions
Although HTTP is itself a stateless protocol, one way to maintain the state for a user is through the use of cookie-based sessions. In this approach, a cookie (in this case, one named rack.session) is stored client-side and used to house data related to the activity in the current user’s session.
### Setting/Fetching a Session

```
require 'sinatra'

configure do
  enable :sessions
end

before do
  content_type :txt
end

get '/set' do
  session[:foo] = Time.now
  "Session value set."
end

get '/fetch' do
  "Session value: #{session[:foo]}"
end
```
### Destroying a Session

```
require 'sinatra'

configure do
  enable :sessions
end

before do
  content_type :txt
end

get '/set' do
  session[:foo] = Time.now
  "Session value set."
end

get '/fetch' do
  "Session value: #{session[:foo]}"
end

get '/logout' do
  session.clear
  redirect '/fetch'
end
```
## Cookies
Cookies are small amounts of metadata stored client-side. There are essentially two types of cookies: session and persistent. They differ by the points at which they expire; session cookies are destroyed when the user closes his browser (or otherwise ends his session), and persistent cookies expire at a predetermined time that is stored with the cookie itself.

```
require 'sinatra'

get '/' do
  response.set_cookie "foo", "bar"
  "Cookie set. Would you like to <a href='/read'>read it</a>?"
end

get '/read' do
  "Cookie has a value of: #{request.cookies['foo']}."
end

get '/delete' do
  response.delete_cookie "foo"
  "Cookie has been deleted."
end
```
## Attachments
```
require 'sinatra'

before do
  content_type :txt
end

get '/attachment' do
  attachment 'story.txt'
  "Here's what will be sent downstream, in an attachment called 'story.txt'."
end
```
## Streaming
The streaming support abstracts away the differences in different Rack-compatible servers, leaving you to focus solely on developing the functionality you desire as opposed to the plumbing that makes it possible.

```
require 'sinatra'

before do
  content_type :txt
end

get '/consume' do
  stream do |out|
    out << "Wanna hear a joke about potassium?\n"
    sleep 1.5
    out << "K.\n"
    sleep 1.5
    out << "I also have one about sodium!\n"
    sleep 1.5
    out << "Na.\n"
  end
end
```
# How it works
Once you understand what is going on backstage, it becomes significantly easier to take full advantage of the available API (or even extend it).
## The Inner Self
```
require 'sinatra'

outer_self = self
get '/' do
  content_type :txt
  "outer self: #{outer_self}, inner self: #{self}"
end
```

```
outer self: main, inner self: #<Sinatra::Application:0x00007fbd188ae9e0>
```
## Helpers and Extensions
### Creating Sinatra Extensions
Define an extension that can handle both GET and POST requests to a particular URL
post_get.rb
```
require 'sinatra/base'

module Sinatra
  module PostGet
    def post_get(route, &block)
      get(route, &block)
      post(route, &block)
    end
  end
  register PostGet
end
```
server.rb
```
require 'sinatra'
require './post_get'

post_get '/' do
  "Hi #{params[:name]}"
end
```
### Helpers
Helpers and extensions are something like cousins: you can recognize them both as being from the same family, but they have quite different roles to play. Instead of calling register to let Sinatra know about them, you pass them to helpers. Most importantly, they’re available both in the block you pass to your route and the view template itself, making them effective across application tiers.

link_helper.rb
```
require 'sinatra/base'

module Sinatra
  module LinkHelper
    def link(name)
      case name
      when :about then '/about'
      when :index then '/index'
      else "/page/#{name}"
      end
    end
  end
  helpers LinkHelper
end
```
server.rb
```
require 'sinatra'
require './link_helper'

get '/' do
  erb :helper
end
```
./views/helper.erb
```
<html> 
<head> 
<title>Link Helper Test</title> 
</head> 
<body> 
<nav> 
<ul> 
<li><a href="<%= link(:index) %>">Index</a></li> 
<li><a href="<%= link(:about) %>">About</a></li> 
<li><a href="<%= link(:random) %>">Random</a></li> 
</ul>
</nav> 
</body> 
</html>
```
### Helpers Without Modules
Sometimes you need to create a helper or two that are only going to be used in one application or for a specific purpose.
```
require 'sinatra'

helpers do
  def link(name)
    case name
    when :about then '/about'
    when :index then '/index'
    else "/page/#{name}"
    end
  end
end

get '/' do
  erb :index
end

get 'index.html' do
  redirect link(:index)
end

__END__

@@index
<a href="<%= link :about %>">about</a>
```
## Request and Response
The next step in understanding Sinatra’s internals is to examine the flow of a request, from parsing to delivery of a response back to the client.
### Rack
It’s an extremely simple protocol specifying how an HTTP server (such as Thin, which we’ve used throughout the book) interfaces with an application object, like Sinatra::Application, without having to know anything about Sinatra in particular.
### Sinatra Without Sinatra
```
module MySinatra
  class Application

    def self.call(env)
      new.call(env)
    end

    def call(env)
      headers = {'Content-Type' => 'text/html'}
      if env['PATH_INFO'] == '/'
        status, body = 200, 'hi'
      else
        status, body = 404, "Sinatra doesn't know this ditty!"

      end

      headers['Content-Length'] = body.length.to_s
      [status, headers, [body]]
    end
  end
end

require 'thin'
Thin::Server.start MySinatra::Application, 4567
```
## Rack It Up
The Rack specification supports chaining filters and routers in front of your application. In Rack slang, those are called middleware. This middleware also implements the Rack specification; it responds to call and returns an array as described above. Instead of simply creating that array on its own, it will use different Rack endpoint or middleware and simply call call on that object. Now this middleware can modify the request (the env hash), modify the response, decide whether or not to call the next endpoint, or any combination of those.
### Middleware
Rack has an additional specification for middleware. Middleware is created by a factory object. This object has to respond to new; new takes at least one argument, which is the endpoint that will be wrapped by the middleware. Finally, the middleware returns the wrapped endpoint.
config.ru
```
MyApp = proc do |env|
  [200, {'Content-Type' => 'text/plain'}, ['ok']]
end

class MyMiddleware
  def initialize(app)
    @app = app
  end

  def call(env)
    if env['PATH_INFO'] == '/'
      @app.call(env)
    else
      [404, {'Content-Type' => 'text/plain'}, ['not ok']]
    end
  end
end

# this is the actual configuration
use MyMiddleware
run MyApp
```
Run
```
rackup -p 4567 -s thin
```
### Sinatra and Middleware
you can use any Sinatra application as middleware.
```
require 'sinatra' 
require 'rack'

# A handy middleware that ships with Rack 
# and sets the X-Runtime header 
use Rack::Runtime

get('/') { 'Hello world!' }
```

The class, Sinatra::Application, is the factory creating the configured middleware instance (which is your application instance). When the request comes in, all before filters are triggered. Then, if a route matches, the corresponding block will be executed. If no route matches, the request is handed off to the wrapped application. The after filters are executed after we’ve got a response back from the route or wrapped app. Thus, your application is Rack middleware.
## Dispatching
Sinatra relies on the “one instance per request” principle. However, when running as middleware, all requests will use the same instance over and over again. Sinatra performs a clever trick here: instead of executing the logic right away, it duplicates the current instance and hands responsibility on to the duplicate instead. Since instance creation (especially with all the middleware being set up internally) is not free from a performance and resources standpoint, it uses that trick for all requests (even if running as endpoint) by keeping a prototype object around.
# Modular Applications
Normal Sinatra applications actually live in Sinatra::Appli cation, which is a subclass of Sinatra::Base. If we do, we usually don’t use Sinatra::Application, but instead we create our own subclass of Sinatra::Base. This style is called a modular application, as opposed to classic applications that are using the Top Level DSL.

But why would one want to use modular style? If you activate the Top Level DSL (by requiring sinatra), Sinatra extends the Object class, somewhat polluting the global namespace.
## Subclassing Sinatra
Creating your own Subclass
```
require "sinatra/base"

class MyApp < Sinatra::Base
  get '/' do
    "Hello from MyApp!"
  end
end
```
Routes outside of the class body
```
require "sinatra/base"

class MyApp < Sinatra::Base; end
MyApp.get '/' do 
  "Hello from MyApp!" 
end
```
### Running Modular Applications
Serving a modular application with run!
```
require "sinatra/base"

class MyApp < Sinatra::Base
  get '/' do
    "Hello from MyApp!"
  end
  run!
end
```

Only start a server if the file has been executed directly
my_app.rb
```
require "sinatra/base"

class MyApp < Sinatra::Base
  get '/' do
    "Hello from MyApp!"
  end

  # $0 is the executed file
  # __FILE__ is the current file
  run! if __FILE__ == $0
end
```
config.ru
```
require "./my_app" 
run MyApp
```
Run with rackup
```
rackup -s thin -p 4567
```
## About Settings
Reading and writing settings
```
require 'sinatra'

set :title, "My Website"

# configure let's you specify env dependent options
configure :development, :test do
  enable :admin_access
end

if settings.admin_access?
  get('/admin') { 'welcome to the admin area' }
end

get '/' do
  "<h1>#{ settings.title }</h1>"
end
```
### Settings and Classes
Another short code survey reveals that settings is actually just an alias for the current application class. Moreover, settings is available both as class and as instance method

```
# Access settings defined with Base.set. 
def self.settings 
  self 
end

# Access settings defined with Base.set. 
def settings 
  self.class.settings 
end
```
## Subclassing Subclasses
### Route Inheritance
Inherited routes
``` 
require 'sinatra/base'

class GeneralApp < Sinatra::Base 
  get '/about' do 
    "this is a general app" 
  end
end

class CustomApp < GeneralApp 
  get '/about' do 
    "this is a custom app" 
  end
end

# This route will also be available in CustomApp 
GeneralApp.get '/' do 
    "<a href='/about'>more infos</a>" 
end

CustomApp.run!
``` 

