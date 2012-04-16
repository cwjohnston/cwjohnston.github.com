---
layout: post
title: "Using deploy_wrapper to set the stage for deployments via Chef"
date: 2012-04-10 13:25
comments: true
categories: chef deploy_wrapper
---
Chef's `deploy` and `deploy_revision` resources provide a useful mechanism for deploying applications as part of a chef-client or chef-solo run, without depending on an external system (e.g. Capistrano.) Many Chef users learning to use these resources for the first time will find that they also need to install an SSH deploy key and an SSH wrapper script for Git before they can make effective use of these deploy resources, and that the Chef wiki doesn't provide much documentation around this issue.

Enter [`deploy_wrapper`](http://github.com/cwjohnston/chef-deploy_wrapper): a Chef definition which handles the installation of an SSH deploy key and SSH wrapper script to be used by a `deploy` or `deploy_revision` resource.

Before `deploy_wrapper`, a recipe to configure the required resources to make an automated `deploy` or `deploy_revision` possible might look something like this:

``` ruby

directory '/root/.ssh' do
  owner "root"
  group "root"
  mode 0640
end

directory '/opt/myapp/shared' do
  owner "root"
  group "root"
  mode 0755
  recursive true
end

deploy_key = data_bag_item('keys', 'myapp_deploy_key')

template "/root/.ssh/myapp_deploy_key" do
  source "deploy_key.erb"
  owner "root"
  group "root"
  mode 0600
  variables({ :deploy_key => deploy_key })
end

template "/opt/myapp/shared/myapp_deploy_wrapper.sh" do
  source "ssh_wrapper.sh.erb"
  owner "root"
  group "root"
  mode 0755
  variables({
    :deploy_key_path => "/root/.ssh/myapp_deploy_key"
  })
end

deploy_revision "/opt/myapp" do
  repository node['myapp']['repository']
  revision node['myapp']['revision']
  ...
  ssh_wrapper "/opt/myapp/shared/myapp_deploy_wrapper.sh"
end

```

Not counting the source to template files for these resources, thats almost 30 lines of code just to set the stage for a deployment. It didn't take long for me to grow tired of reusing this rather verbose pattern across a growing number of recipes.

Here's how I accomplish the same thing with the `deploy_wrapper` definition:

``` ruby
deploy_key = data_bag_item('keys', 'myapp_deploy_key')

deploy_wrapper "myapp" do
  ssh_wrapper_dir "/opt/myapp/shared"
  ssh_key_dir "/root/.ssh"
  ssh_key_data deploy_key
  sloppy true
end

deploy_revision "/opt/myapp" do
  repository node['myapp']['repository']
  revision node['myapp']['revision']
  ...
  ssh_wrapper "/opt/myapp/shared/myapp_deploy_wrapper.sh"
end
```

Much better, right? Well, a lot shorter anyway. Now let's talk about what the `deploy_wrapper` parameters used in the above example are doing. 

The `ssh_key_dir` and `ssh_wrapper_dir` parameters specify directories which will be created by Chef. In the case of `ssh_wrapper_dir`, the git SSH wrapper script will automatically be created in this directory following the pattern "APPNAME_deploy_wrapper.sh", using the value of the name parameter (in this case, `myapp`) in place of "APPNAME". 

Similarly, an SSH key file containing the data passed to the `ssh_key_data` parameter will be created in the directory specified as the value for the `ssh_key_dir` parameter. The key file will be named following the pattern "APPNAME_deploy_key", using the value of the name parameter (`myapp`) in place of "APPNAME".

The `sloppy` parameter is the only optional one. Because the default configuration of most most ssh installations is to require manual verification when accepting a remote host's key for the first time, the `sloppy` parameter allows one to toggle key checking (`StrictHostKeyChecking`) on or off. 

When the value for `sloppy` is `true`, the wrapper script will accept any host key without prompting. The default value for `sloppy` is `false`, meaning that additional Chef resources, or ... *\*gasp\** ... manual intervention, will be required in order to set up a `known_hosts` file before deployments can run successfully.
