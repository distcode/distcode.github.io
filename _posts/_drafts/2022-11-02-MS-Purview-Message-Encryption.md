---
title: "MS Purview Message Encryption"
categories:
  - Microsoft
  - MIP
tags:
  - Microsoft
  - MIP
toc: false
---

<!-- NO TITLE IN ARTICLE !!! -->

This post describes how to work with MS Purview Message Encrpytion formerly knows as OME (Office Message Encryption). Activation, configuration and usage of that service is described in this article.
<!--more-->

MS Purview Message Encryption (MPME) is one way to encrypt messages in Office 365. This service is built on Azure Rights Management, which is part of Azure Information Protection. You can send encrypted mails to receipients in- and outside of your organization, regardless of the destination email address.

Two other ways to encrypt messages in Microsoft 365 is S/MIME and Information Rights Management (IRM). For more information on all types see the [documentation](https://learn.microsoft.com/en-us/microsoft-365/compliance/email-encryption?view=o365-worldwide).

### What could you expect from MPME?

In short:

- sending encrypted messages
- sending encrypted messages and prevent forwarding and printing
- receive encrypted messages and read the content without any other actions
- non-Microsoft mail accounts must use a web portal to read a mail. Authentication is required.
- setting the encryption manually by users or automatically via mail flow rules.

### MPME vs. MP advanced ME

MPME is available in two different form: MPME and Microsoft Purview Advanced Message Encryption. Second one offers in addition multiple templates, message revocation and message expiration.

| Type | License requirement |
| --- | --- |
| MPME (basic) | Azure Information Protection Plan 1 to the following plans: Exchange Online Plan 1 or 2, Office 365 F3, Microsoft 365 Business Basic or Standard, or Office 365 Enterprise E1 To receive Microsoft Purview Message Encryption
| MPME (advanced) | Microsoft E3/E5 or Office 365 E3/E5 |
[Source](https://learn.microsoft.com/en-us/office365/servicedescriptions/exchange-online-service-description/exchange-online-service-description)

### How to activate MPEM?

### How to create or change message templates?

```powershell
#Require ExchangeOnlineManagement

Set-OMEConfiguration
New-OMEConfiguration
Remove-OMEConfiguration
```

### Possible user actions

### Working with mail flow rules
