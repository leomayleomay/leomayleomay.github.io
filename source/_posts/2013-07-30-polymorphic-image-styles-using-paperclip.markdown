---
layout: post
title: "Polymorphic image styles with Paperclip"
date: 2013-07-30 15:02
comments: true
categories: "rails" "paperclip" "Object Oriented Programming"
---

You may have ran into the following scenario that your rails app having 1+ domain models having image with them, say `Album` has one `cover`, `User` has one `avatar`:

```ruby

class Image < ActiveRecord::Base
  belongs_to :viewable, :polymorphic => true
  
  validate :viewable, :presence => true
  
  has_attached_file :attachment
end

class User < ActiveRecord::Base
  has_one :avatar, :class_name => "Image", :as => :viewable, :dependent => :destroy
end

class Album < ActiveRecord::Base
  has_one :cover, :class_name => "Image", :as => :viewable, :dependent => :destroy
end

```

As you can see, an image belongs to a `viewable` object, the `viweable` object could be an instance of `User` or `Album` or anything else.

The presumption here is that the `cover` is having a set of styles different from with the `avatar`, but we only have one `Image` class, what are we supposed to do? We dispatch. We let the `viewable` object decide the styles of the image.

```ruby
class Image < ActiveRecord::Base
  belongs_to :viewable, :polymorphic => true
  
  validate :viewable, :presence => true
  
  has_attached_file :attachment,
    :styles => Proc.new {|a| a.instance.viewable.image_styles}
end

class User < ActiveRecord::Base
  has_one :avatar, :class_name => "Image", :as => :viewable, :dependent => :destroy
  
  def image_styles
    {
      :thumb => "30x30>",
      :medium => "100x100>"
    }
  end
end

class Album < ActiveRecord::Base
  has_one :cover, :class_name => "Image", :as => :viewable, :dependent => :destroy
  
  def image_styles
    {
      :cart => "50x50>"
      :large => "300x300>"
    }
  end
end
```

We now have one `Image` class having different styles for different `viewable` objects, we can also do the same thing to `default_url` and `path`. Leave me a comment if you have better solution, thanks a lot.