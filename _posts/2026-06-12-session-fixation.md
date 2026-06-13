---
layout: post
title: "Session Fixation in osTicket v1.18.2"
subtitle: "A finding during a internal pentest"
date: 2026-06-12 17:30:00 +0000
background: '/img/posts/01.jpg'
---

During an internal pentest performed for a client, I found that the environment was using the latest available version of osTicket at the time, version 1.18.2. Since I did not find any public vulnerabilities associated with that version, I decided to go deeper into the manual analysis of the application, especially its authentication and session management flows.

From that review, I identified behavior consistent with a **Session Fixation** issue, a vulnerability that directly affects the security of user sessions.

Session Fixation ([CWE-384](https://cwe.mitre.org/data/definitions/384.html)) is a weakness in which an application allows a previously known or fixed session identifier to remain valid even after the user logs in.

In a secure implementation, when a user authenticates, the application should generate a new session identifier. If this does not happen, an attacker could try to take advantage of an already established or predefined session to access the victim’s authenticated account.

These vulnerabilities often go unnoticed when one relies solely on the absence of public CVEs or advisories. However, the real security of an application also depends on reviewing its logic flows, especially in authentication and session management.

This finding shows that an internal pentest is not only useful for confirming known vulnerabilities, but also for identifying logic flaws that do not always appear in public sources.


# PoC

As we can see the initial cookie `OSTSESSID` with the value of `POC-COOKIE` is a `Guest User` context cookie, i.e. not authenticated.

![Session Fixation](/blog/img/posts/session-fixation-01.png)

We  access  the login panel:

![Session Fixation](/blog/img/posts/session-fixation-02.png)

And we log in, as we can see in the server response, it does not set us any new cookie configured to the authenticated context.

![Session Fixation](/blog/img/posts/session-fixation-03.png)

Finally we can see that we have been correctly logged in with the user `benjugat` and the cookie has not been modified or changed by a new one, and it keeps the initial cookie `POC-COOKIE`. The server has changed it from unauthenticated to authenticated context.

![Session Fixation](/blog/img/posts/session-fixation-04.png)