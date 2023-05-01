---
author: Stu
title: Generating subresource Integrity (SRI) hashes from the command line
summary: "How to quickly generate subresource Integrity (SRI) hashes from the command line"
date: 2023-01-03T22:27:22Z
draft: true
tags: [cybersecurity, web, cli]
---

<explain SRI>

Use the following to hash your script and out the string needed for SRI. 

```bash
echo -n "sha256-" && openssl dgst -sha256 -binary ./path/to/file.js | openssl base64 -A; echo
```

<aside>
💡 Note: the hash is base64 encoded and prepended with the hash algorithm, in this case SHA256.
</aside>

Use the following to load your script

```jsx
<script defer src="./path/to/file.js" integrity="sha256-<hash>"></script>
```

## Using scripts from another domain?

If using scripts from another domain such as a CDN you’ll need to include `crossorigin="anonymous"` or the integrity check won’t be performed by the browser.

Use the following to load your script:
```jsx
<script defer src="/assets/js/template-web.min.js" integrity="sha256-0sL5IkMJDqqdc9pElxsYupSMQjKt2+Ua6dCMPoDrwGg=" crossorigin="anonymous"></script>
```

## Further reading

Check out the [Wikipedia article](https://en.wikipedia.org/wiki/Subresource_Integrity) and [SRI Hash Generator](https://www.srihash.org/)