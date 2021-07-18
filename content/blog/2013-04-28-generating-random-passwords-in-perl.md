---
date: 2013-04-28T21:19:40Z
disqus: true
share: true
title: Generating Random Passwords in Perl
url: /2013/04/28/generating-random-passwords-in-perl/
---

Though I've already created <a href="https://lamendo.la/2012/07/05/generating-random-passwords-using-your-systems-random-source/">the same thing in another post</a> using a Bash function, I had a very important need for a Perl script that produced random strings. Once I got that little script working, I decided to rewrite the original Bash password generator function entirely in Perl. This may not be the most efficient way to hack it up, but I'm no Perl monger, so don't expect perfection. This script does work well, however, and requires no additional modules, so it can be run anywhere there is a Perl interpreter.

```perl
#!/usr/bin/perl -w

use warnings;
use strict;

# Default length/number
my $len = 19;
my $num = 1;

# Custom definition for safe symbols
my @symbols = ('-', '_', '!', '@', '#', '$', '%', '^', '&', '*', '/', '\\', '(', ')', '_', '+', '{', '}', '|', ':', '<', '>', '?', '=', 'A'..'Z', 'a'..'z', '0'..'9');

# Alphanumeric-only option
my @alphanumeric = ('A'..'Z', 'a'..'z', '0'..'9');

# Pull in args and list usage
my $numargs = scalar @ARGV; 
    if ($numargs == 0 || $ARGV[0] eq "-h") {
        print "Sal's Random Password Generator\n";
        print "-------------------------------\n";
        print "Usage: pwgen <OPTIONS> [length] [number] (length and number optional)\n\n";
        print "OPTIONS (MUST SPECIFY ONE!):\n";
        print "-s               Add symbols to output (NOT FOR MYSQL!)\n";
        print "-a               Alphanumeric only\n";
        print "-p               Generate phpMyAdmin Blowfish secret (for cookie auth)\n";
        print "-w               Generate Wordpress encryption keys (wp-config.php)\n";
        print "-h               Display this usage information\n\n";
        print "If no length or number are defined, a default length of $len and number of $num will be used.\n";
        exit(0);
    } 

my $pwtype = $ARGV[0];

if (defined $ARGV[1] && $ARGV[1] =~ /^\d+$/) {
    $len = $ARGV[1];
}

if (defined $ARGV[2] && $ARGV[2] =~ /^\d+$/) {
    $num = $ARGV[2];
}

if ($pwtype eq "-s") {
    my $count = 0;
    my $pass;
    while ($count < $num) {
        $pass = "";
        $pass .= $symbols[rand @symbols] for 1..$len;
        print "$pass\n";
        $count++;
    }
}

if ($pwtype eq "-a") {
    my $count = 0;
    my $pass;
    while ($count < $num) {
        $pass = "";
        $pass .= $alphanumeric[rand @alphanumeric] for 1..$len;
        print "$pass\n";
        $count++;
    }
}

if ($pwtype eq "-p") {
    my $pass = "";
    $pass .= $symbols[rand @symbols] for 1..64;
    print "$pass\n";
}

if ($pwtype eq "-w") {
    my $pass1 = "";
    my $pass2 = "";
    my $pass3 = "";
    my $pass4 = "";
    my $pass5 = "";
    my $pass6 = "";
    my $pass7 = "";
    my $pass8 = "";
    $pass1 .= $symbols[rand @symbols] for 1..64;
    print "define('AUTH_KEY',\t\t'$pass1');\n";
    $pass2 .= $symbols[rand @symbols] for 1..64;
    print "define('SECURE_AUTH_KEY',\t'$pass2');\n";
    $pass3 .= $symbols[rand @symbols] for 1..64;
    print "define('LOGGED_IN_KEY',\t\t'$pass3');\n";
    $pass4 .= $symbols[rand @symbols] for 1..64;
    print "define('NONCE_KEY',\t\t'$pass4');\n";
    $pass5 .= $symbols[rand @symbols] for 1..64;
    print "define('AUTH_SALT',\t\t'$pass5');\n";
    $pass6 .= $symbols[rand @symbols] for 1..64;
    print "define('SECURE_AUTH_SALT',\t'$pass6');\n";
    $pass7 .= $symbols[rand @symbols] for 1..64;
    print "define('LOGGED_IN_SALT',\t'$pass7');\n";
    $pass8 .= $symbols[rand @symbols] for 1..64;
    print "define('NONCE_SALT',\t\t'$pass8');\n";
}
```

Save this as `salpwgen.pl` or whatever file name you choose. The first line allows you to make the file executable and run it as `./salpwgen.pl`, assuming your Perl interpreter is in `/usr/bin/perl`.

If you have any suggestions, or if you would like to see some other type of secret/hash generated, please feel free to comment below.
