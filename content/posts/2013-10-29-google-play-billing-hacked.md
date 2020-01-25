---
layout: post
title: Google Play In-App Billing Library Hacked
author: Dominik Schürmann
date: 2013-10-29
slug: google-play-billing-hacked
aliases:
    - "/2013/10/29/google-play-billing-hacked.html"
---

## Introduction
I successfully exploited two bugs in Google Play In-App Billing Library, which allow to impersonate the Google Play billing service and circumvent the signature verification. I was able to retrieve unlimited amounts of in-app items in games like Temple Run 2, which uses this library.

**Edit 0: More information about the disclosure process**  
**Edit 1: Timeline added**  
**Edit 2: Apology from Google**

This blog post was released earlier than previously negotiated with Google, because Google was unable to provide proper attribution (they even stated "we recently discovered" in an email sent to Android developers). Additionally, they ignored questions regarding other bad security practices in this library. More information can be found before the conclusion.

## Screenshots

[![](/images/billinghack1-150x150.png)](/images/billinghack1.png)
[![](/images/billinghack2-150x150.png)](/images/billinghack2.png)
[![](/images/billinghack3-150x150.png)](/images/billinghack3.png)

## Vulnerable libraries
All [Google Play Billing Library v3](http://developer.android.com/training/in-app-billing/preparing-iab-app.html) versions before Oct, 8 distributed via Android SDK and [marketbilling on Googlecode](https://code.google.com/p/marketbilling/).

## Problem description
  * Any app can define a new intent-filter with a high priority to impersonate the official in-app billing service. See my [AndroidManifest.xml](https://github.com/dschuermann/billing-hack/blob/master/AndroidManifest.xml) how to do that.
  * [Signature verification returns true](https://github.com/dschuermann/billing-hack/blob/master/src/org/billinghack/google/util/Security.java#L73) if given ``INAPP_DATA_SIGNATURE`` is an empty String ``""``.

## Proposed fixes
[Browse the diff 7bc191a004483a1034b758e1df0bda062088d840](https://code.google.com/p/marketbilling/source/detail?r=7bc191a004483a1034b758e1df0bda062088d840) and merge the modifications into the appropriate parts of your code.

## Proof of concept
  * Clone [https://github.com/dschuermann/billing-hack](https://github.com/dschuermann/BillingHack), compile the project, and install the APK on your device.
  * Then install Temple Run 2 or similar apps, and go to the in-app items and "buy" some items.

## Remarks about the vulnerabilities
The impersonation vulnerability is quite interesting, because it shows that an Android principle regarding IPC with Intents was ignored. If an app, e.g., Google Play Services, registers an Intent filter providing an AIDL remote service, any other app can also do that using the same name. To prevent collisions, the simplest fix is to restrict the scope of of the Intent used for binding to that service from client side by setting ``bindIntent.setPackage("com.android.vending")``.

The other bug is a typical crypto implementation fail, but there is also a take-home message here. The verify method checks if the signature String is empty [before going on to the actual verification](https://code.google.com/p/marketbilling/source/diff?spec=svn7bc191a004483a1034b758e1df0bda062088d840&amp;r=7bc191a004483a1034b758e1df0bda062088d840&amp;format=side&amp;path=/v3/src/com/example/android/trivialdrivesample/util/Security.java). Unfortunately the method returns true per default at the bottom of the method. In my opinion verification methods should be always programmed with this in mind: always return false, return true only on success!

## Remarks about responsible disclosure process
I reported the bug via security@google.com and they confirmed my findings after 7 days and commited [7bc191a004483a1034b758e1df0bda062088d840](https://code.google.com/p/marketbilling/source/detail?r=7bc191a004483a1034b758e1df0bda062088d840). They told me that they will send emails to "high visibility partners", so they can fix this bug before disclosure and asked for 4 weeks until I go public. I answered that I want to be mentioned in those emails, but got a response that states "[...] we are unable to provide attribution." without further reasons. Besides answering slowly and skipping some questions I sent, this is unfair behavior and clearly a violation of common practices in vulnerability research. 5 days later I received an email to my Google Play developer email account, informing me about the following:

> "If you previously used the In-app billing sample code to build your in-app billing system, please use the recently-updated sample code as it addresses an exploitable flaw we recently discovered (note that this only affects the helper sample code; the core system and in-app billing service itself was not affected)."

Conclusively, I think it's unfair that they were unable to provide attribution, especially as I explicitly asked about mentioning me as a security researcher in prior communication with them. Additionally Google payed no bug bounty, although this library is quite important as many app developers rely on it for in-app billing.

## Google's apology
Google sent me an email yesterday apologizing for not providing proper attribution. I really appreciate this! Thanks for rectifying your behavior:

> "I want to apologize for the initial lack of public credit for your discovery of a security bug in the market billing library, and our improper claim that the issue was internally discovered. This was the result of inadequate communication on our part. It was a mistake, and I want to make sure we correct it. We are now providing proper attribution in the README for the library: [https://code.google.com/p/marketbilling/source/browse/v3/README](https://code.google.com/p/marketbilling/source/browse/v3/README)"

## Timeline
2013-10-02 Report to security@google.com  
2013-10-05 Response that they are "investigating now"  
2013-10-09 Confirmation of this bug, bug fix commit, and they told me that they will inform "high visibility partners"  
2013-10-09 I answered that I want to be mentioned in those mails (besides another question)  
2013-10-16 They answered that they "are unable to provide attribution"  
2013-10-21 I received the email to my developer account address  
2013-10-29 Public disclosure in this blog  
2013-11-08 Apology from Google  

## Advertisement
If you are a programmer, consider working with us on [OpenKeychain](http://www.openkeychain.org) to provide secure emailing for Android. I will help on pull requests and be happy about every commit!
