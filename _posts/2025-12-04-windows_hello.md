---
title: Windows Hello For Business
date: 2025-12-04 17:03:00 +0100
categories: [intune, conditional_access]
tags: [intune, mfa, conditional_access, "365", azure, microsoft, CIS]     # TAG names should always be lowercase
author: mith  
description: How to enable windows hello (WHfB) to work as phishing resistant MFA
toc: true
---

## Context
Seeing that more and more sophisticated phinshing are taking place. The goal if this article is to showcase how to mitigate these attacks. With Windows Hello For Business it becomes more difficult for the threat actor to steal mfa tokens. 
Another added benefit is that the user does not need other mfa devices, since everything is handled by the workstation. 

> **_NOTE:_** If you have on-prem environment/hybrid environment. Windows Hello can work as authentication as well. You need to configure cloud trust. This is not covered here. 

## Setup 

### Windows Hello For Business configuration
*Reference: CIS 97 Windows Hello For Business*
1. log into intune.microsoft.com 
2. navigate to configuration
3. Create new policy
4. Choose 'Windows 10 and later' and 'settings catalog'
5. add: 
    5.1. Enable ESS with Supported Peripherals: '(default and recommended for highest security)'
    5.2. Facial Features Use Enhanced Anti Spoofing: 'True'
    5.3. Require Security Device: 'True'
    5.4. Minimum PIN Length: '6'

### Windows Hello For Business Enrollment setting
This is to ensure that new enrolled devices have Windows hello enabled. 

1. intune.microsoft.com
2. Devices / Enrollment
3. Windows Hello For Business
    3.1. Use the same setting as above


## Test phishing resistant MFA
Before rolling out to all users in the organization it is importent to test the setting. 

1. intune.microsoft.com
2. Devices / Manage Devices / Conditional Access
3. Policies 
4. New policy
    4.1. Give it a descriptive namee eg.: Contoso - Grant - Require - Phishing resistant MFA
    4.2 Users or Agents 
        4.2.1 Select users this policy should apply to eg.: The test user
        4.2.2 At some point this should be all users
        4.2.3 Consider the impact on guest users
    4.3 Grant
        4.3.1 Require Authenticatinon Strenght: Phishing resistant MFA

> **_NOTE:_** Phones shall be a newer model to adhere to this conditional access policy. Also consider how admins should comply with this conditional access, it might not be wise to have WHfB for admin accounts, consider using the FIDO2 protocol for this. 