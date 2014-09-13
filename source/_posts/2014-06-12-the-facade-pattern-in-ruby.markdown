---
layout: post
title: "The Facade Pattern In Ruby"
date: 2014-06-12 13:25:26 -0400
comments: true
published: false
categories: code ruby
---

A few months ago I started implementing a pattern that I picked up from some fellow [Nerds](http://www.bignerdranch.com). I sort of blindly used it, not taking the time to figure out why I liked it so much (and not even knowing it had a name) until [Charlie](https://twitter.com/charliebomber) pressed me for the big question: "great, but why?"

Turns out, it does have a name (duh), and that name is the [facade pattern](http://en.wikipedia.org/wiki/Facade_pattern).

<!--more-->

Let me give you an example. I recently built an app for uploading and sharing your schedule for the Bonnaroo Music Festival. Here's the (gist of a) class that creates schedules:

```ruby
class BonnarooSchedule
  attr_reader :user, :profile_name, :document

  def self.upload(params)
    new(params).upload
  end

  def initialize(params)
    @document = params[:document]
    @profile_name = params[:profile_name]
    @user = params[:user]
  end

  def upload
    events.each do |event|
      if show = Show.find_by(artist_name: artist_name(event))
        # Create the ScheduleShow
      end
    end
  end

  private
  # Some private methods (slike events and artist_name) that
  # make my life easier
end

```

Here's the full <a href="http://www.github.com/gschorkopf/roo-pals" target="_blank">open source project</a>, which I intend to share in a future blog post.
