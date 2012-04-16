---
layout: post
title: "Managing Pingdom service checks with Chef"
date: 2012-03-16 14:08
comments: true
categories: chef pingdom monitoring
---

## Monitoring with Pingdom

Swedish firm [Pingdom](http://pingdom.com/) offers a flexible, affordable service for monitoring the availability and response time of web sites, applications and other services. At Needle we provision an instance of our chat server for each partner we work with, and as a result I've found myself creating a Pingdom service check to monitor each of these instances. As you might imagine, this is a rather repetitive task, and the configuration is basically the same for each service check -- a process ripe for automation!

Thankfully Pingdom provides a [REST API](http://www.pingdom.com/services/api/) for interacting with the service programatically, which has made it possible for me to write a Chef LWRP for creating and modifying Pingdom service checks. Source available here: [http://github.com/cwjohnston/chef-pingdom](http://github.com/cwjohnston/chef-pingdom)

## Requirements

Requires Chef 0.7.10 or higher for Lightweight Resource and Provider support. Chef 0.10+ is recommended as this cookbook has not been tested with earlier versions.

A valid username, password and API key for your Pingdom account is required.

## Recipes

This cookbook provides an empty default recipe which installs the required `json` gem (verison <=1.6.1). Chef already requires this gem, so it's really just included in the interests of completeness.

## Libraries

This cookbook provides the `Opscode::Pingdom::Check` library module which is required by all the check providers.

## Resources and Providers

This cookbook provides a single resource (`pingdom_check`) and corresponding provider for managing Pingdom service checks. 

`pingdom_check` resources support the actions `add` and `delete`, `add` being the default. Each `pingdom_check` resource requires the following resource attributes:

* `host` - indicates the hostname (or IP address) which the service check will target
* `api_key` - a valid API key for your Pingdom account
* `username` - your Pingdom username
* `password` - your Pingdom password

`pingdom_check` resources may also specifiy values for the optional `type` and `check_params` attributes.

The `type` attribute will accept one of the following service check types. If no value is specified, the check type will default to `http`.

* http
* tcp
* udp
* ping
* dns
* smtp
* pop3
* imap

The optional `check_params` attribute is expected to be a hash containing key/value pairs which match the type-specific parameters defined by the [Pingdom API](http://www.pingdom.com/services/api-documentation-rest/#ResourceChecks). If no attributes are provided for `check_params`, the default values for type-specific defaults will be used.

## Usage

In order to utilize this cookbook, put the following at the top of the recipe where Pingdom resources are used:

``` ruby
include_recipe 'pingdom'
```

The following resource would configure a HTTP service check for the host `foo.example.com`:

``` ruby
pingdom_check 'foo http check' do
  host 'foo.example.com'A
  api_key node[:pingdom][:api_key]
  username node[:pingdom][:username]
  password node[:pingdom][:password]
end
```

The resulting HTTP service check would be created using all the Pingdom defaults for HTTP service checks.

The following resource would configure an HTTP service check for the host `bar.example.com` utilizing some of the parameters specific to the HTTP service check type:

``` ruby
pingdom_check 'bar.example.com http status check' do
  host 'bar.example.com'
  api_key node[:pingdom][:api_key]
  username node[:pingdom][:username]
  password node[:pingdom][:password]
  check_params :url => "/status",
               :shouldcontain => "Everything is OK!",
               :sendnotificationwhendown => 2,
               :sendtoemail => "true",
               :sendtoiphone => "true"
end
```

## Caveats

At this time I consider the LWRP to be incomplete. The two major gaps are as follows:

* Changing the values for `check_params` does not actually update the service check's configuration. I have done most of the initial work to implement this (available in the `check-updating` branch on github), but there are still bugs. 
* The LWRP has no support for managing contacts.

## Future

* Add `update` action for service checks which modifies existing checks to match the values from `check_params`
* Add `enable` and `disable` actions for service checks
* Add support for managing contacts (`pingdom_contact` resource)
* Convert `TrueClass` attribute values to `"true"` strings
* Validate classes passed as `check_params` values
* One must look up contact IDs manually when setting `contactids` in `check_params`
