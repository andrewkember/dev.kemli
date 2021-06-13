---
title: "Removing Landscape advert from Ubuntu login"
date: 2009-11-24T19:28:08Z
draft: false
aliases:
    - /articles/removing-landscape-advert-from-ubuntu-login/
---

To remove the text that says: `Graph this data and manage this system at https://landscape.canonical.com/` while keeping the useful system information, edit the script that puts Landscape information into the message of the day (MOTD):

`sudo nano -w /etc/update-motd.d/50-landscape-sysinfo`

The last line currently reads:

`/usr/bin/landscape-sysinfo`

Change this to say (on one line):

`/usr/bin/landscape-sysinfo --exclude-sysinfo-plugins=LandscapeLink`

Save the file (Ctrl-X, Y, Enter)

Test this by running:

`/etc/update-motd.d/50-landscape-sysinfo`

It should look like:

```text
  System information as of Tue Nov 24 13:28:10 <span class="caps">GMT</span> 2009</p>

  System load: 0.0                Memory usage: 37%   Processes:       99
  Usage of /:  22.5% of 12.92GB   Swap usage:   0%    Users logged in: 1
```
