## 7. Server-Side Request Forgery (SSRF)

**Definition:**
Server Side Request Forgery (SSRF) is a vulnerability that occurs when an application retrieves remote resources without validating user input. If an attacker controls the requested resource, they may access internal services, read sensitive data, or compromise the server.

---

### In-Band SSRF

**Test:** Try to access internal services or files via API request redirection.

**Scenario:**
Use an API that fetches external URLs or files (e.g., `POST /api/v1/fetch?url=...`) and submit a URL like:

* `http://localhost/files`
* `http://127.0.0.1:80/admin`

Observe whether the server processes the request and returns sensitive internal content.

**Expected Behavior:**

* The API must validate the `url` parameter against a denylist or allowlist.
* Local or loopback addresses (e.g., `localhost`, `127.0.0.1`, `169.254.*`) should be blocked.
* The server should not return internal file or service data to the client.

---

**Test:** Try to scan ports on internal or adjacent systems via the API.

**Scenario:**
Submit requests with URLs like:

* `http://localhost:22`, `http://localhost:3306`, or `http://internal-service:8080`
* Use timing-based payloads or analyze differences in error messages to infer open/closed ports.

**Expected Behavior:**

* The API should not act as a proxy to internal resources.
* Requests to internal IP ranges (`10.0.0.0/8`, `192.168.0.0/16`, `172.16.0.0/12`) should be rejected.
* Proper SSRF protections must be in place, including DNS rebinding defenses.

---

### Out-of-Band SSRF

**Test:** Try sending a request to an external server (e.g., webhook) you control to confirm SSRF.

**Scenario:**
Use a service like Webhook.site or Burp Collaborator to get a temporary URL, e.g. `https://<id>.webhook.site`.

Submit this URL to a vulnerable endpoint like:

```http
POST /api/fetch
{
  "url": "https://<id>.webhook.site"
}
```

If the server initiates a request to that URL, you’ll receive a hit in your webhook log, confirming SSRF.

**Expected Behavior:**

* The API should restrict outbound requests or only allow safe domains.
* Unexpected requests to external URLs should not be possible.
* Server egress traffic should be monitored and rate-limited.
