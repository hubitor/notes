# Setup
## Rails Generator and Directory system

Rails Generator Source Code: https://github.com/rails/rails/blob/master/railties/lib/rails/generators/app_base.rb

`initializers directory` will run before application sets up, eg: if you wanted to connect to the Google Maps API

`application.rb` controls everything, like a master file

`lib directory` custom rake tasks, custom algorithms etc

### Secret Key Base

`bundle exec rake secret`

```
// config/initializers/secret_token.rb
// check app_name: Rails.application.class.parent_name

YourApp::Application.config.secret_key_base = 'your-secret'
```

## Generators

`rails g controller Pages home about index` will generate the routes, pages, and a pages_controller with those three methods and pages

`rails g scaffold Page name:string` will generate a new model, database table, views and controller, and fill in all CRUD functions
`rails g model Page name:string` will just generate a new model and database table
`rails g resource Pages name:string` will create model, database table, empty controller, and empty views

`rails g migration <AddColumnNameToTableName> columnName:datatype`
`rails g migration <RemoveColumnNameFromTableName> columnName:datatype`

## Customise application.rb
```
// application.rb

module AppName
  class Application < Rails::Application

    # Default generators, default to uuids
    config.generators do |g|
      g.orm :active_record, primary_key_type: :uuid
      g.test_framework :rspec
      g.template_engine :erb
      g.stylesheets false
      g.javascripts false
    end

    # Allows CORS requests
    config.middleware.insert_before 0, Rack::Cors do
      allow do
        origins '*'
        resource '*',
          :headers => :any,
          :methods => [:get, :post, :delete, :put, :patch, :options, :head]
      end
    end

    config.autoload_paths << Rails.root.join('lib')
    config.eager_load_paths << Rails.root.join('lib')

    # Set default timezone
    config.timezone = 'UTC'
  end
end
```

# Coding in Rails
Use `before_action` to set current model for each controller method from params

```
class BlogsController > ApplicationController
  before_action :set_blog, only: [:show]

  def show
  end

  private

  def set_blog
    @blog = Blog.find(params[:id])
  end
end
```

`redirect_to @blog` will redirect to @blog's `show` page, or `blog_path(@blog)`



## Model callbacks

### `before_action`

Use `before_action` to set current model for each controller method from params

```
class BlogsController > ApplicationController
  before_action :set_blog, only: [:show]

  def show
  end

  private

  def set_blog
    @blog ||= Blog.find(params[:id])
  end
end
```

### `redirect_to`
`redirect_to @blog` will redirect to @blog's `show` page, or `blog_path(@blog)`

---

### url(absolute) & path(relative)

`link_to, 'something', something_path` => `/something`

`link_to, 'something', something_url` => `http://localhost:3000/something`

Rails path helper module: `Rails.application.routes.url_helpers`
---

### delete & destroy

Delete will remove the item row in the database and no callbacks will be run.

Destroy will attempt to remove the item, but there are a bunch of callbacks associated with the method. If the `before_destroy` callback throws `:abort` then the action is cancelled and `destroy` returns `false`.

To use delete, pass in the resource path, eg `blog_path`, with the resource as an argument a hash of options that specify the verb `delete`.

Eg: to create a link to delete a blog post:

```
<%= link_to 'Delete Post', post_path(@post), method: :delete, data: { confirm: 'Are you sure? } %>
```
---

### respond_to, respond

Describes what you want to happen after previous action.

```
@blog.destroy

respond_to do |format|
  # respond in different formats
  format.html { redirect_to home_url, notice: 'Blog was destroyed'}
  format.json { key: value }
end
```

---

## Enums

Elegent way to manage changes in state

Create a `status` Enum for `Blog` model.

Going to need a `status` attribute, that will be an Integer, and a default value.

```
rails g migration add_post_status_to_blogs status:integer

def change
  add_colunn :blogs, :status, :integer, default: 0
end

rails db:migrate
```

Create Enum:

```
// blog.rb

enum status: {
  draft: 0,
  published: 1
}
```

This gives you access to some useful functionality:

Firstly, you can see that querying a Blog instance shows the Enum value for it's status:

`Blog.last => #<Blog id: 10, title: "My Blog Post 9", slug: "my-blog-post-9", status: "draft">`

`Blog.last.status => "draft"`

Query the Enum and return a boolean:

`Blog.last.draft? => true`\

Update the Enum with a bang method:

`Blog.last.published!`
```
(0.2ms)  BEGIN
  SQL (9.2ms)  UPDATE "blogs" SET "status" = $1, "updated_at" = $2 WHERE "blogs"."id" = $3  [["status", 0], ["updated_at", "2019-12-01 00:54:46.912725"], ["id", 10]]
(0.4ms)  COMMIT
```
Query the number of Objects with a certain status;

`Blog.draft.count`

```
Blog Load (0.6ms)  SELECT "blogs".* FROM "blogs" WHERE "blogs"."status" = $1  [["status", 0]]
=> 10
```


# Controllers & Routes

## Namespace & Scope

You may wish to organize groups of controllers under a namespace. Most commonly, you might group a number of administrative controllers under an `Admin::` namespace. You would place these controllers under the `app/controllers/admin` directory, and you can group them together in your router:

```
namespace :admin do
  resources :articles, :comments
end
```

This will create a number of routes for each of the `articles` and `comments` controller. For `Admin::ArticlesController`, Rails will create routes prefixed by `/admin`:

`GET	'/admin/articles', to:	'admin/articles#index', as:	'admin_articles_path'`

Note that if you do this, you will need to:
- Create a `controllers/admin/` directory to nest your controllers
- Create a `views/admin/` directory to nest your views
- Prefice your controllers: `Admin::ArticlesController`

If you want to the route to be just `/articles` without the `/admin/` prefix, but still scoped to the `Admin::Articles` controller, use `scope`:

```
scope module: :admin, do
  resources :articles, :comments
end
```

... or for a single case:
```
resources, :articles, module: :admin
```

If you want to route `/admin/articles` to ArticlesController, but without the `Admin::` module prefix, you could use `scope '/admin'`:

```
scope '/admin' do
  resources :articles
end
```

... or for a single case:

```
resources :articles, path: '/admin/articles'
```

Now we can route any request to `admin/articles` directly to the `articles` controller.


## Nested Resources

```
resources :magazines do
  resources :ads
end
```

This should mirror a `belongs_to` `has_many` relationship in your database, and will provide the following routes:

```
GET	'/magazines/:magazine_id/ads', to:	'ads#index'
GET	'/magazines/:magazine_id/ads/:id/edit'	'ads#edit'

// all will route to the ads_controller
```

And will also create url helpers, `magazine_ads_url`, which require an instance of `Magazine` as the first argument: `magazine_ads_url(@magazine)`.


## Member Routes & Collection Routes

Add additional actions to a recourse-based route

|| URL|Helper|Description|
|---|---|---|--------------|
|member | `/resource/:id/action` | `action_resource_path(resource)` | Acts on a specific resource therefore id is required
|collection|`/resources/action` | `actions_resources_path` | Acts on collection of resources, therefore id not required


### Example 1

eg: Given you have `resources :photos` in your `routes.rb` file, say you wanted to add a member route to preview a specific photo, that you could call with `preview_photo_path(@photo)`...
```
// route to a preview method at GET photo/:id

resources :photos do
  member do
    get 'preview'
  end
end

=> preview_photo_path(@photo)
=> localhost:3000/photos/:id/preview
```

Say you wanted to create a route where you could perform a search on a collection of photos ...
```
resources :photos do
  collection do
    get 'search'
  end
end

=> search_photos_path
=> localhost:3000/photos/search
```

### Example 2

Given `resources :user`, say you wanted to define a route that calls a method to toggle status of a user to `:banned`. You would want to access the method at `ban_user_path(@user)`...

```
resources :users do
  member do
    PUT 'ban'
  end
end

=> PUT '/users/:id/ban', to: 'users#ban', as: 'ban_user'
```

... and say you wanted to view a list of banned users

```
resources :users do
  collection do
    GET 'banned'
  end
end

=> GET '/users/banned', to 'users#banned', as: 'banned_users'
```

### Example 3

Given `resources :foo` in your `routes.rb`, say you wanted to add an additional member route ...

```
resources :foo do
  member do
    get 'bar`
  end

  // alternative syntax:
  :member => { :bar => :get }`
end

```
you'd get an additional route of:

`GET    /foo/:id/bar # FooController#bar`

In the same way that RESTful resources for our `FooController` provide a `foo#edit` method for a specific `foo/:id` instance, we now have a new route for exactly the same `food/:id` endpoint, but differentiated by a different controller method:

```
GET    /foo/:id/edit   # FooController#edit

// new route:
GET    /foo/:id/bar    # FooController#bar
```

Collection routes work the same way, but are applied to a non-specific collection of instances

```
resources :foo, { :collection => { :buzz => :get } }
```

.. gives us a new route

`GET    /foo/buzz # FooController#buzz`

## Globbing

`get 'posts/*glob', to: 'controller#method'`

This will route to  `controller#method` for any url that matches `'posts/*'`, so if you have any other paths that you want to route somewhere else with the same `'posts/'` url then make sure to place them *above* the glob in your `routes.rb` file.

# Gems

## Friendly-id

https://github.com/norman/friendly_id

`gem 'friendly_id', '~> 5.2.4'` && `bundle install`

Generate the friendly_id initializer and a new migration:

`rails generate friendly_id`

add a friendly_id slug to Blog:

```
rails g migration AddSlutToBlog slug:uniq
rails db:migrate

// blog.rb
// Configure so that when we save the title for a Blog, friendly_id will create a slug for us automatically.

class User
  extend FriendlyId
  friendly_id title:, use: :slugged
end

// blogs_controller.rb

def BlogsController < ApplicationController
  def set_blog
    @blog ||= Blog.friendly.find(params[:id])
  end
end
```

Add title slug to existing Blogs:

`Blog.find_each(&:save)`

## Sharing data between pages with Rails Sessions

__Do no secure confidential data in sessions, they are not secure!!!__

`Session` is an object, therefore you can set and retrieve session data as you would from hash.

For example, if you could retrieve a referrer id from a query string and use that in your app:

```
# application_controller
before_action :set_referrer

def set_referrer
  session[referrer] = params[:src]
end
```
```
# application.html.erb
<%= yield %>

<% if session[:referrer] %>
  Welcome, thanks for visiting us from <%= session[:referrer] %>
<% end %>
```

# Models & Active Record

## `includes` method

Will collect multiple subsequent queries into a single query. One way of thinking about this is that Active Record will use the inital query for a Model / Collection, and then allow subsequent queries to reference that data rather than having to go back to the database to make a new query.

For example, say you have `Author`, `Book` and `Genre` models.

```
@books = Book.all

@books.each do |book|
  book.title
  book.author.name
  book.author.country
  book.genre.name
end
```
This will result in queries for:
- all `Book` models
- `Author` from `authors` table * `n` times, where `n` is the number of `books`
- `Genre` from `genres` table * `n` times, where `n` is the number of `books`

If you have 3 books, then you will make 7 database queries, one initial query to get all of the `Book` ids, then one query each for `Author` and `Genre` for each of those `books`.

Using `.includes` allows you to make one query for each table, then each subsequent query uses the data from that initial query, meaning that the total number of queries will be the same, no matter how many `books` there are in the database (ie: the number of repetitions in the for loop).

```
@books = Books.includes(:author, :genre)
```

Now our queries are:
- `SELECT "books".* FROM "books"`
- `SELECT "authors".* FROM "authors" WHERE "authors"."id" IN (3, 2, 1)`
- `SELECT "genres".* FROM "genres" WHERE "genres"."id" IN (2, 3, 5)`

Note that we are still fetching multiple `authors` and `genres`, but we are fetching only the resources we need, and in a single query for each model.

## Model Callbacks

`after_initialize`

Runs after an Active Record Object is instantiated, eg: when `Model.new` is called, or when a record is loaded from the database.

eg:

```
class User
  after_initialize :set_defaults

  private

  def set_defaults
    self.profile_image ||= "http://placehold.it/600x400",
  end

end
```

## Database Relationships

Thinking about foreign keys: Which Model should 'own' the other, in other words which Model inserts its `id` into the other's table?

Blog Post
- title: Baseball World Series, topic_id: 1
- title: Superbowl, topic_id: 2
- title: Training, topic_id: 1

Topic Model
- id: baseball
- id: NFL

Trying to do this the other way around (where there is a blog id on each topic) won't work because each topic should be able to have multiple blog IDs.

Therefore this should be a 'Topic has_many Blog Posts' relationship

Therefore __we should have a Topic reference on Blog Post__.

`Rails g migration AddTopicsToBlogs topic:references` => ( Add to Blogs the TopicId )

```
def change
  add_reference :blogs, :topic, foreign_key: true
end
```

### Relationship: `has_many :A, through: :B`

If two tables share a common association, eg: an `Author` has `:books`, and a `Genre` has `:books`, then you can say that:
- an `Author` has `:genres`, through: `:books`
- and a `Genre` has `:authors`, through: `:books`

Therefore:

```
# author.rb

has_many :books, inverse_of :author
has_many :genres, through: :books
```
```
# genre.rb

has_many :books, inverse_of :genre
has_many :authors, through: :books
```

Now we can call:
```
author.genres
genre.authors
```

We can also create a join table where we want straight many-to-many relationships:

```
# user.rb

has_many :user_skills, inverse_of :user
has_many :skills, through: :user_skills
```
```
# skill.rb

has_many :user_skills, inverse_of: :skill
has_many :users, through: :user_skills
```
```
# user_skill.rb

belongs_to :user, inverse_of: :user_skills
belongs_to :skill, inverse_of: :user_skills
```

## Custom Queries

Benefit: You should try to keep all the logic for your Models in the Model file, rather than having SQL / Active Record queries in your controllers. The controller should only realy manage data flow.

#### Option 1: Create a Model class method

```
class Person
  def self.boys
    where(gender: 'male')
  end
end

=> can call with `Person.boys`
```

#### Option 2: Define a Scope

```
scope :ruby_on_rails -> { where(topic: 'Ruby on rails') }

=> can call with `Blog.ruby_on_rails`
```

## Concerns

Since they are in the `/models` directory, concerns should deal with data. A module that doesn't deal with data should most likely go in the `/lib` directory.


### included do block

The `included do` hook is called when you `include Module` into a class, even before instantiating an Object from that class. It is used for defining relations, scopes, validations etc pertinent to the class.


<!-- https://stackoverflow.com/questions/28009772/ruby-modules-included-do-end-block -->

```
module MyModule
  extend ActiveSupport::Concern

  // when someone includes this module, it will have these two methods exposed as instance methods

  def first_method
  end

  def second_method
  end

  // ... this included hook will be called

  included do
    // ... and this method will be executed

    second_class_method
  end

  module ClassMethods
    // ... and these two methods exposed as class methods
    // ... these class methods will be mixed as class methods of the Class this module is included in

    def first_class_method
    end

    def second_class_method
    end
  end
end

class MyClass
  include MyModule
end
```

The methods of `ClassMethods` are automatically mixed as class methods of `MyClass`. This is a common Ruby pattern, that `ActiveSupport::Concern` encapsulates. The non-Rails Ruby code is...

```
module MyModule
  def self.included(base)
    base.extend ClassMethods
  end

  module ClassMethods
    def this_is_a_class_method
    end
  end
end

```

...Therefore you can call `MyClass.first_class_method`

The `included do` hook is effectively the following code:

```
# non-Rails version
module MyModule
  def self.included(base)
    base.class_eval do
      # somecode
    end
  end
end

# Rails version with ActiveSupport::Concerns
module MyModule
  included do
    # somecode
  end
end
```

### Controller Concerns: Add before_filter devise params sanitizer:

```
# controllers/concerns/devise_whitelist.rb

module DeviseWhitelist
  extend ActiveSupport::Concern

  included do
    before_filter :configure_permitted_params if devise_controller?
  end

  private

  def configure_permitted_params
    devise_paramaeter_sanitizer.permit(:sign_up, keys: [:name])
  emd
end
```
```
# application_controller.rb
```


## Nested Attributes

Refers to a way to save attributes on associated models __through__ the parent model with the `accepts_nested_attributes_for` class method.

Consider the following relation:

```
class User
  has_one :avatar
  accepts_nested_attributes_for :avatar
end
```

You can save attributes (ie: create new instances of both Objects) trough nested attributes:

```
params = { user: { name: "Josh", avatar_attributes: { image: "image.png"}}}
User.create(params['user'])
```

Active record may have trouble recognising the inverse model associations, in which case you'll need to declare the `inverse_of`.

### `reject_if` validation

```
class Member
  has_many :posts, inverse_of: :member
  accepts_nested_attributes_for :posts, reject_if proc: {|attrs| attrs['title'].blank? }
end
```
```
// when creating multiple nested attributes, pass an array of hashes:

params = { member: { name: "Josh", posts_attributes: [
  title: 'Some Title',
  title: 'Some Other Title',
  title: ''
]}}
```
```
new_member = Member.create(params['member'])

new_member.posts.count
=> 2
```

### Strong params with `nested_attributes_for`

```
def portfolio_params
  params.require(:portfolio).permit(:title, :subtitle, :body, technologies_attributes: [:name])
end
```

# Authorization - devise

### Installation

`gem devise`

`rails generate devise:install`

`rails g devise:views`

`rails g devise User`

### Customise routes

```
devise_for :users, path: '', path_names: { sign_in: 'login', sign_up: 'register', sign_out: 'logout' }
```



# Forms

Set the default cursor starting point for the form

`autofocus: true`


# Views

## Layouts - Multiple Layouts

To configure styling for different layouts:
- `stylesheet_link_tag` must correspond with file from `stylesheets/` directory
- Make sure to remove the `require tree .` from `application.scss`.
- Add `layout "layout_file_name"` to the corresponding controller
- Precompile assets: Add `Rails.application.config.assets.precompile += %w( blogs.css )` to `config/initializers/assets.rb`

## Layouts - `<head>`

### `stylesheet_link_tag`

Specify which stylesheet to use. Default path is `app/assets/`

`<%= stylesheet_link_tag 'application', media: all, 'data-turbolinks-track': 'reload' %>`

Use a CDN:

`<%= stylesheet_link_tag 'CDN_URL'>`

### Customising Page Title

You can define and render a `@page_tite` in your `layouts/application.html.erb` file, and override it depending on the controller method:

```
# layouts/application.html.erb

<title><%= @page_title %></title>
```
```
# blogs_controller.rb

def show
  @page_title = @blog.title
end
```

### Meta SEO keywords

```
<meta name="keywords" content="my keywords here" />
```

## Layouts - Partials

### Basic Partials

You can pass an object / data to a partial locally (using 'locals'), since the partial is rendered by the view itself.

```
# blogs_controller

def new
  @blog = Blog.new
end
```
```
# views/blogs/new.html.erb

<%= render 'shared/form', blog: @blog %>
```
```
# views/shared/_form.html.erb

<%= form_for(blog) do |f| %>
```

You can also dynamically create classes using locals. For example, pass in the top/bottom location of a navbar to different layout files:

```
# application.html.erb

<%= render 'nav', location: 'page-top' %>
```
```
# _nav.html.erb

<div class="<%= location %>">Navbar</div>
```

### Partials for Data Collections

Rails can render collections by default if you follow this convention:
- Use the `render` method in the view and pass in the plural collection you want to render
- Create a partial named after the singular of the collection
- Must match the name of the controller / model

```
# blogs/index.html.erb

<%= render @blogs %>
```
```
# blogs/_blog.html.erb

<%= blog.title %>
<%= blog.subtitle %>
<%= blog.body %>
```
```
# blogs_controller

def index
  @blogs = Blog.include(:title, :subtitle, :body)
end
```

### Custom Partials for Data Collections

Since Rails maps partials to the current directory, the collection & model name, and the partial file.
If you want to render a collection that doesn't follow that convention you have pass in custom options:

For example, if you have Model `Portfolio`, and you are rendering `@portfolio_items`:
```
# portfolios_controller

def index
  @portfolio_items = Portfolio.all
end
```
```
# portfolios/index.html.erb

<%= render partial: 'portfolio_item', collection: @portfolio_items %>
```
```
# _portfolio_item.html.erb

<%= portfolio_item.title %>
<%= portfolio_item.body %>
```

`partial: 'portfolio_item'` refers to the name of the partial. If the partial is in a different directory then you need to specify the path.

`collection: @portfolio_items` is where you pass in the data collection to render


## View Helpers / Application Helpers

Best to use them when you find yourself adding ruby logic to a layout or view.
Extract that logic out into a method and add to Application Helper Module.

As a rule of thumb, extract HTML to partials, ruby logic (case statements etc) to view helpers.

### Basic view helper

```
# app/helpers/application_helper.rb

def login_helper
  if current_user.is_a?(User)
    lienk_to 'Logout', destroy_user_session_path, method: :delete
  else
    link_to 'Register', new_user_registration_path
      link_to 'Login', new_user_session_path
    <% end %>
end
```
```
# application_hmtl.erb

<%= login_helper %>
```

### with `content_tag`

`content_tag` are very useful for programatically returning HTML, classes, content that would typically clutter up a rails view file.

You can pass in an argument (from the view page) for additional configuration.

```
# application_helper.rb

def source_helper(page)
  message = "Welcome. Thanks for visiting us from #{session[:source]}. You are on the #{page} page."
  content_tag(:p, message, class: 'source-message') if session[:source]
end
```

```
# application.html.erb

<%= source_helper("main") %>
```

### Partials Spacer Template

Create a spacing template called `blog_ruler` for the `_blog.html.erb` partial.

Will not be applied to the last item in the collection.

This only works if you define `partial: @blogs`.

```
# blogs/index.html.erb

<%= render partial: @blogs, spacing_template: 'blog_ruler' %>
```
```
# blog_ruler.html.erb

<hr class="blog-spacing">
```

## Other

### cache do

Will cache html, css, assets on the client browser side to speed up subsequent rendering.
Do not use caching if you need the client to be able to update the page and see different data.

```
<% cache do %>
  <div><%= render @blogs %></div>
<% end %>