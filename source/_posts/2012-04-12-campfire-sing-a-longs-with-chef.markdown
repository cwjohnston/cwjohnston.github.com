---
layout: post
title: "Campfire sing-a-longs with Chef"
date: 2012-04-12 22:01
comments: true
categories: chef lwrp campfire
---

Like many small teams, Needle uses [37signals' Campfire](http://www.campfirenow.com) chat platform for collaborating online. Along with messages exchanged between coworkers, we also use Campfire for announcing new git commits, jira tickets and successful application deployments.

Since the code I've been using to send messages from Chef recipes to Campfire is virtually identical between a number of our cookbooks, I decided to turn that code into a [LWRP](http://wiki.opscode.com/display/chef/Lightweight+Resources+and+Providers+%28LWRP%29) that anyone can use in their own recipes. The cookbook for this LWRP is [available on github](http://github.com/cwjohnston/chef-campfire).

## Requirements

* a Campfire API token (these are unique to each Campfire user, so if you want your messages to come from a particular user, get their token)
* the `tinder` gem (installed by the `campfire::default` recipe)

## Attributes

* `subdomain` - the subdomain for your Campfire instance (required)
* `room` - the name of the room you would like to speak into (requied)
* `token` - authentication token for your Campfire account (required)
* `message` - the message to speak. If a message is not specified, the name of the `campfire_msg` resource is used.
* `paste` - toggles whether or not to send the message as a monospaced "paste" (defaults to false)
* `play_before` - play the specified sound before speaking the message
* `play_after` - play the specified sound after speaking the message
* `failure_ok` - toggles whether or not to catch the exception if an error is encountered connecting to Campfire (defaults to true)

A list of emoji and sounds available in Campfire can be found here: http://www.emoji-cheat-sheet.com/ 

## Usage examples

``` ruby
include_recipe 'campfire'

campfire_msg 'bad news' do
  subdomain 'example'
  room 'Important Stuff'
  token '0xdedbeef0xdedbeef0xdedbeef'
  message "I have some bad news... there was an error: #{some_error}"
  play_after 'trombone'
end
```
