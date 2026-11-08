# Lab: Unprotected admin functionality with unpredictable URL (Portswigger)

**Platform:** Access control Vulnerability class: Broken access control (security through obscurity, information leak in client-side code) 

**Tools:** Web browser (view-source with Ctrl+U), Burp Suite

## Goal

Find the admin panel and delete the user **carlos**. This time the path is random, so it can't be guessed.

## How I found it

I opened the page source with **Ctrl+U** on the lab's shop page and read through the scripts. There was JavaScript that adds an "Admin panel" link to the page, but only if (**isAdmin**). In my case **isAdmin** was false, so the link never appeared. The path was still written in the script, in plain text. I loaded that path directly, the admin panel opened with no login, and I deleted **carlos**.

## What the developer assumed

That a random, unguessable URL, plus only showing the link to admins, would keep the panel private.

## Root cause

The server did not check permissions. It allowed anyone who requested the admin page to open it, without verifying if they were actually an administrator.

The application tried to secure the page using browser checks. The secret path was sent to every visitor inside the page's code, relying on the browser to hide it. Because anything sent to a user's device can be read by them, security rules enforced only in the browser can always be bypassed.

Using a random file name didn't provide real security. While it made the URL harder to guess, it failed completely once the path was leaked inside the page source.

## Fix
Enforce authentication and an admin-role check on the server for every request to the admin area.
Don't ship admin paths or other sensitive details to clients that aren't admins.
Don't treat client-side checks as security. They are for user interface only.

## What I learned
Ctrl+U is a quick way to read what a page contains. Burp is better when I need headers, cookies, background requests, or to modify a request.
Same underlying flaw as the first lab (no server-side access control), with a different way of leaking the path.
