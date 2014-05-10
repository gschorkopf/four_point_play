---
layout: post
title: "Refactoring Complex Validations from Your Models"
date: 2014-05-10 18:55:24 -0400
comments: true
categories: code
---

# app/models/user.rb
class User < ActiveRecord::Base
  validates :password, complexity: true # You may also pass this a method that returns true/false
  # ...
end

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

# some view form that takes password, likely app/views/users/new.html.erb
#...
= f.input :password, label: "New Password", error_method: :to_sentence
#...
# Note: to_sentence is built into ActiveSupport, and helps us show all errors in a user-friendly way.
# (http://apidock.com/rails/ActiveSupport/CoreExtensions/Array/Conversions/to_sentence)

- GPS
