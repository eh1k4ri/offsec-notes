# Cylab - Old Sessions
| Info         | Detail                     |
|--------------|----------------------------|
| Platform     | picoCTF                    |
| Category     | Web Exploitation           |
| Vulnerability| Broken Session Management  |
| Status       | ✅ Completed                |

## Table of Contents
- [Cylab - Old Sessions](#cylab---old-sessions)
  - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
  - [Recon](#recon)
  - [Discovery](#discovery)
  - [Exploitation](#exploitation)
  - [Flag](#flag)
  - [Lessons Learned](#lessons-learned)

---

## Overview

Proper session timeout controls are critical for securing user accounts. If a user logs in on a public or shared computer but doesn't explicitly log out (instead simply closing the browser tab), and session expiration is misconfigured, the session may remain active indefinitely.

This allows an attacker using the same browser later to access the user's account without needing credentials, exploiting the fact that sessions never expire and remain authenticated.

---

## Recon

I started the instance and accessed the site, landing on a login page:

![Login page](./assets/01-login-page.png)

I clicked on **Register** to create a new account:

![Register page](./assets/02-register-page.png)

I created a test account and logged in:

![Homepage logged in](./assets/03-homepage-admin.png)

---

## Discovery

The first thing that caught my attention was a comment from user `mary_jones_8992` mentioning a strange page at `/sessions`. I decided to investigate:

![Sessions page](./assets/04-sessions-page.png)

The `/sessions` endpoint exposed a list of active sessions, including my own — each entry contained a session token and its associated user data (e.g. `'key': 'admin'`). This confirmed that session tokens for **other logged-in users** (including an `admin` account) were being leaked on a publicly accessible page.

---

## Exploitation

With the leaked session tokens in hand, I opened the browser's DevTools and navigated to **Application → Cookies**, then replaced the value of my own `session` cookie with the token belonging to the `admin` user (the first entry from the `/sessions` list):

![Modifying the session cookie in DevTools](./assets/05-devtools-cookie.png)

After updating the cookie, I went back to the homepage. The application now recognized me as the `admin` user, and the flag was displayed directly on the page:

![Flag on homepage](./assets/06-flag.png)

---

## Flag

```
picoCTF{s3t_s3ss10n_3xp1rat10n5_51c526ab}
```

---

## Lessons Learned

- Session tokens must never be exposed on a publicly accessible endpoint — leaking them is functionally equivalent to leaking passwords.
- Sessions should always have a proper expiration policy; sessions that never expire dramatically increase the window of opportunity for hijacking, especially on shared or public devices.
- Client-side cookies should be treated as untrusted input from the server's perspective — but from an attacker's perspective, they're a direct path to impersonating another user if not protected by additional safeguards (e.g. binding sessions to IP/device fingerprint, short expiration, secure random generation).
