%title: Puppet Demo - manifests example install
%author: scott.liao
%date: 2017-11-15

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
resource:          package, service, file

-------------------------------------------------

-> Example install ntp <-
=========

-> ## in master <-

# manifests/site.pp

Define the node


\```
node 'node.puppet.com' {}
\```

-------------------------------------------------

-> Example install ntp <-
=========

-> ## in master <-

# manifests/site.pp

or use regular expression


\```
node */^node\\\.puppet\\\.com$/* {}
\```

-------------------------------------------------

-> Example install ntp <-
=========

-> ## in master <-

# manifests/site.pp

*package* resource define install packages.


\```
node /^node\\\.puppet\\\.com$/ {
  package { 'ntp':
    name   => 'ntp',
    ensure => present,
  }
}
\```

same *apt-get install ntp* or *yum install ntp*

-------------------------------------------------

-> Example install ntp <-
=========

-> ## in master <-

# manifests/site.pp

*file* resource define file status.


\```
node /^node\\\.puppet\\\.com$/ {
  package { 'ntp':
    name   => 'ntp',
    ensure => present,
  }

  *file { '/etc/ntp.conf':
    *ensure => file,
    *owner  => 'root',
    *group  => 'root',
    *mode   => '0644',
    *source => "puppet:///modules/${module_name}/ntp.conf"
  *}
}
\```

- *puppet:* URIs, which point to files in modules or Puppet file server mount points.
- Fully qualified paths to locally available files (including files on NFS shares or Windows mapped drives).
- *file:* URIs, which behave the same as local file paths.
- *http:* URIs, which point to files served by common web servers

^


# files/ntp.conf

\```
tinker panic 0
disable monitor
restrict -4 default kod nomodify notrap nopeer noquery
restrict -6 default kod nomodify notrap nopeer noquery
restrict 127.0.0.1
restrict ::1
server time.google.com iburst
driftfile /var/lib/ntp/drift
\```


-------------------------------------------------
-> Example install ntp <-
=========

-> ## in master <-

# manifests/site.pp

*service* resource define service status.


\```
node /^node\\\.puppet\\\.com$/ {
  package { 'ntp':
    name   => 'ntp',
    ensure => present,
  }

  file { '/etc/ntp.conf':
    ensure => file,
    owner  => 'root',
    group  => 'root',
    mode   => '0644',
    source => "puppet:///modules/${module_name}/ntp.conf"
  }

  *service { 'ntp':
    *name   => 'ntp',
    *ensure => running,
    *enable => true,
  *}
}
\```

^

resource order not define \!\!

-------------------------------------------------

-> Example install ntp <-
=========

-> ## in master <-

# manifests/site.pp

use *->* or *~>* define older


\```
node /^node\\\.puppet\\\.com$/ {
  package { 'ntp':
    name   => 'ntp',
    ensure => present,
  } *->*
  file { '/etc/ntp.conf':
    ensure => file,
    owner  => 'root',
    group  => 'root',
    mode   => '0644',
    source => "puppet:///modules/${module_name}/ntp.conf"
  } *~>*
  service { 'ntp':
    name   => 'ntp',
    ensure => running,
    enable => true,
  }
}
\```

*->* (ordering arrow; a hyphen and a greater-than sign) — Applies the resource on the left before the resource on the right.
*~>* (notifying arrow; a tilde and a greater-than sign) — Applies the resource on the left first. If the left-hand resource changes, the right-hand resource will refresh.

-------------------------------------------------

-> Example install ntp <-
=========

-> ## in master <-

# manifests/site.pp

use *metaparameters* define older


\```
node /^node\\\.puppet\\\.com$/ {
  package { 'ntp':
    name   => 'ntp',
    ensure => present,
    *before => File['/etc/ntp.conf'],*
  }

  file { '/etc/ntp.conf':
    ensure  => file,
    owner   => 'root',
    group   => 'root',
    mode    => '0644',
    source  => "puppet:///modules/${module_name}/ntp.conf",
    *require => Package['ntp'],*
    *notify  => Service['ntp']*
  }

  service { 'ntp':
    name      => 'ntp',
    ensure    => running,
    enable    => true,
    *subscribe => File['/etc/ntp.conf'],*
  }
}
\```

*before* — Applies a resource before the target resource.
*require* — Applies a resource after the target resource.
*notify* — Applies a resource before the target resource. The target resource refreshes if the notifying resource changes.
*subscribe* — Applies a resource after the target resource. The subscribing resource refreshes if the target resource changes.


-------------------------------------------------




-> _*Demo End*_ <-


-> Github: https://github.com/shazi7804/puppet-training <-
-------------------------------------------------