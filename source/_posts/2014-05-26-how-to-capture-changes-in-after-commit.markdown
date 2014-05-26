---
layout: post
title: "How to capture changes in after_commit"
date: 2014-05-26 07:43
comments: true
categories: Rails ActiveRecord
---

@flyerhzm has done a great job on when you should use `after_commmit` instead of `after_save` in this [post](http://rails-bestpractices.com/posts/695-use-after_commit). One thing is sometimes you may want to check if there's any changes to a specific column before executing the code within the callback. Sadly, since it is in `after_commit` block, all the changes have been persisted into the database, you will never got a chance to check if there's any changes to any columns, that's the problem we are going to attack.


```ruby
class Person < ActiveRecord::Base
  after_commit :notify_police, only: [:create, :update]
  
  private
  def notify_police
    if self.name_changed? or self.dob_changed?
      PoliceNotification.notify(self)
    end
  end
end
```

Obviously, you don't want to bother the police every time you change your hair style is changed, believe me, you don't want to do that. The fact is we need to let the police know once you change your name or your DOB. You will find a tricky bug in the above code, the `PoliceNotification.notify(self)` will never get triggered because in the `after_commit` block, you will never see any changes to any columns. So, the way I fix this problem is:


```ruby
class Person < ActiveRecord::Base
  after_save :prepare_to_notify_police
  after_commit :notify_police, only: [:create, :update]
  
  private
  def prepare_to_notify_police
    @name_or_dob_changed = self.name_changed? or self.dob_changed?
  end

  def notify_police
    if @name_or_dob_changed
      PoliceNotification.notify(self)
    end
  end
end

```

As you can see, I set a flag `@name_or_dob_changed` within the `after_save` block, as long as you do not reload the object, you can do the check based on the flag in the `after_commit` even after all the changes are persisted into database.

