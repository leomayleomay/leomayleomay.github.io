---
layout: post
title: "Rails 4 Gotcha: nested form created new child object every time parent object is updated"
date: 2014-05-26 08:46
comments: true
categories: Rails
---

I've been playing with Rails 4 recently, the strong parameter, as you all know, is a new feature just appeared in Rails 4. It will get really tricky when you are posting a nested form.


```ruby
# question.rb
class Question < ActiveRecord::Base
  has_one :answer, dependent: :destroy
  accepts_nested_attributes_for :answer, reject_if: Proc.new {|a| a["body"].blank?}
end

# answer.rb
class Answer < ActiveRecord::Base
  belongs_to :question
end



# questions_controller.rb
class QuestionsController < ApplicationController
  def update
  	@question = Question.find(params[:id])
  	
  	if @question.update_attributes(question_params)
  	  redirect_to questions_path, notice: "Question updated."
  	else
  	  render :edit
  	end
  end
  
  protected
  def question_params
   	params.require(:question).permit(:content, answer_attributes: [:body])
  end
end

# questions/edit.html.haml

= form_for @question do |f|
  = f.text_field :content
  
  = f.fields_for :answer, @question.answer || @question.build_answer do |builder|
    = builder.text_field :body
    
  = f.submit "Update", disable_with: "Updating ..."
```

The above code are really straightforward Rails 4 code for updating a given question with its answer, the problem right now is every time I updated the question, no matter if updated the associated answer or not, it will destroy the old answer object, and create a new answer object for me, that's really annoying, the gotcha here is you need have the answer's `id` field go through for strong parameter, otherwise, it will be filtered out, and the old answer will be destroyed, and a new answer object will be created silently. The revision is something like:


```ruby
# questions_controller.rb
class QuestionsController < ApplicationController
  ...
  
  protected
  def question_params
   	params.require(:question).permit(:content, answer_attributes: [:id, :body])
  end
end
```

Happy coding.