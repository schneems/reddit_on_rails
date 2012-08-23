## Reddit on Rails: A Hero's Journey Part 2

## Last Week

Last week in our [UT on Rails Course](http://schneems.com/ut-rails), we used user stories to prototype the link posting ability in a site that replicates some of the functionality form [reddit](http://reddit.com). We broke problems into smaller problems until we were able to solve them. We built MVCr and had a pretty good start to our project. When you step away from a project, it can be difficult to remember where you left off. If possible, I always like to end with a reminder of where I stopped. For some people that means writing an unfinished test, or writing a physical note in a notebook. Since you can't exactly read my notebook right now, we ended with some unfinished user stories so before we try to continue, let's see exactly where we ended. After we implemented most of the link submitting functionality we wrote these two user stories:

    "A logged in user visits the home page, sees a user submitted link and votes on it.
     The user should end up on the home page and the vote count should change up or down
     according to how they voted"

and

    "A logged in user visits the home page, clicks the comments link under a user submitted
     link, there they can add a comment message to that link. Other users can view that
     comment message by visiting the link's show page."


Now we have a place to start. Looking at the first sentence in both of those stories, a user logs in (we already implemented this functionality) and then when they visits the home page they will see links and comments. Since we haven't built comments yet, lets populate our front page with some links.

Like last week you'll be required to use critical thinking and everything you've learned to build out our app. You will run into exceptions and strange behavior. Use docs, logs, the console, previous projects, and anything else you can get your hands on. Make small changes and verify every step of the way. You excited? I am, let's do this thing!

## Links on the Homepage

We've already built a good deal of functionality for our Link feature. We have a link model, a links controller, link views, and routes. We also have a home page we built last time in `pages#index`, so we need to pull links out of our database and put them on our front page. We probably want to start in the console to make sure that we've got some links. Fire up the `rails console` and verify you have links in your database:

    $ rails console
    > Link.count
    # => 3

If you have one or fewer links make some right now using `Link.create` remember links have a `url` and a `title` if you ever need to see the schema for a given model you can just type it into the console.

    $ rails console
    > Link
    # => Link(id: integer, user_id: integer, url: string, created_at: datetime, updated_at: datetime, title: string)

Now you should have a handful of links in your database for testing purposes. Start your server and visit the home page [http://localhost:3000](http://localhost:3000). You should see a sign in or sign out link depending on your status as well as a link labeled "add a link". So what do we need to do to get links displayed on our home page? We need to pull them out of the database and then display them some how. Write a SQL query that populates an instance variable `@links` in the `pages#index` controller. Then in your view display all of the links on your site. Eventually we will want to order these and paginate them, but for now displaying will be enough. You will want to display the title, and when someone clicks the title they are taken to the submitted url. If you're confused about the functionality visit [reddit](http://reddit.com)


    <ul>
      <% @links.each do |link| %>
        <li>Put your logic here</li>
      <% end %>
    </ul>


Once you've got that working a user should be able to refresh the page, click on a title and be redirected to the given url. Once you've got that working commit to git.


Let's take a step back and think about what we just did. We pulled links from the database and displayed them on our home page. What we just did isn't exactly new. We've pulled users and products, and all sorts of things out of databases and displayed them in lists, but this is the first time we've done so in a custom controller. Remember we made the `pages#index` controller and action to satisfy an earlier user story when we were adding devise. While we arguably could have used the `links#index` controller and action we aren't bound to just using controllers that are directly associated with models in the database. In fact, in any well designed Rails app you'll find that you will have many more controllers than models.

While we are on the home page lets break from our user stories for just a second and build in pagination and set the order of our links.

## Pagination

In the last section we pulled all of our Links from the database and displayed them on our homepage. Now we're going to add some pagination (the "next" and "previous" links you see when you browse sites like Google or Amazon). To do this we will use the [will paginate gem](https://github.com/mislav/will_paginate) follow the instructions to add the gem to your gemfile then run `$ bundle install`. Now you should be able to go into your console and run this:

    $ rails console
    > Link.order('created_at DESC').page(1).per_page(1)

You should get an array with one link, if you change that to the second page you should get the next link in the series

    $ rails console
    > Link.order('created_at DESC').page(2).per_page(1)

Change your active record query in your controller to use `page`, `order`, and `per_page`. You can pull values from the params hash `params[:page]` and `params[:per_page]`. You will need to provide some default values for instance if you use

    params[:per_page] ||= 25
    params[:page]     ||= 1

Then the page number will be supplied by the user and if it is not it will be set to 1. Similarly if a per_page is not supplied it will default to 25 (which is how many links show up when you visit reddit).

Add this functionality to your controller now. To finish up pagination we can use will paginate's view helper. Add this to your view:


    <%= will_paginate @links %>


If you have more links in your will paginate collection than are currently showing you will get a convenient "next" and "previous" links where appropriate. You can test this out by visiting the home page and supplying `per_page` and `page` parameters [http://localhost:3000?per_page=1&page=1](http://localhost:3000?per_page=1&page=1).

Save and commit to git.

## Comments

Okay we got some of this user story finished

    "A logged in user visits the home page, sees a user submitted link..."

We even added a bit more with pagination. We could of course write a user story around pagination if you wanted but it's such a common feature typically saying "paginate the links" would be enough. From here you could refine your pagination with different styles, but for now we're done with that feature. While the end of the sentence talks about votes, I would like to build a comments feature first. To help us we're going to switch user stories that we are working on. While it feels weird that we are leaving a story in mid sentence, remember that we are using them to gnererate small discrete tasks. If we were to break these into a to-do list we could have put listing all the links on the homepage under one to-do. Staying flexible and working on the most pressing/interesting problems/features will help you stay sharp and focused. Let's take a look at that other user story.


    "A logged in user visits the home page, clicks the comments link under a user submitted
     link, there they can add a comment message to that link. Other users can view that
     comment message by visiting the link's show page."


Okay, so a logged in user can already visit the home page, but there is no comments link under a user submitted link. So let's add one. Before we do finish the rest of the story, we need to understand the full scope of what that link should do and where it should go. The last sentence gives us a clue, we should see comments for a given link by visiting that link's show page. So right now in your home page add this under each of your user submitted links.

    <%= link_to "comments", link %>

Where link is an active record object. If you prefer you could also run `$ rake routes` and find that the link helper is called `link_path()` so you could alternatively use `link_path(link)` or `link_path(link.id)` or `link_path(:id => link.id)` which are all similar. Once you've added your link visit your [Home Page](http://localhost:3000) and refresh, you should see a "comments" link. Click one of them and you should be directed to `links#show`, you can confirm via the url or using your rails logs.
Once you have this working save and commit to git.

Taking a look back at our story we see that have some nouns without associated models. We have "comment" and that comment has a "message" and we need to know what link the comment is on. We will also want to store the author who left that comment. So right now we know that a comment will have exactly one and only link, that a link can have many comments, a link will belong to only one user and a user can have many comments. Let's see what a model might look like:

      Comment Table
        message:    text
        user_id:    integer
        link_id:    integer
        created_at: datetime
        updated_at: datetime

Here we are setting message to `:text` instead of a `:string`. By default a string will be limited to 255 characters, while `text` has no limitation on length.


If you're confused, especially about why we want a `user_id` and a `link_id` remember week two of our [UT on Rails](http://schneems.com/ut-rails) course  where we talked about [modeling relationships](http://schneems.com/post/25503708759/databases-rails-week-2-modeling-relationships-and). You can also view the first part of last week's exercise where we modeled our links. You might remember that in this type of association (has_many & belongs_to) we must store the relationship somehow in our database, and it is far easier to store it with the record that "belongs" to the other record. So since a comment belongs to both a "user" and a "link" we will add a foreign key of `user_id` and `link_id` to use with our association.

Now would be a good time to discuss open source libraries. We used an open source library for user authentication and for pagination so why not for comments? If you browse [the ruby toolbox](https://www.ruby-toolbox.com/categories/rails_comments), you'll see there are more than a few comment gems out there. The top gem: [Acts as Commentable with Threading](https://github.com/elight/acts_as_commentable_with_threading) is one attractive option. When deciding if I desire to use a given open source library to solve a problem, I ask myself about the problem, and about the library. With authentication, do I really want to write code that could affect the security of my site? With pagination, the ordering and the offset is easy enough in plain SQL but I will_paginate gives me a nice syntax and some convenient helper methods, like the view helper that gave us the next and previous butttons. With comments I already understand the problem well enough to have an idea what the model needs to be, so I'm not sure if a library will actually help me with the implementation. So the next part would be looking at the library. Using the Ruby Toolbox or your own detective skills, you can decide if a library is active and people are using it based on the number of watchers, the last time it was updated, and the quality of documentation. Devise and will_paginate both have quite a bit of watchers and you can see download activity on [Rubygems.org](http://rubygems.org/gems/will_paginate). We aren't looking for raw numbers, we're looking for signs that a developer has tried the library and then liked it enough to come back and use it again.

In this scenario [Acts as Commentable with Threading](https://github.com/elight/acts_as_commentable_with_threading) doesn't look like a bad option, but it would be better for us to have more control over our code for now. By writing custom commenting code, you will be better equipped in the future to decide if you should use a library or not.

## Comment Model Implementation

Now that we've got an idea how we want to implement our comments, we'll need MVCr. We already know what our model should look like so we will need the ruby code in `app/models` and we will also need a migration. You can use a generator for both or build the ruby model manually and then generate the migration using a generator. Since last time we used `$ rails generate model`. We will build our model manually and then make a migration using a generator.

First make a new file `app/model/comment.rb` remember that models are always singular. Inside of your model you can copy the code we have in our `link` model:

    class Link < ActiveRecord::Base
      attr_accessible :title, :url
    end

But we will need to change a few things:

    class Comment < ActiveRecord::Base
      attr_accessible :message, :user_id, :link_id
    end

Now we need some fields in our database run:

    $ rails generate migration create_comments

You'll see something like this:

    class CreateComments < ActiveRecord::Migration
      def up
      end

      def down
      end
    end


When we make a migration, we need to tell Rails how to undo the changes we've made (like last week when we rolled some of our migrations back). If we add a column when we are migrating up, we need to tell it to drop that column when we are rolling back or migrating down. If you look in the migration `add_devise_to_users.rb` you will see there is code in both an up and a down method. If you glance at some other migrations you'll see that instead of `up` and `down` we simply have a `change` migration. This is relying on Rails to be smart about our changes. If we tell it to add a column then it will assume that is what we want to do while going up, and we want to do the opposite while going down (remove that column). Not all changes can be reversed automatically so, thats why Rails gives us the option of using `up` and `down`. While I advocate knowing where to find the docs and being able to write a migration yourself from scratch, I personally find it quicker to look at older migrations when writing my own. Go into the `create_links.rb` migration and copy the change code:

      def change
        create_table :links do |t|
          t.integer :user_id
          t.string  :url

          t.timestamps
        end
      end

Replace `links` with `comments` and then add the columns we need (message, user_id, link_id). Don't forget that `message` is a `text` column and not a `string` column. When you are done run the migrations `$ rake -T` if you can't remember the command. Once you have migrated the data you should be able to run this in the console:

    $ rails console
    > comment = Comment.new
    > comment.user_id = 1
    > comment.link_id = 1
    > comment.message = "Hello world"
    > comment.save

If you can't then try rolling back your migration and fixing any problems or error messages you encounter. Once you get it working we'll want to add the association to our comments. Add `has_many` and `belongs_to` in the appropriate models `app/models/comments.rb`, `app/models/link.rb` and `app/models/user.rb`. Once you have them specified correctly you should be able to run this in the console:

    $ rails console
    > Comment.last.link
    > Comment.last.user
    > User.last.comments
    > Link.last.comments

Once you can run those commands in your console correctly then save and commit the results to git.

Now we've got a pretty good model on our hands let's give our user's some views, controllers, and routes to allow them to add comments without the command line.


## Comments VCr: Controller

Going back to our story:

    "A logged in user visits the home page, clicks the comments link under a user submitted
     link, there they can add a comment message to that link. Other users can view that
     comment message by visiting the link's show page."

A user will need to add a comment on a link's show page, this is where we will add our view for the form and for showing all the comments. That means we will need a controller action to handle creation of a comment and an associated route.

First start out by creating a file `app/controllers/comments_controller.rb` (while models are singular, controllers are plural). From there we can copy the structure in our links controller and replace the needed parts:

    class LinksController < ApplicationController
      def show
        # ...
      end

      def new
        @link = Link.new
      end

      def create
        # ...
      end

    end


We need to change the name of our controller and we will only need the create action:

  class CommentsController < ApplicationController
    def create
      # ...
    end
  end

When a comment is created, we can just redirect the user back to the same view so we can use this code:

    def create
      @comment = Comment.create(params[:comment])
      redirect_to :back
    end

## Protecting our user_id (fyi)

Based on this, we can construct a comment based on a user supplied `:user_id`, `:link_id` and `:message`. That means somewhere in our form we have to have a field (hidden or otherwise) for those fields. Do you see any problem having our controller accept a user_id from our user? When we build html in ruby and send to a user, our user could use a tool like chrome's inspector to edit that html and submit a form. So even if we have a hidden field like this:

    <input id="user-id" name="comments[user_id]" value="9" type="hidden">

A malicious user could edit our html to change that value from 9 to an arbitrary `user_id`. This means that anyone could post a comment as any other user. To get around this we can use devise to make sure a user is logged in when hitting our controller action and then we can use our `current_user` variable to make sure only the currently logged in user is making comments that are tied to their own account.

Looking at the the [Devise Docs](https://github.com/plataformatec/devise/) you will find that we can require a user to be logged in by adding this in our controller:

    before_filter :authenticate_user!

So we can add this to our controller like this:


    class CommentsController < ApplicationController
      before_filter :authenticate_user!

      def create
        @comment = Comment.create(params[:comment])
        redirect_to :back
      end
    end

Rails give us the [before_filter](http://guides.rubyonrails.org/action_controller_overview.html#filters) method we can use in our controllers. What we are saying here is that devise has given us a method called `authenticate_user!` that will run before any of our methods. If `authenticate_user!` redirects, then the user will never hit the `create` action. Basically the code inside of `authenticate_user!` checks to see if a user has a valid session and if they are logged in it will let them continue, otherwise it will redirect them to the sign in path.

Before filters can take options, for example if we only wanted this to run on the create action we could use this instead:

    before_filter :authenticate_user!, :only => [:create]

You can make any action into a before filter, you don't have to rely on devise. For instance you could make one called `redirect_back_unless_logged_in`:


    class CommentsController < ApplicationController
      before_filter :redirect_back_unless_logged_in

      def create
        @comment = Comment.create(params[:comment])
        redirect_to :back
      end

      def redirect_back_unless_logged_in
        redirect_to :back unless current_user.present?
      end
    end

But since devise has already given us a very useful method we can just use that instead. So now that we have ensured a user is correctly logged in. We can make sure that only a `current_user` (which we get from devise) will be allowed to make new comments under their own account. Since we can call `User.last.comments` and `User.last.comments.create(:message => "foo")` we can use this syntax in our controller:


    class CommentsController < ApplicationController
      before_filter :authenticate_user!

      def create
        @comment = current_user.comments.create(params[:comment])
        redirect_to :back
      end
    end

By calling `current_user.comments` we are telling rails that the foreign key in comments will have to point at current_user's primary key. This also means now we don't have to include `user_id` in the form on our website. Sweet.

You may be wondering about how we get associate our link in our form and how we can make sure that the user is only submitting a comment for the correct link. As we hinted at earlier we will use a hidden field in our form to supply the controller with a `link_id`. For the next part checking that the user didn't modify that id in the html...we won't worry about it. If a logged in user wants to add a comment on a different link that's fine, it's not going to affect anyone except for the currently logged in user. This is different from the `user_id` problem we had earlier where a user modifying their html would result in another user seeing strange behavior.

If you're a bit lost, don't worry, if your website is small or if it is for an internal tool, then you'll likely never run into the problem of opening this type of security hole. If you decide to take up Rails development for a living, then you will need to understand this risk very well.

## Comments VCr: Routes

By far the easiest part in the MVCr to implement if you're sticking to defaults add this to your routes:

      resources :comments

If you wanted you could scope this to only produce a route for `create`:

      resources :comments, :only => [:create]

Typically I don't bother with this as I'm likely to add other functionality later.


## Comments VCr: View

So now we've got our model, controller, and routes we just need to add a view. We already settled that we would have our form in our `links#show` view. So let's copy an existing form from `links#new` and then modify it. Open up `app/views/links/show.html.erb` and paste in the copied form:


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

First we'll need to change `link` to `comment`:

    <%= form_for(@comment) do |f| %>
      <% if @comment.errors.any? %>
        <div id="error_explanation">
          <h2><%= pluralize(@comment.errors.count, "error") %> prohibited this comment from being saved:</h2>

          <ul>
          <% @comment.errors.full_messages.each do |msg| %>
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

Next we'll change `:title` to `:message` and remove `:url`:

    <%= form_for(@comment) do |f| %>
      <% if @comment.errors.any? %>
        <div id="error_explanation">
          <h2><%= pluralize(@comment.errors.count, "error") %> prohibited this comment from being saved:</h2>

          <ul>
          <% @comment.errors.full_messages.each do |msg| %>
            <li><%= msg %></li>
          <% end %>
          </ul>
        </div>
      <% end %>

      <div class="field">
        <%= f.label :message %><br />
        <%= f.text_field :message %>
      </div>
      <div class="actions">
        <%= f.submit %>
      </div>
    <% end %>

Notice that our `:message` is a `text_field` this what we used for our `title` and `url` in the link model. While this makes sense for entering in strings, `:message` is stored in a `text` column. So it would be easier to add more text if we use a `text_area` instead:

        <%= f.text_area :message %>

As a bonus, we can add some placeholder text:

        <%= f.text_area :message, :placeholder => "Add a comment" %>



Now we'll just need to populate our `@comment` instance variable. Open up your links controller and add `@comment = Comment.new` into your `show` action:

    def show
      @link    = Link.find(params[:id])
      @comment = Comment.new
    end

By now we should have enough to render our view. Visit a link's page like [links/1](localhost:3000/links/1) and a form for our comment should render. If you get errors, fix them.


We also want to add a hidden field for our `link_id`

        <%= f.hidden_field :link_id, :value => @link.id %>

Add this code, and refresh your page, you won't see anything change but if you view source or use a tool like chrome's inspector you should see something like this in the html:

    <input id="comment_link_id" name="comment[link_id]" type="hidden" value="1" />

Now when we submit the form we will get `:link_id => 1` added to our `params[:comment]` hash.

Log in if you aren't already, now enter in a comment, and click submit. You should be redirected back to the same link show page. You can check your logs to make sure that we hit the right action:


    Started POST "/comments" for 127.0.0.1 at 2012-08-05 16:00:59 -0500
    Processing by CommentsController#create as HTML
      Parameters: {"utf8"=>"✓", "authenticity_token"=>"b7ZKLfYSbuzzfa1/X+GU0p3sH5l0B7ekFgeXtndeLj0=", "comment"=>{"link_id"=>"1", "message"=>"hello world"}, "commit"=>"Create Comment"}
      User Load (0.3ms)  SELECT "users".* FROM "users" WHERE "users"."id" = 1 LIMIT 1
       (0.1ms)  begin transaction
      SQL (11.4ms)  INSERT INTO "comments" ("created_at", "link_id", "message", "updated_at", "user_id") VALUES (?, ?, ?, ?, ?)  [["created_at", Sun, 05 Aug 2012 21:00:59 UTC +00:00], ["link_id", 1], ["message", "hello world"], ["updated_at", Sun, 05 Aug 2012 21:00:59 UTC +00:00], ["user_id", 1]]
       (0.9ms)  commit transaction
    Redirected to http://localhost:3000/links/1
    Completed 302 Found in 109ms (ActiveRecord: 14.5ms)


Here notice that we called the `CommentsController` and the `create` action (`CommentsController#create`) and that our parameters contain a `comment` hash:

     "comment"=>{"link_id"=>"1", "message"=>"hello world"}

Also notice that this hash has both a `link_id` and a `message`. You can verify that the comment was correctly created in the console:

    $ rails console
    > Comment.last
    # => #<Comment id: 2, user_id: 1, link_id: 1, message: "hello world", created_at: "2012-08-05 21:00:59", updated_at: "2012-08-05 21:00:59">

Furthermore we can verify it has the proper link and user associations.

    $ rails console
    > Comment.last.user
    # => #<User id: 1, email: "richard ...
    > Comment.last.link.url
    # => "http://schneems.com"

So now that we are sure our comment was saved correctly we want to finish out our user story:

    "A logged in user visits the home page, clicks the comments link under a user submitted
     link, there they can add a comment message to that link. Other users can view that
     comment message by visiting the link's show page."

We have to allow other users to view those comments. You can do that by adding some code similar to this in your link show view:

    <h2>Comments:<h2>
    <div>
      <% @link.comments.each do |comment| %>
        <p><%= comment.message %></p>
      <% end %>
    </div>

Once you've done that then our comments user story is complete. Ensure that comments show up when you refresh the page and attempt to make new comments. Fix any errors you encounter. Save and commit to git. You should commit to git frequently in your own projects so if you get something working and then break it while working on a new feature you can inspect the most immediate changes using a tool like gitx for the github app.


At this point and time it might be a good idea to write a new comment user story or or to add to the existing one. You can visit [reddit](http://reddit.com) for some inspiration. We could add votes to our comments, we could add comment threading, but at very least we can put the author's name and how long ago the comment was posted on the form. I'll leave the exercise of writing a user story up to you including everything else you want. We may do more with comments later but for now we need to finish our existing user stories. We still haven't finished voting.


## Voting

Take a look at the remaining user story:


    "A logged in user visits the home page, sees a user submitted link and votes on it.
     The user should end up on the home page and the vote count should change up or down
     according to how they voted"

This tells us we need to have a "vote" noun, which implies that we need a "vote" model. So it begins the great MVCr dance. We'll use the same methods we used with links and comments. Lets first start building the model. Reading the story we know that a vote should belong to a user and should be on a link, so we need some associations there. We also know that the vote should be either positive (up) or negative (down). First let's look at the associations, how would you describe them to someone.

A user and a vote are associated somehow. Can a user have a single vote? Yes. Can a user only have a single vote (i.e. only vote once ever)? No, a user can vote on multiple links so they should have many votes. Can a vote have multiple users? No. Can a vote have one user? Yes. So based on these questions we now know that users has_many votes and that votes belongs_to user. Sometimes it can be confusing to which syntax we should use. I've seen people say incorrectly that a user has_many votes and that votes has_one user but this isn't quite right. Your foreign key goes on the model that belongs_to the other model, so without one of your models belonging to one of your other models there wouldn't be a foreign key!! Unlike in fight club, the things you own do not end up owning you. If a user has many products, one of those products does not "have" control over their user, but simply belongs to that user.

Let's do a similar exercise with a vote and a link. Can a link have more that one vote? Yes. Is a link limited to one vote total? No. Can a single vote belong to a link? Yes. Can a single vote belong to multiple links? It could but that would be weird, so...no.

Simple questions like these can help you define the proper associations to make in your Rails app.

Putting that all together, a user has many votes, and a vote belongs to a user. A link has many votes and a vote belongs to a link. This is very similar to the relationships we just built for comments.

Next we'll need to figure out how to store whether a vote is an "up" vote or a "down" vote. We could store this in any manner of ways. We could have a string column that we populate with "up" or "down" or we could have an integer column that is `1` for an up vote and `-1` for a down vote, or we could have a boolean column that is `true` when you have voted something up and `false` when you have voted something down. Any of those would work so we'll just chose one. I like booleans for this since you don't have to worry about misspelling anything and we're really only storing two different states anyway. If you wanted to allow people to rate something from 0 to 5 in the future instead of a simple boolean vote, picking integers might be of value, but for for now simpler is better so we'll use boolean values.

    Vote Table
      user_id: integer
      link_id: integer
      up:      boolean

Here we will store `up` as true if the user has voted the link up and false if the user has voted the link down. Now that we've know what we want our model to look like we need to make a migration and a `app/models/vote.rb` file. You can do this anyway you chose, using a generator or manually. Look back in the directions if you need help, by now you should be able to make your own migrations and model files. When you are done you should be able add a vote to the database:


    $ rails console
    > vote = Vote.new
    > vote.user_id = 1
    > vote.link_id = 1
    > vote.up = true
    > vote.save
    # => true

You should also be able to query the vote from the database based on associated models:

    $ rails console
    > Vote.last.user
    > Vote.last.link
    > Link.last.votes
    > User.last.votes

Note that user and link are singular while `votes` is plural. This is convention. Once you can do all of that with no errors and get back expected results save and commit the results to git.


## Votes VCr:

Generate routes for votes, see previous instruction or use the routes files.

Generate a controller for votes. For now it only needs a `create` action. It should belong in `app/controllers/votes_controller.rb` (controllers are plural). You can copy the comments controller and go from there, but instead of `current_user.comments` we will need `current_user.votes`. When a user submits a vote they should go back to the same page they started on. We can do that using `redirect_to :back`.

Now we've got a model, a controller, routes but no view. Back to our user story:

    "A logged in user visits the home page, sees a user submitted link and votes on it.
     The user should end up on the home page and the vote count should change up or down
     according to how they voted"


So we have links to add votes from the home page. Let's open up `pages#index`. We should have something that looks similar to this:

    <% @links.each do |link| %>
      <%= link_to link.title, link.url %>
      <%= link_to "comments", link %>
    <% end %>

Where we are displaying all of our paginated links. We want to add an up-vote and a down vote button here. Let's do that now. If we run `$ rake routes` we will see that we can access the create action using this route:

                   votes GET    /votes(.:format)               votes#index
                         POST   /votes(.:format)               votes#create

So we can use the `votes_path` helper and we must specify that we use the POST http request. Let's add an up vote to our view right now:

      <%= link_to "+", votes_path(:vote => {:link_id => link.id, :up => true}), :method => :post %>

Before you refresh your [home page](http://localhost:3000) make sure that `link_id` and `up` are both accessible in your vote model. Now visit the [localhost:3000](http://localhost:3000) and you should see a "+" next to each link. Click on one of them, you should be redirected back to the same page. If you get any errors, fix them and try again. You get a successful post you should see something like this in your log:


    Started POST "/votes?link%5Blink_id%5D=1&link%5Bup%5D=true" for 127.0.0.1 at 2012-08-05 17:43:14 -0500
    Processing by VotesController#create as HTML
      Parameters: {"authenticity_token"=>"b7ZKLfYSbuzzfa1/X+GU0p3sH5l0B7ekFgeXtndeLj0=", "vote"=>{"link_id"=>"1", "up"=>"true"}}
      User Load (0.2ms)  SELECT "users".* FROM "users" WHERE "users"."id" = 1 LIMIT 1
       (0.1ms)  begin transaction
      SQL (0.6ms)  INSERT INTO "votes" ("created_at", "link_id", "up", "updated_at", "user_id") VALUES (?, ?, ?, ?, ?)  [["created_at", Sun, 05 Aug 2012 22:43:14 UTC +00:00], ["link_id", nil], ["up", nil], ["updated_at", Sun, 05 Aug 2012 22:43:14 UTC +00:00], ["user_id", 1]]
       (2.1ms)  commit transaction
    Redirected to http://localhost:3000/
    Completed 302 Found in 7ms (ActiveRecord: 3.0ms)


Verify the controller and action are correct, and verify that the parameters contain a "link" hash.

    "vote"=>{"link_id"=>"1", "up"=>"true"}

Now we can confirm that a vote was created by checking the console

    $ rails console
    > Vote.last
    # => #<Vote id: 3, up: true, user_id: 1, link_id: 1, created_at: "2012-08-05 22:51:12", updated_at: "2012-08-05 22:51:12">

If you don't see a `user_id` a `link_id` and an `up` value, then you need to figure out what went wrong and fix it. Now that we have our votes working we can add a vote count and make it possible to down vote a link:

      <%= link_to "-", votes_path(:vote => {:link_id => link.id, :up => false}), :method => :post %>

Note that we changed the value of `:up` to false. We can also show the number of total up votes and down votes.

      Up Votes: <%= link.votes.where(:up => true).count %>
      Down Votes: <%= link.votes.where(:up => false).count %>

Though likely we only want to show total vote count which would be up votes minus down votes:

    Votes: <%= link.votes.where(:up => true).count - link.votes.where(:up => false).count %>

Now if we refresh the page we can see the vote count that a link has on it! Go ahead and try down voting and up voting links to make sure that the value changes. Once you get all that working save and commit to git. Lets check our story:


    "A logged in user visits the home page, sees a user submitted link and votes on it.
     The user should end up on the home page and the vote count should change up or down
     according to how they voted"

It looks like everything is in good working order, but if you played around with our  voting feature you may have noticed that you can vote a link up once and only once. Our app doesn't have the same restriction, so one user could rig the vote. Before we fix this lets take a second to meditate on how this will affect users by writing another user story.

    "A logged in user that votes for a link cannot vote for the same link in the same
     direction. A logged in user can see that which links they have voted for and in which
     direction up/down"

## Vote Validations

The easiest thing we can do to prevent duplicate votes from saving is to add validations to our vote model. Open up `app/models/vote.rb` and add this validation:


    validates :user_id, :uniqueness => { :scope => :link_id }


You can view the [rails validation docs](http://guides.rubyonrails.org/active_record_validations_callbacks.html) for more info. Essentially what we are telling our vote model is that it cannot save any new votes unless the `user_id` and `link_id` combo haven't been taken created before. If we were going to leave the scope part out then it would verify that all `user_id` rows in our vote table are unique which would prevent users from voting on anything more than once...ever. Since we don't want that we only want to prevent users from voting on one link over and over again.


Now that we have new validation code in place we might already have some bad entries that have duplicate `user_id` and `link_id` saved to the database. You can get rid of these by iterating through all the votes in your system and deleting them if they are invalid. You can use the `find_each` method to get all votes (this uses batched find so you will not run out of memory even on a huge number of votes). This method takes a block that will be yielded to for each vote. [Find Each docs](http://apidock.com/rails/ActiveRecord/Batches/ClassMethods/find_each). This should do the trick

    Vote.find_each {|vote| vote.destroy unless vote.valid? }

Once you've done this you'll remove all duplicate votes. Now, with the validation code in your model, refresh your home page and click the "+" button a few times. What happens? Nothing? Good. If the count on the vote changes then double check your validations.

Now try to change your vote from an up vote, to a down vote. What happened? Nothing? Bad. A user should be able to switch their vote, why isn't this working? We're hitting the vote controller and the create action (double check with your log) so let's look at that code:

    def create
      @vote = current_user.votes.create(params[:vote])
      redirect_to :back
    end


So our down vote isn't saving but why? Let's debug with `puts` change your code to this:

    def create
      @vote = current_user.votes.create(params[:vote])
      puts "======================"
      puts @vote.errors.inspect
      redirect_to :back
    end

The first puts with the equals signs i call a tracer bullet, I add it just to make my second puts easier to find in the log file. The second puts will output any errors on our @vote object. If a validation prevents an object from saving to the database it will add `errors` to that model. Try voting something down again and let's see the output in the log.


In the log we can find our tracer and the errors:

    ======================
    #<ActiveModel::Errors:0x007fcb1b2d7fd8 @base=#<Vote id: nil, up: false,
      user_id: 1, link_id: 1, created_at: nil, updated_at: nil>,
      @messages={:user_id=>["has already been taken"]}>

So here we have the problem, our user_id has already been taken. This makes sense since we added a validation but we're adding a down vote, not an up vote...why are we getting this error?

We can see that from the inspect that we were trying to save a user_id of `1` and link_id `1` (note your numbers might be different). Open up a console and look for a vote with those credentials:


    $ rails console
    > Vote.where(:link_id => 1, :user_id => 1).first
    # => #<Vote id: 3, up: true, user_id: 1, link_id: 1, created_at: "2012-08-05 22:51:12", updated_at: "2012-08-05 22:51:12">

Okay so there is already a vote with that user id and link id in the database. How do we change the attribute? Rather than trying to create every time in the controller we could try to find a link before updating its attributes! We could do something like this:

    @vote = Vote.where(:link_id => params[:vote][:link_id], :user_id => current_user.id).first
    if @vote
      @vote.up = params[:vote][:up]
      @vote.save
    else
      @vote = current_user.votes.create(params[:vote])
    end
    redirect_to :back


Or we could get fancy with our boolean logic:

    @vote = current_user.votes.where(:link_id => params[:vote][:link_id]).first || current_user.votes.create(params[:vote])
    @vote.update_attributes(:up => params[:vote][:up])
    redirect_to :back

Or written another way we could do

    @vote = current_user.votes.where(:link_id => params[:vote][:link_id]).first
    @vote ||= current_user.votes.create(params[:vote])
    @vote.update_attributes(:up => params[:vote][:up])
    redirect_to :back

All of these are doing roughly the same thing. Change the code in your controller to the method you like best. You should be able to refresh the page and click either "+" or "-". Make sure you removed invalid votes from the database as above. If you get any errors fix them, then save and commit the results to git.

So we're making some good progress but still haven't quite finished our story:

    "A logged in user visits the home page, sees a user submitted link and votes on it.
     The user should end up on the home page and the vote count should change up or down
     according to how they voted"

We need to let a user know when they have voted for a link and whether they voted up or down. To do this reddit changes the up vote arrow color. Since we don't have votes yet we can change the "+" to a "*" and make it not click-able. You can do that using SQL like this:



      <% if current_user && current_user.votes.where(:link_id => link.id, :up => true).present? %>
        *
      <% else %>
        <%= link_to "+", votes_path(:vote => {:link_id => link.id, :up => true}), :method => :post %>
      <% end %>

Go ahead and add that to your home page view. Make sure that behavior matches what you would expect. Repeat the process for the down vote link. Test the behavior and make sure everything works as you desire. Save and commit to git.


Now are user story is complete! We'll need to write some more user stories and some more code next week, but for now sit back, look at what you've created and be proud.

Take a second to deploy your changes to Heroku where anyone you want can get to your website.


## Fin

We've done quite a bit today. We prototyped and built two new models along with some complex VCr behind the scenes. While our site could certainly use some polish, it's beginning to have some of the base features we would expect from a service similar to [Reddit](http://reddi.com). Hopefully you're feeling a bit more confident in your ability to dissect a user story and pull out the needed elements to build a model. If you don't get it right the first time, you can always rollback or add columns. Comments and votes are pretty important pieces in many web sites, you can find different variations of them on facebook, youtube, and of course Reddit. Understanding how to take an abstract concept like a "vote" or a "like" and translate that into a working web feature is much easier if we break it into pieces with the help of user stories. There will be one final exercise where we add some polish to our sites. The one difference between the process I've been walking you though and what I would actually do is user testing. As I'm building features i'm constantly asking my friends and family if my user stories make sense, and when I get working prototypes like what we have here, I ask them to use it with minimal direction to see if they find any new bugs or problems. Non-technical friends can be more helpful than technical friends when it comes to validating your designs and implementation. Hopefully you're working on your own project in addition to following these tutorials right now. If you are, let go of your ego and put it in front of someone with little or no explanation. You will learn more in 10 minutes of watching another user use your product than in hours of reading up on usability best practices.

You've come along way in a short amount of time. You should be proud of your work you deserve a break, go check out [/r/pics](http://www.reddit.com/r/pics/) and kill some time (caution user submitted though typically awesome pictures).