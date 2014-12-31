---
layout: post
title: "Silencing Sensu with a Chef Handler"
date: 2013-09-25 10:43
comments: true
categories: chef sensu
---

# Introduction

I have a lot of warm feelings for [Sensu](http://www.sensuapp.org/), a flexible, scalable open source monitoring framework. At [Needle](http://www.needle.com) our team has used Chef to build a Sensu instance for each of our environments, allowing us to test our automated monitoring configuration before promoting it to production, just like any other code we deploy.

Speaking of deploying code, isn't it obnoxious to see alerts from your monitoring system when you know that your CM tool or deploy method is running? We think so too, so I set about writing a [Chef handler](https://docs.chef.io/handlers.html) to take care of this annoyance.

# Sensu API and Stashes

Among Sensu's virtues is its [RESTful API](http://sensuapp.org/docs/0.11/api) which provides access to the data Sensu servers collect, such as clients & events.

The API also exposes an interface to [stashes](http://sensuapp.org/docs/0.11/api-stashes). Stashes are arbitrary JSON documents, so any JSON formatted data can be stored under the `/stashes` API endpoint.

Sensu handlers are expected to check the stashes under the `/stashes/silence` path when processing events, and silence events whose client has a matching stash at `/stashes/silence/$CLIENT` or whose client and check match a stash at `/stashes/silence/$CLIENT/$CHECK`.

# Chef

Chef's handler functionality can be used to trigger certain behaviors in response to specific situations during a chef-client run. At this time there are three different handler types implemented by `Chef::Handler`:

* start handlers, triggered when the defined aspect of a chef-run starts
* exception handlers, triggered when the defined aspect of a chef-run fails
* report handlers, triggered when the defined aspect of a chef-run succeeds

# Tying it all together

Combined, Sensu's stash API endpoint and Chef's exception and report handlers provide an excellent means for Chef to silence Sensu alerts during the time it is running on a node.

We achieved our goal by implementing [`Chef::Handler::Sensu::Silence`](https://github.com/needle-cookbooks/chef-sensu-handler/blob/master/files/default/handlers/sensu_handlers.rb#L40-L50), which runs as a start handler, and [`Chef::Handler::Sensu::Unsilence`](https://github.com/needle-cookbooks/chef-sensu-handler/blob/master/files/default/handlers/sensu_handlers.rb#L52-L62), which runs as both an exception and a report handler. All of this is bundled up in our [`chef-sensu-handler`](https://github.com/needle-cookbooks/chef-sensu-handler) cookbook.

The cookbook installs and configures the handler using the `node['chef_client']['sensu_api_url']` attribute. Once configured, the handler will attempt to create a stash under `/stashes/silence/$CLIENT` when the Chef run starts, and delete that stash when the Chef run fails or succeeds.

We also wanted to guard against conditions where Chef could fail catastrophically and its exception handlers might not run. To counter that possibility, the handler writes a timestamp and owner name into the stash it creates when silencing the client:

```
{ "timestamp": 1380133104, "owner": "chef" }
```

We then authored a Sensu plugin, [`check-silenced.rb`](), which compares the timestamp in existing silence stashes against a configurable timeout (in seconds). Once configured as part of our Sensu monitoring system, this plugin acts as a safety net which prevents clients from being silenced too long.


