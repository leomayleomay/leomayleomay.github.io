---
layout: post
title: "take a REST"
date: 2013-05-07 23:16
comments: true
categories: Refactor Ruby Rails REST
---

[REST](https://en.wikipedia.org/wiki/Representational_state_transfer) is a buzzword in the modern web development world. "Be RESTful", "Keep up to the RESTful rules" is usually thought as one of the best practices we need to keep in mind.

We are all blessed as a [Ruby](http://www.ruby-lang.org/en/) & [Rails](http://rubyonrails.org/) developer, since Rails has already had REST baked within its heart, for example,

```ruby
# config/routes.rb

resources :photos

```

will generate 7 routes for us:

| HTTP Verb	| Path	| action used for |
|:---------	|:-----	|----------------:|
|GET	/photos				| index				| display a list of all photos
|GET	/photos/new 		| new 				| return an HTML form for creating a new photo
|POST	/photos				| create			| create a new photo
|GET	/photos/:id			| show		       	| display a specific photo
|GET	/photos/:id/edit	| edit            	| return an HTML form for editing a photo
|PUT	/photos/:id	       	| update 			| update a specific photo
|DELETE	/photos/:id     	| delete 			| delete a specific photo
---



Awesome! we can also adjust which actions we want via editing the resources defined in the `config/routes.rb`:

```ruby
# config/routes.rb

resources :photos, :only => [:index, :show]


```

Now we have only two actions left for the `PhotosController`, and they are `GET /photos` and `GET /photos/:id`, any other requests except these two will result in an error. So far so, good, until one day, the boss came over and told you to add comment to the photos, we want the user to comment the photos, who doesn't? OK, roll up the sleeves, and change the `config/routes.rb` to:

```ruby
# config/routes.rb

resources :photos, :only => [:new, :create] do
  member do
    post :comment
  end
end


```

that will give us a new routes `POST /photos/:id/comment`, and we add the corresponding action to the `PhotosController`:


```ruby
class PhotosController < ApplicationController
  before_filter :find_the_current_user
  before_filter :find_the_current_photo

  def comment
    comment = current_user.comments.new(params[:comment])
    comment.commentable = current_photo
  
    if comment.save
      redirect_to current_photo, notice: "new comment added!"
    else
      redirect_to current_photo, notice: "oops, xxit happens!"
    end
  end
end

```

After done with spec, and all green, we are happy to close the ticket and move on to next issue. Wait a minute, someone is whispering "Be RESTful",  sounds like we are doing something wrong, we are not RESTful enough! We all know the above code will work and do the job, the QA will happily accept your work and everything seems to be working flawlessly. Just one thing, we are not keeping up to RESTful rules, we are adding new action besides the 7 actions given to us, that's not good enough. Let's fix it:

* remove the action `comment` defined in the `PhotosController`
* edit the `config/routes.rb` and change it to:

```ruby
  resources :photos, :only => [:new, :create]
  resources :comments, :only => [:create]
```
* add a new `CommentsController` and it should have action `create` within it
* move the `photos_controller#comment` spec to `comments_controller#create`

that's all, the refactor is all about moving things around, nothing more. But it's important for two reasons:

* we keep the `PhotosController` only concerned photos, that's [SRP](http://en.wikipedia.org/wiki/Single_responsibility_principle)
* we make it clear that the comments related action should go to its own controller, we are telling the developer reading this code later after us that we have two resources defined in our app, one is photo, another is comment.

Yes, RESTful is all about resource.




