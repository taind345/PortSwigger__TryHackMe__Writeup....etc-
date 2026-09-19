# I)Intro 
## What is SSRF?

**Server-Side Request Forgery (SSRF)** is a vulnerability that allows an attacker to cause the server-side application to make HTTP requests to a destination of the attacker's choosing. In a typical SSRF attack, the attacker manipulates a parameter that the application uses to construct a server-side request, redirecting it to an internal service, a cloud metadata endpoint, or an external server under their control.

SSRF exploits the trust that internal systems place in the application server. Backend services, databases, and cloud infrastructure often accept requests from the server without additional authentication, because they assume any request arriving from a trusted internal IP address is legitimate. An attacker who can control where the server sends its requests effectively inherits that trust.

## Types of SSRF

There are two categories of SSRF vulnerability, and the distinction affects how exploitation is approached.

|Type|Response Visible?|Description|
|---|---|---|
|Regular SSRF|Yes|The response from the back-end request is returned in the application's front-end response. The attacker can directly read the output.|
|Blind SSRF|No|The application makes the back-end request but does not return the response. The attacker must use indirect methods to confirm exploitation.|

With a regular SSRF, if an attacker forces the server to fetch an internal admin page, the contents of that page appear directly in the HTTP response. This provides immediate, readable output.

With a Blind SSRF, the application may display a fixed success message regardless of the back-end outcome. However, blind SSRF can still be exploited. An attacker can confirm the vulnerability by directing the request to a server they control (using a tool such as Burp Collaborator) and observing whether a callback arrives. Differences in response time or error messages between reachable and unreachable hosts can also reveal information about internal infrastructure.

## Impact

The impact of SSRF depends on what internal services are reachable from the application server.

|Impact|Description|
|---|---|
|Access to internal endpoints|Admin panels, configuration interfaces, and monitoring dashboards that are not exposed to the internet become reachable. IP-based access controls are bypassed because the request originates from the server itself.|
|Sensitive data exposure|Backend databases, private APIs, and internal tooling that trust the server's network position may return customer data, organisational records, or application secrets.|
|Internal network reconnaissance|By sending requests to different IP addresses and ports, an attacker can map internal hosts and services using variations in response time, status codes, and error messages.|
|Cloud metadata theft|Cloud providers such as AWS, GCP, and Azure expose instance metadata at `169.254.169.254`. An attacker who reaches this endpoint can retrieve temporary credentials, IAM role details, and instance configuration data.|
|Credential and token leakage|Authentication tokens and secrets passed between internal services can be intercepted, particularly where back-end communication runs over unencrypted HTTP.|

In the following tasks, we will examine how SSRF manifests in different application features, how to identify it, and how to bypass common defences.

> [!NOTE]
> phần này nói về những thứ đã biết

# 2-

![[Pasted image 20260919145554.png]]![[Pasted image 20260919145543.png]]