---
layout: post
title: "Managing Loggly devices and inputs via Chef"
date: 2011-12-12 15:19
comments: true
categories: chef lwrp logging
---

# Introduction

Recently I have been experimenting with the logging-as-a-service platform at [Loggly](http://loggly.com/). It seems pretty promising, and there's a free tier for those who are indexing less than 200MB per day.

Since I am using Chef to manage my systems, I decided I would take a crack at writing a [LWRP](http://wiki.opscode.com/display/chef/Lightweight+Resources+and+Providers+%28LWRP%29) that would allow me to manage devices and inputs on my Loggly account through Chef. This makes it possible for new nodes to register themselves as Loggly devices when they are provisioned, without requiring me to make a trip to the Loggly control panel. The resulting cookbook is available here: [http://github.com/cwjohnston/chef-loggly](http://github.com/cwjohnston/chef-loggly)

# Requirements

* Valid Loggly account username and password
* `json` ruby gem

## Required node attributes

* `node['loggly']['username']` - Your Loggly username.
* `node['loggly']['password']` - Your Loggly password.

In the future these attributes should be made optional so that usernames and passwords can be specified as parameters for resource attributes.

# Recipes

* `default` - simply installs the `json` gem. Chef requires this gem as well, so it should already be available.
* `rsyslog` - creates a loggly input for receiving syslog messages, registers the node as a device on that input and configures rsyslog to forward syslog messages there. 

# Resources

## `loggly_input` - manage a log input

### Attributes

* `domain` - The subdomain for your loggly account
* `description` - An optional descriptor for the input
* `type` - The kind of input to create. May be one of the following:
    * `http`
    * `syslogudp`
    * `syslogtcp`
    * `syslog_tls`
    * `syslogtcp_strip`
    * `syslogudp_strip`

### Actions

* `create` - create the named input (default)
* `delete` - delete the named input

### Usage

``` ruby
loggly_input "production-syslog" do
    domain "examplecorp"
    type "syslogtcp"
    description "syslog messages from production nodes"
    action :create
end
```

## `loggly_device` - manage a device which sends logs to an input

The name of a `loggly_device` resource should be the IP address for the device. Loggly doesn't do DNS lookups, it just wants the device's IP.

### Resource Attributes

* `username` - Your Loggly username. if no value is provided for this attribute, the value of `node['loggly']['username']` will be used.
* `password` - Your Loggly password. if no value is provided for this attribute, the value of `node['loggly']['password']` will be used.
* `domain` - The subdomain for your loggly account
* `input` - the name of the input this device should be added to

### Resource Actions

* `add` - add the device to the named input (default)
* `delete` - remove the device from the named input

### Usage

``` ruby
loggly_device node[:ipaddress] do
    domain "examplecorp"
    input "production-syslog"
    action :add
end
```
