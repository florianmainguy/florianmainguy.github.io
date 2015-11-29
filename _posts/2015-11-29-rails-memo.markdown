---
layout: post
title:  "Git memo"
date:   2015-11-29 20:30:00
categories: jekyll update
---
A simple memo to remind me some basic rails commands.

{% highlight ruby %}
# Migrate up
bundle exec rake db:migrate
# Reverse migration or 'Migrate down'
bundle exec rake db:rollback

# Load development environment in sandbox
rails console --sandbox

# New model object
user = User.new(name: "florian", email: "flo@mainguy.com")
user.save
# Equivalent
user = User.create(name: "florian", email: "flo@mainguy.com")

# Update object
user.name = "ginny"
user.save

# Find object from database
User.find('id')
User.find_by(email: "flo@mainguy.com")
User.first
User.all

# Integration test
bundle exec rake test:integration
# Models test
bundle exec rake test:models
{% endhighlight %}

Last update: 29 Nov 2015