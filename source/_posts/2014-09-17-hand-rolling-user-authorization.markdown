---
layout: post
title: "Hand Rolling User Authorization"
date: 2014-09-17 13:25:26 -0400
comments: true
categories: code ruby rails
---

When writing a Rails app, you'll almost always have to employ some form of user authorization. Peeps wanna sign in and be able to see their personal awesomeness. And often enough, certain folks (admins, managers, <a href="http://2.bp.blogspot.com/-W303-_EO37M/TVw1bxG_gRI/AAAAAAAAAjY/cjHl5LiOe3c/s1600/math2.jpg" target="_blank">bad-ass M.C.s</a>) will have specialized permissions. Today, we're going to hand roll authorization into our web app.

<!--more-->

Popular Ruby gems for authorization include <a href="https://github.com/ryanb/cancan" target="_blank">CanCan</a>, <a href="https://github.com/RolifyCommunity/rolify" target="_blank">Rolify</a>, and my personal favorite, <a href="https://github.com/elabs/pundit" target="_blank">Pundit</a>. Today we're going to hand roll our own authorization system, taking some naming queues from the latter.

I'm going to assume you've already handled authentication somewhere in your app. For clarity, authentication is confirming the identity of a user, while authorization is determining the permissions of said user.

<hr/>

<h3>The Task at Hand</h3>

Let's say our app, I don't know, <a href="http://www.github.com/gschorkopf/frolfr/" target="_blank">keeps track of disc golf scorecards</a>. Each scorecard has many (usually 18) "turns," which is representative of a set of shots on a given hole for a given course. Let's say the object looks something like this:

```ruby
# db/schema.rb
create_table "turns", force: true do |t|
  t.integer  "user_id"
  t.integer  "hole_id"
  t.integer  "score"
  t.integer  "par", default: 3
end
```

For simplicity, let's say we only want the turn's player (user) to be able to <i>update</i> their own score / par / etc.

Let's get to work.

<hr/>

<h3>Writing a Turn Policy Object</h3>

The key to building out good authorization is writing an understandable policy object. As my buddy <a href="https://twitter.com/srbiv" target="_blank">Stafford</a> says, "naming things is the hardest thing we do in programming."

I personally like the naming conventions of the Pundit library. A policy object's method and class titles closely mirror the naming of Rails' RESTful resources and model conventions, respectfully.

Let's test drive this shiz.

```ruby
# spec/policies/turn_policy_spec.rb
describe TurnPolicy do
  subject(:policy) { described_class.new(current_user, turn) }
  let(:current_user) { double('User') }

  context 'the turn belongs to the current user' do
    let(:turn) { double('Turn', user: current_user) }

    it 'can update the turn' do
      expect(subject.update?).to eq true
    end
  end

  context 'the turn belongs to another user' do
    let(:turn) { double('Turn', user: double('User')) }

    it 'can not update the turn' do
      expect(subject.update?).to eq false
    end
  end
end
```

Simple enough! The spec creates a mock current user (the signed in user), a mock other user, and a mock turn. That's all we really need to get started.

When building out the TurnPolicy class, I like to use Structs. They're simple, and we're looking for simplicity here.

```ruby
# app/policies/turn_policy.rb
class TurnPolicy < Struct.new(:current_user, :turn)
end
```

Note the naming: the class ```TurnPolicy``` mirrors the ```Turn``` model, we place the policy in a policies directory, and pass it our current user (which you will for any policy object you create) and a turn variable. Lovely.

Next, we'll need to build out an ```update?``` method. As you might have guessed based on the name, this can be used in many places: our ```TurnsController#update``` action, our turns' edit page, and anywhere else it's needed. Neat, eh?

```ruby
# app/policies/turn_policy.rb
class TurnPolicy < Struct.new(:current_user, :turn)
  def update?
    turn.user == current_user
  end
end
```

Tada! Now let's apply our ```update?``` method elsewhere.

<hr/>

<h3>Applying our Object to Requests</h3>

This is what our turn's update action looks like now:

```ruby
# app/controllers/turns_controller.rb
class TurnsController < ApplicationController
  def update
    @turn = Turn.find(params[:id])
    # Do the updating
  end
end
```

Now we need to raise some sort of error when a bad user attempts to reach this action. Let's make the error reuseable across all our controllers and put it in our ApplicationController. The error can be anything, as long as it prevents the user from doing any updating.

```ruby
# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  # ...
  def authorize(policy)
    raise KaboomError unless policy
  end
end
```

So we've passed a method called ```authorize``` a boolean value, called ```policy```. If the turn belongs to the user, all is well. If not, well, KABOOM.

Now let's put this method to work.

```ruby
# app/controllers/turns_controller.rb
class TurnsController < ApplicationController
  # ...
  def update
    @turn = Turn.find(params[:id])

    authorize policy(turn).update?

    # Do the updating
  end

  def policy(turn)
    TurnPolicy.new(current_user, turn)
  end
end
```

We define our policy in a new method. We then use the ```authorize``` method and our ```update?``` method to authorize the action.

<hr/>

<h3>Applying our Object to Views</h3>

Finally, let's say we have a scorecard page, which is also our turns#index action. It's a simple table with all turns for that scorecard, as well as the ability to edit any score in place. Any user can view the scorecard; it's public. But only the turn's owner can edit a turn. In our TurnsController, we can turn our ```policy``` method into a nifty helper method.

```ruby
# app/controllers/turns_controller.rb
class TurnsController < ApplicationController
  # ...
  helper_method :policy
  def policy(turn)
    TurnPolicy.new(current_user, turn)
  end
end
```

So our view would could now something like this:

```ruby
# app/views/turns/index.html.slim
- @users.each do |user|
  tr
    td = user.initials
    - user.turns.each do |turn|
      - if policy(turn).update?
        td = link_to turn.score, turn_path(turn), method: :put
      - else
        td = turn.score
```

<hr/>

<h3>Conclusion</h3>

Hand rolling your own tools can remove dependencies, allows programmers unfamiliar with certain libraries to check out the policies within your own source code, and is just fun to try out.

- GPS
