---
layout: post
title:      "Displaying Form Entry Errors to the User Through a View with Sinatra!"
date:       2018-06-11 20:49:59 -0400
permalink:  displaying_form_entry_errors_to_the_user_through_a_view_with_sinatra
---
## or authentication and validation errors off the "rubyonrails"

### Or how I built a Custom Furniture Project Manager for Makers Using Sinatra

#### Tags: Ruby, ActiveRecord, Sinatra, validation, authentication

In a previous life, I explored being a custom wood furniture builder here in Brooklyn. I had been a wooden boat builder, loved design and had enjoyed building some furniture for myself and some friends. I was onto something here! 

I gave myself a year to try it out and check back in with myself afterwards to see how things went. Needless to say, I enjoyed the woodworking part more than managing a business. I ended up actually working part time in the shops of friends who also built custom interior projects to learn from them. 

From this experience I gathered that there are two parts to having a custom furniture business, the actual design and building of the projects and the management of the business details of clients and their projects. For me at the time I excelled in the first point but lacked in the latter. 

A quick fast forward to the present, I am now learning web development at the Flatiron School to grow my skillset and I need to build an independent project using the Sinatra framework for Ruby. Oh no, what kind of domain should I build? After much deliberation, I decided to build a simple custom furniture project manager, that I, or other custom interior furniture makers could possibly use. 

CustomFurniturePM(project manager), available on [GitHub](https://github.com/C1RCSAW/CustomFurniturePM), was created utilizing Sinatra, a web application framework that uses Ruby and MVC conventions. Makers that use CustomFurniturePM, can create a list of their clients names and contact information as well as track certain attributes of the projects they have for each client. The application goes further to track itemized costs for each project for their clients and calculate the User's earnings as each cost is added to the project.

For this post I want to focus on authentication and validation errors and how to expose them in a readable format with a view in the browser for the end user.

A user's account and associated content is protected through the use of the bcrypt gem. In order to use the gem, the User model must include 'has_secure_password' and the associated Users migration table must include a 'password_digest' attribute.

*Read more about bcrypt [here](https://en.wikipedia.org/wiki/Bcrypt)*

```
class User < ActiveRecord::Base

  has_secure_password

end
```
```
class CreateUsers < ActiveRecord::Migration[5.1]
  def change
    create_table :users do |t|
      t.string :name
      t.string :username
      t.string :email
      t.string :password_digest
    end
  end
end
```
Great, the code above makes it possible to create a User and add them to the database, but then what prevents someone else who is not that user from accessing their content? the 'has_secure_password' macro adds #authenticate to the methods available to an incidence of the User class. This is where the controller part of MVC comes in.

```
class UsersController < ApplicationController

...
get '/login' do
    if !logged_in? ##<= helper method defined in ApplicationController 
      erb :'login'
    else
      redirect '/clients'
    end
  end

  post '/login' do
    user = User.find_by(:username => params[:username])
    if user && user.authenticate(params[:password])
      session[:user_id] = user.id
      redirect '/clients'
    else
      @login_error = "invalid username or password"
      erb :'login'
    end
  end
	...
	
end
```
when the UsersController accesses the get '/login' action it does one of two things. If the user is not logged in, the controller will render the login form.  Otherwise, if the user is already logged in, they are redirected to their clients page which lists all their clients with links to deeper content on each client. Let's say our user isnt logged in, they are served the form and they either dont enter the correct username and password combo or they try to log into another users account with the incorrect password. At this point the controller accesses the post '/login' do action. 

Its here where we see #authenticate in action, the data entered into the form by the user is stored temporarily in the params hash on the server, the post action looks up the user in the database based on the username entered by the user then the password entered by the user is validated against the password for that user that has been stored in the database using #authenticate. If everything matches, the user_id is saved in the params hash and used in later methods to validate access to further resources specific to that user in the database. Once a successful login has occured the authenticated user is then redirected to their clients list.  

*For deeper reading on params check out [This Rails Guide](http://guides.rubyonrails.org/action_controller_overview.html#parameters)*

Ok cool, so what happens when things dont go so well during authentication? if the user leaves one or more of the fields in the form blank or enters the wrong username/password combo, the controller will render the login form again to the user, however this time there will be a difference. why not redirect the user to the '/login' path you ask? 

As you may know, html is stateless in that data cannot be persisted from one html request to the next. there are however ingenious workarounds to this that are now common in nearly all web applications. one of which we have barely touched on is the sessions hash. 

In this case, instead of using redirect we are rendering erb, therefore a new html request is not being made and any stored variables in the action can be stored and used in the rendered form. You will notice that just before the login form is rendered and just after a login attempt fails, the variable @login_error is set to the string "invalid username or password". the scope of this variable is elevated with "@" so it can be accessed in the rendered login form. you will see that the login form begins with logic that only displays "invalid username or password" if a login attempt fails the authentication conditions.

```
<form action="/login" method="post">
  <h2>Log In</h2>
  
  <%= @login_error if @login_error %>

  <h3> Username: <input type="text" name="username"> </h3>
  <h3> Password: <input type="text" name="password"> </h3>
  <h3> <input type="submit"> </h3>
</form>
```

signaling a login error like this to the user is pretty straight forward, we can get away with hard coding a @login_error variable to a static string like "invalid username or password". But what about displaying a greater variety of error information to a user in the browser from a form with multiple fields where this user entered data will be validated before generating an ActiveRecord instance thats persisted to a database? Great question, glad you asked!

Sticking with our User model for now, lets look a little more at how the User class is defined.

```
class User < ActiveRecord::Base
  has_many :clients ## => @user.clients
  has_many :projects, through: :clients ## => @user.client.projects
  has_secure_password

  validates :name, :username, :email, presence: true
  validates_uniqueness_of :username, :email

  validates :name, format: { with: /\A[a-zA-Z\s]+\z/i,
                             message: 'can only have letters'}

  validates :username, format: { with: /\A[a-zA-Z0-9]+\z/i,
                                 message: 'only allows letters and numbrs' }

  validates :email, format: { with: /\A([^@\s]+)@((?:[-a-z0-9]+\.)+[a-z]{2,})\z/i,
                              message: 'must be in the form of an email address' }
															
...
   ...
end
```

Since this project also uses ActiveRecord to connect our Model part of our MVC to the SQL database, we can use ActiveRecord validation helpers! the validations for the User class above help prevent bad data such as blanks or data not in the format expected to be saved to the database. The presence of a name, username and email is required, username and email must be unique and name, username and email must be in the format specified by their assigned regular expression. note the use of the custom message attribute. ActiveRecord validations have default error messages that are saved to an errors hash. The 'message:' attribute for 'validates' allows you to override the default message and write your own special error messages. 

So now that our model has validations how do we then display them to our user when they make a goof or enter something our validations don't like? Well, we are going to need our Controller and our View for that, the other two parts of our MVC. 

To begin to answer this question. lets look at when validations actually occur in our application. In CustomFurniturePM this happens in when our new user form triggers a POST request that gets handled by our UsersController before it gets sent to the server.

```
class UsersController < ApplicationController
...
  get '/signup' do
    if logged_in?
      redirect '/clients'
    else
      erb :'users/create_user'
    end
  end

  post '/signup' do
    @user = User.new(params)
    if @user.save
      redirect '/login'
    else
      erb :'users/create_user'
    end
  end
	...
end
 ```
	
In ActiveRecord validations are triggered whenever a new obect gets pushed to be saved in the database. Most commonly this is accomplised with #create, #save or #update. each of these have different behaviors with our objects but they all trigger ActiveRecord validations if they are present on the Class they are called on. 

*For more details on ActiveRecord validations and helper methods check out this [guide](http://guides.rubyonrails.org/active_record_validations.html)*

In our post '/signup' route and action, we are first creating a new instance of a user with the user entered data from the form that was stored in the params hash. note we are using #new instead of #create. If #create was used, the data would both be used to create our new instance AND push a new row to our users table in the database. This would also trigger our validations and either fail or be a success. If create is a success then great, the user did what we were expecting and the application can move on to the next thing. however, we want to hold on to the errors if there were any, so we need to split this process into two parts to both hold onto the errors if there will be any AND allow for an if statement to provide flow control to our application. We are talking about code in a controller after all. Using #new here allows this to be split up we can hold onto an instance of our User class and trigger validations with #save on that instance later in the if statement.

Just like our previous example with user login validation, we dont want to send a redirect to the server, we would lose our error information. therefore if @user.save fails validation and returns false, we want to render erb: 'users/create_user' so we can access the validation errors and display them to the user in the browser with a View. 


```
<h1>Sign Up for an Account</h1>

<% if @user && @user.errors.any? %>
  <%= @user.errors.full_messages.join("<br/>") %>
<% end %>

<form action="/signup" method="post">
  <p> Name: <input type="text" name="name" value=""> </p>
  <p> Email: <input type="text" name="email" value=""> </p>
  <p> Username: <input type="text" name="username" value=""> </p>
  <p> Password: <input type="text" name="password" value=""> </p>
  <p> <input type="submit" name="" value="Sign Up"> </p>
</form>

```


Hence we have closed the MVC loop on displaying validation errors to our user through thier browser. The errors will only display in the view of this form if theres a @user variable that has been set AND that @user instance failed to be saved thus containing errors. If this is the case the form will then iterate through the errors one by one and display them with full_messages with a line break between each one so they can be more easily read instead of wrapped together in a run on string. An example of the output of this re rendered form with errors woud look like this:

![](https://imgur.com/SarSJdR)

This is of course a simple example. The embedded ruby in the form used to display the validation errors if they are present could be sandwiched in your own custom div class and styled with CSS. the possibilites are, well, endless. Hope you find this post helpful in generating your own lightweight, interactive app using Sinatra! happy coding!


	



