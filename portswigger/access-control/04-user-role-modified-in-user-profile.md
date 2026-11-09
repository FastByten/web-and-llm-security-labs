# Lab: User role can be modified in user profile (Portswigger)

**Topic:** Access control
**Vulnerability class:** Broken access control (mass assignment)

**Tools:** Burp Suite (Proxy, Repeater)

## Goal

Find a way to access the admin panel and delete the user `carlos`.

## How I found it

I logged in as `wiener` and changed my email address through the account page, from `wiener@normal-user.net` to `wiener@admin-user.net`. The response to that request came back as JSON containing my full user object, including a field I hadn't set myself:

```json
{
  "username": "wiener",
  "email": "wiener@admin-user.net",
  "apikey": "...",
  "roleid": 1
}
```

That told me the server tracks a `roleid` per user, and that the `/my-account/change-email` endpoint returns it, even though the feature is only supposed to change an email address.

I found the request itself in Proxy history (`POST /my-account/change-email`), sent it to Repeater, and looked at its body:

```json
{
"email":"wiener@admin-user.net"
}
```

I added a `roleid` field that the form never sends on its own:

```json
{
"email":"wiener@admin-user.net",
"roleid": 2
}
```

First attempt returned a `500 Internal Server Error`. Two mistakes in my edited request caused it: a missing comma in the JSON, and a stale `Content-Length` header left over from the original, shorter body. After fixing the JSON and removing the `Content-Length` header (so Burp could set the correct one), the request succeeded with a `302` and the response showed `"roleid": 2`.

Because this request had actually changed my role server-side, rather than just claiming a false value in a cookie, my existing session was now a real admin session. Loading `/admin` directly in the browser worked without needing to edit any cookie. I deleted `carlos` from there.

## What the developer assumed

That the `/my-account/change-email` endpoint would only ever receive the one field the UI's form sends (`email`), so it was safe to save whatever object the request body contained.

## Root cause

**Mass assignment.** The endpoint took the entire submitted object and saved it, including fields the client was never intended to be able to set. The UI only exposes an email field, but nothing on the server enforced that restriction, it trusted the shape of the request instead of validating which fields a user is allowed to modify.

This is a different mechanism from the previous lab in this series ("User role controlled by request parameter"), where the server read an existing client-held cookie value with no check at all. Here, the server has a real, server-side role, but a different endpoint let an attacker overwrite it through an unrelated feature.


## Fix

- On the server, explicitly define and validate which fields each endpoint is allowed to update (an allow-list), rather than saving whatever the request body contains.
- Never let privileged fields (`roleid`, `isAdmin`, permissions, etc.) be set through an endpoint whose actual purpose is something else.
- Treat any field appearing in a response that wasn't part of the original request as a hint worth investigating, it can reveal server-side data structures you didn't know you could influence.

## What I learned

- When hand-editing a raw request body in Burp, check for: valid JSON (commas, matching braces) and a correct `Content-Length`, a stale length after an edit can itself cause a 500 that has nothing to do with the vulnerability being tested.
- A response can leak the existence of a field (like `roleid`) even when the request that produced it never set that field, worth reading full responses, not just the status code.
- "Change my role" doesn't have to go through an obvious admin endpoint. It can hide inside a completely unrelated feature (changing an email) if the backend isn't strict about what it accepts.
