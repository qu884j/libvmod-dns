============
libvmod_dns
============

----------------------
Varnish DNS Module
----------------------

:Author: Kenneth Shaw
:Date: 2013-04-01
:Version: 0.1
:Manual section: 3

SYNOPSIS
========

import dns;

DESCRIPTION
===========

Provides some DNS functions to Varnish.

FUNCTIONS
=========

resolve
-------

Prototype
        ::

                resolve(STRING str)
Return value
	STRING
Description
	Resolves the hostname to its dns entry
Example
        ::

                set resp.http.x-dns-example = dns.resolve("www.example.com");

rresolve
--------

Prototype
        ::

                rresolve(STRING str)
Return value
	STRING
Description
	Reverse resolve an IP
Example
        ::

                set resp.http.x-dns-reverse = dns.rresolve("127.0.0.1");

INSTALLATION
============

This vmod is based off the libvmod-example source, and thus works / compiles
in the same fashion.

This vmod was originally built with the intention of detecting bots with bad
User-Agent strings. See the example below.

Usage::

 ./configure

Make targets:

* make - builds the vmod
* make install - installs your vmod
* make check - runs the unit tests in ``src/tests/*.vtc``


If you are installing this on a Debian or Debian-derivative, and using the
[Debian packages](https://www.varnish-cache.org/installation/debian), then
doing the following (as root) should be all that is necessary to build and
install this VMOD::

 # change to working source directory
 cd /usr/local/src

 # get varnish and libvmod-dns
 apt-get source varnish
 git clone https://github.com/kenshaw/libvmod-dns

 # build libvmod-dns
 cd libvmod-dns
 ./configure

 # install
 make && make install


In your VCL you could then use this vmod along the following lines::

        import dns;

        # do a dns check on "good" crawlers
        sub vcl_recv {
            if (req.http.user-agent ~ "(?i)(googlebot|bingbot|slurp|teoma)") {
                # do a reverse lookup on the client.ip (X-Forwarded-For) and check that its in the allowed domains
                set req.http.X-Crawler-DNS-Reverse = dns.rresolve(req.http.X-Forwarded-For);

                # check that the RDNS points to an allowed domain -- 403 error if it doesn't
                if (req.http.X-Crawler-DNS-Reverse !~ "(?i)\.(googlebot\.com|search\.msn\.com|crawl\.yahoo\.net|ask\.com)$") {
                    return (synth(403, "Forbidden"));
                }

                # do a forward lookup on the DNS
                set req.http.X-Crawler-DNS-Forward = dns.resolve(req.http.X-Crawler-DNS-Reverse);

                # if the client.ip/X-Forwarded-For doesn't match, then the user-agent is fake
                if (req.http.X-Crawler-DNS-Forward != req.http.X-Forwarded-For) {
                    return (synth(403, "Forbidden"));
                }
            }
        }

HISTORY
=======

This module was created in an effort to detect/prevent/stop clients User-Agent
strings claiming to be googlebot/msnbot/etc.

COPYRIGHT
=========

This document is licensed under the same license as the
libvmod-dns project. See LICENSE for details.

* Copyright (c) 2013 Kenneth Shaw
