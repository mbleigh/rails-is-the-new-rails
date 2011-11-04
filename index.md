# Rails is the new Rails - Michael Bleigh - Ruby Midwest 2011

!SLIDE

# <span style="font-weight:300; font-size: 80%;">Rails is the new Rails <br/> <small>Michael Bleigh</small></span>

## Ruby Midwest 2011

!SLIDE

# [@mbleigh](http://twitter.com/mbleigh)

!SLIDE

# [@intridea](http://twitter.com/intridea)

!SLIDE

## Follow Along

# <a href='http://mbleigh.com/rails-is-the-new-rails' style='font-weight: 300; font-size: 80%;'>mbleigh.com/rails-is-the-new-rails</a>

!SLIDE

# Not a Cool Talk

!SLIDE

# Just a Bunch of Tips

!SLIDE

# Rails Safari?

!SLIDE

# 3 Minute Philosophy

!SLIDE

# Time is Sanity

!SLIDE

# Story Time

!SLIDE

# Always Be Optimizing

!SLIDE

# On to the Code!

!SLIDE theme

# The Router

!SLIDE tip

# Routing Constraints

!SLIDE

## Routes for Logged In Users

```ruby
# config/routes.rb
constraints User do
  resource :account
  resources :friends
end

# app/models/user.rb
class User < ActiveRecord::Base
  def self.matches(request)
    request.session[:user_id].present?
  end
end
```

!SLIDE

## Routes for Subdomained Accounts

```ruby
# config/routes.rb
constraints Account do
  resources :users
end

# app/models/account.rb
class Account < ActiveRecord::Base
  def self.matches(request)
    Account.find_by_subdomain(request.subdomain).present?
  end
end
```

!SLIDE

## Reversal Syntactic Sugar

```ruby
module ReverseConstraint
  extend ActiveSupport::Concern

  module ClassMethods
    def not(constraint)
      lambda{|request| !self.matches(request)}
    end
  end
end

# app/models/user.rb

class User < ActiveRecord::Base
  include ReverseConstraint

  def matches(request)
    request.session[:user_id].present?
  end
end

# config/routes.rb

constraints User.not do
  root to: 'splash#show'
end
```

!SLIDE

# Considered Harmful? Maybe.

!SLIDE tip

# RegExp Params

!SLIDE

## GitHub-Style Routing?

```ruby
  # config/routes.rb
  resources :repos, path: '', id: /[a-z0-9]+\/[a-z0-9]+/i do
    resources :issues
    resource :wiki
  end
```

!SLIDE theme

# Application Config

!SLIDE tip

# Customize Generators

!SLIDE

## Disable Helper Generation

```ruby
# config/application.rb
config.generators.helper = false
```

!SLIDE

## Switch SCSS Style to SASS

```ruby
# config/application.rb
config.sass.preferred_syntax = :sass if config.sass
```

!SLIDE theme

# ActiveRecord

!SLIDE tip

# Migration Shortcuts

!SLIDE

## Better Associations in Migrations

```ruby
# in your migration...
def change
  create_table :comments do |t|
    t.text :body
    t.belongs_to :user
    t.belongs_to :commentable, :polymorphic => true
  end
end
```

* Indexes are created automatically.
* Both columns for polymorphic tables are created.

!SLIDE

## Migration Generator Shorthand

```sh
rails g model comment body:text user:belongs_to
```

You can do this when generating migrations to add columns:

```sh
rails g migration add_spam_to_comments spam:boolean
```

!SLIDE tip

# Mass Assignment Roles

!SLIDE

## Scope Your Assignment Rules

```ruby
# app/models/user.rb
class User < ActiveRecord::Base
  attr_protected :verified
end

# app/controllers/users_controller.rb
def update
  # won't update the "verified" column
  @user.update_attributes(params[:user])
end

# app/controllers/admin/users_controller.rb
def update
  # will update the "verified" column
  @user.update_attributes(params[:user], as: :admin)
end
```

!SLIDE tip

# Touching

!SLIDE

## Touching for Cache Keys

```ruby
# app/models/post.rb
class Post < ActiveRecord::Base
  belongs_to :section, touch: true
end
```

```html
<!-- app/views/sections/index.html.erb -->
<% for section in @sections %>
  <% cache "section/#{section.id}/#{section.updated_at}" do %>
    <%= render partial: "posts/post", collection: section.posts %>
  <% end %>
<% end %>
```

!SLIDE theme

# Action Pack

!SLIDE tip

# Superfast HTTP Basic

!SLIDE

## Beta Version Guard

```ruby
class ApplicationController < ActionController::Base
  http_basic_authenticate_with name: ENV['BETA_USER'], password: ENV['BETA_PASS']
end
```

!SLIDE tip

# HTML5 Data Tags

!SLIDE

## Embed Data in the DOM

```html
<%= content_tag :span, 
  :data => @user.attributes.select{|k,v| %w(id name avatar).include?(k) } do %>
  <%= @user.nickname %>
<% end %>
```

!SLIDE tip

# Actions as Rack Endpoints

!SLIDE

## Use With Middleware

```ruby
# config/initializers/some_middleware.rb
Rails.application.config.middleware.use SomeMiddleware, 
  on_error: ErrorsController.action(:show)
```

!SLIDE tip

# Capturing Output

!SLIDE

```ruby
# app/helpers/shouting_helper.rb
module ShoutingHelper
  def shout(model, &block)
    with_output_buffer(&block).upcase.html_safe
  end
end
```

```html
<%= shout do %>
  <p>Why speak in normal tones when you can shout?</p>
<% end %>
```

!SLIDE theme

# Bonus Tips!

!SLIDE tip

# ActiveSupport::Concern

!SLIDE

```ruby
module MyCoolModule
  def self.included(base)
    base.extend ClassMethods
    base.send :include, InstanceMethods

    base.class_eval do
      # some_code
    end

    module ClassMethods; end
    module InstanceMethods; end
  end
end
```

Using `ActiveSupport::Concern`, this becomes:

```ruby
module MyCoolModule
  extend ActiveSupport::Concern

  included do
    #some_code
  end

  module ClassMethods; end
  module InstanceMethods; end
end
```

!SLIDE tip

# Personal Asset Gem

!SLIDE

## The Setup

```sh
bundle gem mycompany-assets
```

```ruby
# mycompany-assets.gemspec
gem.add_dependency 'rails', '~> 3.1.1'

# lib/mycompany-assets.rb
module Mycompany
  class Engine < Rails::Engine
  end
end
```

Now put all of your favorite Javascript/CSS plugins and hacks into
`vendor/assets` and push it to GitHub.

!SLIDE

## The Payoff

In a fresh Rails app:

```ruby
# Gemfile
gem 'mycompany-assets', git: 'git://github.com/mycompany/mycompany-assets.git'
```

```javascript
# app/assets/javascripts/application.js
//= require modernizr
//= require slider
```

!SLIDE tip

# Stdlib Timeouts

!SLIDE

## Keeping in Control

```ruby
  require 'timeout'

  begin
    Timeout::timeout(5) do
      result = SomeLibrary.call_without_timeout_options
    end
  rescue Timeout::Error
    Rails.logger.error "SomeLibrary service call timed out.
  end
```

!SLIDE tip

# Fun With Times

!SLIDE

## Enumerators are Useful, ActiveSupport is Fun!

```ruby
5.times.to_a # => [0,1,2,3,4]

WORDS = %w(a bunch of random words)
def random_word
  WORDS.sort_by{ rand }.first
end

def random_sentence(length = (5 + rand(8)))
  words = length.times.map{ random_word }
  words.first.capitalize!
  words.join(' ') + '.'
end
```

!SLIDE

# What's Your Optimization?

!SLIDE

# Thanks!
