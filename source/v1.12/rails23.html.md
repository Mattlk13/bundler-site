---
title: Using Bundler with Rails 3
---

Rails 2.3 comes with its own gem handling. We're going to
override that and replace it with support for Bundler.

<b>NB:</b> This <i>may</i> work with Rails versions lower than 2.3.
The Bundler team has not tested those versions, and will not provide
support for anyone on Rails older than 2.3, but feel free to try it. :)


## Using Bundler with Rails 2.3

If you don't have a Rails 2.3 app yet, generate one

    $ rails myapp
    $ cd myapp

Insert the following code at the bottom of config/boot.rb,
right above the line `Rails.boot!`

~~~ ruby
class Rails::Boot
  def run
    load_initializer

    Rails::Initializer.class_eval do
      def load_gems
        @bundler_loaded ||= Bundler.require :default, Rails.env
      end
    end

    Rails::Initializer.run(:set_load_path)
  end
end
~~~

Create a new file, <code>config/preinitializer.rb</code>,
and insert the following. That is <code>config</code> <strong>NOT</strong>
<code>config/initializers</code>.

~~~ ruby
begin
  require 'rubygems'
  require 'bundler'
rescue LoadError
  raise "Could not load the bundler gem. Install it with `gem install bundler`."
end

if Gem::Version.new(Bundler::VERSION) <= Gem::Version.new("0.9.24")
  raise RuntimeError, "Your bundler version is too old for Rails 2.3.\n" +
   "Run `gem install bundler` to upgrade."
end

begin
  # Set up load paths for all bundled gems
  ENV["BUNDLE_GEMFILE"] = File.expand_path("../../Gemfile", __FILE__)
  Bundler.setup
rescue Bundler::GemNotFound
  raise RuntimeError, "Bundler couldn't find some gems.\n" +
    "Did you run `bundle install`?"
end
~~~

Get all config.gem declarations from your application, and place
them into the Gemfile. If you have declarations in development.rb,
for instance, place them in a named group. Make sure to include
Rails itself and a default gem source.

~~~ ruby
source 'https://rubygems.org'
gem 'rails', '~> 2.3.5'
gem 'sqlite3-ruby', :require => 'sqlite3'

# bundler requires these gems in all environments
# gem 'nokogiri', '1.4.2'
# gem 'geokit'

group :development do
  # bundler requires these gems in development
  # gem 'rails-footnotes'
end

group :test do
  # bundler requires these gems while running tests
  # gem 'rspec'
  # gem 'faker'
end
~~~

<a class="btn btn-primary" href="/groups.html">Learn More: Groups</a>

Once you have everything set up, you can use script/console,
script/server, and other Rake tasks as usual. From this point
on, you can follow the instructions in the Rails 3 guide

    $ bundle exec rake db:migrate

<a class="btn btn-primary" href="/rails3.html#shared_with_23">Learn More: Rails 3</a>
