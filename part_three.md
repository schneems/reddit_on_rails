## Reddit on Rails: A Hero's Journey Part 3
## Last Week

Last week in our [UT on Rails Course](http://schneems.com/ut-rails), we prototyped and built two new models along with some complex VCr behind the scenes. While our site could certainly use some polish, it's beginning to have some of the base features we would expect from a service similar to [Reddit](http://reddit.com). Hopefully you're feeling a bit more confident in your ability to dissect a user story and pull out the needed elements to build a model. If you don't get it right the first time, you can always rollback or add columns.


## This week

Since we've been focusing on getting our barebones features to work, this week we will work on some polish. We'll add some styling, do some testing to ensure we don't accidentally break our site, add a site search, and talk a little about how you could move forward with the site.


## Make it Pretty

Looking at [Reddit](http://reddit.com) you might not think that looks matter too much, but users consistently will choose to use a more attractive service if everything else is equal. If you've ever gone to a hackathon, or other competitive coding event, you're likely used to the winners having a polish, flashy UI. If you're building a professional quality web app, I recommend hiring a designer to help with wire-frames, comps, and assets generation.

While many sites might look like they're not designed (google, facebook, etc.) they put a great deal of time, money, and energy in maintaining that clean look. If you're interested in learning a little about design I recommend [the non designers design book](http://www.amazon.com/The-Non-Designers-Design-Book-Typographic/dp/1566091594) for an intro to design basics, for css I recommend anything by [Khoi Vinh](http://www.subtraction.com/2010/11/05/i-wrote-a-book), [Dan Cederholm](http://simplebits.com/#books), or [Zeldman](http://www.zeldman.com/).

Design, like programming, isn't something you can just pick up over night, it takes practice and effort to get good at. If you're really interested in this stuff, find a local user group, buy books, subscribe to blogs, and work at it.

Don't forget there is more to design than just looking pretty. Interaction and usability design is crucial for having a good application. You can go just as deep into usability as visual design. As a rails developer, you don't need to be the best at all of these things, but if you have a wide breadth of knowledge you will be more successful in coding and in life. Even if you never have to design a day in your life, understanding how to work with those that do will.



## Bootstrap

We will be using a CSS framework called [bootstrap](http://twitter.github.com/bootstrap/), released and by [Twitter](http://twitter.com). It is a collection of components that make it easier for people to design websites. Like Rails, they've decided on a set of defaults that you can change. There are a number of components but my favorite feature of bootstrap is a grid system. [Grids have been used to layout webpages](http://en.wikipedia.org/wiki/Grid_(page_layout) and other media for some time, but html and css have never had a built in grid system. Bootstrap helps us out by giving us one using standard css classes and html tags.

If you have bootstrap css loaded from http://twitter.github.com/bootstrap/assets/css/bootstrap.css then you could do something like this:


    <div class="row">
      <div class="span4">...</div>
      <div class="span8">...</div>
    </div>

And the result is that you have different elements of different column sizes (span4 and span8) inside of a row. You can go to town with this your grid system, for more info check out http://twitter.github.com/bootstrap/scaffolding.html#gridSystem. To do this next part you don't need to know bootstrap, but the more you know the easier it will be.


## Boilerplate Bootstrap Layout

You can cheat a bit and replace the code in your `app/views/layouts/application.html` layout with this:


    <!doctype html>
    <html>
      <head>
        <meta charset="utf-8">
      <title>RedditOnRails</title>

        <meta content="IE=edge,chrome=1" http-equiv="X-UA-Compatible">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <%= csrf_meta_tags %>


        <link href="http://twitter.github.com/bootstrap/assets/css/bootstrap.css" rel="stylesheet">
        <link href="http://twitter.github.com/bootstrap/assets/css/bootstrap-responsive.css" rel="stylesheet">

        <%= stylesheet_link_tag    "application", :media => "all" %>

        <script src="http://heroku.github.com/template-app-bootstrap/jquery-1.7.1.min.js"></script>
        <script src="http://heroku.github.com/template-app-bootstrap/jquery.replacetext.min.js"></script>

        <!--
          IMPORTANT:
          This is Heroku specific styling. Remove to customize.
        -->
        <link href="http://heroku.github.com/template-app-bootstrap/heroku.css" rel="stylesheet">
        <!-- /// -->

        <!--
          IMPORTANT:
          To do the replacement  example for the real name application
        -->
        <script src="http://heroku.github.com/template-app-bootstrap/heroku.js"></script>
        <!-- /// -->


      </head>

      <body>
        <div class="navbar navbar-fixed-top">
          <div class="navbar-inner">
            <div class="container">
              <a href="/" class="brand">Reddit on Rails</a>
              <div class='nav-collapse'>
                <ul class='nav'>
                  <%# user is logged in, show log out link %>
                  <% if current_user.present? %>
                    <li><%= link_to 'Sign Out', destroy_user_session_path, :method => :delete %></li>
                  <%# user is not logged in, show signup and login links %>
                  <% else %>
                    <li><%= link_to 'Sign In', new_user_session_path %></li> |
                    <li><%= link_to 'Register Now!', new_user_registration_path %></li>
                  <% end %>
                </ul>
              </div>
              <!--
                IMPORTANT:
                This is Heroku specific markup. Remove to customize.
              -->
              <a href="/" class="brand" id="heroku">by <strong>heroku</strong></a>
              <!-- /// -->
            </div>
          </div>
        </div>

        <div class="container" id="getting-started">
          <div class="row">
            <div class="span6 offset2">
            <div class="page-header"></div>
              <%= yield %>
            </div>
          </div>
        </div>
        <%= javascript_include_tag "application" %>
      </body>
    </html>



Browse over the code and try to make sense of it. We're loading the styles at the top of the page before any body content, and javascript last so the page appears to load faster and our dom is fully available by the time the javascript gets loaded. We have some Heroku branding that you can take or leave, but I think it looks nice. Take a look at the classes specified in the layout, these are what you can use to control the look and flow of your website with bootstrap.




## Add the Flash

It is important that you don't forget the flash message in your layout I add this code to show a flash messages right above my `<%= yield %>` in the `application.html.erb` layout:

    <% [:notice, :error, :alert].each do |level| %>
      <% unless flash[level].blank? %>
        <div data-alert="alert" class="alert alert-<%= level %> fade in">
          <a class="close" data-dismiss="alert" href="#">&times;</a>
          <%= content_tag :p, flash[level] %>
        </div>
      <% end %>
    <% end %>



## Sidebar

You might have noticed this curious line of code:


    <%= yield :sidebar %>


We already have a yield on the page so what is up with this `yield :sidebar` ? Rails gives us the ability to render from a variety of locations using yield and `content_for`. You can put content in that area from any page. For example if you want the "add a link" link to go in the sidebar from the main page you could do something like this:

    <%= content_for :sidebar do %>
      <%= link_to "Add a link", new_link_path %>
    <% end %>

Now when you refresh the link will show up in the sidebar. Using named yield like this is a great way to keep your views clean and to allow individual files to add content to layouts. If you didn't want to show that whole section when no sidebar content was available you can check to see if any content has been written.

    <% if content_for?(:sidebar) %>
      <div class="span2">
        <div class="page-header"></div>
        <h2>Sidebar</h2>
        <%= yield :sidebar %>
      </div>
    <% end %>

Now the sidebar will only be shown if you are rendering content for it. Chances are we always want to show content in the sidebar so this might not be needed, but just remember that you can always check for presence of content from a `content_for`.

In addition to sidebars I usually have a number of these yields in my layout including but not limited to

    <%= yield :head_stylesheets %>
    <%= yield :header %>
    <%= yield :footer %>
    <%= yield :js %>
    <%= yield :analytics %>

You can also use it to set the `id` and `class` of your main body with something like this:

    <body id="<%= yield :body_id %>"
          class="<%= yield :body_class %>
          <%= current_user ? 'logged-in' : 'not-logged-in' %>">

This can be useful for styling your whole layout differently depending on the page the user is viewing and if they are logged in or not. I also like to add the controller action and name to the body id by default. By adding in this hook, you could change the styling of your page later in an individual view file without needing a separate layout.

If you call content_for multiple times it will add multiple elements, this can be very convenient behavior. Try it out now by adding multiple links to the sidebar

    <%= content_for :sidebar do %>
      <%= link_to "Add a link", new_link_path %><br />
    <% end %>

    <%= content_for :sidebar do %>
      <%= link_to "Awesome Blog", 'http://schneems.com' %>
    <% end %>

Of course you could have just put both of those links in one `content_for` but sometimes it is nice to add content from different places and know that it is just getting added and won't over-ride the previous values. This is one of my favorite Rails features, and I don't see it used very much. If you find your view files becoming very large and the need for multiple layout files, consider if you might be able to dry some code up using `yield` and `content_for`.

Now that we've got our site looking like we want it to, let's add a piece of functionality that has come to be expected in just about every website these days. Search!


## Search it!

Lucky for us we're working with the ever so awesome [Postgres](http://postgresguide.com/) database in production on Heroku which has built in full-search capability. While one option could be matching strings with a `like` query:

    $ rails console
    > Link.where("title like '%ruby programming%'")

There is a better way. This is a very expensive, slow and not very accurate method for search. There are several alternatives out there to the built in postgres full-text search such as [web solr](https://addons.heroku.com/websolr) and [elastic search](https://addons.heroku.com/bonsai). If you have high load on your database it might be a good idea have your full text search on another service, though it would require another service on your machine. Heroku provides a free shared Postgres database in production so that makes using Postgres full text search an even better fit. They also offer many different tiers of [production postgres instances](https://postgres.heroku.com/pricing) when you're ready to grow.

It's always a good idea to keep your production and development environments as close as possible, so you'll want to use Postgres locally. Make sure that you've got a version of postgres of 9.1 or above locally:

    $ postgres --version
    postgres (PostgreSQL) 9.1.2

If you don't have postgres you will need to [install postgres locally](https://devcenter.heroku.com/articles/local-postgresql).

You'll also need to have the `pg` gem in your `Gemfile`

    gem 'pg'

And remove your sqlite gem. Then `$ bundle install`. Next specify the appropriate database adapter in your `config/database.yml`:

    development:
      adapter: postgresql
      encoding: utf8
      database: reddit_on_rails
      pool: 5
      host: localhost


If you're just switching over to Postgres now, you will lose any data you had stored in sqlite. This shouldn't be a problem, it just means you'll have to sign up and add some more links afterwards. Create a database by running:

    $ rake db:create

Now we need our schema in the database so you can run

    $ rake db:migrate

You should be set now, you can double check by restarting your rails server or opening up your console and performing a query.

While we could search all kinds of things, lets focus on searching for links right now, specifically the title of links. We could do this writing our own sql, but whenever you're doing something you feel might be fairly common it is always a good idea to research if there are any libraries you can use. To make using full text search with Postgres easier I decided to use the [Texticle Gem](https://github.com/texticle/texticle).

To get started add it to your Gemfile:

    gem 'texticle', :require => 'texticle/rails'

Then run:

    $ bundle install

Now you should be ready to go, fire up a console session and try it out. The gem puts a method `search` on all ActiveRecord objects.

    $ rails console
    > Link.create(:title => "UT on Rails", :url => "http://schneems.com/ut-rails")
    > Link.search(:title => "rails")
    => [#<Link id: 2, user_id: nil, url: "http://schneems.com/ut-rails", ... ]

So looks like it worked, to see the output you can call `.to_sql` on the method

    $ rails console
    > Link.search(:title => "rails").to_sql
    => SELECT "links".*, ts_rank(to_tsvector('english', "links"."title"::text), to_tsquery('english', 'rails'::text))
      AS "rank0.19344968583449806" FROM "links"  WHERE (to_tsvector('english', "links"."title"::text) @@ to_tsquery(
       'english', 'rails'::text)) ORDER BY "rank0.19344968583449806" DESC

As you can see from the query there is quite a bit going on here, but at the end of the day the `texticle` gem is using SQL that Postgres can read in and output a search. Don't expect this sql to work on another RDBMS like mysql, it is custom to Postgres. Back in the day people used to strive for cross database compliance on websites (the ability to switch between mysql and postgresql for example) but most people have realized that switching once a site has already been deployed to production is fairly un-common, and also these custom features are awesome. Postgres has some amazing custom features like the key-value column type [hstore](http://schneems.com/post/19298469372/you-got-nosql-in-my-postgres-using-hstore-in-rails).

If you're not convinced that using full-text search with Postgres (through the texticle gem). It can do many advanced things, one of the most useful is text stemming. Let's say we've got a link:

    > Link.create(:title => "the home of richard schneeman", :url => "http://schneems.com")

If you were to search for `schneemans`, which means snowmen in German, in google you would expect to find this link, but we can't with a `like` query:

    > Link.where("title like '%schneemans%'")
    ==> []

Now if you try the search method, postgres will stem `schneemans` and return a link that contains a common lexical root `schneeman` in a word:

    > Link.search(:title => "schneemans")
    => [#<Link id: 4, user_id: nil, url: "http://schneems.com" ... ]

Wunderbar! It worked just like we wanted it to. Make note though that there is quite a bit of configuration available, you should check out the gem's documentation and take a look at the [postgres documentation for full text search](http://www.postgresql.org/docs/8.3/static/textsearch-intro.html). Once you've got the search running you'll want to make sure it runs fast. We can add an index to speed up search.


## Search, All the Things!

Now that we've got search let's build a user story around a search feature.

    "A user visits our site, enters a search for 'Internet homepage' and is directed to a page that includes a link to 'reddit.com'"

From this description we can see that we will need a search box, a controller action to handle any searching logic, and then a view to display the results. Let's make a controller & routes first. Generally rails controllers are plural while models should always be singular. In this case though, we'll make an exception and make our controller the `search_controller` you can run:

    $ rails g controller search

Or I just prefer to build the file manually under `app/controllers`. Once you've got a controller you a choice or two to make, what action should we use to issue our search? Should we make a custom action called `search` ? Can we use one of our "RESTful" routes? What url should it be at?

Before you pop a blood-vessle thinking about all of that, ask what you as a user would expect. Then ask what you as a developer would expect if you had never seen this project before. As a user I guess I would expect a search to direct me to a `/search` url. As a developer I expect no custom named actions unless there is a good reason. So let's look at our potential "RESTful" actions and see if we can make one fit.

    show   # displays a singular item i.e. /users/1 would display only one user
    index  # displays lists of things i.e. /users would display all users
    new    # holds the form for a create action
    create # makes a new item
    edit   # holds the form for the update action
    update # modifies an existing item
    delete # destroys an existing item

If it makes sense to use one of these routes it will help us and other developers down the road since we are following some kind of standard. It also helps us know what type of HTTP action can access the method (GET, PUT, POST, ...) by convention. I've seen far too many projects littered with too many custom methods in their controllers.

I once worked on a project that had a controller with 64 methods. SIXTY -- FOUR. That's beyond acceptable. The problem is when you add one custom named method, two doesn't seem so bad. When you have two custom named methods, three doesn't seem so bad ... The cycle continues. The default RESTful routes already give us 7, most of the time you can make do with those. I say 8 or fewer public methods per controller (including: show, index, new, create, edit, update, delete) for a happy project. If you need more methods you might really just need a new controller. Think about it.

Back to the task at hand. Looking at all those actions delete, update, edit, create, and new are all out since they modify data, and searching shouldn't modify your database. You might be tempted to say we are "creating" a search, but since the search term isn't being saved to a database to be accessed later I would say "no" to that interpretation. That leaves us with `show` and `index` my two favorite controller methods.

When we search will we be returning one or more values? While our user story is a bit vauge, most people expect to see a list of (multiple) values after they search, so I would say that an `index` action is more appropriate.

Now that we've finished and decided to paint our bikeshed blue (it's a programmer thing, look up "bikeshedding" if you're not familiar) lets get to work. Add a route to your `config/routes.rb`

    get 'search' => 'search#index'

And add an index method to `app/controllers/search_controller.rb`

    def index
    end

Finally make a new view file in `app/views/search/index.html.erb`. It might be tempting to re-use your home-page since it's already displaying a list of links, but I would rather start off writing duplicate view code and then optimize later by refactoring to just one view instead of trying to shoe-horn two different view features into one another. This doesn't mean it is a bad idea, just that it might not be until you're 5 nested if statements deep into your one template that you realize you really needed two templates. Most gross view code i've seen is the result of premature "optimization". While Rails incourages you to be DRY (don't repeat yourself) there is a limit to this motto, as a rubyist once said you want to stay DRY but just don't get chaffed.

Now that we've got the basics we can add a search form. This is the first form in this entire course that hasn't been backed by a model. In the past we've used `form_for` and always passed in an object such as `@user` or `@link` so how do we build a form using rails without an object from our database?

If you googled the question you'll find that there is another view helper called `form_tag` we can use. It accepts a path as its first argument, in this case we want it to go to our `search_path` (SearchController#index). Then since we're not using a `form_for` we will have to use tag helpers to generate our form fields. So instead of `f.text_field` we will use `text_field_tag`. The final product would look like this:

    <%= form_tag(search_path, :method => "get", :style => "margin: 5px 0 0 0") do %>
      <%= text_field_tag(:q, nil, :placeholder => "search") %>
      <%= submit_tag("Search") %>
    <% end %>


It is very important that you include the `:method => "get"` in this form helper otherwise the default of forms it to submit POST requests. If you're confused by the difference between `form_for` and `form_tag`, don't worry I was confused for a really long time. I just try to remember that `form_for` is super exclusive and requires a database object while kids playing `tag` are happy to let anyone join in on the fun. It might not be the best way to remember, but when in doubt look at the docs, especially the [Rails form guides](http://guides.rubyonrails.org/form_helpers.html).

Add the above code to your layout so a user can search from anywhere on your site. Refresh your page and put in a search term like "foo". Hit enter and take a look at your logs. You should see something like this:

    Started GET "/search?utf8=%E2%9C%93&q=foo&commit=Search" for 127.0.0.1 at 2012-10-03 21:54:45 -0400
    Processing by SearchController#index as HTML
      Parameters: {"utf8"=>"✓", "q"=>"foo", "commit"=>"Search"}
      Rendered search/index.html.erb within layouts/application (0.3ms)
    Completed 200 OK in 9ms (Views: 8.0ms | ActiveRecord: 0.0ms)

Here you'll see that we hit the right controller and action and that our search term came through under the param "q". Why is that? If you'll remember in our search form we had a `text_field` using the `:q` symbol like so:

    <%= text_field_tag(:q) %>

**Note:** The "nil" (refers to the default value) and ":placeholder => 'search'" is optional

This maps to a top level param of `:q` being sent to the server. While you don't have to use `q` you could use "query", "search_term", or "anything_you_want"; though `q` is short and fairly common for legacy reasons in search actions on websites. So why break with tradition?

Now that our form works, we'll want to finish writing our controller action and view. Since we know we're sending the query over in `params[:q]` it would be pretty easy to search our database like this:

    @links = Link.search(:title => params[:q])

Now that we've got an array of links let's add them to our view:

    Here are all the links that matched "<%= params[:q] %>"
    <ul>
      <% @links.each do |link| %>
        <li>
          Votes: <%= link.votes.where(:up => true).count - link.votes.where(:up => false).count %>

          <% if current_user && current_user.votes.where(:link_id => link.id, :up => true).present? %>
          *
          <% else %>
            <%= link_to "+", votes_path(:vote => {:link_id => link.id, :up => true}), :method => :post %>
          <% end %>


          <% if current_user && current_user.votes.where(:link_id => link.id, :up => false).present? %>
            *
          <% else %>
              <%= link_to "-", votes_path(:vote => {:link_id => link.id, :up => false}), :method => :post %>
          <% end %>

          <%= link_to link.title, link.url %>
          <%= link_to "comments", link %>
        </li>
      <% end %>
    </ul>


Save your controller and your index and search for something that should be in your database. You can add a fresh link if you like:

    $ rails c
    > Link.create(:title => "Homepage of the Internet", :url => "http://reddit.com")

Then search for it with something like "internets homepage" you should see your link. You can play around with different variations and you'll see that Postgres isn't fool-proof. For example searching for "home page of the internet" got me nothing. Search is an inherently hard problem and while humans are great at picking out similar or "fuzzy" matches, machines need all the help they can get. You'll find for any generic searching storage solution there will be quite a bit of configuration, if your site depends largely on search, take some time researching your options and once you've picked a storage solution, spend some time researching how to optimize and configure it. For us the default configuration should work just fine, besides we can't make the search "too good" otherwise it wouldn't be a true reddit clone (reddit search is famously horrible).

Now that search works you want to make sure that user's don't get overwhelmed with links and that your server isn't crushed by pulling too many entries from your database. You can do this by adding pagination. Since the `search` method returns and `ActiveRecord::Relation` we can just use methods from the `will_paginate` library that we used in the PagesController.


        @links = Link.search(:title => params[:q]).page(params[:page]).per_page(20)

Then you just need to add the will_paginate view helper and you should be good to go! Once you've got that working and committed, let's make sure we don't accidentally break our site. Lest we end up having to show our users this:

![reddit is down](http://f.cl.ly/items/2F0Z1h3f2e2l2s1h0c1X/Screen%20Shot%202012-10-27%20at%2010.52.31%20AM.png)

## Testing

The single best thing you can do to improve the quality of code you write, is to test it. Tests have some negative connotation in society. Being "put to the test" isn't exactly a good thing, who wants their patience tested, and no one celebrates when a teacher announces "big test tomorrow!". The testing we do in programming is different, failing a test is just as important as passing. What exactly is a test, and why to they help? Let's take a step back and consider our morning routines.

When you wake up in the morning you likely have a routine that might involve brushing your teeth. You don't know it but while you're doing this task you're constantly running hundreds of small tests in your head to make sure things are going as planned. When you pick up the toothpaste, you might check the label for a certain slogan, or wait for a hint of minty freshness to waft through the air before you put it on a brush and stick it in your mouth. This behavior is natural, almost instinctual, and is certainly a good thing. If you didn't perform these seemingly pointless checks, you might one day find yourself with a mouth full of hair creme, or athlete's foot medication. As you drive, you check your speed, as you write you check your spelling. It is only natural that when you code you check your code. Unlike driving a car or brushing your teeth, coding is unique in that we can write automated tests that perform these checks for us faster and more consistently than we can.

What might you want to test? Does your app rely on new users signing up? Perhaps you should test that. What about accepting a payment or completing a sale, you should definitely check that. Take a step back and consider the elements that are important to your application and then make sure you don't get the digital equivalent of a mouth full of foot creme by breaking signup and not realizing it for several days.

Tests are great for verifying that the code we write is actually solving the problem we want it to solve. Even better it serves as a living documentation of the intent of the code. If you see some gnarly code someone else wrote and want to fix it, how do you know you got it right? If that code is tested, go ahead and make your changes, run the tests to see if they still pass and you're good to go. If you get a failure, not to worry, the test can help guide you on what is missing. Tests can be like living specification over what the code is _supposed_ to do.

Enough with the talk, let's dive into writing a test. We can use our user stories to help us test, if you go back to the first one we made:

    "A user visits the home page of the website, they then log in. They click a "submit link"
     button and enter a url and a title. They submit that link and title and then end up on a
     page that shows the title, link, votes for that link and any comments."

It's likely when you coded this feature you manually tested it to make sure it was working. It's always good to verify manually, but let's add an automated test so that no matter what we change, we can always verify this feature is still working like we expect.


## Integration Testing with Capybara

To accomplish this type of test we will be using an "integration" testing tool called Capybara. The reason this testing is "integration" is we are testing all of our components (as they are integrated) together: MVC(r). Capybara can actually run a headless version of a browser and then simulate different user actions, such as filling out forms and clicking on links.

Add capybara to your gemfile:

    group :test do
      gem 'database_cleaner'
      gem 'capybara', '~> 1.1.2'
    end

We'll also need to use the `database_cleaner` gem, you can read the [Capybara docs](https://github.com/jnicklas/capybara) for more info. Run `$ bundle install` and then add this code to the end of your `test/test_helper.rb` file:


    require 'capybara/rails'

    # Transactional fixtures do not work with Selenium tests, because Capybara
    # uses a separate server thread, which the transactions would be hidden
    # from. We hence use DatabaseCleaner to truncate our test database.
    DatabaseCleaner.strategy = :truncation

    class ActionDispatch::IntegrationTest
      # Make the Capybara DSL available in all integration tests
      include Capybara::DSL

      # Stop ActiveRecord from wrapping tests in transactions
      self.use_transactional_fixtures = false

      teardown do
        DatabaseCleaner.clean       # Truncate the database
        Capybara.reset_sessions!    # Forget the (simulated) browser state
        Capybara.use_default_driver # Revert Capybara.current_driver to Capybara.default_driver
      end
    end

You can use Capybara with a number of different testing frameworks, we will use the built in framework that Rails provides. Even if you chose to use a different framework later, the concepts are very similar.

Create a `test/integration` folder if it does not already exist. This is where our Capybara tests will live. They don't have to go in that folder, but it helps if we are all on the same page. We are going to test our user story now.


    "A user visits the home page of the website, they then log in. They click a "submit link"
     button and enter a url and a title. They submit that link and title and then end up on a
     page that shows the title, link, votes for that link and any comments."

We have a few distinct steps, but over all we are going to test the creation of links. We want the file name of our test to reflect this so we can name it `link_create_test.rb` making sure that it ends in `_test.rb` this is how Rails (and `Test::Unit`) knows it is a test. Inside of that test add this skeleton code:


    require 'test_helper'

    class LinkCreateTest < ActionDispatch::IntegrationTest

    end


Notice that we are inheriting from `ActionDispatch::IntegrationTest` which is the class we modified in our `test_helper.rb`. Since we inherit from this class we can define tests by adding a test block that looks like this:

    test "the world is round" do
      assert true
    end

Without adding any more code, go ahead and run your tests. You can do this in a number of ways, you could run:


    $ rake test

You may need to add `bundle exec` before `rake test`. You should get an output similar to this:

    # Running tests:

    Finished tests in 0.004353s, 0.0000 tests/s, 0.0000 assertions/s.

    0 tests, 0 assertions, 0 failures, 0 errors, 0 skips
    Run options:

    # Running tests:

    Finished tests in 0.002594s, 0.0000 tests/s, 0.0000 assertions/s.

    0 tests, 0 assertions, 0 failures, 0 errors, 0 skips
    Run options:

    # Running tests:

    .

    Finished tests in 0.037623s, 26.5795 tests/s, 26.5795 assertions/s.

    1 tests, 1 assertions, 0 failures, 0 errors, 0 skips


You can see that we had one test, with one assertion and zero failures, errors, or skips. Since testing is so important it is the default rake task, you can get the same response by running:

    $ rake


Congrats, you just wrote a passing test, but it doesn't tell us much, let's have it follow our user story. First we need to log in as a user. Make a new test:


    test "logged in user submits valid links" do
    end



We need to simulate a user creating a link. Since we don't care about the signup process for this test we can just make a user directly in our database, and then walk through the sign in process. First we will create a user with a random unique email:

    user = User.create(:email => "#{Time.now.to_f}@example.com", :password => "password")

**Note:** It needs to be a unique email otherwise it won't save to the database and your test will likely fail accidentally

Try running your rake task to make sure you get no errors. Next we need to visit the user sign in page. We can use capybara's `visit` method and pass in a path using our rails routes:

    visit new_user_session_path

You can manually specify a path if you want, like this:

    visit 'users/sign_in'

But it is more brittle in the long run. Try running your rake task to make sure you get no errors.

Next we can verify we are on the page we expect

    assert_equal '/users/sign_in', current_path


or

    assert_equal new_user_session_path, current_path


The code `assert_equal` takes to entries and compares them together if they are equal it will pass the test, otherwise it will fail the test. You may wonder why we didn't just use something like this:


    assert '/users/sign_in' == current_path

The method `assert_equal` knows that you expect the current_path (the second argument) to equal `'/users/sign_in'`. If it does not it will fail and tell you the value of expected versus actual. Using `assert` will only tell you that they did not equal and cannot give you the two different values. Basically `assert_equal` gives you more information and a better test experience in this case.

It is nice to verify and test each step of the way, so we don't get any unexpected results, your code should look like this:

    require 'test_helper'

    class LinkCreateTest < ActionDispatch::IntegrationTest

      test "logged in user submits valid links" do
        user = User.create(:email => "#{Time.now.to_f}@example.com", :password => "password")
        visit new_user_session_path
        assert_equal '/users/sign_in', current_path
      end

    end

Save and run tests, they should be passing, if not fix your test or your code. At any time you want capybara to show  you what it is doing you can add this line in to debug:

    save_and_open_page

Go ahead and try it:


    require 'test_helper'

    class LinkCreateTest < ActionDispatch::IntegrationTest

      test "logged in user submits valid links" do
        user = User.create(:email => "#{Time.now.to_f}@example.com", :password => "password")
        visit new_user_session_path
        assert_equal '/users/sign_in', current_path
        save_and_open_page
      end

    end


The sign in page should show up in your browser, this is a good way to make sure your test is going as planned. Now we are on the sign up page we need to fill in the email and password for our user. We will need to target the text boxes so we can tell capybara where to fill in text, view source on the page and grab the id's of the text input elements. Now we can capybara to fill in the email:

    fill_in 'user_email', :with => user.email


This will target the element with an id of `user_email` and add in the email our user has in the database. Now we need to fill in the password:

    fill_in 'user_password', :with => 'password'

and finally submit the form:

    click_button 'Sign in'

Since a valid sign in will result in a different page and an invalid sign in will return the user to the same page we can add an assertion to make sure the sign in worked


    refute_equal new_user_session_path, current_path

Where we are saying the `current_path` should not be equal to the user sign in page. Your code should now look something like this:

    require 'test_helper'

    class LinkCreateTest < ActionDispatch::IntegrationTest

      test "logged in user submits valid links" do
        user = User.create(:email => "#{Time.now.to_f}@example.com", :password => "password")
        visit new_user_session_path
        assert_equal '/users/sign_in', current_path

        fill_in 'user_email', :with => user.email
        fill_in 'user_password', :with => 'password'

        click_button 'Sign in'
        refute_equal '/users/sign_in', current_path

        save_and_open_page
      end

    end

Save and run the tests. Your tests should pass, and you should now be on the homepage as a logged in user. You now need to visit `new_link_path`, enter a link and a url, and then verify they saved correctly.


first

    visit new_link_path

then

    title = "Random Title: #{Time.now.to_f}"
    fill_in 'link_title', :with => title


finally

    fill_in 'link_url', :with => 'http://schneems.com'
    click_button 'Create Link'

Now we want to verify the page and the content:


    link = Link.last
    assert_equal link_path(link), current_path

    assert has_content?(link.title)
    assert has_content?(link.url)


Save and run your tests, fix them or your code until they pass. You might be tempted to add hundreds of assertions to one test, but don't go overboard just yet. Though this test seems straightforward it might be brittle in very subtle ways, for instance if we change the user sign in path all the assertions to `'/users/sign_in'` would fail. Ideally you want to make the minimum number of assertions required to verify behavior, but beyond that you're just making your tests more brittle. This is a hard balance to achieve, you will get better at it over time the more you write and run your tests. If your test keeps failing even though the feature still works, it is a good indication that it is too brittle. If your test does not fail, but a feature breaks, you likely need to add extra code.


Another thing to consider when writing tests is speed and maintainability. Tests are code too and need to be maintained, duplicating testing logic, or over testing areas of your app will end up with a very large very slow test suite that is difficult to maintain. As you advance in your testing knowledge you will likely come to love testing all sorts of code and maybe even adopt TDD (test driven development). It is important to remember that tests are a means to an end. Tests are there to help you with refactoring, feature stability, and for ensuring interfaces (more on this later) are consistent. If a given test or suite of tests ever starts to impair your ability to run your code, you should refactor or remove those tests.

## Integration is a Good Thing

Some people detest integration tests as they claim they are slow (they aren't) and they are brittle. Brittle means they break even when there is no user facing breakage. If you look over our test code we could introduce a breakage simply by changing our html, by specifying a different element `id` for instance. To this I say: use conventions. Designers should never style using html id's (that's a whole different talk) and if you're a developers should have no reason to change an element's id, so I say those are pretty safe. Other areas like targeting specific text "Sign in" can be brittle, so I often add an id to the to make testing easier `:id => "signUserInButton"`. Even with precautions your tests might suddenly break and give you a false positive. When this happens, just fix it and talk to your team to try to find out why it broke and to come up with some better conventions or training so that it rarely happens in the future.

Integration tests take longer to run each test since it requires spinning up a headless webserver and actually interacting with forms. How on earth could it not be "slow" then? Since you're testing the full stack you can get away with fewer tests, and your tests will be much more "human" like and capable of testing real failures. I've worked on projects obsessed with unit tests (small tests for individual methods) that took 30 minutes or more to run, that did nothing to catch several crippling bugs deployed to production. My gem [oPRO](https://github.com/opro/opro) almost exclusively uses Integration tests, runs in under 2 minutes total and has over 90% code coverage (pretty decent).


## Breaking code


Now that you've got a test, it is important that you run it frequently to check to make sure you didn't accidentally break something. At very least, I run my test suites before deploying to production on [Heroku](http://heroku.com). If i'm working with a team, I run my tests before I push my changes up to my shared repository (usually on Github). Let's experiment with changing our code to break our tests to see what happens. Open up `app/controllers/link_controller.rb` and comment out everything in the `create` action. Now run your tests again and see what happens. When I run this test I get an output like this:

    test_logged_in_user_submits_valid_links(LinkCreateTest) [/test/integration/link_create_test.rb:25]:
    <"/links/980190962"> expected but was
    <"/links">.


This tells me exactly what line my test failed at which happens to be this one:


    assert_equal link_path(link), current_path

It doesn't tell me exactly what is wrong, but it does give me a hint. Since we know everything up to this point worked, we can add a debugging `save_and_open_page` or we could tail our development logs while running the test again:


    $ tail log/test.log


A test will tell you where there is a problem, but it won't always point you to the exact point, see if you can use debugging skills to determine that the problem lies in your `links_controller.rb`.


## Interfaces and Tests

What we just tested here was the interface between your user and the application. What is an interface? It is the point of contact between a human and a technology. A car's interface is a steering wheel and gas pedal. A phone's would be the buttons (or touch screen) and speakers. Just like those physical technologies, our digital ones have interfaces. For our web app, the interface is a browser. More specifically we are using URLs and form elements within a browser to manipulate our application. While we are used to interfaces and technologies being fixed, if you've ever used a hammer before, it's likely you'll still use it the same way a year from now. But digital technologies are slightly different because the interface is just a representation of the underlying application. If we change our html, we can still have the same effective app, but our interface changed. Similarly we can change our app without changing the interface.

So when we use Capybara to navigate to a web page, click on buttons and fill in elements, we aren't really testing our application code directly, but through this web interface. I like testing my web apps like this because it is how users will interact with an app, and if you do something to trivial that would impact a user like accidentally removing a link, integration tests can help you catch the mistake (sometimes). There are other types of tests, you can test controllers by themselves with functional tests where the interface is a controller method such as `create` a HTTP verb such as `POST` and params such as `{:title => "schneems"}`. You can test your routes, and you can also test your models directly using unit tests. After integration tests, unit tests tend to be my favorite, they don't test the interface a user will use, but rather they test coding interfaces. When you define a method name that takes arguments, you are defining an interface for calling the code in that method, if you change the interface (changing the method name for examlple) then you will break other code unless it changes too. As you get more advanced in coding, unit testing can greatly help you cleanly define these "interfaces".


We only tested one of our user stories, but you could easily test all of them. Isolate the behavior you want to test, then move towards the goal step by step as a user would.

It is hard to show just how important tests are in a walk-through like this since there aren't stakes on the line. If you deploy a bug, you don't have hundreds of users that you are letting down. You don't have any co-workers you can upset by breaking features. Just remember that we play like we practice. If you treat these examples like a production quality application, it means when you deploy your baby to the world (wide web) it will be in good, tested, hands.

To put it one way: I know a lot of _bad_ programmers who _don't_ test their code much, but I don't know any programmers that test their code frequently and are still bad. Testing is immediate feedback in code form. If you're going to work on open source Ruby projects, tests are a requirement.


## Does Search Work? Are you sure search works?

Since our site's search is much better, we'll assume that our user's will rely heavily on it and that they would be really bummed if something happened and it quit working. To make sure nothing happens to our precious feature let's write a test for it. Not only will this help us to catch bugs, but it can also help us experiment with different search back ends or configuration if we wanted.

Make a new file `test/integration/search_test.rb`. I typically cheat when making new test files and just copy the contents of another test file if it exists, you could copy `link_create_test.rb` and then make a few changes:

    require 'test_helper'

    class SearchTest < ActionDispatch::IntegrationTest
      test "search for item in database works" do

      end
    end

Now we need to go back to our user story

        "A user visits our site, enters a search for 'Internet homepage' and is directed to a page that
         includes a link to 'reddit.com'"


Simple enough, let's start out by visiting the home page. Go ahead and write the code for that while I grab a drink, remember you can cheat off of your old test file. You can use `save_and_open_page` at the end of the test to verify that you're on the right page if you want.

Now would be a good time to verify that you're using Postgres in your `test` environment:

    test:
      adapter: postgresql
      encoding: utf8
      database: reddit_on_rails_test
      pool: 5
      host: localhost

Also verify that all migrations on the test environment have been run:

    $ bundle exec rake db:migrate RAILS_ENV=test

You can run all the tests by running:

    $ bundle exec rake test

Or just that specific file by running

    $ rake test TEST=test/integration/search_test.rb

Once that works you want to make a link in your database with a given title (you can just use `Link.create` for simplicity). Then you want to target the search field with capybara and enter in a query that is similar but not identical to the title of the link you saved in the database. Then tell capybara to hit the submit button.

Run your test and verify that you see a link on the search page. If you don't, debug using `puts` taking it one step at a time. Did your link save to the database? Are you typing in the right field? If you add that same database entry and search in a `$ rails console` session do you get a return?

Once you're satisfied that your test works correctly you want to make sure that you've got some assert statements. I like to assert the url that I'm currently on. In this situation it also makes sense to look for text on the page that matches the text in our link. Make sure you're not asserting generic content that is always going to be there like `div` or content that might change. A general rule of thumb I use, is to only assert content if it's content I've defined in the specific test. So if i make a user with an email of "captain_rocket_pants29@schneems.com" I could reasonably expect that string to be on their account page with little fear that it would accidentally be there or accidentally be removed.

Run your test and make sure it passes. Try commenting out your `SearchController#index` action and re-run just to make sure that the test fails if you were to accidentally break something. It might seem silly but it is really easy to write a test that isn't actually testing anything. For example if you picked a link titled "Sign In" it would find that text on any page for a logged out user.

Remove your commented out code and make sure all your tests run

    $ bundle exec rake test

Once you're happy with your tests, commit everything to git (if you haven't been doing so already) and we're ready to deploy your reddit competitor to the cloud

## Deploy

Tests and deployment go together like peanut butter and jelly. Run your tests throughout development to help you build the features you want, then run the whole test suite before you deploy to make sure you're not introducing bugs into production. While you won't be able to fully replace manual testing ever, writing automated tests will save you time and help you sleep sound at night. When you do run into production bugs, write tests for them so they never happen again. You can consider practicing [5 whys](http://en.wikipedia.org/wiki/5_Whys) and asking yourself why a test didn't catch the failure previously. Tests might not be magic bullets, but they're certainly ammo to make better products and coders.

Take this opportunity to deploy and try to introduce errors into your own product in production, when you do write a test that would fail if that bug continues to exist in your website. You won't always be able to catch every bug with tests, but you'll catch no bugs with tests if you don't write tests.


## The End of the Beginning

Sad to say, we're done for this exercise. If you feel pretty amazing like you just completed a monumentous task that mere non-programming mortals can only dream of, it's because you should. Take a step back and look at what you've done. You've cloned a good portion of functionality from one of the top sites on the Internet. You mastered votes, conquered user sign-up, demolished front-end design, obliterated full-text search and wrapped it all up in a tested package. Not just anyone could make it this far, you may now call yourself a Rails programmer. As you're congratulating yourself take some time to think to the future and of the past. How did you get here? What pushed you to follow 10 weeks of videos, lectures, quizzes, and exercises? This "Reddit on Rails" exercise is nearly 20,000 words and you read them all. Surely you must have a good reason to put yourself through all of that. Do you have an app idea burning a hole in your pocket? Did you want to make more money, or get a better job? Do you want to change the world through creation and code? Whatever your reason, you're here and you should hold on to that and not stop.

Learning is a life long journey my friends, and programming is no different. Now that you're done with this course try to plan your own next steps. Join a user group. Pick up a Ruby (with or without Rails) programming book like [Metaprogramming Ruby](http://pragprog.com/book/ppmetr/metaprogramming-ruby). Build the product of your dreams, or maybe go through some [other rails tutorials](http://ruby.railstutorial.org/). Don't forget to subscribe to [my blog](http://schneems.com) and follow me on twitter [@schneems](http://twitter.com/schneems). You might also want to subscribe to [reddit.com/r/ruby](http://reddit.com/r/ruby) and [ruby weekly](http://rubyweekly.com). Maybe you've had enough of Ruby and Rails and you want to [dive into front end development](http://www.abookapart.com/) or maybe you fancy yourself an [iPhone programmer](http://www.bignerdranch.com/book/ios_programming_the_big_nerd_ranch_guide_rd_edition_), or perhaps you want to become a ballet dancer.

Whatever it is that moves you, hold onto it and go with it.