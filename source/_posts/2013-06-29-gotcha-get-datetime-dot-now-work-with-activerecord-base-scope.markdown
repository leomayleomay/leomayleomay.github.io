---
layout: post
title: "gotcha: get DateTime.now work with ActiveRecord::Base scope"
date: 2013-06-29 19:12
comments: true
categories: gotcha rails active_record scope
---

```ruby

class Invitation < ActiveRecord::Base
  scope :expired, where("invitations.expired_at < ?", DateTime.now)
end

```

It looks nothing wrong with the above code at the first glance. Let's have an interesting experiment,

```bash
1.9.3p327 :001 > Invitation.expired.to_sql
 => "SELECT `invitations`.* FROM `invitations`  WHERE (invitations.created_at < '2013-06-29 12:00:16')"
1.9.3p327 :002 > DateTime.now.utc
 => Sat, 29 Jun 2013 12:35:50 +0000
```

As you can see, the `DateTime.now.utc` returns "12:35:50", while the SQL statement in the first line says "12:00:16", there's a mysterious gap between two of them. Actually, the "12:00:16" is the time I started the rails console, to be more precise, it's the time the `app/models/invitation.rb` is loaded, it's the time the `DateTime.now` in the `expired` scope was evaluated. This could be a serious problem for some of you, espeically under production environment, since once you deployed your application, the `DateTime.now` will keep what it was when the application was fully loaded by the application server, and it will be a constant value instead of a variable value as you imagined, the only chance it will get changed is the next time you deploy the application.

The problem seems to be mysterious, but the solution to the problem is pretty straightforwarding, `lambda`, have `lambda` work with the scope will make the statements in the closure evaluated dynamically every time the scope is called.

```ruby

class Invitation < ActiveRecord::Base
  scope :expired, lambda {
    where("invitations.expired_at < ?", DateTime.now)
  }
end

```

```bash
1.9.3p327 :001 > Invitation.expired.to_sql
 => "SELECT `invitations`.* FROM `invitations`  WHERE (invitations.created_at < '2013-06-29 12:40:11')"
1.9.3p327 :002 > DateTime.now.utc
 => Sat, 29 Jun 2013 12:40:12 +0000
```

It seems to be working this time. It's time to add more spec to cover this.

```ruby
describe Invitation do
  descirbe '.expired' do
	it "should find all the expired invitations" do
	  invitation_1 = Factory(:invitation, :expired_at => DateTime.now - 1.seconds)
	  invitation_2 = Factory(:invitation, :expired_at => DateTime.now + 1.seconds)
	  
	  Invitation.expired.to_a.should == [invitation_1]
	end
  end
end
```

The above test will fail if you not using `lambda`, just keep in mind, test will save your ass every time.