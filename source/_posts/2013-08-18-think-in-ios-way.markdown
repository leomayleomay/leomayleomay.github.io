---
layout: post
title: "As a Rails dev, you may want to think in the iOS way"
date: 2013-08-18 16:02
comments: true
categories: OO rails
---

I am not sure this is a good idea, but I think it worths a post to share the idea to the people who're suffering/fighting messy dependencies within their applications.

Anyone who's been programming in Objective-C should be familiar with the delegate/protocol, you can barely make any practical iOS application without knowing the delegate/protocol. For those of you who have never been in the iOS world, this is the quick definition from [apple official document](http://developer.apple.com/library/ios/documentation/cocoa/conceptual/ProgrammingWithObjectiveC/WorkingwithProtocols/WorkingwithProtocols.html)


>In the world of object-oriented programming, it’s important to be able to define a set of behavior that is expected of an object in a given situation. As an example, a table view expects to be able to communicate with a data source object in order to find out what it is required to display. This means that the data source must respond to a specific set of messages that the table view might send.

I am not a language expert, I have no idea who invented this, but this is definitely a brilliant idea in my opinion, you just fire a method on your collaborator, and you will be notified what to do next. You don't ask, you tell, sounds familiar? Yeah, people in Hollywood are always saying:

>You don't call us, we will call you

It's also well known as [Tell, don't ask](http://c2.com/cgi/wiki?TellDontAsk).

You know what, you don't have to be an iOS developer to benefit from this, this is generic you can apply it to your Rails applications.


```ruby

class PostsController < ApplicationController
  def new
    @post = Post.new
  end

  def create
    @post = Post.new(params[:post])

    if @post.save
      redirect_to posts_path, :notice => "Post created succesfully!"
    else
      render :new
    end
  end
end

```

The above snippet is a classical `PostsController` that does nothing wrong, the coming part is the same logic implemented with delegate/protocol.

```ruby

class PostsController < ApplicationController
  def new
    @post = Post.new
  end

  def create
  	service = PostService.new
  	service.delegate = self
  	service.create_new_post(params[:post])
  end
  
  # PostServiceProtocol begins
  def new_post_created_successfully(post)
  	redirect_to posts_path, :notice => "Post created succesfully!"
  end
  
  def new_post_not_created_because_of(errors)
    render :new
  end
  # PostServiceProtocol ends
end

class PostService
  attr_writer :delegate
  
  def create_new_post(params)
  	post = Post.new(params)
  	
  	if post.save
  	  self.delegate.new_post_created_successfully(post)
  	else
  	  self.delegate.new_post_not_created_because_of(post.errors)
  	end
  end

  protected
  def delegate
    @delegate ||= NullPostServiceDelegate.new
  end
end

class NullPostServiceDelegate
  def new_post_created_successfully(post)
	# no-op
  end
  
  def new_post_not_created_because_of(errors)
    # no-op
  end
end

```

If you are still with me, you may ask: "why bother?", why we want to do this as long as the first version is working perfectly, my explanation is you never know who's going to call `PostService#create_new_post`. For now, it's just `PostsController` doing that. Imagine the Rails application is providing API backend for a mobile app, someone is going to create new post via API, are we going to duplicate the logic as well as the tests? I don't think so.

```ruby

class Api::PostsController < Api::Base
  def create
  	service = PostService.new
  	service.delegate = self
  	service.create_new_post(params[:post])
  end
  
  # PostServiceProtocol begins
  def new_post_created_successfully(post)
    render :json => post.to_json
  end
  
  def new_post_not_created_because_of(errors)
    render :json => {
      :errors => errors
    }.to_json
  end
  # PostServiceProtocol ends 
end

```

You can see from the above example, I am having the same logic for creating post for the web endpoint and API endpoint, it's consitent.

For now, there's no way in plain Ruby to define protocol for a class, there's also no way to have a class corform to specific protocol, I am considering have a gem for that, if you are interested pair with me, ping me.
