# Varnish

[![build
Status](https://travis-ci.org/finalsite/finalsite-varnish.png?branch=master)](https://travis-ci.org/finalsite/finalsite-varnish)

#### Table of Contents

1. [Overview](#overview)
3. [Setup - The basics of getting started with varnish](#setup)
    * [What varnish affects](#what-varnish-affects)
    * [Setup requirements](#setup-requirements)
    * [Beginning with varnish](#beginning-with-varnish)
4. [Usage - Configuration options and additional functionality](#usage)
5. [Reference - An under-the-hood peek at what the module is doing and how](#reference)
5. [Limitations - OS compatibility, etc.](#limitations)
6. [Development - Guide for contributing to the module](#development)

## Overview

Installs, configures and runs the Varnish and VarnishNCSA services.


## Setup

This module installs varnish from whatever repository you have configured
on your system already. To be clear: this module does *not* attempt to
install a package repository for you.

To install, simply run:

    puppet module install finalsite-varnish

### What varnish affects

* Varnish Package
* Varnish & Varnishncsa services
* /etc/varnish/default.vcl
* /etc/varnish/varnish.params
* /etc/varnish/secret
* /etc/systemd/system/varnishncsa.service
* /etc/sysconfig/varnishncsa

### Setup Requirements

Required Puppet Modules:

* ripienaar/module_data
* puppetlabs/stdlib

### Beginning with Varnish

As puppet classes

```puppet

  class { 'varnish::config':
    package_ensure  => '4.1.3-1.el7',
    service_manage  => false,
    vcl_template    => 'mymodule/my_default_vcl.erb',
    vcl_config_file => '/etc/varnish/myconfig.vcl',
    listen_address  => $::ipaddress_eth0,
    listen_port     => 8080,
    storage_spec    => malloc,80%,
    parameters      => { min_threads => 50, max_threads => 1000, thread_timeout => 300 },
    runtime_options => [
      '-j unix:varnish',
      '-t 120'
    ]
  }
```

or using hiera yaml files:

```yaml
---
classes:
  varnish

varnish::config::package_ensure: '4.1.3-1.el7'
varnish::config::service_manage: false
varnish::config::vcl_template: 'mymodule/my_default_vcl.erb'
varnish::config::vcl_config_file: '/etc/varnish/myconfig.vcl'
varnish::config::listen_address: '%{::ipaddress_eth0}'
varnish::config::listen_port: 8080
varnish::config::storage_spec: 'malloc,80%'
varnish::config::parameters:
  min_threads: 50
  max_threads: 1000
  thread_timeout: 300
varnish::config::runtime_options:
  - '-j unix:varnish'
  - '-t 120'
```

#### Storage Specifications
The `storage_spec` parameter is normalized prior to being included in the final configuration, with percentage
based sizes converted for `malloc`, `file` and `persistent` storage backend types.

This varnish modules provides additional facter facts to facilitate this for file based storage backends. These
facts are based on the existing mount points on your system for filesystem types including `ext[234]`, `btrfs`,
`xfs`, `reiser4`, and `zfs`. If the filesystem you're backing varnish with isn't included in that list, then this won't
work for you.

If you are using `malloc`, percentages will be based on the total available capacity of the server as reported
by the `memorysize_mb` fact. If you are using `file`, or `persistent` storage backends and you provide a size,
then the facts provided by this module will be used to determine the capacity of the path, and whether or not
there is sufficient space for the requested size you provided.

### Usage

All parameters are optional, but at minimum it is suggested you set a
secret.

|Parameter|Type|Defaults|Description|
|---------|----|--------|-----------|
|package_name|String|`varnish`|Name of the varnish package to install|
|package_ensure|String|`installed`|Version to pass to ensure field (see: [package#ensure](https://docs.puppet.com/puppet/3.7/reference/type.html#package-attribute-ensure)]|
|package_manage|Bool|`true`|Whether or not to let puppet manage the varnish package|
|service_ensure|String|`running`|Should varnish be running? (see: [service#ensure](https://docs.puppet.com/puppet/3.7/reference/type.html#service-attribute-ensure))|
|service_enable|String|`true`|Should varnish be enabled at boot time? (see: [service#enable](https://docs.puppet.com/puppet/3.7/reference/type.html#service-attribute-enable))|
|service_manage|Bool|`true`|Whether or not to let puppet manage varnish services|
|vcl_config_file|Path|`/etc/varnish/default.vcl`|Path to the varnish VCL config file|
|secret_file|Path|`/etc/varnish/secret`|Path to the file containing the CLI admin secret|
|secret_pass|String|`<autogenerated>`|Password to use with varnish admin CLI|
|params_path|Path|`/etc/varnish/varnish.params`|Startup and runtime parameters config file|
|params_template|String|`module/varnish/varnish.params.erb`|Path to an alternate varnish.params ERB template|
|listen_address|IP Address|`0.0.0.0`|IP Address for Varnish to listen on|
|listen_port|Integer|`6081`|Port for Varnish to listen on|
|admin_listen_address|IP Address|`127.0.0.1`|IP Address for Varnish Admin CLI to listen on|
|admin_listen_port|Integer|`6082`|Port for Varnish Admin CLI to listen on|
|storage_spec|String|`malloc`|Varnish storage specification (see: [Varnish Storage Backends](https://www.varnish-cache.org/docs/4.1/users-guide/storage-backends.html))|
|vcl_content|String|`undef`|VCL configuration (mutually exclusive with vcl_template and vcl_source)|
|vcl_template|String|`undef`|Puppet template for VCL configuration (mutually exclusive with vcl_content and vcl_source)|
|vcl_source|String|`undef`|Puppet source file for VCL configuration (mutually exclusive with vcl_content and vcl_template)|
|runtime_user|String|`varnish`|User to run varnish as|
|default_ttl|Integer|120|Default TTL to use for all cached objects|
|parameters|Hash|`{"min_threads" => 5, "max_threads" => 500, "thread_timeout" => 300}|Various parameters to pass to varnish (see: [Runtime Parameters](https://www.varnish-cache.org/docs/4.1/reference/varnishd.html#run-time-parameters))|
|runtime_options|Array|['-r cc_command,vcc_allow_inline_c,vmod_dir', '-t 120', '-j unix,user=varnish']|Runtime Options to pass to the varnish daemon on startup (see: [Runtime Options](https://www.varnish-cache.org/docs/4.1/reference/varnishd.html#options))|

## Limitations

*Tested and verified working on CentOS 7 only.*

## Development

* Always run `bundle install` and `bundle exec rake spec`
* Always create tests for functionality changes or additions
* Squash all commits prior to submitting Pull Requests
* Install `rabbitt-githooks` and attach them to your fork.
    ```
    # if you've run bundle install, then this is unnecessary
    gem install rabbitt-githooks

    # run this from the root of the varnish module directory
    githooks attach -p .githooks
    ```
