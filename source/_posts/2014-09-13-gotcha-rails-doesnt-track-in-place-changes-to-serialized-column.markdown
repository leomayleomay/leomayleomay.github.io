---
layout: post
title: "Gotcha: Rails doesn't track in place changes to serialized column"
date: 2014-09-13 21:20
comments: true
categories: Rails
---

Believe it or not, it's been 6 years since the [ticket](https://rails.lighthouseapp.com/projects/8994/tickets/360-dirty-tracking-on-serialized-columns-is-broken) was created about "Rails doesn't track in place changes to serialized column", it's really a tricky problem rare people will run into. Consider the following scenario:

```ruby
class Answer < ActiveRecord::Base
  serialize :options, Array
end
```

```ruby

\>> a = Answers.new(options: [1, 2, 3])

\>> a.options

=> [1, 2, 3]

\>> a.changed?

=> false

\>> a.options << 4

=> [1, 2, 3, 4]

\>> a.changed?

=> false
```

As commented by Jose Valim [link](https://rails.lighthouseapp.com/projects/8994/tickets/360-dirty-tracking-on-serialized-columns-is-broken#ticket-360-16), it's a feature of Rails and it's documented somewhere. The workaround is issue another `options_will_change!` to mark the object changed.


```ruby

\>> a = Answers.new(options: [1, 2, 3])

\>> a.options

=> [1, 2, 3]

\>> a.changed?

=> false

\>> a.options << 4

=> [1, 2, 3, 4]

\>> a.options_will_change!

\>> a.changed?

=> true

```

It gets more tricky especially when you are using `accepts_nested_attributes_for`, the Rails will ditch the changes to the children objects if `changed?` returns false.

```ruby
class Question < ActiveRecord::Base
  has_many :answers, dependent: :destroy
  accepts_nested_attributes_for :answers
end
```

```ruby

\# spec/requests/questions_controller_spec.rb

describe QuestionsController do
  describe "PUT /update" do
    it "updates the question with its answers" do
      q1 = Question.create
      a1 = Answer.create(question: q1, options: [1, 2, 3])

      put :update, id: q1.id, question: {
        answers_attributes: {
          "0" => {id: a1.id, options: [1, 2, 3, 4]}
        }
      }
      
      a1.reload
      expect(a1.options).to eq [1, 2, 3, 4] # FAILED!!
    end
  end
end

```

It really got me and hope this will help somebody like me, happy hacking!!