---
layout: post
title: "Sortable bootstrap table with STI"
date: 2014-03-09 20:17
comments: true
categories: Rails
---

You are working on a Rails 3 project and your client asks you to sort a list of items and display the items in order, the first thing come to your mind might be acts_as_list and jQuery sortable, you are not alone. I found that acts_as_list requires the items to be sorted belong to a parent, sadly, that's not the case I am facing, the items I am trying to sort belongs to nothing, [RankedModel](https://github.com/mixonic/ranked-model) come to rescue. 

> ranked-model is a modern row sorting library built for Rails 3 & 4. It uses ARel aggressively and is better optimized than most other libraries.

Lucky me, i found [this great post](http://benw.me/posts/sortable-bootstrap-tables) introducing how to get RankdedModel work with Bootstrap table and jQuery sortable.

The requirement I am facing is bit more complex than the one explained in the above article, I have a list of items, which is an array of items of different types stored in one single tables, say, I have the following object layout:

```ruby


class Employee < ActiveRecord::Base
  include RankedModel
  ranks :title
end

class Manager < Employee
end

class Salesman < Employee
end

```

Say, I have a list of employees, including managers, salesman shown in order, I can now drag and drop to sort the items in the list, great, ready to charge the customer? Not yet, you will find it will only sort the managers and salesmen separately, it will not sort them as a whole entity, dig into the source of RankedModel, i found something really interesting,

```ruby


#lib/ranked-model/ranker.rb
def instance_class
  ranker.class_name.nil? ? instance.class : ranker.class_name.constantize
end

```


This where the RankedModel decides which class to use as the finder, the problem it could not sort managers and salesmen as a whole entity is because it will use `Manager` and `Salesman` separately to do the sorting, RankedModel found this problem before I do, and they provide an option `class_name` to unify the finder lookup, the solution is:

```ruby


class Employee < ActiveRecord::Base
  include RankedModel
  ranks :title, :class_name => "Employee"
end

```

Back to the items listing page, viola, it sorts the `Manager` and `Salesman` as a whole entity.

Happy hacking!