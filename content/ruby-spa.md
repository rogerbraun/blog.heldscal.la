
## Our Stack

We will use the following software:

- Rails 4
- Grape
- Pundit
- Roar / Representable
- Devise
- AngularJS

## Why use Rails at all?

We won't be using any Rails controllers or Rails views. So why use Rails at all? Here are some reasons:

- You get a familiar project structure
- ActiveRecord is already configured, complete with migrations
- Testing is easy to configure
- Tons of gems you can use

Now, could we have all this without using Rails? Of course, but there would be a
lot of things you'd need to configure yourself. So, as long as there is no
'Rails for Grape', I think actually using rails and mounting Grape is the best
thing we have.

## Creating our Rails project

The Rails app we'll be creating will be a bit anemic: We don't need the asset
pipleline or views, as all the frontend stuff will be handled by our AngularJS app, and
we also don't need controllers, because we'll use Grape to handle the requests.
We'll also be using RSpec, so we use this magic incantation:

`rails new -T -S -J akasha`

This will skip Test::Unit (-T), Sprockets, the asset pipeline (-S) and all the
Javascript files (take a guess).

## Adding our secret spices

Let's now add all the gems we want to use. Modify your Gemfile and add the following lines:

```
gem 'pundit'
gem 'grape'
gem 'devise'
gem 'roar'

group :development, :test do
  gem 'shoulda'
  gem 'rspec-rails'
end
```

Install them with `bundle install`. After that, we'll also need to run a few generators:

```
rails g pundit:install
rails generate rspec:install
rails generate devise:install
rails generate devise User
```

Devise will generate a migration, so run `rake db:migrate` to get your database set up.
