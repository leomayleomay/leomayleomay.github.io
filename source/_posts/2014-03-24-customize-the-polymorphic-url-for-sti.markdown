---
layout: post
title: "Customize the polymorphic URL for STI"
date: 2014-03-24 18:19
comments: true
categories: Rails
---

One thing I love Rails is it provides really handy URL helpers to the developers. Today, I am going to present you guys how to use [polymorphic_url](http://api.rubyonrails.org/classes/ActionDispatch/Routing/PolymorphicRoutes.html) like a ninja.

Say, I've got the following models:

```ruby

# app/models/vehicle.rb
class Vehicle < ActiveRecord::Base
  ...
end

# app/models/bicycle.rb
class Bicycle < Vehicle
  ...
end

# app/models/motercycle.rb
class Motorcycle < Vehicle
  ...
end


```

As you can see, we've have a STI table named `vehicles` in the database, and it stores all the bicycles, motorcycles and the other new vehicles we may have in the future, like `unicycle`.

We have a collection of vehicles, `@vehicles`, it contains bicycles and motercycles. What we are going to do is iterating the `@vehicles`, and display a link to the vehicle detail page. With `polymorphic_url`, we can write down the following code:

```ruby

<% @vechicles.each do |v| %>
  <%= link_to "Vehicle Detail", polymorphic_url(v) %>
<% end %>

```

instead of:

```ruby

<% @vehicles.each do |v| %>
  <% if v.is_a? Bicycle %>
  	<%= link_to "Vehicle Detail", bicycle_page(v) %>
  <% elsif v.is_a? Motercycle %>
   	<%= link_to "Vehicle Detail", motorcycle_page(v) %>
  <% end %>
<% end %>

```

Once we add a new kind of vehicle, we don't need to open the above view file and add another `elsif`, it complies to the [Open Closed Principle](http://en.wikipedia.org/wiki/Open/closed_principle) perfectly!

The request to bicycle detail will go to:

```ruby

# app/controllers/bicycles_controller.rb
class BicyclesController < ApplicationController
  def show
    @bicycle = Bicycle.find(params[:id])
  end
end

# app/views/bicycles/show.html.erb
<h1>Here comes the bicycle details</h1>


```

And the request to motorcycle detail will go to:

```ruby

# app/controllers/motorcycles_controller.rb
class MotorcyclesController < ApplicationController
  def show
    @motorcycle = Motorcycle.find(params[:id])
  end
end

# app/views/motorcycles/show.html.erb
<h1>Here comes the motorcycle details</h1>


```

What if we want to have the same controller rendering the same view for all the vehicles? Say, we only want one `VehiclesController#show` for all the vehicles's detail, here we go:

```ruby

# app/controllers/vehicles_controller.rb
class VechiclesController < ApplicationController
  def show
    @vehicle = Vehicle.find(params[:id])
  end
end

# app/views/vechicles/show.html.erb
<h1>Here comes the motorcycle details</h1>

```

You may say we can just use `vehicle_url(v)` to generate the URL, I can not agree more, but we are exploring something deep inside of Ruby on Rails, so, bare with me, :)

To have the `polymorphic_url`, generate the URL like `vechicle_url`, we need to overwrite `self.model_name` for the `Vehicle`, here's how I found it out:

1. the `polymorphic_url` will call `build_named_route_call` to generate the URL
2. the `build_named_route_call` will call `RecordIdentifier.__send__("plural_class_name", record)` to find out reource name, say, a record of `Bicycle` will generate `bicycles`
3. the `plural_class_name` will call `model_name_from_record_or_class` to determine the model name of the record
4. `model_name_from_record_or_class` will call the record's class `model_name` method to find out the model name of the class

We need to override the `self.model_name` to let `polymorphic_url` pick up the right model name:

```ruby

# app/models/vehicle.rb
class Vehicle < ActiveRecord::Base
  ...

  def self.model_name
    ActiveModel::Name.new(self, nil, "Vehicle")
  end
end

```

If we want a custom URL and view template for specific vehicle, we can also write something like:

```ruby
# app/models/unicycle.rb
class Unicycle < Vehicle
  def self.model_name
    ActiveModel::Name.new(self, nil, "Unicycle")
  end
end

```

With the following customization, `polymorphic_url(@unicycle)` will generate the URL `unicycle_url(@unicycle)`.