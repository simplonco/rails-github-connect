##Github connect [![Build Status](https://travis-ci.org/simplonco/rails-github-connect.svg?branch=master)](https://travis-ci.org/simplonco/rails-github-connect)

###How to install login connect for Github via Omniauth

[Source](railscasts.com/episodes/360-facebook-authentication)

####Gemfile

gem 'omniauth'
gem 'omniauth-github'

####config/initializers/omniauth.rb

```
OmniAuth.config.logger = Rails.logger
Rails.application.config.middleware.use OmniAuth::Builder do
    provider :github, ENV['ID'], ENV['SECRET']
end

```

####Terminal

rails g model user provider uid name oauth_token oauth_expires_at:datetime
rake db:migrate

####models/user.rb 
```
def self.from_omniauth(auth)
    where(auth.slice(:provider, :uid)).first_or_initialize.tap do |user|
        user.provider = auth.provider
        user.uid = auth.uid
        user.name = auth.info.name
        user.oauth_token = auth.credentials.token
        user.oauth_expires_at = Time.at(auth.credentials.expires_at)
        user.save!
    end
end
```

####config/routes.rb 
```
match 'auth/:provider/callback', to: 'sessions#create'
match 'auth/failure', to: redirect('/')
match 'signout', to: 'sessions#destroy', as: 'signout'
```

####Terminal
```
rails g controller sessions
```

####session_controller.rb 
```
	class SessionsController < ApplicationController
		def create
			 user = User.from_omniauth(env) session[:user_id] = user.id redirect_to root_url end def destroy session[:user_id] = nil redirect_to root_url 
		end 
	end
```

####application_controller.rb 
```
    private
    def current_user
        @current_user ||= User.find(session[:user_id]) if session[:user_id]
    end
    helper_method :current_user
```

####layouts/application.html.erb
```
    <div id="user_nav">
        <% if current_user %>
            Signed in as <strong><%= current_user.name %></strong>!
            <%= link_to "Sign out", signout_path, id: "sign_out" %>
        <% else %>
            <%= link_to "Sign in with Github", "/auth/github", id: "sign_in" %>
        <% end %>
    </div>

```

###Optionnel

####app/assets/javascripts/facebook.js.coffee.erb 

```
jQuery ->
    $('body').prepend('<div id="gh-root"></div>')
    $.ajax
        url: "#{window.location.protocol}//connect.facebook.net/en_US/all.js"
        dataType: 'script'
        cache: true
window.GHAsyncInit = ->
    GH.init(appId: '<%= ENV["APP_ID"] %>', cookie: true)
    $('#sign_in').click (e) ->
        e.preventDefault()
        GH.login (response) ->
            window.location = '/auth/facebook/callback' if response.authResponse
    $('#sign_out').click (e) ->
        GH.getLoginStatus (response) ->
            GH.logout() if response.authResponse
        true
```
