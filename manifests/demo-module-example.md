%title: Puppet Demo - manifests module example
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

-------------------------------------------------

-> Example ntp module <-
=========

-> ## in master <-

modules architecture

\```
ntp
 ├── files
 |   └── ntp.conf
 └── manifests
     ├── init.pp
     ├── install.pp
     ├── config.pp
     └── service.pp
\```

-------------------------------------------------

-> Example ntp module <-
=========

-> ## in master <-

# modules/ntp/manifests/init.pp

initial first class

\```
class ntp {
  contain ntp::install
  contain ntp::config
  contain ntp::service

  Class['::ntp::install']
  \-> Class['::ntp::config']
  ~> Class['::ntp::service']
}
\```

-------------------------------------------------

-> Example ntp module <-
=========

-> ## in master <-

# modules/ntp/manifests/install.pp

ntp install class

\```
class *ntp::install* {
  package { 'ntp':
    name   => 'ntp',
    ensure => present,
  }
}
\```

-------------------------------------------------

-> Example ntp module <-
=========

-> ## in master <-

# modules/ntp/manifests/config.pp

ntp config class

\```
class *ntp::config* {
  file { '/etc/ntp.conf':
    ensure  => file,
    owner   => 'root',
    group   => 'root',
    mode    => '0644',
    source  => "puppet:///modules/${module_name}/ntp.conf",
    require => Package['ntp'],
    notify  => Service['ntp'],
  }
}
\```

^

# modules/ntp/files/ntp.conf

ntp config file

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

-> Example ntp module <-
=========

-> ## in master <-

# modules/ntp/manifests/service.pp

ntp service class

\```
class *ntp::service* {
  service { 'ntp':
    name      => 'ntp',
    ensure    => running,
    enable    => true,
    subscribe => File['/etc/ntp.conf'],
  }
}
\```

-------------------------------------------------

-> Example ntp module <-
=========

-> ## in master <-

and then ?

^

node code is clean \!\!

\```
node /^node\\\.puppet\\\.com$/ {
  *include ::ntp*
}
\```

-------------------------------------------------




-> _*Demo End*_ <-


-> Github: https://github.com/shazi7804/puppet-training <-
-------------------------------------------------