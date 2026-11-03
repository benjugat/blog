---
layout: post
title: "Stored Open Redirect in a File Download Workflow (Bug Bounty)"
subtitle: "A finding during a bugbounty journey"
date: 2026-11-03 09:30:00 +0000
background: '/img/posts/05.jpg'
---

During a recent bug bounty assessment, I found a stored open redirect vulnerability affecting the file download workflow of a platform that allows users to publish and distribute digital content.

The issue was particularly interesting because the malicious URL was not directly exposed to the victim. Instead, it was stored as part of the published content and later used by the application's backend when another user initiated a download.

Vulnerability Overview

When publishing content, the application submitted metadata about the downloadable files to an API endpoint.

One of the parameters included in the request was similar to the following:

```
{
  "modelUrl": "https://cdn.example.com/files/model.3mf"
}
```

![Stored Open Redirect](/blog/img/posts/stored-open-redirect-01.png)

The modelUrl value represented the location from which the associated file would eventually be downloaded.

While testing the publishing workflow, I noticed that this parameter could be modified to contain an arbitrary external URL.

For example:

```
{
  "modelUrl": "https://attacker.example/payload"
}
```

The application accepted the modified value without performing any meaningful validation of the destination.

More importantly, the URL was persisted by the application.

This transformed what could otherwise have been a simple open redirect into a stored open redirect affecting other users interacting with the published content.

# Reproduction

The exploitation flow was straightforward.

First, I created a new piece of content using the normal publishing functionality of the application.

While submitting the publication form, I intercepted the API request responsible for storing the content metadata.

A simplified version of the request looked like this:

```
PUT /api/v1/content/draft/<ID> HTTP/2
Host: target.example
Content-Type: application/json
```

Within the JSON body, the application included the file location:

```
{
  "modelUrl": "https://legitimate-cdn.example/file"
}
```

I replaced this value with a server under my control:

```
{
  "modelUrl": "https://attacker.example/file"
}
```

The server accepted the request and stored the supplied value.

After publishing the content, I accessed it using a separate user session and attempted to download the corresponding file through the legitimate application interface.

![Stored Open Redirect](/blog/img/posts/stored-open-redirect-02.png)

Instead of retrieving a file from the expected infrastructure, the application's download system generated a redirect toward the attacker-controlled domain.

![Stored Open Redirect](/blog/img/posts/stored-open-redirect-03.png)

Conceptually, the flow looked like this:

```
Victim
  |
  | Clicks "Download"
  v
Legitimate Platform
  |
  | HTTP 302
  v
https://attacker.example/file
```

From the victim's perspective, the interaction originated entirely from the legitimate application's download interface.

