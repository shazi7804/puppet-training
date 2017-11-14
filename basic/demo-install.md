%title: Puppet Demo - Master and Agent
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

-------------------------------------------------

-> Domain setting <-
=========

-> ## in master & node <-


*scott@master*:$ cat /etc/hosts

127.0.0.1       localhost
172.16.0.10     master.puppet.com
172.16.0.11     node.puppet.com

-------------------------------------------------

-> Time setting <-
=========

-> ## in master & node <-


*scott@master*:$ sudo ntpdate time.stdtime.gov.tw

*scott@master*:$ sudo timedatectl set-timezone Asia/Taipei

-------------------------------------------------

-> Install Puppet Master <-
=========

-> ## in master <-


*scott@master*:$ wget https://apt.puppetlabs.com/puppet5-release-xenial.deb
^

*scott@master*:$ sudo dpkg -i puppet5-release-xenial.deb
^

*scott@master*:$ sudo apt-get update
^

*scott@master*:$ sudo apt-get install puppetserver

-------------------------------------------------

-> Modify Master memory <-
=========

-> ## in master <-


*scott@master*:$ cat /etc/default/puppetserver
JAVA_ARGS="-Xms2g -Xmx2g"

-------------------------------------------------

-> Modify Master config <-
=========

-> ## in master <-


*scott@master*:$ cat /etc/puppetlabs/puppet/puppet.conf

[main]
dns_alt_names = master.puppet.com
server = master.puppet.com
certname = master.puppet.com
environment = production

-------------------------------------------------

-> Configuration Settings <-
=========

-> ## in master <-


*dns_alt_names*       list of alternate DNS names for Puppet Server.
*certname*            When a node requests a certificate from the CA puppet master.
*server*              Puppet master server.
*environment*         Puppet running environment.







[Configuration Reference](https://puppet.com/docs/puppet/5.3/configuration.html)

-------------------------------------------------

-> Start Master service <-
=========

-> ## in master <-


*scott@master*:$ sudo systemctl start puppetserver

*scott@master*:$ sudo systemctl enable puppetserver

-------------------------------------------------

-> Verify Master service <-
=========

-> ## in master <-


*scott@master*:$ ss -tunlp | grep 8140
tcp  LISTEN  0  50  :::8140  :::*  users  (("java",pid=27866,fd=32))

-------------------------------------------------

-> Initial Master CA credentials <-
=========

-> ## in master <-



*scott@master*:$ sudo /opt/puppetlabs/bin/puppet master --verbose --no-daemonize

The command initializes the CA certificate in `/etc/puppetlabs/puppet/ssl/ca`

-------------------------------------------------

-> Enable Firewall 8140 port <-
=========

-> ## in master <-



*scott@master*:$ sudo ufw allow 8140

Policy allow TCP 8140.

-------------------------------------------------

-> Install Puppet agent <-
=========

-> ## in node <-


*scott@node*:$ wget https://apt.puppetlabs.com/puppet5-release-xenial.deb
^

*scott@node*:$ sudo dpkg -i puppet5-release-xenial.deb
^

*scott@node*:$ sudo apt-get update
^

*scott@node*:$ sudo apt-get install puppet-agent

-------------------------------------------------

-> Modify Node config <-
=========

-> ## in node <-


*scott@node*:$ cat /etc/puppetlabs/puppet/puppet.conf

[main]
server = master.puppet.com
certname = node.puppet.com
environment = production
runinterval = 2h

-------------------------------------------------

-> generator node certificate <-
=========

-> ## in node <-


*scott@node*:$ sudo /opt/puppetlabs/bin/puppet agent --test

-------------------------------------------------

-> Master trust node certname <-
=========

-> ## in master <-

*scott@master*:$ sudo /opt/puppetlabs/bin/puppet cert list -all
^
+ "node.puppet.com" (SHA256) 5D: ...

Waiting signin node certificate

^
*scott@master*:$ sudo /opt/puppetlabs/bin/puppet cert sign agent.puppet.com
^
Notice: node.puppet.com has a waiting certificate request
Notice: Signed certificate request for node.puppet.com

-------------------------------------------------

-> node pull catalogs <-
=========

-> ## in node <-

*scott@node*:$ sudo /opt/puppetlabs/bin/puppet agent --test
^
## Info: Using configured environment 'production'
## Info: Retrieving pluginfacts
## Info: Retrieving plugin
## Info: Loading facts
## Info: Caching catalog for node.puppet.com
## Info: Applying configuration version '1510671695'
Notice: Applied catalog in 1.87 seconds

Successful update catalog.

-------------------------------------------------



-> Demo End <-



-------------------------------------------------