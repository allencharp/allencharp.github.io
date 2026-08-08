---
layout: post
title:  "wfuzz and some bugs"
date:   2019-01-15 22:31:34 +0800
author: allencharp
tags: [pentest, wfuzz, fuzzing]
---

# Wfuzz

[Wfuzz](https://github.com/xmendez/wfuzz) is a powerful brute-force / fuzzing tool for web applications. I also integrated Python Wfuzz into the ZAPv2 library, which can do a lot of amazing jobs.

# Basic usage

Wfuzz marks injection points with the **FUZZ** keyword. Point it at a dictionary with `-z` and filter the noise with `-f`:

```bash
# install
pip install wfuzz

# fuzz a path segment with a wordlist
wfuzz -z file,/usr/share/wordlists/common.txt http://target.com/FUZZ

# fuzz POST data and filter by response code
wfuzz -z file,users.txt -d "username=FUZZ&password=test" --hc 404 http://target.com/login

# filter responses by a string in the body
wfuzz -z file,ids.txt --hs "not found" http://target.com/api/item?id=FUZZ
```

Key options:

* `-z <dict>` — payload source (file, list, range, ...)
* `-d <data>` — POST body with FUZZ placeholder
* `-H <header>` — extra header
* `-f / --filter` / `--hc`, `--hl`, `--hh`, `--hs` — filter responses by code/lines/chars/string to keep only interesting results

# A known bug: can't set Content-Type when posting data

When using `postdata` to send an HTTP POST, don't change the content-type. Looking at the source in [src/wfuzz/externals/reqresp/Request.py](https://github.com/xmendez/wfuzz/blob/master/src/wfuzz/externals/reqresp/Request.py):

```python
elif name == "postdata":
    if self.ContentType == "application/x-www-form-urlencoded":
            return self.__variablesPOST.urlEncoded()
    elif self.ContentType == "multipart/form-data":
            return self.__variablesPOST.multipartEncoded()
    else:
            return self.__uknPostData
```

If we change the ContentType to `application/json`, the post data will be corrupted to `__uknPostData` — the payload encoder only understands form-urlencoded and multipart bodies, so JSON data is not serialized correctly.

# Workarounds

If you need to send a JSON (or other custom) body, the reliable paths are:

```bash
# 1) use -d with the default content type (form-urlencoded) and don't fight it
wfuzz -z file,payloads.txt -d 'field=FUZZ' http://target.com/api

# 2) send a raw body with --data (treated as opaque payload) and set the header explicitly
wfuzz -z file,payloads.txt --data '{"q": "FUZZ"}' -H "Content-Type: application/json" http://target.com/api

# 3) or just script it with requests/httpx in Python and skip the buggy code path
```

Depending on the wfuzz version, `--data` keeps the body untouched while `-H "Content-Type: ..."` sets the header directly, avoiding the `__uknPostData` fallback entirely.

# Summary

Wfuzz is a fast, scriptable fuzzer, but its post-data handling is form-centric. Know the `__uknPostData` trap, and for non-form bodies prefer `--data` + explicit `Content-Type` header (or a few lines of Python) over fighting the built-in encoder.
