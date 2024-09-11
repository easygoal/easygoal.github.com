---
layout: post
category: tutorial
tagline: ""
lang: en_US
tags: [nginx, tutorial]
title: The "Holy Grail" Solution for Removing ".html" in nginx
---
{% include JB/setup %}

Here is a "holy grail" solution for ``.html`` redirects in nginx.

However, I'll give an example and explain how it works. Here is the code:
```
location / {
    if ($request_uri ~ ^/(.*)\.html$) {
        return 302 /$1;
    }
    try_files $uri $uri.html $uri/ =404;
}
```

What's happening here is a pretty ingenious use of the ``if`` directive. Nginx runs a regex on the ``$request_uri`` portion of incoming requests. The regex checks if the URI has an .html extension and then stores the extension-less portion of the URI in the built-in variable ``$1``.

From the docs, since it took me a while to figure out where the $1 came from:

> Regular expressions can contain captures that are made available for later reuse in the $1..$9 variables.

The regex both checks for the existence of unwanted .html requests and effectively sanitizes the URI so that it does not include the extension. Then, using a simple return statement, the request is redirected to the sanitized URI that is now stored in ``$1``.

The best part about this, as original author [cnst](https://stackoverflow.com/users/1122270/cnst) explains, is that

> Due to the fact that $request_uri is always constant per request, and is not affected by other rewrites, it won't, in fact, form any infinite loops.

Unlike the rewrites, which operate on **any** ``.html`` request (including the invisible internal redirect to ``/index.html``), this solution only operates on external URIs that are visible to the user.

## What does "try_files" do?
You will still need the ``try_files`` directive, as otherwise Nginx will have no idea what to do with the newly sanitized extension-less URIs. The ``try_files`` directive shown above will first try the new URL by itself, then try it with the ".html" extension, then try it as a directory name.

The Nginx docs also explain how the default ``try_files`` directive works. The default ``try_files`` directive is ordered differently than the example above so the explanation below does not perfectly line up:

> Nginx will first append ``.html`` to the end of the URI and try to serve it. If it finds an appropriate ``.html`` file, it will return that file and will **maintain the extension-less URI**. If it cannot find an appropriate ``.html`` file, it will try the URI without any extension, then the URI as a directory, and then finally return a 404 error.

## What does the regex do?
The above answer touches on the use of regular expressions, but here is a more specific explanation for those who are still curious. The following regular expression (regex) is used:

```
^/(.*)\.html$
```
This breaks down as:

``^:`` indicates beginning of line.

``/:`` match the character "/" literally. Forward slashes do NOT need to be escaped in Nginx.

``(.*):`` capturing group: match any character an unlimited number of times

``\.:`` match the character "." literally. This must be escaped with a backslash.

``html:`` match the string "html" literally.

``$:`` indicates end of line.

The capturing group ``(.*)`` is what contains the non-".html" portion of the URL. This can later be referenced with the variable ``$1``. Nginx is then configured to re-try the request (``return 302 /$1;``) and the ``try_files`` directive internally re-appends the ".html" extension so the file can be located.

## Retaining the query string
To retain query strings and arguments passed to a ``.html`` page, the ``return`` statement can be changed to:
```
return 302 /$1?$args;
```
This should allow requests such as ``/index.html?test`` to redirect to ``/index?test`` instead of just ``/index``.

**Note that this is considered safe usage of the `if` directive.**

From the Nginx page If Is Evil:

> The only 100% safe things which may be done inside if in a location context are:
> return ...;
> rewrite ... last;

**Also, note that you may swap out the '302' redirect for a '301'.**

A ``301`` redirect is permanent, and is cached by web browsers and search engines. If your goal is to permanently remove the ``.html`` extension from pages that are already indexed by a search engine, you will want to use a ``301`` redirect. However, if you are testing on a live site, it is best practice to start with a ``302`` and only move to a ``301`` when you are absolutely confident your configuration is working correctly.

