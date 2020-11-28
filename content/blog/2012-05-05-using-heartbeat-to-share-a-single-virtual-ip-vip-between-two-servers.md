---
date: 2012-05-05T10:45:08Z
disqus: true
share: true
title: Using Heartbeat to Share a Single Virtual IP (VIP) Between Two Servers
url: /2012/05/05/using-heartbeat-to-share-a-single-virtual-ip-vip-between-two-servers/
---

Just did a high availability cluster setup a few weeks ago and though the web servers use the traditional heartbeat/ldirectord setup, there needed to be high availability between the two master/master database servers as well.

Yes, this can be done with ldirectord for MySQL, but if you also have other services like Memcached, or Redis, etc, you may just want to share a VIP between the servers.

This is especially true if you're connecting to these servers internally. Connecting via the VIP will only connect to one server, so this is not a load-balancing setup, but rather simply a high availability solution (failover).

You just need to install heartbeat and add lines in /etc/hosts with each node's real address (so they can talk to each other without DNS lookups and also so they use the right interface if it's an actual resolving hostname that routes to a different one). Then configure the following files in /etc/ha.d:

<strong>authkeys:</strong>
<pre>
auth 1
1 sha1 SoMeSeCrEtYoUcAnUsEtOkEePsAfE
</pre>
This is just a key that authenticates other hosts. It must remain the same between all servers.

<strong>ha.cf:</strong>
<pre>
logfacility     local0
auto_failback on
keepalive 2
deadtime 10
udpport 694
ucast eth0 db2.yourhostname.com
node    db1.yourhostname.com
node    db2.yourhostname.com
</pre>
Here you specify options that determine things such as:
<ul>
	<li>Automatically falling back to first host (auto_failback)</li>
	<li>Delay between heartbeats (keepalive)</li>
	<li>Amount of failed heartbeat time before server takes the IP (deadtime)</li>
	<li>Port to connect on - no reason to change this (udpport)</li>
	<li>Host to look for and what interface to use to connect (ucast)</li>
	<li>Nodes within the cluster (can also be specified on one line for ugly configs)</li>
</ul>
<strong>haresources:</strong>
<pre>
db1.yourhostname.com 200.200.200.200/26/eth0
</pre>
This file should match on both hosts and contains the hostname (uname -n) of the main server followed by the shared IP (VIP)/prefix/interface. Normally, you include the actual resource which should be monitored at the end of the line. Since you're just moving an IP back and forth for this setup, you can leave the last field blank.

<strong>NOTE: Don't forget to make your services listen on 0.0.0.0 !</strong>
