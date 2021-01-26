---
layout: post
title:  "WinPEAS.exe update"
date:   2020-12-27 19:32:09 +0100
categories: hacking winpeas privesc
---

While playing CTFs on [HackTheBox.eu][hackthebox-link] and [Vulnhub][vulnhub-link],<br />
I frequently used [PEASS - Privilege Escalation Awesome Scripts SUITE][peass-link].<br />
I really like this set of tools, and I had some ideas about improving them, so I contacted the author [carlospolop][carlospolop-link].<br />
After some discussion, I've implemented <b>Version 2</b> update, which will contain: <br />

- redesigned search - will be way faster
- logging to file
- watson update
- system version / KB enumeration update
- PS default transcripts - enhancement / fix
- LmCompatiblityLevel check
- credential manager enumeration
- saved wifi credentials enumeration
- updated / fixed scheduled applications
- updated search lists
- search for linux subsystem / WSL
- updated Autorun locations
- applocker enumeration
- search for hidden files in c:\users
- search for .bat, .exe, .ps1  in non-default folders
- debugging information - used memory / time
- bugfixes - clipboard, other bugs

## Coming Soon!

[hackthebox-link]: https://hackthebox.eu/
[vulnhub-link]: https://vulnhub.com/
[peass-link]: https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite
[winpeas-link]: https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS
[carlospolop-link]: https://github.com/carlospolop

Thanks [carlospolop][carlospolop-link] for answering my questions and creating [PEASS - Privilege Escalation Awesome Scripts SUITE][peass-link]!