# Popify Security Assessment

## Scope
- Target: popify.live
- Date: 2026-09-02
- Tester: Robin
- Tools: Kali Linux, Burp Suite
- Accounts: Two self-controlled test accounts

## Objective
Assess the application's API for common authorization and
access-control vulnerabilities.

## Methodology
- API endpoint discovery
- HTTP traffic analysis
- Authentication testing
- Authorization / BOLA testing
- Controlled A/B account testing
- Input and parameter analysis

## Findings

### No confirmed vulnerabilities
No confirmed vulnerabilities have been identified within the
endpoints tested so far.

### Positive authorization control
Endpoint: `authenticated-profile-data`

Changing the `sellerId` from Account A's resource to Account B's
resource while authenticated as Account A resulted in HTTP 401
and no Account B data was returned.

## Outstanding Tests
- Shipping authorization
- Listing/offer authorization
- Seller mutation authorization
- Remaining state-changing endpoints
