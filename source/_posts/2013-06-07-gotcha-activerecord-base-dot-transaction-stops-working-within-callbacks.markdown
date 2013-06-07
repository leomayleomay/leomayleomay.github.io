---
layout: post
title: "Gotcha: ActiveRecord::Base.transaction stops working within callbacks"
date: 2013-06-07 22:45
comments: true
categories: gotcha rails active_record transaction
---

If you are not familiar with `ActiveRecord::Base.transaction`, i highly recommend you go through the [document](http://api.rubyonrails.org/classes/ActiveRecord/Transactions/ClassMethods.html).

As you can see,
> Exceptions will force a ROLLBACK that returns the database to the state before the transaction began.

that means the following code should rollback the transaction if any of the `deposit!` fails

```ruby
# transaction.rb
class Transaction < ActiveRecord::Base
  after_create :transfer_credits

  attr_accessible :credits
  
  belongs_to :send_account, :class_name => "Account"
  belongs_to :recv_account, :class_name => "Account"
 
  private
  def transfer_credits
    begin
      ActiveRecord::Base.transaction do
      	send_account.deposit!(-credits) 
      	recv_account.deposit!(credits)
      end
    rescue => e
      Rails.logger.info e.message
    end
  end
end
 
# transaction_spec.rb
context "exception raised" do
  before do
    @send_account = create :account, :credits => 1000

    @transaction = Transaction.new
    @transaction.send_account = @send_account
    @transaction.credits = 1

    recv_account = double(:account).as_null_object
    @transaction.stub(:recv_account) {recv_account}
    recv_account.should_receive(:deposit!).and_raise(StandardError.new("Transfer failed!"))
  end
  
  it "should not transfer the credits" do
    @transaction.save

    @send_account.reload

    @send_account.credits.should == 1000
  end
end

```

But the result is the spec fails and says

```bash

Failures:
 
  1) Transaction status exception raised should not transfer the credits
     Failure/Error: @send_account.credits.should == 1000
       expected: 1000
            got: 999 (using ==)
     # ./spec/models/transaction_spec.rb:185:in `block (4 levels) in <top (required)>'
 
Finished in 1.73 seconds
1 example, 1 failure

```

It did deduct the credits from the `send_account` by 1 while i was expecting it should do nothing to it. What's going wrong, the doc says it will rollback the transaction, then I tried to test if calling `transfer_credits` explicitly is going to work. And yes, it works. Here's the revised version:


```ruby

# transaction.rb
class Transaction < ActiveRecord::Base
  attr_accessible :credits
  
  belongs_to :send_account, :class_name => "Account"
  belongs_to :recv_account, :class_name => "Account"

  def transfer_credits
    begin
      ActiveRecord::Base.transaction do
      	send_account.deposit!(-credits)
      	recv_account.deposit!(credits)
      end
    rescue => e
      Rails.logger.info e.message
    end
  end
end
 
# transaction_spec.rb
describe "#transfer_credits" do
  before do
    @send_account = create :account, :credits => 1000

    @transaction = Transaction.new
    @transaction.send_account = @send_account
    @transaction.credits = 1

    recv_account = double(:account).as_null_object
    @transaction.stub(:recv_account) {recv_account}
    recv_account.should_receive(:deposit!).and_raise(StandardError.new("Transfer failed!"))
  end
  
  it "should not transfer the credits" do
    @transaction.transfer_credits
 
    @send_account.reload

    @send_account.credits.should == 1000
  end
end

```

GOTCHA: DO NOT place ATOMIC operations within callbacks, like `before_*`, `after_*`