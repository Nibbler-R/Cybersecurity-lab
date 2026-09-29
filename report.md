# Uusfam.fi Security Assessment Report

**Assessment Type:** Authorized Web Application Security Assessment
**Target:** `uusfam.fi`
**Assessment Date:** 2 September 2026
**Tester:** Robin Uus
**Testing Environment:** Kali Linux, Burp Suite Community Edition, OWASP ZAP
**Scope:** Uusfam storefront and client-facing functionality

---

## 1. Executive Summary

An authorized security assessment was conducted against the Uusfam.fi storefront to identify common web application security weaknesses, with particular focus on client-side manipulation, cart functionality, input validation, business logic, API traffic, and security configuration.

Testing combined **manual penetration testing using Burp Suite Community Edition with automated security scanning using OWASP ZAP**.

The assessment focused on functionality that could be safely tested without affecting other customers, real payments, or third-party infrastructure outside the Uusfam storefront.

### Overall Result

**No confirmed security vulnerabilities were identified during this assessment.**

Several security controls behaved as expected, including:

* Negative cart quantities were rejected.
* Invalid cart line references were rejected.
* Invalid discount codes did not produce a discount.
* Cart prices remained server-controlled.
* Cart totals remained mathematically consistent.
* Security-related HTTP headers were present.
* Shopify and PayPal telemetry endpoints were identified and excluded from active security testing.
* No obvious client-side price manipulation vulnerability was identified.
* OWASP ZAP automated scanning did not result in a confirmed vulnerability.

This assessment does **not** establish that Uusfam.fi is completely secure. Testing was limited to the functionality and attack surface accessible during the assessment period.

---

# 2. Scope

The following areas were examined:

* Public Uusfam storefront
* Product pages
* Shopping cart
* Cart quantity manipulation
* Cart line manipulation
* Discount-code handling
* Server-side price handling
* Storefront GraphQL requests
* Client-side/API traffic
* HTTP security headers
* Automated web vulnerability scanning
* Shopify-related storefront functionality
* Basic business-logic behavior

### Out of Scope

The following were intentionally excluded:

* Shopify infrastructure itself
* PayPal infrastructure
* Other customers' accounts or data
* Real payment manipulation
* Destructive testing
* Denial-of-service testing
* Brute-force attacks
* Inventory depletion
* Accessing or modifying unrelated customer information

---

# 3. Methodology

Testing was performed using both automated and manual security-testing techniques.

### Manual Testing — Burp Suite

Burp Suite Community Edition was used as an HTTP proxy to:

1. Capture browser traffic.
2. Identify application and API endpoints.
3. Inspect requests and responses.
4. Identify Shopify storefront functionality.
5. Modify controlled requests.
6. Test cart input validation.
7. Test business-logic behavior.
8. Examine HTTP security headers.

Testing was performed against the tester's own cart/session wherever state-changing functionality was involved.

### Automated Testing — OWASP ZAP

OWASP ZAP was also used to perform automated web application security scanning against the Uusfam storefront.

The ZAP assessment was used to identify potential:

* Security-header issues
* Web configuration issues
* Common web application vulnerabilities
* Unexpected exposed resources
* Other findings detectable through automated passive/active scanning

Automated scanner results were treated as **potential findings rather than confirmed vulnerabilities** and were considered together with the manual Burp Suite testing.

The automated testing did not result in a confirmed exploitable vulnerability within the tested scope.

---

# 4. Attack Surface Discovery

During traffic analysis, several types of requests were observed.

### 4.1 Shopify Monorail Telemetry

```text
/.well-known/shopify/monorail/unstable/produce_batch
```

This was identified as Shopify telemetry/analytics traffic rather than an application business-logic endpoint.

No security testing was performed against this endpoint because it belongs to Shopify's telemetry functionality and is outside the Uusfam application security scope.

### 4.2 Analytics Collection

Requests to:

```text
/api/collect
```

were identified as analytics-related traffic.

No vulnerability was identified or pursued through this endpoint.

### 4.3 Storefront GraphQL

Requests to:

```text
/api/2026-01/graphql.json
```

were observed.

The `GetStorefrontData` operation was inspected and appeared to retrieve normal storefront information.

A separate `limitedCartQuery` operation was also observed. This operation read cart information but did not expose an obvious mutation mechanism.

No vulnerability was identified through these requests.

### 4.4 Shopify Cart Endpoints

The following cart functionality was observed:

```text
/cart.json
/cart/change
/cart/update
```

These endpoints were used for controlled cart-security testing.

---

# 5. OWASP ZAP Automated Assessment

OWASP ZAP was used as a secondary automated testing tool against the storefront.

The purpose of the scan was to identify common web application security weaknesses that might not be immediately apparent through manual browsing.

Potential scanner observations were evaluated against the application's actual behavior where applicable.

No confirmed exploitable vulnerability was identified from the ZAP assessment.

### Result

**PASS — No confirmed vulnerability identified**

---

# 6. Security Tests

## 6.1 Cart Quantity Manipulation

A controlled test was performed against the `/cart/change` endpoint.

### Test Cases

| Test               | Result       |
| ------------------ | ------------ |
| Quantity `2`       | Accepted     |
| Quantity `0`       | Item removed |
| Quantity `-1`      | Rejected     |
| Invalid line `999` | Rejected     |

A negative quantity resulted in an HTTP 400 response.

### Result

**PASS**

No negative-quantity manipulation vulnerability was identified.

---

# 7. Invalid Cart Line Testing

A request referencing a nonexistent cart line was tested.

The server returned an HTTP 400 response and did not perform an unintended cart modification.

### Result

**PASS**

---

# 8. Server-Side Price Validation

Price manipulation was investigated to determine whether product prices could potentially be controlled by client-supplied values.

A cart operation involving a high-value product was examined.

The cart modification request contained cart line and quantity information but did not contain a client-supplied price parameter.

The server independently returned pricing information, including:

* Original price
* Discounted price
* Line price
* Final price
* Cart total

The returned values were consistent with the actual product price.

### Result

**PASS — No client-side price manipulation vulnerability identified**

---

# 9. Discount-Code Validation

An intentionally invalid test discount code was submitted.

The server accepted the request at the HTTP level but marked the discount as not applicable.

The cart total remained unchanged and no unauthorized discount was applied.

### Result

**PASS**

---

# 10. HTTP Security Headers

Several security-related HTTP response headers were observed, including:

```text
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
Strict-Transport-Security
Content-Security-Policy
X-Permitted-Cross-Domain-Policies: none
```

These provide additional protection against several common browser-based attacks.

### Result

**PASS / Positive Security Observation**

---

# 11. Findings Summary

| ID      | Finding                             | Severity        | Status            |
| ------- | ----------------------------------- | --------------- | ----------------- |
| UUS-001 | Negative cart quantity manipulation | None            | Not vulnerable    |
| UUS-002 | Invalid cart line manipulation      | None            | Not vulnerable    |
| UUS-003 | Client-side price manipulation      | None identified | Pass              |
| UUS-004 | Invalid discount-code manipulation  | None            | Not vulnerable    |
| UUS-005 | Cart price consistency              | None            | Pass              |
| UUS-006 | Security headers                    | Informational   | Positive          |
| UUS-007 | Shopify/analytics endpoint exposure | Informational   | Expected behavior |
| UUS-008 | OWASP ZAP automated scan            | None confirmed  | Pass              |

### Confirmed Vulnerabilities

**None identified.**

---

# 12. Limitations

The assessment was performed using Burp Suite Community Edition, OWASP ZAP, and manual testing.

The following areas were not fully assessed:

* Authenticated customer-account authorization
* Cross-account access-control testing
* Order-history authorization
* Customer profile authorization
* Checkout/payment workflow
* Inventory race conditions
* Coupon enumeration
* Shopify application/backend configuration
* Server-side custom applications
* Administrative interfaces
* Source-code review
* Dependency vulnerability analysis

The absence of findings in these areas should therefore not be interpreted as evidence that no vulnerabilities exist.

---

# 13. Recommended Next Steps

For a future assessment, the following areas would provide the greatest additional coverage:

### Priority 1 — Authorization Testing

Use two controlled customer accounts to verify that one account cannot access or modify the other account's:

* Profile
* Orders
* Account information
* Customer-specific resources

### Priority 2 — GraphQL Authorization

Investigate authenticated GraphQL operations involving customer and order functionality.

### Priority 3 — Checkout Logic

Using controlled test products/orders, verify that:

* Cart totals remain consistent during checkout.
* Shipping costs are server-calculated.
* Discounts are validated server-side.
* Product quantities are revalidated before checkout.
* Product prices cannot be modified between cart and checkout.

### Priority 4 — Client-Side Code Review

Review publicly delivered JavaScript for:

* Hardcoded application secrets
* Unexpected API endpoints
* Debug functionality
* Development endpoints
* Sensitive information exposed to unauthenticated users

---

# 14. Conclusion

The Uusfam.fi storefront was subjected to an authorized security assessment combining **manual penetration testing with Burp Suite Community Edition and automated scanning with OWASP ZAP**.

The assessment covered several common web application and business-logic attack vectors, including cart manipulation, input validation, discount handling, price integrity, API traffic, and security configuration.

**No confirmed vulnerabilities were identified during the testing performed.**

The strongest positive observations were the rejection of invalid cart quantities and cart references, server-side price handling, proper invalid discount handling, and the presence of several important HTTP security headers.

The most valuable area for a future assessment would be **authenticated authorization testing**, particularly customer-account and order-access controls using two controlled accounts.

### Final Assessment

**No vulnerabilities identified within the tested scope and methodology.**
