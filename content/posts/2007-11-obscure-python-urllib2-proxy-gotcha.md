---
title: "Obscure python urllib2 proxy gotcha"
date: 2007-11-21T19:28:08Z
draft: false
disclaimer: represent
aliases:
    - /articles/obscure-python-urllib2-proxy-gotcha/
---

This is going to be very obscure, technical and humourless[^1], so unless you suspect your environment-set http proxy is messing with your python, you can stop reading now.

This is actually two problems, and three solutions.

## Problem 1

My python script bombs out with:

```python

File "c:\Python23\lib\urllib2.py", line 506, in proxy_open
    if '@' in host:
TypeError: iterable argument required
```

## Solution 1

Your `http_proxy` environment variable must include `http://` at the start.

## Problem 2

But why’s it happening in the first place? I’m not even _using_ the environment-set proxy – I’m defining my own, thus:

```python

proxy_handler = urllib2.ProxyHandler({'http': 'http://myproxy.net:3128'})
opener = urllib2.build_opener()
opener.add_handler(proxy_handler)
```

## Solution 2

Ah! But `urllib2.build_opener()` has a set of default handlers, one of which is a proxy handler. Guess what the default behaviour of the proxy handler is, when you don’t give it a proxy url? It gets it from the environment.

So you’re ending up with _two_ proxy handlers. The default one gleaned from the environment, and your custom one. It seems that it only uses the custom one (try it using [Wireshark](http://www.wireshark.org/)) but it still processes the environment one. So if the format of the environment one is wrong, it breaks your script. This is a bit unpredictable and brittle, so try this instead:

```python
proxy_handler = urllib2.ProxyHandler({'http': 'http://myproxy.net:3128'})
opener = urllib2.build_opener(proxy_handler)
```

That will give you one and only one proxy handler.

## [Updated 03/03/2008 to add]

To get _no_ proxies – i.e. to disable proxies, do this:

```python
proxy_support = urllib2.ProxyHandler({})
opener = urllib2.build_opener(proxy_support)
urllib2.install_opener(opener)    # Optional - makes this opener default for urlopen etc.
```

## Solution to end all solutions

I tried this on Python 2.5 and it exhibited none of these problems. I’ve looked at the new urllib2 code which has much more detailed and careful proxy url parsing, allowing a more liberal approach to defining proxies.

## [Updated 09/03/2010 to add]

Ciprian Trofin has emailed me to point out that while the parsing of proxy urls in environment variables improved in Python 2.5, the `build_opener` still uses default handlers, and gets its proxy from the environment, when none is specified. This is the case with Python 2.6 – I guess it’s an awkward feature, not actually a bug. Thanks to Ciprian for the update and clarification!

[^1]: Okay – we’re done. You can look at [funny things](http://xkcd.com/282/) now.
