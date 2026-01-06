# 🛡️ Technical Write-up: Sophisticated Stored HTML Injection via UI Redressing

![Security Research](https://img.shields.io/badge/Focus-Vulnerability%20Research-red)
![Vulnerability](https://img.shields.io/badge/Type-Stored%20HTML%20Injection-orange)
![Platform](https://img.shields.io/badge/Framework-Hotwire%20/%20Turbo-blue)

## 📌 Executive Summary
During a security assessment of a collaborative SaaS platform (**[REDACTED]**), I identified a **Critical Stored HTML Injection** vulnerability. By bypassing client-side validation, an attacker can inject arbitrary, styled HTML into the application’s UI. 

This isn't just a cosmetic issue. Because the application utilizes **Hotwire/Turbo Stream** for real-time updates, the injected payload integrates seamlessly into the trusted DOM. This allows for sophisticated **UI Redressing** attacks that are virtually indistinguishable from legitimate system alerts.

---

## 🔍 Technical Analysis

### 1. The Root Cause: Client-Side Trust
The application relies on client-side editors for sanitization. However, the backend fails to re-validate or sanitize the `comment[body]` or `filename` parameters upon submission.
* **Vector:** Intercepting the `POST` request to `/comments` allows an attacker to bypass editor restrictions and inject raw, malicious HTML.

### 2. CSP Exploitation & UI Redressing
The existing **Content Security Policy (CSP)** includes `'unsafe-inline'` in the `style-src` directive. I leveraged this to:
* Apply complex inline CSS to override existing UI elements.
* Create a "pixel-perfect" fake UI overlay (e.g., fake login prompts).
* Hide legitimate application content to force user interaction with the malicious payload.

### 3. Exploiting Turbo Streams
The use of **Turbo Streams** amplifies the impact. Updates are pushed to all active users viewing a "card" in real-time. The payload executes immediately without a page refresh, bypassing the user's natural suspicion.

---

## 🛠 Proof of Concept (PoC)

### Reproduction Steps:
1. Navigate to any collaborative card on the platform.
2. Prepare a comment and intercept the request using **Burp Suite**.
3. Replace the `comment[body]` parameter with the following URL-encoded payload.
   *(Note: This payload includes a hidden tracking pixel to verify execution via Burp Collaborator)*:
   ```html
   %22%3E%3Ch1%20style%3D%22color%3Ared%3Bfont-family%3Asans-serif%22%3ESession%20Expired%3C%2Fh1%3E%3Ca%20href%3D%22https%3A%2F%2Fattacker.com%22%20style%3D%22background%3Ared%3Bcolor%3Awhite%3Bpadding%3A10px%3Btext-decoration%3Anone%3Bborder-radius%3A5px%22%3ELogin%20Again%3C%2Fa%3E%3Cbr%3E%3Cimg%20src%3D%22https%3A%2F%2Fc5id6i5fep1tfce1y4o7xpl5cwin6fu4.oastify.com%22%20style%3D%22display%3Anone%22%3E%22


   Forward the request. The payload renders strictly within the application context.

🖼️ Visual Evidence & OAST Verification

To verify the vulnerability without harming real users, I utilized Burp Collaborator to confirm Out-of-band (OAST) interaction.

1. Payload Injection via Burp Repeater: Figure 1: Intercepted request showing the injected HTML/CSS payload and the successful 200 OK response from the server.

2. Execution Confirmed (OAST Interaction): Figure 2: Evidence of the application executing the injected payload and initiating HTTP/DNS requests to the attacker-controlled server.
🎯 Impact Assessment

    Credential Theft: By mimicking native alerts, attackers can trick users into re-entering credentials on phishing pages.

    Bypassing Trust: Attacks originate from the trusted domain, bypassing browser phishing filters.

    Organizational Risk: The payload is persistent and affects every team member viewing the shared resource.

✅ Recommended Remediation

    Server-Side Sanitization: Use a robust backend library (e.g., Ruby Sanitize gem) with a strict whitelist.

    CSP Hardening: Remove 'unsafe-inline' from style-src.

    Context-Aware Encoding: Ensure all user data is encoded before DOM rendering.

Disclaimer: This report is for educational and professional portfolio purposes only. The vulnerability was reported responsibly.
   
