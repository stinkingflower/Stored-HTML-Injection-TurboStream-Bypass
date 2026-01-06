# Technical Write-up: Sophisticated Stored HTML Injection via UI Redressing

![Security Research](https://img.shields.io/badge/Focus-Vulnerability%20Research-red)
![Vulnerability](https://img.shields.io/badge/Type-Stored%20HTML%20Injection-orange)
![Platform](https://img.shields.io/badge/Framework-Hotwire%20/%20Turbo-blue)

## 📌 Executive Summary
During a security assessment of a collaborative SaaS platform (**app.fizzy.do**), I identified a **Critical Stored HTML Injection** vulnerability. By bypassing client-side validation, an attacker can inject arbitrary, styled HTML into the application’s UI. 

This isn't just a cosmetic issue. Because the application utilizes **Hotwire/Turbo Stream** for real-time updates, the injected payload integrates seamlessly into the trusted DOM. This allows for sophisticated **UI Redressing** attacks that are virtually indistinguishable from legitimate system alerts.

---

## 🔍 Technical Analysis

### 1. The Root Cause: Client-Side Trust
The application relies on client-side editors (Trix/Lexical) for sanitization. However, the backend fails to re-validate or sanitize the `comment[body]` or `filename` parameters upon submission.
* **Vector:** Intercepting the `POST` request to `/comments` allows an attacker to bypass editor restrictions and inject raw, malicious HTML.

### 2. CSP Exploitation & UI Redressing
The existing **Content Security Policy (CSP)** includes `'unsafe-inline'` in the `style-src` directive. I leveraged this to:
* Apply complex inline CSS.
* Create a "pixel-perfect" fake UI overlay (e.g., fake login prompts or session alerts).
* Override existing elements to hide legitimate content.

### 3. Exploiting Turbo Streams
The use of **Turbo Streams** amplifies the impact. Since updates are pushed to all active users viewing a "card" or "workspace" in real-time, the payload executes immediately on the victim's screen without a page refresh, increasing the likelihood of a successful social engineering attack.

---

## 🛠 Proof of Concept (PoC)

### Reproduction Steps:
1. Navigate to any collaborative card on the platform.
2. Prepare a comment and intercept the request using **Burp Suite**.
3. Replace the `comment[body]` parameter with the following URL-encoded payload:
   ```html
   %22%3E%3Ch1%20style%3D%22color%3Ared%3Bfont-family%3Asans-serif%22%3ESession%20Expired%3C%2Fh1%3E%3Ca%20href%3D%22https%3A%2F%2Fattacker.com%22%20style%3D%22background%3Ared%3Bcolor%3Awhite%3Bpadding%3A10px%3Btext-decoration%3Anone%3Bborder-radius%3A5px%22%3ELogin%20Again%3C%2Fa%3E

Forward the request. The payload renders a fake "Session Expired" button for all users.

Visual Evidence:

    [!IMPORTANT] Below are the intercepted requests and the resulting UI redressing.

Burp Suite Request Interception: Figure 1: Bypassing client-side filters via proxy.

Malicious UI Rendering: Figure 2: The fake system alert rendered inside the trusted application domain.
🎯 Impact Assessment

    Credential Theft: By mimicking native "Session Expired" alerts, attackers can trick users into re-entering credentials on external phishing pages.

    Bypassing Trust: Since the attack originates from app.fizzy.do, it bypasses standard browser phishing filters and user suspicion.

    Organizational Risk: The payload is persistent. Any team member viewing the shared resource becomes a potential victim.

✅ Recommended Remediation

    Server-Side Sanitization: Implement a robust backend sanitization library (e.g., Ruby Sanitize gem) with a strict whitelist.

    CSP Hardening: Remove 'unsafe-inline' from style-src to prevent unauthorized UI modifications.

    Context-Aware Encoding: Ensure all user-supplied data is properly encoded before being rendered in the DOM.

Disclaimer: This report is for educational and professional portfolio purposes only. The vulnerability was reported responsibly according to the platform's security policy.




   
