## Layout your ditty with Sinatra

### Introduction

I recently had the chance to do a web application for a group of users where they needed some forms to be deployed online with some common features such as signin, registration, self service etc to be made available. They also needed one of the forms to be submit and mailed. I had reasonable experience with Ruby (which I love for its grace, elegance and flexibility) and had done some work in Rails. I wanted to give Sinatra a try which I had heard a lot about for its simple DSL for developing web applications quickly. It was an absolute pleasure.

The below is a Sinatra application in its simplest form
```ruby
require 'sinatra'

get '/' do
	"Welcome to Sinatra !"
end

get '/:name' do
	"Hello #{params[:name]}"
end

``` 
If we run this application and point our browser to ```http://localhost:4567``` (default Sinatra applications runs on port 4567) it shows
```html
Welcome to Sinatra !
```
If we say ```http://localhost:4567/James```
it shows
```html
Hello James
```


Sinatra does not force you to develop you applications according to any set pattern, you are free to choose from a varity of styles (including MVC) depending on the complexity and needs of your application. Blake Mizerany and Konstantin Haase have done a commendable work in their efforts.

I went on to integrate Activerecord for ORM Cucumber and other tools for BDD and Bootstrap for UI development. Most of the time adapting these with Sinatra was effortless and with excellent community support for these technologies not many questions where left unanswered.

### Layout of the Sinatra application.

```
├───config
│   ├───data
│   └───form_validations
├───helpers
├───logs
├───migrations
│   └───sql
├───model
├───public
│   └───style
│       └───bootstrap
│           ├───css
│           ├───img
│           └───js
├───test
│   ├───config
│   │   └───data
│   ├───data
│   ├───evidence
│   ├───features
│   │   ├───factory
│   │   ├───step_definitions
│   │   └───support
│   │       └───pages
│   └───helpers
├───vendor
│   └───cache
└───views
app.rb
Rakefile
README.md
gemfile
gemfile.lock
config.ru
.gitignore
```

I will now give you a brief summary of what the individual directories hold. The application root held the files ```app.rb``` along with ```Rakefile, README, gemfile, config.ru``` and the usual ```.gitignore```. I will discuss these later.

### Directories

#### config
This directory holds the any application configuration files and data that is needed. For example the database connection parameters in ```database.yml```
```yaml
development:
  adapter: mysql2
  encoding: utf8
  database: vt14
  username: db_user
  password: db_user_password
  host: localhost
  port: 3306
```
I also choose to store my reference data for form validations and some data for my HTML select lists.

#### helpers
This directory is used to store helper methods that need to be used in the application. Helper methods have been defined inside a module. The modules are registered within a sinatra applications using the sintra DSL syntax ```helpers Module_name```.

#### migrations
This directory is used to store and define the Activerecord migrations to build the database Schema from scratch using Activerecord.

#### model
This directory holds the active-record classed for the database tables, these tables are the subclasses of ```ActiveRecord::Base```, they also are the place to store any model specific validations. For example for a unique user email address we say 
```ruby
validates :user_email_address, uniqueness: { case_sensitive: false , message: "Email address has already been taken."}
```

#### public 
This directory is the place where all the public files such as any static HTML, CSS or java scripts are held. I have stored the Bootstrap CSS files here.

#### test
This directory holds all the BDD tests (cucumber .feature files, step definitions etc). I will expalnd on the details of the test directory later.

#### vendor
This directory hold the gem files needed by the application , ready for deployment.

#### views
This folder holds the views, the views can be static .erb files or .haml files among many other formats. I choose .haml.

### Sinatra Application
There are two ways to write a Sinatra application , they both work equally well and I believe that in terms of performance they are the same. They are known as the Classic application or the Modular application. Classic applications have a ```require sinatra``` statement at top which triggers the creation of a classic application, they use the top level DSL directly and the application itself is a ```Sinatra::Application``` object. Modular applications on the other hand are classes that are a subclass of ```Sinatra::Base``` class. For modular applications we require ```sinatra/base``` in the ruby class file.

Classic applications Vs Modular application

```ruby
require 'sinatra'
get '/' do
	"Hello world"
```

```ruby
require 'sinatra/base'

class MySite < Sinatra::Base

	get '/' do
		"Hello World"
	end
end
```

For larger applications it may be a good idea to go for a modular approach so that each piece of related 'controller' behaviour can be encapsulated it its own class.

#### configuration
Sinatra offers DSL to configure the applications in the form of a ```configure``` block. See below example.
```ruby
[...]
	configure :development do
		enable :logging
		set :session_secret , 'av3rys3cr3tk3y'
		set :bind, '0.0.0.0'
		set :haml, :format => :html5 
		set :environment, :development

		set :dump_errors, false
		set :raise_errors, false
		set :show_exceptions, false
	end
[...]	
```
Here we have enabled logging, set a session key, set the template engine to haml. Further I have set :dump_errors, :raise_errors and :show_exceptions to false in order for us to handle the sinatra errors manually using ```error env['sinatra.error']``` and then log them if needed.

#### Extensions and Helpers
There are two ways to extent a Sinatra application. They are called extensions and helpers. In order to keep our code DRY. We write any code that needs to be used repeatedly in modules and then make them available to Sinatra using ```helpers``` block. 
```ruby
module HelperModule
	def camel_case (a_string)
		split('_').map{|e| e.capitalize}.join
	end
end
``` 
the in the Sinatra application we say.
```ruby
class MyApp < Sinatra::Base
	helpers HelperModule
end
```

To register extensions with Sinatra we use the ```register``` method.

```ruby
module MyExtension

	def hello
		"Hello world"
	end

end

class MyApp < Sinatra::Base
	register MyExtension
end
```

#### Views in Sinatra
Sintra offers use of various template engines to render views, they can include haml, erb and Nokogiri among others. The view files are by default placed at /views folder. To render a haml view we call ```haml :login```, this will render the login.haml view. To pass local variables into view use ```haml :my_form, locals = {:name => "James", :age => "33"}```

#### REST Routes in Sinatra

The Sinatra DSL allows the creation of REST style routes with ease in a Sinatra application.

```ruby
[...]
	get '/country/:info' do
		info_type = params[:info]
		case info_type
		when "city": haml :city
		when "capital": haml :capital
		else haml :info_not_available
		end
	end
[...]

	get '/login' do
		haml :login
	end	
[...]
```

#### Running Sinatra applications

Runnng a classic sinatra application is just like running any ruby program, just execute ```ruby app.rb``` for modular applications we use ```ApplicationName.run!```. To run on a particular port using a specific webserver use
```ruby
ApplicationName.run! :host => 'localhost', :port => 4567, :server => 'thin'
```

#### References
1. [Sinatra] (http://www.sinatrarb.com/)
2. [Sinatra Rdoc] (http://rubydoc.info/gems/sinatra/frames)
