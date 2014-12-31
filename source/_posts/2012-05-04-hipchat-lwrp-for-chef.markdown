---
layout: post
title: "HipChat LWRP for Chef"
date: 2012-05-04 12:22
comments: true
categories: chef lwrp hipchat
---

Since releasing a Campfire LWRP for Chef a few weeks ago, my team has evaluated
and subsequently transitioned to [Atlassian's HipChat service](http://www.hipchat.com). 
Luckily I was able to reuse the framework I had already created for Campfire as the
basis for a HipChat LWRP.

The LWRP should work with any modern version of Chef. When you use `include_recipe`
to access the LWRP in your own recipes, the default recipe for this cookbook will install
the required ['hipchat' gem](http://rubygems.org/gems/hipchat).

## Attributes
* `room` - the name of the room you would like to speak into (requied).
* `token` - authentication token for your HipChat account (required).
* `nickname` - the nickname to be used when speaking the message (required).
* `message` - the message to speak. If a message is not specified, the name of the `hipchat_msg` resource is used.
* `notify` - toggles whether or not users in the room should be notified by this message (defaults to true).
* `color` - sets the color of the message in HipChat. Supported colors include: yellow, red, green, purple, or random (defaults to yellow).
* `failure_ok` - toggles whether or not to catch the exception if an error is encountered connecting to HipChat (defaults to true).

## Usage example
```ruby
include_recipe 'hipchat'

hipchat_msg 'bad news' do
  room 'The Pod Bay'
  token '0xdedbeef0xdedbeef0xdedbeef'
  nickname 'HAL9000'
  message "Sorry Dave, I'm afraid I can't do that: #{some_error}"
  color 'red'
end
```

## Availability

You can find this cookbook [on github](http://www.github.com/cwjohnston/chef-hipchat) or on the [Opscode community site](http://community.opscode.com/cookbooks/hipchat).
