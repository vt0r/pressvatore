---
date: 2012-07-05T09:30:21Z
disqus: true
share: true
title: Generating Random Passwords in Bash Using Your System's Random Source
url: /2012/07/05/generating-random-passwords-using-your-systems-random-source/
---

Today, we will re-invent the wheel with a simple bash function to generate some passwords and a few hashes as well. Why would I bother coding something when several good programs already exist to perform the same functionality? Well, that's simple: I use this most on systems where I cannot install software. Any user that has SSH access can edit their own .bashrc , so I decided since random password generation is a daily task I perform at my job and elsewhere, it made sense to have a good way to generate passwords. There are a few hashes you can use this for as well. This function will generate the hash used for cookie authentication in phpMyAdmin and it will also generate the hashes used to secure authentication via Wordpress (to be used in wp-config.php). Paste this function in its entirety into your .bashrc or /etc/profile , /etc/bashrc , /etc/bash.bashrc (wherever you feel it should go):

``` bash
function pwgen() {
    case "$1" in
    -s|--symbols)
        export SYMBOLS="1" PHPMA="0" WORDPRESS="0" YUBIKEY="0"
    ;;
    -a|--alphanumeric)
        export SYMBOLS="0" PHPMA="0" WORDPRESS="0" YUBIKEY="0"
    ;;
    -p|--phpmyadmin)
        export PHPMA="1" WORDPRESS="0" YUBIKEY="0"
    ;;
    -w|--wordpress)
        export WORDPRESS="1" PHPMA="0" YUBIKEY="0"
    ;;
    -y|--yubikey)
        export YUBIKEY="1" PHPMA="0" WORDPRESS="0"
    ;;
    -h|--help|*)
        echo $"Usage: pwgen <OPTIONS> [length] [number] (length and number optional)"
        echo $""
        echo $"OPTIONS (MUST SPECIFY ONE!):"
        echo $"--symbols (-s)               Add symbols to output (NOT FOR MYSQL!)"
        echo $"--alphanumeric (-a)          Alphanumeric only"
        echo $"--phpmyadmin (-p)            Generate phpMyAdmin Blowfish secret (for cookie auth)"
        echo $"--wordpress (-w)             Generate Wordpress encryption keys (wp-config.php)"
        echo $"--yubikey (-y)               Generate Public/Private/Secret keys for Yubikey OTP"
        echo $"--help (-h)                  Display this usage information"
        echo $""
        echo $"If no length or number are defined, a default length of 19 and number of 1 will be used."
        return 1
    ;;
    esac

if [ -z "$2" ]
then
     export LEN="19"
else
     export LEN="$2"
fi

if [ -z "$3" ]
then
     export NUM="1"
else
     export NUM="$3"
fi

if [ "$PHPMA" == "1" ]
then
     tr -dc 'a-zA-Z0-9-_!@#$%^&*/\()_+{}|:<>?=' < /dev/random | fold -w 46 | head -n 1
     return 0
elif [ "$WORDPRESS" == "1" ]
then
     WP1=$(tr -dc 'a-zA-Z0-9-_!@#$%^&*/\()_+{}|:<>?=' < /dev/random | fold -w 64 | head -n 1)
     WP2=$(tr -dc 'a-zA-Z0-9-_!@#$%^&*/\()_+{}|:<>?=' < /dev/random | fold -w 64 | head -n 1)
     WP3=$(tr -dc 'a-zA-Z0-9-_!@#$%^&*/\()_+{}|:<>?=' < /dev/random | fold -w 64 | head -n 1)
     WP4=$(tr -dc 'a-zA-Z0-9-_!@#$%^&*/\()_+{}|:<>?=' < /dev/random | fold -w 64 | head -n 1)
     WP5=$(tr -dc 'a-zA-Z0-9-_!@#$%^&*/\()_+{}|:<>?=' < /dev/random | fold -w 64 | head -n 1)
     WP6=$(tr -dc 'a-zA-Z0-9-_!@#$%^&*/\()_+{}|:<>?=' < /dev/random | fold -w 64 | head -n 1)
     WP7=$(tr -dc 'a-zA-Z0-9-_!@#$%^&*/\()_+{}|:<>?=' < /dev/random | fold -w 64 | head -n 1)
     WP8=$(tr -dc 'a-zA-Z0-9-_!@#$%^&*/\()_+{}|:<>?=' < /dev/random | fold -w 64 | head -n 1)
     echo "define('AUTH_KEY',         '$WP1');"
     echo "define('SECURE_AUTH_KEY',  '$WP2');"
     echo "define('LOGGED_IN_KEY',    '$WP3');"
     echo "define('NONCE_KEY',        '$WP4');"
     echo "define('AUTH_SALT',        '$WP5');"
     echo "define('SECURE_AUTH_SALT', '$WP6');"
     echo "define('LOGGED_IN_SALT',   '$WP7');"
     echo "define('NONCE_SALT',       '$WP8');"
     return 0
elif [ "$YUBIKEY" == "1" ]
then
     #echo "Public Identity: vv$(tr -dc 'a-f0-9' < /dev/random | fold -w 10 | head -n 1 | tr "[0123456789abcdef]" "[cbdefghijklnrtuv]")"
     #echo "Private Identity: $(tr -dc 'a-f0-9' < /dev/random | fold -w 12 | head -n 1)"
     #echo "Secret Key: $(tr -dc 'a-f0-9' < /dev/random | fold -w 32 | head -n 1)"
     echo "OTP AES Key: $(tr -dc 'a-f0-9' < /dev/random | fold -w 40 | head -n 1)"
elif [ "$SYMBOLS" == "1" ]
then
     tr -dc 'a-zA-Z0-9-_!@#$%^&*/\()_+{}|:<>?=' < /dev/random | fold -w "$LEN" | head -n "$NUM"
else
     tr -dc 'a-zA-Z0-9' < /dev/random | fold -w "$LEN" | head -n "$NUM"
fi
}
```

Now, you can name this whatever you like. There is already a very good program called pwgen which I use quite often, so on my system, this is actually called 'salpwgen' to avoid conflicts. Once you have saved this function into your .bashrc, you just need to run the following command to read from .bashrc:
<pre>. .bashrc</pre>
You may notice we're using /dev/random as the random source. This may cause blocking and take long amounts of time if you don't have sufficient entropy available. You have two choices in this situation:

1. You can install an amazing piece of software known as <a title="HAVEGED" href="http://www.issihosts.com/haveged/index.html" target="_blank">haveged</a> which will automatically push random data into /dev/random which passes all FIPS tests.
2. You can use /dev/urandom, which is a non-blocking source, but can contain repeats of previously used data (that's how it prevents from blocking).

The choice is yours, but haveged is highly recommended as you never know when you will need randomness!

If you have any suggestions on ways I can improve my simple password generator, please feel free to leave a comment and I might consider adding a particular password or hash type.
