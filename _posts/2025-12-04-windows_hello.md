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

## Setup 
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

## Test