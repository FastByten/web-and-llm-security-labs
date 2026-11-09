# Security Labs

Hands-on notes and write-ups from my self-taught path into application security, with a longer-term focus on LLM / AI security.

## About

I'm based in Bulgaria and learning cybersecurity from scratch. My goal is a junior role in application security or penetration testing, then specializing in LLM security (prompt injection, agent and RAG risks).

Everything here is written in my own words, and only covers labs, practice platforms, and systems I'm allowed to test. I only test systems I own or have explicit permission to test: PortSwigger Web Security Academy labs, my own virtual machines, and similar practice platforms.

## Progress

- [x] Linux fundamentals (filesystem, permissions, processes, packages)
- [x] Networking fundamentals (IP, DNS, ports, TCP/UDP, TLS)
- [x] HTTP fundamentals (methods, status codes, cookies, authentication vs authorization, headers)
- [x] Burp Suite basics (Proxy, HTTP history, Intercept, Repeater)
- [ ] PortSwigger Web Security Academy: Access control (in progress)
- [ ] PortSwigger Web Security Academy: SQL injection, XSS, authentication, SSRF, API testing
- [ ] Python basics (HTTP requests, JSON, small automation scripts)
- [ ] LLM security: OWASP Top 10 for LLM Applications, prompt injection, Garak

## Lab write-ups

| # | Lab | Vulnerability class | Write-up |
|---|-----|---------------------|----------|
| 1 | Unprotected admin functionality | Broken access control | [read](portswigger/access-control/01-unprotected-admin-functionality.md) |
| 2 | Unprotected admin functionality with unpredictable URL | Broken access control | [read](portswigger/access-control/02-unpredictable-admin-url.md) |
| 3 | User role controlled by request parameter | Broken access control | [read](portswigger/access-control/03-user-role-controlled-by-request-parameter.md) | 
| 4 | User role modified in user profile | Broken access control | [read](portswigger/access-control/04-user-role-modified-in-user-profile.md) |

## LLM security

| # | Test | Tool | Write-up |
|---|------|------|----------|
| 1 | First prompt injection scan | Garak | [read](garak/experiments/01-prompt-injection/README.md) |


## Environment

- Kali Linux (VirtualBox VM and WSL 2)
- Burp Suite Community Edition
- curl, Python, Git
- Garak (LLM vulnerability scanner)


## Next steps

1. Finish the access control labs, then move on to authentication, SQL injection, XSS, and SSRF.
2. Learn enough Python to script HTTP requests and parse JSON.
3. Return to Garak with a better understanding of what each probe and detector does, and investigate individual failures by hand.
4. Work through the OWASP Top 10 for LLM Applications with small hands-on experiments.

 
 
### _NO SECRETS, CREDENTIALS, OR SESSION TOKENS ARE COMMITTED TO THIS REPOSITORY!_ 
