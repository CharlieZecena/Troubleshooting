# Lab 01: Investigating a DNS Query Timeout

## Objective
Distinguish an unsuccessful DNS query from a general internet
connectivity problem by comparing working and failing requests.

## Environment
- MacBook running macOS
- Tools: Terminal, ping, nslookup, curl, and dig
- Controlled simulation; no system network settings were changed

## 1. Establish a healthy baseline

### Test IP connectivity
Command:
`ping -c 4 1.1.1.1`

Result:
Four packets transmitted, four received, and 0% packet loss.
Average round-trip time: 14.391 ms.

Interpretation:
The Mac could reach this external IP address without DNS resolution.

### Test DNS resolution
Command:
`nslookup google.com`

Result:
The configured DNS resolver returned six IPv4 addresses.

Interpretation:
The resolver successfully answered the lookup. A
“non-authoritative answer” is normal for a recursive resolver.

### Test HTTPS connectivity
Command:
`curl -I --connect-timeout 10 https://google.com`

Result:
HTTP/2 301, with a redirect to https://www.google.com/.

Interpretation:
The HTTPS request received an HTTP response. The redirect
was not a connection failure.

## 2. Investigate an unexpected result

Command:
`dig @192.0.2.1 google.com +time=2 +tries=1`

Expected:
A timeout, because the destination is a documentation address
where a working DNS service was not expected.

Observed:
NOERROR, six answers, and a 13 ms response.

Interpretation:
The result contradicted the prediction. DNS interception or
redirection was a possible explanation, but was not verified.
The output alone did not establish which device answered.

## 3. Simulate an unavailable DNS endpoint

Command:
`dig @127.0.0.1 -p 55333 google.com +time=2 +tries=1`

Result:
“connection timed out; no servers could be reached”

Interpretation:
No DNS response arrived from the selected localhost endpoint
within the time allowed. No DNS service was expected on that
port, though a timeout alone does not prove a service is absent.

This command affected only its own query, not the Mac's DNS settings.

## 4. Verify normal DNS still works

Command:
`dig google.com +time=2 +tries=1`

Result:
- Status: NOERROR
- Answers: 1
- Returned address: 142.251.45.174
- Query time: 16 ms
- Configured resolver used port 53

Interpretation:
The configured DNS resolver continued to answer successfully.

## Conclusion
The simulated timeout was specific to the endpoint selected
for that query. Baseline tests demonstrated working IP
connectivity, DNS resolution, and an HTTPS response.

No system configuration repair was necessary. Removing the
command-specific endpoint override allowed the next query
to use the working configured resolver.

## Lessons Learned
- Test IP connectivity and DNS resolution separately.
- A timeout does not automatically mean the internet is down.
- “1 server found” in dig does not confirm a working DNS service.
- Follow observed results rather than assuming a test will fail.
- Record uncertainty when the cause has not been verified.
