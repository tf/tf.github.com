---
title: Making Rails 4 and Sinatra share a Session
layout: post
extract: A tale of encrypted cookies.
---

In our latest project, I wanted to restrict access to the
[resque](https://github.com/resque/resque) web interface by reusing
the
[Warden](https://github.com/hassox/warden)/[Devise](https://github.com/plataformatec/devise)/[CanCan](https://github.com/ryanb/cancan)
based authorization stack of the main Rails app.  I quickly found
[some](http://neovintage.blogspot.de/2011/11/mounting-resque-web-in-rails-3-using.html)
[posts](http://labnote.beedesk.com/sinatra-warden-rails-devise)
explaining how to mount the resque Sinatra app side by side with the
Rails app in the `config.ru` file. Still, I always ended up with an
empty `env['rack.session']` inside the Sinatra app.

As it turns out, Rails 4 changes the session middleware to use
encrypted cookies.  It no longer relies on
[`Rack::Session::Cookie`](https://github.com/rack/rack/blob/master/lib/rack/session/cookie.rb#L47),
but uses
[`ActionDispatch::Session::CookieStore`](https://github.com/rails/rails/blob/master/actionpack/lib/action_dispatch/middleware/session/cookie_store.rb#L54)
instead.

Unfortunately, the new `CookieStore` middleware cannot live on its own
since the `ActionDispatch::Cookies::CookieJar` it uses to get cookies
[requires an `action_dispatch.key_generator`](https://github.com/rails/rails/blob/master/actionpack/lib/action_dispatch/middleware/cookies.rb#L219)
to be present on the `env`.  If it is missing, the session cookie
cannot be decrypted leading to silent failure.

`Rails::Application` objects place the key generator on an
[`env_config` hash](https://github.com/rails/rails/blob/master/railties/lib/rails/application.rb#L176),
which is
[merged into the request's env](https://github.com/rails/rails/blob/master/railties/lib/rails/engine.rb#L512)
before processing a request.  To make the Rails session available to
our Sinatra app, we have to insert a middleware which mimics this
behaviour:

    # rails_env_config_middleware.rb 
    class RailsEnvConfigMiddleware
      def initialize(app)
        @app = app
      end
    
      def call(env)
        env.merge!(Rails.application.env_config)
        @app.call(env)
      end
    end
    
The final rack up file then looks like this:

    # config.ru
    require ::File.expand_path('../config/environment',  __FILE__)
    
    map '/' do
      run Rails.application
    end
    
    map '/resque' do
      # This is the important line
      use RailsEnvConfigMiddleware
    
      use ActionDispatch::Session::CookieStore, :key => '_<app_name>_session', 
        :path => '/', :secret => '<secret_key_base>'
    
      use Warden::Manager do |manager|
        manager.failure_app = Pageflow::Application
        manager.default_scope = Devise.default_scope
      end
    
      run AdminResqueServer.new
    end

Now we can access the authenticated user via warden and authorize with
CanCan:

    # admin_resque_server.rb
    require 'resque/server'
    require 'resque_scheduler/server'
    
    class AdminResqueServer < Resque::Server
      before do
        redirect '/admin/login' unless authenticated?
        redirect '/' unless can?(:manage, Resque)
      end
    
      private
    
      def can?(*args)
        Ability.new(user).can?(*args)
      end
    
      def authenticated?
        request.env['warden'].authenticated?
      end
    
      def user
        request.env['warden'].user
      end
    end
    
In our production environment, I faced a final bug.  For some reason the
[`rack-protection`](https://github.com/rkh/rack-protection) middleware
which comes as
[part of](https://github.com/sinatra/sinatra/blob/master/lib/sinatra/base.rb#L1742)
the Sinatra stack kept
[dropping the session](https://github.com/rkh/rack-protection/blob/master/lib/rack/protection/base.rb#L88)
as a reaction to some falsely detected attack.  Deactivating
protection inside the Sinatra app solved the issue for now. 

    class AdminResqueServer < Resque::Server
      disable :protection
      
      ...
    end

This is of course not the desired solution. I'll post an update once I
have figured out the details.

   
