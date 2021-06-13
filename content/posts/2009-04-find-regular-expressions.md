---
title: "Find regular expressions"
date: 2009-04-06T19:28:08Z
draft: false
# disclaimer: represent|removed
aliases:
    - /articles/find-regular-expressions/
---

> "Why don't my regular expressions work with the 'find' utility in Linux/Ubuntu/Unix/Cygwin/Posix-environment?"

Short answer: You need `-regextype posix-extended`

E.g. To find files with either of two file extensions, use:
`find . -regextype posix-extended -regex &#39;.*\.(xsd|java)&#39;`

Want to know the differences between POSIX Extended Regular Expressions and basic ones? Read [this excellent resource about regular expressions](http://www.regular-expressions.info/posix.html). Want to test your regular expressions, live, in the browser? Try [Regexpal](http://regexpal.com/).

Similarly, use `egrep` instead of `grep` to enable extended regex functionality and use `sed -r` instead of `sed`.
