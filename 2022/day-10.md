---
title: Improve your site security with this one weird trick
author: @quannt
date: 12-03-2022
---

Content Security Policy (CSP) is a HTTP response header that tells browsers what they can and cannot do with pretty much anything on a given web page. This gives developers a great tool at disposal to mitigate common attack techniques like XSS, click jacking.

**How to use CSP**: since it's a HTTP response header, all you have to do is making sure your web servers returns the appropriate header, browsers would take care of the rest.

Let's go over an example

`Content-Security-Policy: default-src 'self' http://example.com; connect-src 'self';`

In this example, the server issues 2 policies, separated by a semicolon.

The first policy: `default-src` is called *directive*. `'self' http://example.com` is called *value*. This tells browser to only load resources (js, HTML, CSS, fonts, AJAX requests, etc) from the origin domain (self) and example.com.

The second policy:  `connect-src` tells browsers to only make AJAX requests to certain domains, `'self'` means that domain is the origin domain. You may have noticed by now that both `connect-src` and `default-src` enforces policy on AJAX requests, in this case, the policy from `connect-src` would override `default-src`.

If you want to learn more: just Google with the keyword or ask ChatGPT.
