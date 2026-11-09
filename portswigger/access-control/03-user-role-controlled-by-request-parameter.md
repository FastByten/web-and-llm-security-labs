# Lab: User role controlled by request parameter (Portswigger)

**Topic:** Access control
**Vulnerability class:** Broken access control (client-controlled authorization data)

**Tools:** Burp Suite (Proxy, Repeater, browser dev tools)

## Goal

Find a way to access the admin panel and delete the user `carlos`.

## How I found it

After logging in, I checked the request to `/my-account` in Burp's Proxy history. The `Cookie` header included:

```
Cookie: Admin=false; session=<session id>
```

I sent the request to Repeater, changed `Admin=false` to `Admin=true`, and resent it. The response came back `200 OK` and included an `<a href="/admin">Admin panel</a>` link in the page, which the normal response didn't have.

Repeater only sends a one-off copy of the request, it doesn't change what my browser actually sends. To finish the lab, I opened Developer Tools in Burp's browser, went to Application > Cookies, and changed the real `Admin` cookie to `true` directly, then loaded `/admin`. The admin panel opened and listed both my account and `carlos`, with a delete option. I deleted `carlos`.

## What the developer assumed

That the `Admin` cookie was safe to trust as proof of admin status, since the application itself was the one that originally set it.

## Root cause

The server checked a value the client fully controls instead of a value the server owns. A cookie is just a string the browser stores and resends; nothing stops the client from editing it before sending it back. The session ID in the same request authenticates who I am, but the `Admin` flag was being used for authorization, and it was never tied to anything verified server-side (such as a role looked up from a database using the session ID).

This is different from the first two access control labs in this series. Those had no access control check at all on the admin page. This one has a check, it's just checking a value the attacker controls.

## Fix

- Store authorization data (roles, permissions) server-side, keyed to the session.
- Never let a client-supplied value (cookie, hidden form field, query parameter, or an editable claim in a token) determine what a user is allowed to do.
- On each privileged request, look up the user's role from the session on the server, don't read it off the request.

## What I learned

- Repeater is for testing a request in isolation; changing my browser's actual cookie (via dev tools) is what's needed to interact with the resulting page normally.
- "The client sent it" is never proof of anything the server should trust for authorization. Only server-held state is trustworthy.
- Progression across the three access control labs so far:

