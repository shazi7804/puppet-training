%title: Puppet Demo - How to use eyaml-hiera
%author: scott.liao
%date: 2017-12-06

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
hiera encrypt:     eyaml-hiera

-------------------------------------------------

-> eyaml-hiera install <-
=========

-> ## in master <-

# install eyaml-hiera for command

\```
$ sudo gem install hiera-eyaml
\```

# install eyaml-hiera for puppet master

\```
$ sudo puppetserver gem install hiera-eyaml
\```

-------------------------------------------------

-> eyaml-hiera config <-
=========

-> ## in master <-

# Initial key

\```
$ cd /etc/puppetlabs/puppet
$ eyaml createkeys
\```

*./keys
  *├── private_key.pkcs7.pem
  *└── public_key.pkcs7.pem

^

# eyaml-hiera config for command

\```
$ tee /etc/eyaml/config.yaml
\---
pkcs7_private_key: /etc/puppetlabs/puppet/keys/private_key.pkcs7.pem
pkcs7_public_key:  /etc/puppetlabs/puppet/keys/public_key.pkcs7.pem
\```

*/etc/eyaml/config.yaml* or *~/.eyaml/config.yaml*

-------------------------------------------------

-> eyaml-hiera config <-
=========

-> ## in master <-

# hiera support eyaml-hiera

\```
\---
version: 5

defaults:
  datadir: data
  data_hash: yaml_data

hierarchy:
  *- name: "Secret data: per-node, per-datacenter, common"
    *lookup_key: eyaml_lookup_key
    *paths:
      *- "secrets/nodes/%{trusted.certname}.eyaml"
      *- "secrets/environment/%{::environment}.eyaml"
      *- "common.eyaml"
    *options:
      *pkcs7_private_key: /etc/puppetlabs/puppet/keys/private_key.pkcs7.pem
      *pkcs7_public_key:  /etc/puppetlabs/puppet/keys/public_key.pkcs7.pem

  \- name: "Per-node data"
    glob: "nodes/**/%{::trusted.certname}.yaml"
  ...
\```

-------------------------------------------------

-> eyaml-hiera use <-
=========

-> ## in master <-

# eyaml encrypt `string`

\```
$ eyaml encrypt -s time.google.com
\```

^

\```
*block: >
    *ENC[PKCS7,...]
\```

^

## insert to `ntp::servers`

\```
*ntp::servers: >
    *ENC[PKCS7,...]
\```

-------------------------------------------------

-> eyaml-hiera use <-
=========

-> ## in master <-

# eyaml other command

*decrypt string*

\```
$ eyaml decrypt -s 'ENC[PKCS7,...]'
\```

*encrypt file*

\```
$ eyaml encrypt -f filename
\```

*decrypt file*

\```
$ eyaml decrypt -f filename
\```

*editing eyaml files*

\```
eyaml edit filename.eyaml
\```

-------------------------------------------------




-> So *hiera* provides Puppet flexible parameter configuration <-

^

-> *eyaml-hiera* to achieve private in puppet <-

^





-> *THANKS* <-

-> Github: https://github.com/shazi7804/puppet-training <-
-> Gitbook: https://www.gitbook.com/book/shazi7804/puppet-manage-guide/details <-
-------------------------------------------------