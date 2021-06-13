---
title: "Can't run Crashplan desktop under Ubuntu"
date: 2009-11-09T19:28:08Z
draft: false
# disclaimer: represent|removed
aliases:
    - /articles/can-t-run-crashplan-desktop-under-ubuntu/
---

I've installed Crashplan on my Ubuntu server (it installed its own JVM) and now I'm trying to start the Crashplan desktop client using X11 forwarding.  I get:<!--more-->

```bash
$ cat /usr/local/crashplan/log/ui_error.log
com.backup42.desktop.CPDesktop main
SEVERE: Failed to launch CPDesktop; java.lang.UnsatisfiedLinkError: no swt-pi-gtk-3448 or swt-pi-gtk in swt.library.path, java.library.path or the jar file
java.lang.UnsatisfiedLinkError: no swt-pi-gtk-3448 or swt-pi-gtk in swt.library.path, java.library.path or the jar file
```

**Solution: Install Java.** Specifically the JRE version 1.5. [Unlike this report here](http://ubuntuforums.org/archive/index.php/t-628277.html) it has nothing to do with 32/64 bit conflicts, because it's a 32 bit machine. If you don't want to install Java, and its associated clutter, then you could try following the advice in the Crashplan readme file about running the UI locally and connecting to the remote crashplan service via the service port:

> Remote GUI Config of CrashPlan on a shell account:
> What if you have a remote shell account on a box that has SSH access, but no X windows interface or GUI? We're going to show you how to attach your local desktop CrashPlan UI to the remote CrashPlanEngine on a remote box.
>
> We'll do this by using SSH to forward local traffic to the GUI control port of the service on the remote machine.
>
> Step 1. Install & start engine on host 1.2.3.4 (this is the far away text only server)
>
> Step 2. Install CrashPlan on your local desktop. (Mac, Windows, Linux, doesn't matter)
>
> Step 3. In the CrashPlan folder of your local install, there is a folder called "conf", edit the file called my.ui.properties. Add this as a new line at the bottom of the file and save it. Shut down the GUI, and add this as a new line at the bottom of the file and save it. (If you are running a CrashPlanPRO blue client, substitute 4283 for 4200 in the commands below.)
>
> `servicePort=4200`
>
> Step 4. We need to forward port 4200 locally to our remote server using SSH. Type this in terminal:
>
> `ssh -L 4200:localhost:4243 yourusername@1.2.3.4`
>
> Step 5. Run your CrashPlan UI. You're now connected to remote CrashPlanEngine and can configure it at will.
>
> Step 6. (optional) Pointing your UI back to local crashplan. Edit the "my.ui.properties" and comment out the servicePort change by putting a "#" in front and saving it. That's it! Next time you use UI it will connect to your local CrashPlan again.
>
> `#servicePort=4200`

Let me know if it works for you!
