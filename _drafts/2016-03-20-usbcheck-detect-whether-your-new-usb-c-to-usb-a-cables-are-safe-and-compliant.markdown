---
layout: post
title: USBCheck – Detect whether your new USB-C to USB-A cables are safe and compliant
---
USB-C is a fairly new USB standard which we're now seeing roll out to new mobile devices.

However many cheap (and not so cheap) cables from [Amazon](http://www.amazon.co.uk/s/ref=nb_sb_noss?url=search-alias%3Daps&field-keywords=USB-C+to+USB-A) and other online retailers are cutting corners and producing dangerous cables which could damange your device and cause electrical fire. 

These cables fail to have the correct electrical components in them to stop the client trying to draw more power than the host can provide. 

A Google engineer named [Benson Leung](https://plus.google.com/+BensonLeung) has taken it upon himself to [review and rate](https://plus.google.com/collection/s0Inv) USB-C cables avaiable through Amazon to help identify which cables are safe and which you should avoid.

However there may come a time where you need to use a USB-C cable but want piece of mind that it isn't going to destroy your phone or cause fire. I have just published an Android app to the Play Store named [USBCheck](https://play.google.com/store/apps/details?id=jacob.uk.com.usbcheck) for this very task.

[USBCheck](https://play.google.com/store/apps/details?id=jacob.uk.com.usbcheck) uses low level Linux driver information to detect whether the connected USB-C to USB-A cable is drawing less than 3As of power from the host (anything less than 3A is considered safe and compliant).

The app is [available on the Play Store]([USBCheck](https://play.google.com/store/apps/details?id=jacob.uk.com.usbcheck)) now, it's entirley open source but currently only supports the below devices:

- Huawei Nexus 6P
- LG Nexus 5X

The code is [open source on GitHub](https://github.com/imjacobclark/USBCheck) and I would encourage fellow developers to contribute support for other Android devices on the market with USB-C hardware.

<a href="https://play.google.com/store/apps/details?id=jacob.uk.com.usbcheck&utm_source=global_co&utm_medium=prtnr&utm_content=Mar2515&utm_campaign=PartBadge&pcampaignid=MKT-Other-global-all-co-prtnr-py-PartBadge-Mar2515-1"><img alt="Get it on Google Play" src="https://play.google.com/intl/en_us/badges/images/generic/en-play-badge.png" width="150px" /></a>
