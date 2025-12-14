---
title: Certificate Authentication
date: 2025-12-04 17:03:00 +0100
categories: [intune, conditional_access]
tags: [intune, mfa, conditional_access, "365", azure, microsoft]     # TAG names should always be lowercase
author: mith
description: How to enable phishing resistant mfa with certificates, and example of why it works
toc: true
---

## Context
Seeing that more and more sophisticated phinshing are taking place. The goal if this article is to showcase how to mitigate these attacks. With certificate based MFA it becomes more difficult for the threat actor to steal tokens. This is also a spin off, off the other article about Windows Hello For Business. The difference is that certifiacte based authentication does not only work on windows devices, but for the most part this works on all devices. Currently I have only tested it on Windows and Ubuntu. 

## Setup 
(smallstep)[https://smallstep.com/docs/step-cli/installation/#debianubuntu]
1. Start a new VM. This will be our certificate issuer

## Test