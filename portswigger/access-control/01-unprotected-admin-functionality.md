# Lab: Unprotected admin functionality (Portswigger)

**Platform:** Access control Vulnerability class: Broken access control (security through obscurity) 

**Tools:** Burp Suite (browser, HTTP history), web browser

## Goal

Find the admin panel and delete the user **carlos**.

## How I found it

The shop had no visible link to an admin page. Sites often publish standard files that describe their own structure, so I checked **/robots.txt** on the lab's address. It contained a **Disallow:** line that named the admin panel's path. I loaded that path directly and the admin panel opened with no login. From there I deleted **carlos.**

I also tried guessing **/admin** first, which returned a 404. A guess only works when the path happens to be common, which is why checking files the site publishes comes before guessing.

## What the developer assumed

That an admin page nobody links to would stay hidden, and that listing it under Disallow would keep people away from it.

## Root cause

The website relied on security through obscurity. It assumed that keeping the link unlinked on the front page was enough security, but it never actually checked if the person requesting the page was an authorized admin.

Putting sensitive pages inside robots.txt made it worse. That file is completely public and acts like a sign advertising where the hidden paths are located.

## Fix
Require authentication and an admin-role check on the server for every request to the admin area.
Don't list sensitive paths in robots.txt.
Don't rely on an unlinked URL as protection.

## What I learned
Always check easy public resources like robots.txt, page source, and sitemaps before wasting time trying to guess URLs manually.
