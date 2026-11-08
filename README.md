Security Labs

Hands-on notes and write-ups from my self-taught path into application security, with a longer-term focus on LLM / AI security.

About

I'm based in Bulgaria and learning cybersecurity from scratch. My goal is a junior role in application security or penetration testing, then specializing in LLM security (prompt injection, agent and RAG risks).

Everything here is written in my own words, and only covers labs, practice platforms, and systems I'm allowed to test. I only test systems I own or have explicit permission to test: PortSwigger Web Security Academy labs, my own virtual machines, and similar practice platforms.

Progress
 Linux fundamentals (filesystem, permissions, processes, packages)
 Networking fundamentals (IP, DNS, ports, TCP/UDP, TLS)
 HTTP fundamentals (methods, status codes, cookies, authentication vs authorization, headers)
 Burp Suite basics (Proxy, HTTP history, Intercept, Repeater)
 PortSwigger Web Security Academy: Access control (in progress)
 PortSwigger Web Security Academy: SQL injection, XSS, authentication, SSRF, API testing
 Python basics (HTTP requests, JSON, small automation scripts)
 LLM security: OWASP Top 10 for LLM Applications, prompt injection, Garak
Lab write-ups
#	Lab	Vulnerability class	Write-up
1	Unprotected admin functionality	Broken access control	read
2	Unprotected admin functionality with unpredictable URL	Broken access control	read
LLM security
#	Test	Tool	Write-up
1	First prompt injection scan	Garak	read

Environment
Kali Linux (VirtualBox VM and WSL 2)
Burp Suite Community Edition
curl, Python, Git
Garak (LLM vulnerability scanner)

Scope and ethics

 **No secrets, credentials, or session tokens are committed to this repository.**

Next steps
Finish the access control labs, then move on to authentication, SQL injection, XSS, and SSRF.
Learn enough Python to script HTTP requests and parse JSON.
Return to Garak with a better understanding of what each probe and detector does, and investigate individual failures by hand.
Work through the OWASP Top 10 for LLM Applications with small hands-on experiments.
