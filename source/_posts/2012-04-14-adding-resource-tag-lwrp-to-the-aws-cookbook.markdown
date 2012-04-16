---
layout: post
title: "Adding resource_tag LWRP to the aws cookbook"
date: 2012-04-14 15:08
comments: true
categories: chef, lwrp, aws
---

This weekend I decided that I'd had enough with reusing the same pattern for manipulating tags on EC2 instances across multiple recipes. Since Opscode already publishes an `aws` cookbook with providers for other AWS resources, I figured it would be worthwhile to create a provider for manipulating these tags and contribute it back upstream.

The result of this Saturday project is the `resource_tag` LWRP. Source available [here](https://github.com/cwjohnston/chef-opscode-aws/commit/fac8434abc380a903397be2605c48919fd128e6f), Opscode ticket [here](http://tickets.opscode.com/browse/COOK-1195).

## Actions

* `add` - Add tags to a resource.
* `update` - Add or modify existing tags on a resource -- this is the default action.
* `remove` - Remove tags from a resource, but only if the specified values match the existing ones.
* `force_remove` - Remove tags from a resource, regardless of their values.

## Attribute Parameters

* `aws_secret_access_key`, `aws_access_key` - passed to `Opscode::AWS:Ec2` to authenticate, required.
* `tags` - a hash of key value pairs to be used as resource tags, (e.g. `{ "Name" => "foo", "Environment" => node.chef_environment }`,) required.
* `resource_id` - resources whose tags will be modified. The value may be a single ID as a string or multiple IDs in an array. If no `resource_id` is specified the name attribute will be used.

## Usage

`resource_tag` can be used to manipulate the tags assigned to one or more AWS resources, i.e. ec2 instances, ebs volumes or ebs volume snapshots.

``` ruby
include_recipe "aws"
aws = data_bag_item("aws", "main")

# Assigining tags to a node to reflect it's role and environment:
aws_resource_tag node['ec2']['instance_id'] do
  aws_access_key aws['aws_access_key_id']
  aws_secret_access_key aws['aws_secret_access_key']
  tags({"Name" => "www.example.com app server",
        "Environment" => node.chef_environment})
  action :update
end

# Assigning a set of tags to multiple resources, e.g. ebs volumes in a disk set:
aws_resource_tag 'my awesome raid set' do
  aws_access_key aws['aws_access_key_id']
  aws_secret_access_key aws['aws_secret_access_key']
  resource_id [ "vol-d0518cb2", "vol-fad31a9a", "vol-fb106a9f", "vol-74ed3b14" ]
  tags({"Name" => "My awesome RAID disk set",
        "Environment" => node.chef_environment})
end
```

When setting tags on the node's own EC2 instance, I recommend wrapping `resource_tag` resources in a conditional like `if node.has_key?('ec2')` so that your recipe will still run on Chef nodes outside of EC2 as well.
