---
title: "Batch file comments"
date: 2009-09-05T19:28:08Z
draft: false
aliases:
    - /articles/batch-file-comments/
---

In Windows batch files, you can use a double-colon (`::`) as a comment. I've just spend a few hours trying to figure out why my batch file says, "The syntax of the command is incorrect." when I run it. The answer? I've got a `::` comment as the last statement after a group of `IF` statements. Obscure? Yes. Flaky? Certainly. Oh &mdash; hang on, it might be because I've got the `::` before a `for` statement. For some arrangements of the `::` comment, I'm also getting "The system cannot find the drive specified."

`::` is a bad, bad thing. My advice: Stick to `REM`
