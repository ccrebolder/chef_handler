# chef_handler Cookbook

[![Build Status](https://travis-ci.org/chef-cookbooks/chef_handler.svg?branch=master)](https://travis-ci.org/chef-cookbooks/chef_handler) [![Cookbook Version](https://img.shields.io/cookbook/v/chef_handler.svg)](https://supermarket.chef.io/cookbooks/chef_handler)

Provides a resource for installing and automatically loading [Chef report and exception handlers](http://docs.chef.io/handlers.html).

## Requirements

### Platforms

- Debian/Ubuntu
- RHEL/CentOS/Scientific/Amazon/Oracle
- Windows

### Chef

- Chef 12.7+

### Cookbooks

- none

## Attributes

`node['chef_handler']['handler_path']` - location to drop off handlers directory, default is a folder named 'handlers' in Chef's file cache directory

## Resources

### chef_handler

Requires, configures and enables handlers on the node for the current Chef run. Also has the ability to pass arguments to the handlers initializer. This allows initialization data to be pulled from a node's attribute data.

It is best to declare `chef_handler` resources early on in the compile phase so they are available to fire for any exceptions during the Chef run. If you have a base role you would want any recipes that register Chef handlers to come first in the run_list.

#### Actions

- `:enable:` Enables the Chef handler for the current Chef run on the current node
- `:disable:` Disables the Chef handler for the current Chef run on the current node

#### Attribute Parameters

- `class_name:` name attribute. The name of the handler class (can be module name-spaced).
- `source:` full path to the handler file. can also be a gem path if the handler ships as part of a Ruby gem. can also be nil, in which case the file must be loaded as a library.
- `arguments:` an array of arguments to pass the handler's class initializer
- `type:` type of Chef Handler to register as, i.e. :report, :exception or both. default is `:report => true, :exception => true`

#### Example

```ruby
  # register the Chef::Handler::JsonFile handler
  # that ships with the Chef gem
  chef_handler 'Chef::Handler::JsonFile' do
    source 'chef/handler/json_file'
    arguments path: '/var/chef/reports'
    action :enable
  end

  # do the same but during the compile phase
  chef_handler 'Chef::Handler::JsonFile' do
    source 'chef/handler/json_file'
    arguments path: '/var/chef/reports'
    action :nothing
  end.run_action(:enable)

  # handle exceptions only
  chef_handler 'Chef::Handler::JsonFile' do
    source 'chef/handler/json_file'
    arguments path: '/var/chef/reports'
    type exception: true
    action :enable
  end

  # enable the MyCorp::MyLibraryHandler handler which was dropped off in a
  # standard chef library file.
  chef_handler 'MyCorp::MyLibraryHandler' do
    action :enable
  end
```

## Usage

### default

This cookbook's default recipe has been deprecated. To setup your own handlers you should instead use the chef_handler resource documented above within your own cookbook to define handlers.

### json_file

Leverages the `chef_handler` LWRP to automatically register the `Chef::Handler::JsonFile` handler that ships as part of Chef. This handler serializes the run status data to a JSON file located at `/var/chef/reports`.

## Unit Testing

chef_handler provides built in [chefspec](https://github.com/chefspec/chefspec) matchers for assisting unit tests. These matchers will only be loaded if ChefSpec is already loaded. Following is an example of asserting against the jsonfile handler:

```ruby
  expect(runner).to enable_chef_handler('Chef::Handler::JsonFile').with(
    source: 'chef/handler/json_file',
    arguments: { path: '/var/chef/reports' },
    type: { exception: true }
  )
```

## License & Authors

**Author:** Cookbook Engineering Team ([cookbooks@chef.io](mailto:cookbooks@chef.io))

**Copyright:** 2011-2017, Chef Software, Inc.

```
Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
