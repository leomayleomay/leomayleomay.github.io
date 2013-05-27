---
layout: post
title: "Fancy login with red light"
date: 2013-05-27 17:53
comments: true
categories: OSS ruby rails
---


I am not saying you are wrong if you are still redirecting the visitor to http://your_killer_app/login to input his/her credentials before letting him/her access the authentication required resource. I am just introducing another way of showing login page to you, the fancy way.

It's fancy because it detects the links and forms requiring authencation in a fancy way. I am going to use [Devise](https://github.com/plataformatec/devise) for authentication in the following example app, and you will learn how to have fancy login within your app.

Generate the app

```bash
$ rails new fancy_login_app
```

Generate the home controller

```bash
$ rails g controller home index
```

Generate the controller requiring authentications

```bash
$ rails g controller posts new create
```

Add [Devise](https://github.com/plataformatec/devise) and [red_light](https://github.com/leomayleomay/red_light) to the Gemfile

```ruby
gem 'devise'
gem 'red_light'
```

Install the gems

```bash
$ bundle install
$ rails g devise:install
$ rails g red_light:install
```

Configure Devise views

```bash
$ rails g devise:views
```

The `config/routes.rb` should be something like

```ruby
devise_scope :user do
  devise_for :users, :controllers => { :sessions => "sessions" }

  get "/login", :to => "sessions#new"
end

resources :posts, :only => [:new, :create]

root :to => "home#index"
```

Change `app/controllers/posts_controller.rb` to something like

```ruby
class PostsController < ApplicationController
  before_filter authenticate_user!
  
  def new
  end
  
  def create
  end
end
```

Change `app/views/home/index.html.erb` to something like

```ruby
<%= link_to "New Post", new_post_path %>
```

Fire the webrick

```bash
$ rails s
```

Visit `localhost:3000`, inspect the "New Post" link, you fill find the interesting `rel`

{% img left /images/fancy_login_rel.jpg 685 24 'fancy_login_rel' %}

`red_light` add that `rel` to the link requiring authentication auto-magically. With the `rel`, you can play some javascript tricks to present a modal view when the link is clicked, here's a [demo](https://fancy-login.herokuapp.com).

You can also customize the `config/initializers/red_light.rb` to tell the `red_light` which `before_filter` should be auto blocked, for now, it's only `authenticate_user!` there.


