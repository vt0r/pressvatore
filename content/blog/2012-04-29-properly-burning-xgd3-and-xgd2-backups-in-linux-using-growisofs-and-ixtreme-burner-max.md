---
date: 2012-04-29T22:38:32Z
disqus: true
share: true
title: Properly Burning XGD3 (and XGD2) Backups in Linux Using Growisofs and iXtreme
  Burner Max
url: /2012/04/29/properly-burning-xgd3-and-xgd2-backups-in-linux-using-growisofs-and-ixtreme-burner-max/
---

<strong>UPDATE 2013-02-27: I have added some basic file size checking to these functions, so you should re-add them to ~/.bashrc !</strong>

This should be a short and sweet posting since the extremely long title basically sums it up.

So, if you're reading this, it's likely you're at least fairly technically-inclined (unless you're using Ubuntu, in which case it could go either way). Based on that assumption, I'll go straight into the details.

You will obviously need to have c4eva's excellent iXtreme Burner Max firmware installed on your Liteon iHAS x24-series burner and will also need growisofs installed. To make it simpler, I've always written a small function for burning the games, since in the past it required using many command-line options and I'm lazy. So here are the functions you should add to your ~/.bashrc:

``` bash
function burnx360iso_xgd3() {
    if [ `stat -c%s "$1"` -lt 8738846720 ]
    then
        echo "This is most likely not an XGD3 image. Use burnx360iso_xgd2 instead."
    else
        growisofs -use-the-force-luke=break:2133520 -speed=4 -Z /dev/sr0=$1 ;
    fi
}

function burnx360iso_xgd2() {
    if [ `stat -c%s "$1"` -eq 8738846720 ]
    then
        echo "This looks like an XGD3 image to me. Use burnx360iso_xgd3 instead."
    else
        growisofs -use-the-force-luke=dao -use-the-force-luke=break:1913760 -dvd-compat -speed=4 -Z /dev/sr0=$1 ;
    fi
}
```

There is a slight chance you'll experience an error when using the `-dvd-compat` and `-use-the-force-luke=dao` options with XGD2 backups, in which case you should just remove that option. Very simple. Speaking of simple, here's the usage (same for both):
<pre>burnx360iso_xgd3 filename.iso</pre>
