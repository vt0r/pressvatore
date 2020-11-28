---
date: 2012-06-05T21:37:40Z
disqus: true
share: true
title: Getting Juniper Network Connect to Work on 64-bit Linux
url: /2012/06/05/getting-juniper-network-connect-to-work-on-64-bit-linux/
---

So let's say you've got a Juniper MAG or SRX or similar unit at work that supports SSL VPN. Now let's say you're running 64-bit Linux at home and need to connect to said VPN. I doubt it makes much difference which distro you use, but I'm going to assume you're really cool and are using a Debian-based one. So there are a few necessary steps required to get this Java-based VPN client running, and if you follow this tutorial, you'll have it working in no time.

First, you will need to ensure the 'nosudo' option is NOT enabled for /home in /etc/fstab. Please don't curse at me. I didn't write this app. The reason you have to take a major step back in security is because the app needs to modify routing tables, overwrite your /etc/hosts and /etc/resolv.conf, load the tun module and create a tun device. It will install two setuid root binaries into ~/.juniper\_networks/network_connect/. If the option is enabled, remove the option and remount /home:
<pre>
mount -o remount /home
</pre>

Next, you will need to ensure you have all the gcc and g++ 32-bit packages installed (gcc-multilib and g++-multilib in Debian/Ubuntu) as well as the 32-bit libraries (ia32-libs) and libstdc++ (libstdc++6 on Debian unstable).
<pre>
apt-get install gcc-multilib g++-multilib ia32-libs libstdc++
</pre>

Now install xterm. Yes, xterm. I'm not kidding. 
<pre>
apt-get install xterm
</pre>

Also make sure sudo is installed and configured for your user, though it's rare to find a distro that doesn't install sudo by default anymore.

And now for the fun part. You should probably already have the 64-bit Java JRE installed, and we'll assume you install it manually to stay up to date, so let's unzip it into /opt/jre1.7.0_04-64 if you don't already have it somewhere. Also get the 32-bit version and put it in /opt/jre1.7.0_04-32 . Move /opt/jre1.7.0_04-64/bin/java to /opt/jre1.7.0_04-64/bin/java.orig and create a bash script at /opt/jre1.7.0_04-64/bin/java :

``` bash
#!/bin/bash

VERSION="1.7.0_04"

if [ $3x = "NCx" ]
then
    /opt/jre${VERSION}-32/bin/java "$@"
else
    /opt/jre${VERSION}-64/bin/java.orig "$@"
fi
```

Make sure your new script is executable:
``` bash
chmod 0755 /opt/jre1.7.0_04-64/bin/java
```

The final step is to create a symlink to the 64-bit version of Java's JRE plugin in Firefox's plugins directory (which you will create if it doesn't exist).

This assumes you're using 64-bit Firefox and you've installed it in /opt/firefox:
``` bash
mkdir /opt/firefox/plugins 2>/dev/null
ln -s /opt/jre1.7.0_04-64/lib/amd64/libnpjp2.so /opt/firefox/plugins/libnpjp2.so
```


Problem solved!

<strong>UPDATE: Tested and Juniper NC works with the latest (1.7.0_04) JRE as of this writing. Updated version numbers above</strong>

Credit for the script goes to the following forum post on Ubuntu forums:

<a href="http://ubuntuforums.org/showpost.php?p=11189826&postcount=441">http://ubuntuforums.org/showpost.php?p=11189826&postcount=441</a>
