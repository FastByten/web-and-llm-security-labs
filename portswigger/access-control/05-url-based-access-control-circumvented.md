# Lab: URL-based access control can be circumvented

**Platform:** PortSwigger Web Security Academy
**Topic:** Access control
**Vulnerability class:** Broken access control (front-end/back-end routing mismatch)
**Tools:** Burp Suite (Proxy, Repeater)

## Goal

Access the admin panel at `/admin` and delete the user `carlos`. The lab description states the setup directly: a front-end system blocks external access to `/admin`, but the back-end framework supports an `X-Original-URL` header.

## How I found it

The lab description named the mechanism outright, so this one was about using it correctly rather than discovering it.

A plain `GET /admin` returned `403 Forbidden` with body `"Access denied"`, confirming the front end blocks that exact path. A few path tricks I tried first did **not** work and are worth recording, since this app's front end turned out to be stricter than other labs' naive path matching:

- `GET /admin/` (trailing slash) → still 403
- `GET /ADMIN` / `/Admin` (case variation) → still 403
- Appending a sub-path directly, e.g. `/admin/deleteUser/carlos` → reached a real endpoint but still returned 403 `"Access denied"`, meaning the back-end itself still enforced a check at that path

The actual bypass, since this app supports `X-Original-URL`:

1. Send the request-line path as something harmless the front end won't block, e.g. `GET / HTTP/2`.
2. Add a header: `X-Original-Url: /admin`.
3. The front end only inspects the literal path on the request line (`/`), lets it through, and the back-end framework routes based on the `X-Original-URL` header instead, serving the real `/admin` page. This returned `200 OK` with the admin panel's content.

To actually delete `carlos`, I needed the delete action's path and parameter, both visible in the admin panel's own HTML/response once I could view it: `/admin/delete?username=carlos`. Getting this working took another round of trial and error:

- First attempt: `X-Original-Url: /admin/delete?username=carlos` (query string inside the header) → `400 Bad Request`, `"Missing parameter 'username'"`.
- This showed the framework reads `X-Original-URL` for the **path only**; it doesn't parse a query string placed inside that header value.
- Fix: move the query string back onto the real request line, and keep only the path in the header:
  - Request line: `GET /?username=carlos HTTP/2`
  - Header: `X-Original-Url: /admin/delete`
  - This returned `302 Found` (a redirect after the delete action), and the lab marked itself solved.

## What the developer assumed

That blocking a single, literal path string (`/admin`) at the front end was sufficient to protect everything the back end considers part of the admin area, and that trusting a client-supplied routing header (`X-Original-URL`) at the back end was safe because the front end was "already" filtering requests.

## Root cause

**A trust mismatch between two layers that don't agree on what a request actually is.** The front end makes its access decision based on the literal request-line path. The back end makes its own routing decision based on a different, attacker-controlled signal (`X-Original-URL`). Neither layer is wrong on its own, but together they create a gap: a request can look harmless to the layer doing the blocking and look privileged to the layer doing the work, because they're reading different parts of the same request.

This is a different shape of bug than the earlier access control labs in this series. Those had either no check at all, or a check that read an attacker-controlled *value* (a cookie, a request-body field). This one has a real check, enforced correctly by the front end, undermined because the back end doesn't honor the same boundary the front end thinks it's enforcing.

| | Root of the trust problem |
|---|---|
| Lab 1/2 | No access control check existed at all |
| Lab 3 | Check existed, but read a client-controlled cookie |
| Lab 4 | Check existed, but an unrelated endpoint let the client overwrite the trusted value directly |
| Lab 5 (this one) | Check existed and was correct, but a *different layer* trusted a client-controlled header that let requests skip it entirely |

## Fix

- Don't rely on a front-end proxy as the only enforcement point for access control; the back end should independently verify authorization for sensitive paths, regardless of how it was routed to them.
- If a back-end framework supports routing/override headers like `X-Original-URL` or `X-Rewrite-URL`, strip them at the edge (the front end) unless they're explicitly required and trusted, since they let a client override how its own request gets interpreted downstream.
- Keep the front end and back end in agreement about what "the path" means for a given request. A mismatch between the two is the actual vulnerability, not just the header's existence.

## What I learned

- This was the hardest lab so far. The bypass itself needed several corrections before it worked: getting the path/header split right, then separately getting the query string in the right place once the delete action needed a parameter.
- A header that controls routing (`X-Original-URL`) is a client-supplied value like any other, and a back end trusting it without re-validating authorization defeats the purpose of having a front-end block in the first place.
- When an endpoint says "missing parameter," it's a strong clue about *where* the framework expects a piece of data to come from (query string vs. a custom header vs. the path), not just that the data is absent, worth treating as diagnostic information rather than just a failure.
- Wrong guesses (trailing slash, case variation, naive sub-paths) are a normal and useful part of narrowing down which specific mechanism a lab is actually testing, especially once the generic "try the obvious tricks" approach stops working and the lab description has to be read closely for its actual hint.
