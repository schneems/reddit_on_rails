## Reddit on Rails: A Hero's Journey

In past exercises for [UT on Rails](http://schneems.com/ut-rails) I've given you a starter project to work on, and I've given very step by step instructions with the intent of teaching specific concepts and skills. By now you have the knowledge to accomplish much with Rails. We will use that knowledge to build a website from scratch similar in functionality to http://reddit.com.

Instead of focusing on specific concepts and introducing ideas, in this exercise we will be looking at Rails from a high level. We will be doing things that you've already done, it is your job as a student to try to anticipate what we should be doing next before we do it, and then to execute the implementation with full understanding. If you have any doubts over why we are doing something, you should ask someone immediately, or write them down to investigate later. While you can get through the exercise by blindly following instructions, you will be more lost when attempting to implement things by yourself. If you want to get the full bennefit of this exercise, try to not copy any code. Type in code as it appears, and fix any mistakes that you make. This exercise is not about the final product but about the journey.

As we go along you'll get a glimpse into my thought process for making websites. While yours might differ in time I find seeing someone else's work flow always strengthens mine.

Sometimes I will be missing steps between instructions, such as "stop your server". Other times I will give high level instruction. This is not laziness on my part, but rather a chance for you to exercise what you've learned as a programmer. It is your job to verify the output of commands you type into the console. This should not be a passive exercise.

As a side note, I usually have 3 or more console tabs open to the current directory I'm working in so I can run my server and console and any rake commands I want at the same time.

## Install

You will need ruby 1.9 on your system.

    $ ruby -v

You will also need rails and bundler. You can find install directions and materials in previous exercises http://schneems.com/ut-rails. This is not a beginner exercise, but rather an intermediate trial by fire, new developers are not expected to be able to finish. Good luck.

## Let's get Started

First we will take a look at what we want to build, make a game plan on how to make it, and then iterate very slowly until our product becomes what we desire.

If you're not familiar with http://reddit.com, go there right now. Take a look around, click some links. Reddit is a very popular social-ish network for sharing links. The basic idea is simple, a user submits a link with a title that they find interesting. If others like the link they vote it up. If they dislike the link, they vote it down. Top links should show up on the front page.

Before I begin writing any code for a project I will craft user stories around the interactions. If we take those user stories and break out the nouns (people, places, things) this will give us a good idea of what our models need to be. An example of a user story for this Reddit clone might be something like:

    "A user visits the home page of the website, they then log in. They click a "submit link"
    button and enter a url and a title. They submit that link and title and then end up on a
    page that shows the title, link, votes for that link and any comments."


As I develop a Rails site I will move in as small of steps as I possibly can and validate they are working each step along the way as I would expect. On a high level I will need MVCr (model, view, controller, and routes) to form a full resource that can be used on our website. Since we have to start somewhere I typically start with the models. I'll go back to my user story and pull out all of my nouns:


    "user", "home page", "web site", "submit link button", "link", "url", "title", "page", "votes",
    "comments"

There are some duplicates and sum such as "web site" that aren't very helpful. When looking at these nouns I've got some top level nouns such as:

    "user", "link", "votes", "comments"

And then I've got sum nested nouns, for example we mentioned "url" and "title", but these should likely be attributes of a link. You might be questioning why "votes" isn't an attribute of "link", while you could store a vote_count with each link, we need some way to associate a user with a vote, so each user can only vote once. I know this because i've used reddit a fair bit, but we could have come to this conclusion by writing a user story around voting on links.

Now that we've got our top level nouns, we can turn them into proper models. Since we're going to go in small steps, we'll only work on one model at a time, fleshing out MVCr as we go along.

We'll start out with the "user" model:

## User Model

While you could call this "person" or "human" or something similar, "user" has become a pretty standard term in web applications. To have a user in our application we want some standard attributes. To help get these we could imagine what you would expect when you view your own profile. Likely you would want some way of logging in, so the system would have a username or email, or both, and it would have a password to keep strangers from pretending to be you.

Even if we have a username, it makes sense to keep a user's email since they might forget their password and need a reset link sent to them. If you want to add more attributes or functionality later, you can. So don't worry about getting everything. So our user model now looks something like this:

      User Table
        username:   string
        email:      string
        password:   string
        created_at: datetime
        updated_at: datetime

There is one small problem with this model. Storing passwords in plain text is a horrible practice, insecure at best and negligent at worse. We could store our own password "fingerprint" using a hashing library like [bcrypt](http://en.wikipedia.org/wiki/Bcrypt) which is similar to MD5 but more secure. Then we could compare our stored "fingerprint" to a user submitted password that has been run through our hashing (fingerprinting) library. This is a commonly accepted practice, but we are lazy programmers. People smarter than us have already written this functionality in other libraries. So rather than store our password in plain text we will use a ruby library to give us this functionality.

Currently [Devise](https://github.com/plataformatec/devise/) is the industry standard for user authentication, but it wasn't always that way. If you didn't know about it, you could find good library by googling for `rails user authentication library`. Another good resource is the [Ruby Toolbox](https://www.ruby-toolbox.com/) which will help compare libraries with similar functions. You can see their section on [Rails Authentication](https://www.ruby-toolbox.com/categories/rails_authentication) and you will see that at the time of this writing Devise is on top.


## Rails New

So we have a good idea on how we want to start but haven't written any code at all, this is fine but let's at least make a new rails project. In your projects folder run the command to make a new rails project call it `reddit_on_rails`. If you don't know how to do that in your project directory try running:

    $ rails --help

Once you've generated a new rails app go into that directory:

    $ cd reddit_on_rails/

You can verify that you are in a Rails app by checking the file system:

    $ ls
    Gemfile   README.rdoc app   config.ru doc   log   script    tmp
    Gemfile.lock  Rakefile  config    db    lib   public    test    vendor

If you run `$ rails --help` now you will see a different output:

    $ rails --help
    Usage: rails COMMAND [ARGS]

    The most common rails commands are:
     generate    Generate new code (short-cut alias: "g")
     console     Start the Rails console (short-cut alias: "c")
     server      Start the Rails server (short-cut alias: "s")
     dbconsole   Start a console for the database specified in config/database.yml
                 (short-cut alias: "db")
     new         Create a new Rails application. "rails new my_app" creates a
                 new application called MyApp in "./my_app"

    In addition to those, there are:
     application  Generate the Rails application code
     destroy      Undo code generated with "generate" (short-cut alias: "d")
     benchmarker  See how fast a piece of code runs
     profiler     Get profile information from a piece of code
     plugin       Install a plugin
     runner       Run a piece of code in the application environment (short-cut alias: "r")

    All commands can be run with -h (or --help) for more information.

If you get stuck using a rails command you can always use this help, you can also drill down to get more help. If you didn't know what generate did you could run

    $ rails generate --help

And deeper still

    $ rails generate model --help

You're not expected to know all the syntax of all the Rails commands by heart, but you do need to know where to look for help.


Now that we're in our project you might want to initialize a git repository and commit. This makes it easier to see what changes you do to your app.

We already laid out our user model above so lets make it. We will need a model file in `app/models` and a migration in `/db/migrate`. We can manually make the model file but making migrations manually is frowned upon. Lets make both with a generator. Generate a model using:

    $ rails generate model user

Here we could have specified the fields for our user like $ rails generate model user email:string ...`, but we can add them in manually. I prefer a hands on approach that gives me control and visibility into what is going on.

We should now have a `app/model/user.rb` file and a `db/migrate/...._create_users.rb` file. The number in front is a time stamp and is used by rails to know what migrations we've already run.

Open up the migration file, you'll see there is already some code in there but we need to add our `email` and `username` fields (created_at and updated_at) were added via the `t.timestamps` by rails. Rails will automatically add a primary key of `id` as well.

If you don't know how to add fields to this file you can search for `rails migrations docs` on google and you will find the [API Docs](http://api.rubyonrails.org/classes/ActiveRecord/Migration.html) as well as the [Guide Docs](http://guides.rubyonrails.org/migrations.html) both are good resources. You might also choose to look at an old project of yours, this isn't cheating...it's using your resources effectively.


Looking in the API docs we see we can add a string column by adding this line inside of our `create_table` block:

    t.string  :email
    t.string  :username

Add those lines to your migration document now. We'll then verify they were added correctly by running:

    $ rake db:migrate

This will create our users table and columns. To verify this we can start a new rails console and create a user:

    $ rails console
    > u = User.new
    > u.email = "foo@bar.com"
    > u.username = "schneems"
    > u.save

If you get an error like this:

    > User.create(:email => "foo@bar.com", :username => "schneems")
      ActiveModel::MassAssignmentSecurity::Error: Can't mass-assign protected attributes: email, username

That's okay, copy the error into google and try to figure out why we got it, in this case we might find this [document on mass assignment security](http://api.rubyonrails.org/classes/ActiveModel/MassAssignmentSecurity/ClassMethods.html) which will tell you that we have to whitelist attributes on our models. Open up your `app/models/user.rb` and add an `attr_accessor` to include :email, and :username.


    attr_accessible :email, :username

You can restart your console or you can reload all of the classes by calling:

    > reload!

Either way, you should now be able to use mass assignment to create a user:

    > User.create(:email => "foo@bar.com", :username => "schneems")
     (0.1ms)  begin transaction
      SQL (0.7ms)  INSERT INTO "users" ("created_at", "email", "updated_at", "username") VALUES (?, ?, ?, ?)  [["created_at", Sun, 29 Jul 2012 19:08:02 UTC +00:00], ["email", "foo@bar.com"], ["updated_at", Sun, 29 Jul 2012 19:08:02 UTC +00:00], ["username", "schneems"]]
       (2.0ms)  commit transaction
     => #<User id: 2, email: "foo@bar.com", username: "schneems", created_at: "2012-07-29
          19:08:02", updated_at: "2012-07-29 19:08:02">


By now you should be able to make new users as you please, if you cannot fix any errors you get. Save your code and commit to git. Congrats - you just made a User model.


## User Log In

We mentioned before that we'll use Devise to handle the sign in and registration process of our app. We already created the model for user so we still need views, controllers, and routes to finish the login and signup process. Devise has all of these baked in. Open up the [Devise documentation](https://github.com/plataformatec/devise/) and we will follow along. Go to the "Getting Started" section. First we must add this line to our `Gemfile`:

    gem 'devise'

Now run:

    bundle install

Devise comes with built in generators that it adds to Rails. If you run help on rails generators now you'll see the new generators:

    $ rails generate --help
      ...
      Devise:
        devise
        devise:install
        devise:views

Following along with the devise documentation we will first run:

    $ rails generate devise:install

It will create some files, most importantly the `config/initializers/devise.rb` and it will give us some warnings:


    1. Ensure you have defined default url options in your environments files. Here
       is an example of default_url_options appropriate for a development environment
       in config/environments/development.rb:

         config.action_mailer.default_url_options = { :host => 'localhost:3000' }

       In production, :host should be set to the actual host of your application.

    2. Ensure you have defined root_url to *something* in your config/routes.rb.
       For example:

         root :to => "home#index"

    3. Ensure you have flash messages in app/views/layouts/application.html.erb.
       For example:

         <p class="notice"><%= notice %></p>
         <p class="alert"><%= alert %></p>

    4. If you are deploying Rails 3.1 on Heroku, you may want to set:

         config.assets.initialize_on_precompile = false

       On config/application.rb forcing your application to not access the DB
       or load models when precompiling your assets.

It is important to read output of any new commands you are running, they often have important information. Let's follow these instructions quickly.

Open `config/environments/development.rb` and add this line to it:

    config.action_mailer.default_url_options = { :host => 'localhost:3000' }

We don't have a production url yet, but when we do we will need to set that in production.rb.

Now it says we have to have `root_url` defined. But we've got a problem, we don't have any controllers yet. We can make a controller to store our homepage, the example they gave uses the `home_controller` but it could be anything. I like to have a controller called `pages_controller` in my projects, we can then use `pages#index` for our homepage, and then if we want to add additional views like about, privacy_policy, etc. we can use the `show` action to render those pages. While it makes sense to have `home#index` be the homepage, i think it makes less sense to have `home#show` render an about page. Create a pages controller now. You can either do this manually by adding a file `app/controllers/pages_controller.rb` or you can run a generator:

    $ rails generate controller pages

For rails all, models are singular and controllers are plural. FYI - if you mess this up you can always reverse a generation by running `destroy` and trying again `$ rails destroy controller pages`, but don't run that - it will erase our controller.


Now we've got a controller in `app/controllers/pages_controller.rb`, if you're making your own, it should look like this:

    class PagesController < ApplicationController
    end


 Super simple. Add an index action


    class PagesController < ApplicationController
      def index
      end
    end

And then add the routes. In `config/routes.rb` add:

    resources :pages

And:

    root :to => "pages#index"

You can verify your routes by running:

    $ rake routes

Now from MVCr on this we don't need a model, we've got our controller and our routes, so we need to add our views. Make a file `app/views/pages/index.html.erb` and put some text in it to let you know it rendered:

    <h2>I AMA Homepage</h2>

To verify this all worked we can spin up a rails server

    $ rails server

And visit [http://localhost:3000](http://localhost:3000) and you should see:

    I AMA Homepage

If you see the "Welcome Aboard" file, you can remove `index.html` from your public folder. Try refreshing, and you should now see your view instead. If you don't, go back over the steps and see what you did wrong. Use the logs and google if you run into errors.

So by now we've got 2 of those suggested items in the output done. For the next open `app/views/layouts/application.html.erb` and add these two lines above the `<%= yield %>`:


    <p class="notice"><%= notice %></p>
    <p class="alert"><%= alert %></p>


Finally add this line inside of your `config/application.rb` file.


    config.assets.initialize_on_precompile = false


Now would be a good time to commit to git. We're not quite done with devise just yet. We need to tell devise which model we want to log in with to do this we will run the `rails generate devise` command we can get more information about it by running:

    $ rails generate devise --help

Here we see that we can run `rails generate devise NAME [options]` where NAME is the name of our MODEL that we want to use devise on. In our case this is the user model so go ahead and run:

    $ rails generate devise user

You should see some output from this like:

    invoke  active_record
      create    db/migrate/20120729200407_add_devise_to_users.rb
      insert    app/models/user.rb
       route  devise_for :users

We can verify that we have a new migration, that `app/models/user.rb` has more code in it and that our `config/routes.rb` has new code in it. You can look at these files to see what code devise added. You should run


    $ rake db:migrate

to add the new fields to your user model. You might get an error when you run this. Why?

You should get this error:

    SQLite3::SQLException: duplicate column name: email: ALTER TABLE "users" ADD "email" varchar(255) DEFAULT '' NOT NULL

Since we already have an email column on our user model, go into the last migration and comment out this line:

    t.string :email,              :null => false, :default => ""

Run the migration again. If you have users in your database with duplicate emails, you can destroy them by running

    $ rails console
    > User.destroy_all

Once you've got the migrations running correctly we should be able to start our server (restart if it's already on). You can see the routes that devise added by running

    $ rake routes
              new_user_session GET    /users/sign_in(.:format)       devise/sessions#new
                  user_session POST   /users/sign_in(.:format)       devise/sessions#create
          destroy_user_session DELETE /users/sign_out(.:format)      devise/sessions#destroy
                 user_password POST   /users/password(.:format)      devise/passwords#create
             new_user_password GET    /users/password/new(.:format)  devise/passwords#new
            edit_user_password GET    /users/password/edit(.:format) devise/passwords#edit
                               PUT    /users/password(.:format)      devise/passwords#update
      cancel_user_registration GET    /users/cancel(.:format)        devise/registrations#cancel
             user_registration POST   /users(.:format)               devise/registrations#create
         new_user_registration GET    /users/sign_up(.:format)       devise/registrations#new
        edit_user_registration GET    /users/edit(.:format)          devise/registrations#edit
                               PUT    /users(.:format)               devise/registrations#update
                               DELETE /users(.:format)               devise/registrations#destroy


Here we see that we can make a new user by visiting the [/users/sign_up](http://localhost:3000) path. To see where this view came from read the next optional section:

### FYI Rails Engines (this section is optional)

You should see view containing a signup form. Devise is a Rails engine which means that it can behave much like a mini rails app within your app. These views and controllers are actually contained inside of the devise gem.

To see where this code comes from you can run:

    $ bundle open devise

You may have to specify your editor

    $ EDITOR=subl bundle open devise

If you have sublime installed on your system, you can get the `subl` command installed by following these directions http://www.sublimetext.com/docs/2/osx_command_line.html and googling if you run into problems. If you can't get that working you can run:

    $ gem env

To see where your INSTALLATION DIRECTORY directory is, mine is at `/Users/schneems/.rvm/gems/ruby-1.9.3-p194`. You can go to this directory (note it will be different for you).

    $ cd /Users/schneems/.rvm/gems/ruby-1.9.3-p194

Then into the gems directory

    $ cd gems

You can verify that devise is there by running:

    $ ls

Once you know the version of devise you are using you can cd into that directory:

    $ cd devise-2.1.2

Now you can run:

    $ pwd

To get the current working directory for me it is `/Users/schneems/.rvm/gems/ruby-1.9.3-p194/gems` you can then use a text editor to open that directory. You may need to google to find out how to show hidden files and directories (like .rvm) to find the right files. You can also run

    $ open .

in the console to open the current directory in a finder.


As you can see getting the `bundle open` method to work is well worth the time, and will help quite a bit later.

Now that you have the devise code open you can see where that view is stored in `devise/app/views/devise/registrations/new.html.erb`. The structure of an engine follows that of a Rails app. You can see the registration controller in `controllers/devise/registrations_controller.rb`. Pretty cool huh?

You might be tempted to make modifications to these files directly, don't. They were installed as a rubygem and aren't a part of your project (those changes don't show up in your source control) therefore when you push to production you won't see those changes. Because of this, devise gives us a built in set of generators for modifying its views and controllers. We won't be doing that right now, but you can read more about it here: [Configuring Views](https://github.com/plataformatec/devise/#configuring-views) and [Configuring Controllers](https://github.com/plataformatec/devise/#configuring-controllers).

This section is purely for your own info, it wasn't required...but hopefully gave you a better idea of how those views showed up on your screen.

## Back to the Required work

If you skipped the Rails engine part, go back and read it some time. We'll continue building your app now. Try signing up as a user using your web interface. Once you do you should see your custom home page!

You can check to make sure your user was created by looking in the console:

    $ rails console
    > User.last

Sweet, by now we've got our User model, devise controllers, views and routes for registering and signing in and signing out! That's all the MVCr we need for our user for now. Let's go back to our user story and see how much of it we can do now.

    "A user visits the home page of the website, they then log in. They submit that link and title and then end up on a
    page that shows the title, link, votes for that link and any comments."

Let's try this, first visit your home page [http://localhost:3000](http://localhost:3000). Now follow the next step, "log in". We haven't added any type of log in or log out functionality to our site. Maybe we should do that right now.

Looking at the devise docs you'll see that we should have a `current_user` available if the user is logged in. We can use some logic to add links in our layout. Get the devise routes again by running:

    $ rake routes
            new_user_session GET    /users/sign_in(.:format)       devise/sessions#new
                user_session POST   /users/sign_in(.:format)       devise/sessions#create
        destroy_user_session DELETE /users/sign_out(.:format)      devise/sessions#destroy
               user_password POST   /users/password(.:format)      devise/passwords#create
           new_user_password GET    /users/password/new(.:format)  devise/passwords#new
          edit_user_password GET    /users/password/edit(.:format) devise/passwords#edit
                             PUT    /users/password(.:format)      devise/passwords#update
    cancel_user_registration GET    /users/cancel(.:format)        devise/registrations#cancel
           user_registration POST   /users(.:format)               devise/registrations#create
       new_user_registration GET    /users/sign_up(.:format)       devise/registrations#new
      edit_user_registration GET    /users/edit(.:format)          devise/registrations#edit
                             PUT    /users(.:format)               devise/registrations#update
                             DELETE /users(.:format)               devise/registrations#destroy
                       pages GET    /pages(.:format)               pages#index
                             POST   /pages(.:format)               pages#create
                    new_page GET    /pages/new(.:format)           pages#new
                   edit_page GET    /pages/:id/edit(.:format)      pages#edit
                        page GET    /pages/:id(.:format)           pages#show
                             PUT    /pages/:id(.:format)           pages#update
                             DELETE /pages/:id(.:format)           pages#destroy
                        root        /                              pages#index


So the sign in path is accessed by the `new_user_session` helper and the register is `new_user_registration` and sign out is `destroy_user_session`. Open `app/views/layouts/application.html.erb` and add this logic to your view:


      <%# user is logged in, show log out link %>
      <% if current_user.present? %>
        <%= link_to 'Sign Out', destroy_user_session_path, :method => :delete %>
      <%# user is not logged in, show signup and login links %>
      <% else %>
        <%= link_to 'Sign In', new_user_session_path %> |
        <%= link_to 'Register Now!', new_user_registration_path %>
      <% end %>

Refresh your page, you should see some of your links. If you are signed in, try signing out. Once signed out, try signing in. If you forgot your password make a new user account.

Once you've verified this works, commit to git.

Let's go back to our user story now, we can do our first sentence:

    "A user visits the home page of the website, they then log in."

Great! Let's look at the next one:

    "They click a "submit link" button and enter a url and a title"

Okay, we can't do any of that right now. Before we can, let's go back to our nouns (remember those?). We had:

        "user", "link", "votes", "comments"

We already did MVCr for user noun (model). To do the next part we will need a "link" model of some kind. Let's make that model now.


## Link did all the work, why does Zelda get the legend?

Reading though our User story we know that a link should have a url and a title.


    Link:
      url:   string
      title: string

We also get the feeling that there is a relationship between user and link. If you were going to describe your website to someone you could say correctly that a user can have many links but each link can only have one user. Because of this we need to store the relationship somehow between links and users. If you don't remember when we talked about associations you can always google or check docs. One of my favorites is the [rails association guide](http://guides.rubyonrails.org/association_basics.html). After a quick refresher, you'll see that we need a foreign key in the model that belongs_to the other model. So our link model now looks like this:

    Link:
      user_foreign_key: integer
      url:              string
      title:            string

By default rails expects foreign keys to end with `_id` and begin with the model name it is referencing so for a foreign key that points at the user model we can name it `user_id`

    Link:
      user_id: integer
      url:     string
      title:   string


This sounds like a pretty solid link model to me, let's generate our migration and model. Remember models are singular, and you can always run `destroy` if you misspell something.

    $ rails generate model link

Let's open our migration and add in our fields. To simulate forgetting a field, let's leave out the title field:



    t.integer :user_id
    t.string  :url

Trust me, it's impossible to get your model 100% right the first time. You can always add and remove columns later as we'll see in just a second.

Run:

    $ rake db:migrate

Now we have an incomplete link model, we can verify in the console if you wish. How do we add our missing column?

We'll need a new migration so run:

    $ rails generate migration add_title_to_link

We'll get a new migration that looks like this:

    class AddTitleToLink < ActiveRecord::Migration
      def change
      end
    end

Use the [Rails Active Record Docs](http://api.rubyonrails.org/classes/ActiveRecord/Migration.html) and the `add_column` method to add a string column to the links table.

When you're done you should be able to run `$ rake db:migrate` and then see your column in the console.

    $ rails console
    > link = Link.new
    > link.title = "Schneems sets world record for teaching Rails"
    > link.save

If you mess up and add things you didn't intend in your migration you can always see what rake tasks are available by running `$ rake -T` From there we would usually use

    $ rake db:rollback

To go back one step or:

    $ rake db:rollback  STEP=2

to go back two steps.

Once you've got a working migration it is much easier to add a new migration then to edit an old one. If you've committed a migration to source control you should not modify it unless there is a really good reason. New migrations are easy to make just run `$ rails generate migration ...`

Once you've got the title attribute in your links table, add `attr_accessors` until this works in your rails console:

    $ rails console
    > reload!
    > Link.create(:url => "http://schneems.com", :title => "Awesome")


We also want to verify that our association works. Add

    has_many :links

and

    belongs_to :user

In the appropriate files. You should then be able to run

    $ rails console
    > User.last.links
    > Link.last.user


Once that works, save everything and commit to git. Let's review where we are. We've got our model but no controller, view or routes. We'll need a form for submitting new links, which means a new and create action, we also want to show that link after submission, so a show action. Let's add our routes and controller now. In `config/routes.rb` add:

    resources :links

Then make a new controller `app/controllers/links_controller.rb`. We can start off by copying the contents of our pages controller:

    class PagesController < ApplicationController
      def index

      end
    end

But then we need to change the class name and add some actions


    class LinksController < ApplicationController
      def show
      end

      def new
      end

      def create
      end
    end

We will leave these blank for now, and also create 3 view files under `app/views/links` called `show.html.erb`, `new.html.erb` and `create.html.erb`. To verify they work, I like adding unique text to each and then visiting the routes manually. You can always run `$ rake routes` to find the available routes. You should see this in your output:


         links GET    /links(.:format)               links#index
               POST   /links(.:format)               links#create
      new_link GET    /links/new(.:format)           links#new
     edit_link GET    /links/:id/edit(.:format)      links#edit
          link GET    /links/:id(.:format)           links#show
               PUT    /links/:id(.:format)           links#update
               DELETE /links/:id(.:format)           links#destroy

Note: While I'm coding by myself, I usually don't make controller actions and views ahead of time like this. Typically I'll just code until I get an error that tells me I'm missing a template which I know means I need a view right there. Most of the error messages in Rails are decent at telling you what is missing. Rather than trying to avoid errors at all costs, we can let Rails remind us what we're missing.

That means we can manually visit [/links/1](http://localhost:3000/links/1) and [/links/new](http://localhost:3000/links/new). Before we pre-emptively add code to our controller, let's go back to our user story:

    "They click a "submit link" button and enter a url and a title"

So let's add a button (or link, sometimes our user stories might need updating) on the main page to a page that we can put a form on to make a new link. If you look at your existing routes and controller actions which might be good to hold a form?

Hopefully you picked the `links#new` action. We can open up our `pages/index.html.erb` and add a link to the new view:

    <%= link_to "Add a link", new_link_path %>

With your server on, refresh your home page. Click the link, and we've got half of our user story sentence complete.

Now we'll need a form and some things in our `new` controller action to finish the sentence.

I'll be honest with you - Rails forms were the hardest thing for me to get a handle on. My best recommendation to you is to become well acquainted with the docs, and keep in mind that at the end of the day, rails is building HTML forms. If you can't figure out how to do something, if you can find an example of it on another website you can reverse engineer the logic. I also like cheating off of old code of mine or out of rails generated scaffolding.

All of coding is like an open book test, as long as it works it doesn't matter what resources you used to get there. Google, docs, old code, stack overflow, someone you met at a hackathon, or even your cat randomly hitting keys on your keyboard. So let's get our cheat on. I'm going to open up one of our old projects and copy the form out of `app/views/user/_form.html.erb`.This was generated from Rails scaffolding, so it can't be too wrong...


 If you don't have any other projects, you could always start a new junk project and run a scaffold generator `$ rails generate scaffold foo name:string`. You can also build your own looking at the [Action View Form Helper Docs](http://api.rubyonrails.org/classes/ActionView/Helpers/FormHelper.html).

 If you chose to make your own, you can skip this next bit until the next section. If you copied out of an old project you should get something like this:

     <%= form_for(@user) do |f| %>
      <% if @user.errors.any? %>
        <div id="error_explanation">
          <h2><%= pluralize(@user.errors.count, "error") %> prohibited this user from being saved:</h2>

          <ul>
          <% @user.errors.full_messages.each do |msg| %>
            <li><%= msg %></li>
          <% end %>
          </ul>
        </div>
      <% end %>

      <div class="field">
        <%= f.label :name %><br />
        <%= f.text_field :name %>
      </div>
      <div class="field">
        <%= f.label :job_title %><br />
        <%= f.text_field :job_title %>
      </div>
      <div class="actions">
        <%= f.submit %>
      </div>
    <% end %>

Hopefully you shouldn't be too lost, but typing up all that code from scratch can be difficult and prone to error. Change "user" to "link"

     <%= form_for(@link) do |f| %>
      <% if @link.errors.any? %>
        <div id="error_explanation">
          <h2><%= pluralize(@link.errors.count, "error") %> prohibited this link from being saved:</h2>

          <ul>
          <% @link.errors.full_messages.each do |msg| %>
            <li><%= msg %></li>
          <% end %>
          </ul>
        </div>
      <% end %>

      <div class="field">
        <%= f.label :name %><br />
        <%= f.text_field :name %>
      </div>
      <div class="field">
        <%= f.label :job_title %><br />
        <%= f.text_field :job_title %>
      </div>
      <div class="actions">
        <%= f.submit %>
      </div>
    <% end %>

Also change `:name` and `:job_title` to `url` and `title`:

     <%= form_for(@link) do |f| %>
      <% if @link.errors.any? %>
        <div id="error_explanation">
          <h2><%= pluralize(@link.errors.count, "error") %> prohibited this link from being saved:</h2>

          <ul>
          <% @link.errors.full_messages.each do |msg| %>
            <li><%= msg %></li>
          <% end %>
          </ul>
        </div>
      <% end %>

      <div class="field">
        <%= f.label :title %><br />
        <%= f.text_field :title %>
      </div>
      <div class="field">
        <%= f.label :url %><br />
        <%= f.text_field :url %>
      </div>
      <div class="actions">
        <%= f.submit %>
      </div>
    <% end %>

Once you've got that saved, refresh the `links#new` page and you should get an error:

    undefined method `model_name' for NilClass:Class

So this error isn't so great, but if you look at the line where it is pointing:

    <%= form_for(@link) do |f| %>

We're passing in a `@link` variable we haven't initialized yet. Let's add that to our `links#new` action in the controller:

    @link = Link.new

Refresh the page, you should now see a form. Fix any errors until you do.

## Create a Link

By now we can render our view form, so we've got our model, routes, view, and most of our controller out of the way. Let's go back to our user story:


    "A user visits the home page of the website, they then log in. They click a "submit link"
    button and enter a url and a title. They submit that link and title and then end up on a
    page that shows the title, link, votes for that link and any comments."

So now we can do the first and second sentence. What about the next one:


    "They submit that link and title and then end up on a page that shows
     the title, link, votes for that link and any comments."

Let's enter a title and url and see what happens. You might end up on a blank create view, but that's not what we wanted, we actually want to populate our database. We'll have to add that to our controller. I want you to code this for yourself. You can use old code, scaffolding, or just write it for yourself. Don't forget to use logs to see the `params` available:


    Started POST "/links" for 127.0.0.1 at 2012-07-29 16:54:44 -0500
    Processing by LinksController#create as HTML
      Parameters: {"utf8"=>"✓", "authenticity_token"=>"b7ZKLfYSbuzzfa1/X+GU0p3sH5l0B7ekFgeXtndeLj0=", "link"=>{"title"=>"Heroku", "url"=>"http://herokou.com"}, "commit"=>"Create Link"}


When you're done you should be able to enter in a title and url and find it in the console.

    $ rails console
    > Link.last.url
    => "http://heroku.com"
    > Link.last.title
    => "Heroku"

When you can do that, save and commit to git. Looking at our user story, we're expected to be able to see the link once we've submitted. To do this, you'll want to redirect to the show action if a save is successful in the create action.

Once a user lands on the show action after submitting their form, we're almost done with our user story. Save and commit to git.


    "They submit that link and title and then end up on a page that shows
     the title, link, votes for that link and any comments."

This last part mentions links, votes, and comments. While we already have links, we don't have votes or comments, and this is the first time they've been mentioned.

That indicates to me that we should have some more user stories before completing the rest of this one. Votes first:

    "A logged in user visits the home page, sees a user submitted link and votes on it.
     The user should end up on the home page and the vote count should change up or down
     according to how they voted"

Now Comments

    "A logged in user visits the home page, clicks the comments link under a user submitted
     link, there they can add a comment message to that link. Other users can view that
     comment message by visiting the link's show page."


This gives us a good starting point for next week. Make sure everything you've got is committed.

## Production

Once everything works follow the appropriate directions here: https://devcenter.heroku.com/articles/rails3 to put your app on Heroku. You should be given a unique URL. You can also reference this guide: http://guides.railsgirls.com/heroku/.

Email me a link to your app's url. Your final project needs to be capable of running on Heroku, now is a good time to get experience configuring an application for production.


## Fin

Hopefully by now you're growing accustomed to the idea of building MVCr and then repeating. User stories can help us focus on what we need to move forward, and running into errors because we haven't implemented parts yet, can be helpful. Programming by nature is reductive. We take a problem and reduce it into smaller manageable pieces until that problem is solved. The more you do this the better you will get. You'll find some of my methods don't work well for you, or you'll find others do things slightly different. At the end of the day only two things matter, that the website behaves as you want, and that the website behaves as a user expects. As you grow, you'll likely get better at programming in general, never forget that while better code means it will be easier to maintain and change, if you neglect the user experience then your web app isn't any better.

You've come a long way in your journey to becoming a Rails master. Many of you are picking up Rails as your first serious form of coding, which is quite impressive. In just a couple of short weeks we've gone from nothing to be able to make a pretty big dent in the functionality of a complex website like Reddit in just one day.


Go celebrate by visiting http://www.reddit.com/r/funny/ for a bit (caution potentially strong language).