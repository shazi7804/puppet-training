%title: Puppet Demo - How to use Hiera
%author: scott.liao
%date: 2017-12-04

-> Environment <-
=========




_Puppet Master:_
\    Domain:        master.puppet.com
\        OS:        Ubuntu 16.04 x86_64
\        IP:        172.16.0.10


_Puppet Node:_
\    Domain:        node.puppet.com
\        OS:        Ubuntu 16.04 x86_64
\        IP:        172.16.0.11


puppet version:    latest
ntp version:       latest
hiera version:     5

-------------------------------------------------

-> hiera.yaml config <-
=========

-> ## in master <-

Hiera searches the hierarchy in order

Global      - */etc/puppetlabs/puppet/hiera.yaml*

Environment - */etc/puppetlabs/code/environments/$ENV/hiera.yaml*

Module      - */etc/puppetlabs/code/environments/$ENV/modules/hiera.yaml*
\            - */etc/puppetlabs/code/modules/hiera.yaml*

*Global* -> *Environment* -> *Module*

-------------------------------------------------

-> hiera.yaml config <-
=========

-> ## in master <-

## example hiera.yaml

\```
\---
version: 5
defaults:
  datadir: data
  data_hash: yaml_data
hierarchy:
\  - name: "Per-node data"
    glob: "nodes/**/%{::trusted.certname}.yaml"

\  - name: "Per-environment data"
    path: "environment/%{::environment}.yaml"

\  - name: "Per-OS defaults"
    path: "os/%{facts.os.family}.yaml"

\  - name: "Per-Virtual data"
    path: "virtual/%{::is_virtual}.yaml"

\  - name: "Common data"
    path: "common.yaml"
\```

## variable use `facter` or puppet data.

\```
$ facter os.family
Debian
\```

so match *os/Debian.yaml*

-------------------------------------------------

-> hiera.yaml config <-
=========

-> ## in master <-

## hiera version, support 3, 4 or 5.

\```
\---
*version: 5*
\```

-------------------------------------------------

-> hiera.yaml config <-
=========

-> ## in master <-

## hiera data directory and data format.

\```
\---
version: 5
*defaults:*
  *datadir: data*
  *data_hash: yaml_data*
\```

-------------------------------------------------

-> hiera.yaml config <-
=========

-> ## in master <-

## `name` and `path` is simple of paths.  

\```
\---
version: 5
*defaults:*
  *datadir: data*
  *data_hash: yaml_data*
hierarchy:
  *- name: "Common data"*
    *path: "common.yaml"*
\```

## layer hierarchy

*├─ hiera.yaml*
*└─ data/common.yaml*

-------------------------------------------------

-> hiera.yaml config <-
=========

-> ## in master <-

## which, when using the `paths` key, is equivalent to :

\```
\---
version: 5
defaults:
  datadir: data
  data_hash: yaml_data
hierarchy:
\  - name: "Hiera data"
    *paths:*
      *- "environment/%{::environment}.yaml"*
      *- "os/%{facts.os.family}.yaml"*
      *- "is_virtual/%{::is_virtual}.yaml"*
      *- "common.yaml"*
\```

*environment* -> *os* -> *is_virtual* -> *common.yaml*

^

## layer hierarchy

*├─ hiera.yaml*
*└─ data*
    *├─ environment/
    *|  ├─ dev.yaml*
    *|  ├─ staging.yaml*
    *|  └─ production.yaml*
    *├─ os/
    *|  ├─ Debian.yaml*
    *|  └─ RedHat.yaml*
    *├─ is_virtual/
    *|  ├─ true.yaml*
    *|  └─ false.yaml*
    *└─ common/
       *└─ common.yaml*

-------------------------------------------------

-> hiera.yaml config <-
=========

-> ## in master <-

## more dynamic ways to define hierarchies and where to look for data files.

## `glob` or `globs`

\```
\---
version: 5
defaults:
  datadir: data
  data_hash: yaml_data
hierarchy:
\  - name: "Per-node data"
    *glob: "nodes/\*\*/%{::trusted.certname}.yaml"*

\  - name: "Per-environment data"
    path: "environment/%{::environment}.yaml"

\  - name: "Per-OS defaults"
    path: "os/%{facts.os.family}.yaml"

\  - name: "Per-Virtual data"
    path: "virtual/%{::is_virtual}.yaml"

\  - name: "Common data"
    path: "common.yaml"
\``` 

-------------------------------------------------

-> hiera.yaml config <-
=========

-> ## in master <-

## Another interesting way to define hierarchies is by using `mapped_paths` key.

\```
\---
version: 5
defaults:
  datadir: data
  data_hash: yaml_data
hierarchy:
\  - name: "Pre-Groups data"
    *mapped_paths: [groups, group, "groups/%{group}.yaml"]
\```

^

## If we had a `$groups` variable containing an array like ['api','sdk'] Hiera would lookup for data in the following files (relative to the defined datadir):

## layer hierarchy

*├─ hiera.yaml*
*└─ data*
    *└─ groups/
        *├─ api.yaml*
        *└─ sdk.yaml*
      
-------------------------------------------------

-> hiera.yaml data <-
=========

-> ## in master <-

## hiera data example, declare variables `$servers`.

# modules/ntp/manifests/init.pp

\```
node /^node\\\.puppet\\\.com$/ {
  class ntp (
    *Array $servers,*
  ){
    package { 'ntp':
      ...
    }

\    file { '/etc/ntp.conf':
\      ...
\      content => template("${module_name}/ntp.conf.erb"),*
\      ...
\    }

\    service { 'ntp':
\      ...
\    }
  }
}
\```

# code/environments/production/templates/ntp.conf.erb

\```
*<% @servers.each do |server| -%>
*server <%= server %> iburst
*<% end -%>
\```

-------------------------------------------------

-> hiera.yaml data <-
=========

-> ## in master <-

## for default data in common.yaml

\```
ntp::servers:
\  - 'time.google.com'
\```

^

## for environments data in environments/production.yaml

\```
ntp::servers:
\  - 'prod.time.com'
\```

^

## for nodes data in nodes/example/node.puppet.com.yaml

\```
ntp::servers:
\  - 'node.time.com'
\```

^

*node* -> *environment* -> *common*

^

*foo.puppet.com* in *production* get *prod.time.com*
^
*bar.puppet.com* in *dev* get *time.google.com*
^
*node.puppet.com* get *node.time.com*


-------------------------------------------------

-> _*Demo End*_ <-


-> Github: https://github.com/shazi7804/puppet-training <-
-> Gitbook: https://www.gitbook.com/book/shazi7804/puppet-manage-guide/details <-
-------------------------------------------------