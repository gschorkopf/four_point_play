---
layout: post
title: "Refactoring Complex Validations from Your Models"
date: 2014-05-10 18:55:24 -0400
comments: true
categories: code rails
---

You've probably heard the phrase "fat models, skinny controllers" before. But when Rails models become a dumping ground for all things even tangentally related to the original class, it's probably time to trim off a few pounds.

<!--more-->

One way to do that is to pare away complex validations. For example, you might think it wise to drum up a method or two to handle passwords (minimum 8 characters! minimum one number! no weird symbols! etc.) So you throw these methods into the private domain of your User class, and call it a day.

But you can (and should) probably move that logic elsewhere. The following is usually how my complexity validator looks:

We have a simple User model.

``` ruby
# app/models/user.rb
class User < ActiveRecord::Base
  validates :password, complexity: true
end
```

Which hides away logic that's not even really related to a User. You can inherit from the ActiveModel::EachValidator class, like so:

``` ruby
# app/validators/complexity_validator.rb
class ComplexityValidator < ActiveModel::EachValidator
  def validate_each(record, attribute, value)
    record.errors.add(attribute, "must have a number")              if value !~ /(?=.*\d)/
    record.errors.add(attribute, "must have a lowercase letter")    if value !~ /(?=.*[a-z])/
    record.errors.add(attribute, "must have an uppercase letter")   if value !~ /(?=.*[A-Z])/
    record.errors.add(attribute, "must have a special character")   if value !~ /(?=.*[\W])/
    record.errors.add(attribute, "must have at least 8 characters") if value.length < 8
  end
end
```

And to tidy things up on the view end, consider the following for error handling:

``` ruby
# app/views/users/new.html.erb
# ...
= f.input :password, label: "New Password", error_method: :to_sentence
# ...
```

to_sentence is built into ActiveSupport, and helps us show all errors in a user-friendly way. [Check it out](http://apidock.com/rails/ActiveSupport/CoreExtensions/Array/Conversions/to_sentence).

- GPS
