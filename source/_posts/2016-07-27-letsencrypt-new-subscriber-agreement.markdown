---
layout: post
title: "Let's Encrypt Subscriber Agreementの比較"
date: 2016-07-27 23:00:00 +0900
comments: true
categories: letsencrypt
---
7月9日(土) に `Let's Encrypt Subscriber Agreement Update` というメールがきていて、
`Let's Encrypt Subscriber Agreement` が8月1日に v1.0.1 から v1.1.1 に更新されるというので、
内容をテキストファイルに落として diff をとってみました。

<!--more-->

## Let's Encrypt からのメール

Let's Encrypt からのメールは必要最低限しかこないのですが、
今のところ staging 環境で取得していた証明書の期限切れ通知メールと
今回の `Let's Encrypt Subscriber Agreement Update` のお知らせメールしか届いていません。

お知らせメールの内容は以下の通りでした。

```text
Let's Encrypt Subscriber,

We're writing to let you know that we are updating the Let's Encrypt Subscriber Agreement, effective August 1, 2016. You can find the updated agreement (v1.1.1) as well as the current agreement (v1.0.1) in the "Let's Encrypt Subscriber Agreement" section of the following page:

https://letsencrypt.org/repository/

Thank you for helping to secure the Web by using Let's Encrypt.

- The Let's Encrypt Team
```

## テキストへ変換

`poppler` に入っている `pdftotext` コマンドで PDF からテキストファイルに変換しました。

## diff

`git diff --no-index LE-SA-v1.*.txt` での比較結果は以下の通りでした。

```diff
diff --git a/LE-SA-v1.0.1-July-27-2015.txt b/LE-SA-v1.1.1-August-1-2016.txt
index d5a0b9d..e5850ef 100644
--- a/LE-SA-v1.0.1-July-27-2015.txt
+++ b/LE-SA-v1.1.1-August-1-2016.txt
@@ -1,6 +1,6 @@
-Version 1.0.1
-July 27, 2015
-Page 1 of 6
+Version 1.1.1
+August 1, 2016
+Page 1 of 7
 
 LET’S ENCRYPT
 SUBSCRIBER AGREEMENT
@@ -38,15 +38,20 @@ Key.
 “Public Key” — In Public Key Cryptography, this is the publicly-disclosed key that is used by the recipient
 to (i) validate Digital Signatures created with the corresponding Private Key and (ii) encrypt messages or
 files to be decrypted with the corresponding Private Key.
+“Key Compromise”— A Private Key is said to be compromised if its value has been disclosed to an
+unauthorized person, an unauthorized person has had access to it, or there exists a practical technique by
+which an unauthorized person may discover its value. A Private Key is also considered compromised if
+methods have been developed that can easily calculate it based on the Public Key or if there is clear
+evidence that the specific method used to generate the Private Key was flawed.
+
+Version 1.1.1
+August 1, 2016
+Page 2 of 7
 “Public Key Cryptography” — A type of cryptography that uses a Key Pair to securely encrypt and
 decrypt messages. One key encrypts a message, and the other key decrypts the message. One key is kept
 secret (the Private Key), and one is made available to others (the Public Key). These keys are, in essence,
 large mathematically-related numbers that form a unique pair. Either key may be used to encrypt a
 message, but only the other corresponding key may be used to decrypt the message.
-
-Version 1.0.1
-July 27, 2015
-Page 2 of 6
 “Repository” — An online system maintained by ISRG for storing and retrieving Let’s Encrypt Certificates
 and other information relevant to Let’s Encrypt Certificates, including information relating validity or
 revocation.
@@ -54,7 +59,7 @@ revocation.
 (“Valid From” or “Activation” date), and ending on the expiration date indicated in such Certificate (“Valid
 To” or “Expiry” date).
 “Your Certificate” — A Let’s Encrypt Certificate issued to You.
-2.!
+2.
 
 Effective Date, Term, and Survival
 2.1
@@ -78,7 +83,7 @@ Sections in this Agreement concerning privacy, indemnification, disclaimer of wa
 liability, governing law, choice of forum, limitations on claims against ISRG, and prohibitions on the use of
 fraudulently-obtained Certificates and expired Certificates shall survive any termination or expiration of
 this Agreement.
-3.!
+3.
 
 Your Warranties and Responsibilities
 3.1
@@ -86,16 +91,10 @@ Your Warranties and Responsibilities
 Warranties
 
 By requesting, accepting, or using a Let’s Encrypt Certificate:
-•!
-
-•!
-
-•!
-•!
-•!
-•!
-•!
-3.2
+•
+•
+•
+•
 
 You warrant to ISRG and the public-at-large that You are the legitimate registrant of the
 Internet domain name that is, or is going to be, the subject of Your Certificate, or that You are
@@ -103,21 +102,30 @@ the duly authorized agent of such registrant.
 You warrant to ISRG and the public-at-large that either (1) You did not obtain control of
 such domain name as the result of a seizure of such domain name, or (2) such domain name
 had no ongoing lawful uses at the time of such seizure.
-You warrant that all information in Your Certificate regarding You or Your domain
-name is accurate, current, reliable, complete, and not misleading.
-You warrant that all information You have provided to ISRG is accurate, current,
-complete, reliable, complete, and not misleading.
-You warrant that You rightfully hold the Private Key corresponding to the Public Key
-listed in Your Certificate.
-You warrant that You have taken all appropriate, reasonable, and necessary steps to secure
-and keep your Private Key secret.
-You warrant that You will not use Your Certificates to attack, defraud or intercept the
-traffic of others.
+You warrant to ISRG and the public-at-large that all information in Your Certificate
+regarding You or Your domain name is accurate, current, reliable, complete, and not
+misleading.
+You warrant to ISRG and the public-at-large that all information You have provided to
+ISRG is, and You agree that all information you will provide to ISRG at any time will be,
+accurate, current, complete, reliable, and not misleading.
+
+Version 1.1.1
+August 1, 2016
+Page 3 of 7
+•
+•
+
+3.2
+
+You warrant to ISRG and the public-at-large that You rightfully hold the Private Key
+corresponding to the Public Key listed in Your Certificate.
+You warrant to ISRG and the public-at-large that You have taken, and You agree that at
+all times You will take, all appropriate, reasonable, and necessary steps to maintain sole
+control of, secure, properly protect and keep secret and confidential the Private Key
+corresponding to the Public Key in Your Certificate (and any associated activation data or
+device, e.g. password or token).
 Changes in Certificate Information
 
-Version 1.0.1
-July 27, 2015
-Page 3 of 6
 If at any time You no longer control the Internet domain names associated with any of Your Certificates, or
 if any of the warranties in Section 3.1 above are no longer true with respect to any of Your Certificates in
 any other way, You will immediately request that ISRG revoke the affected Certificates. You may request
@@ -141,7 +149,7 @@ Key Pair Generation
 Your Key Pair (Public and Private Keys) will be generated by You or Your ACME Client Software on
 Your systems. You will submit the corresponding Public Key to ISRG and it will be incorporated into
 Your Certificate. ISRG will store Your Certificate in its Repository. ISRG will not have access to Your
-Private Key.
+Private Key. Your Private and Public Keys will remain Your property.
 We will use technical methods and protocols to verify that You have exclusive control over the subject
 Internet domain name. This verification is done solely to assist ISRG in determining whether to issue a
 Let’s Encrypt Certificate and is not a service being performed for Your benefit or on Your behalf.
@@ -149,46 +157,59 @@ Let’s Encrypt Certificate and is not a service being performed for Your benefi
 
 Inspection and Acceptance of Certificates
 
-You agree to immediately inspect the contents of Your Certificate (“Initial Inspection”), and to
-immediately request revocation if you become aware of any inaccuracies, errors, defects, or other
-problems (collectively, “Certificate Problems”) with Your Certificate. Your ACME Client Software
-may perform this task for You. You agree that You will have accepted Your Certificate when You
-first use Your Certificate or the corresponding Private Key after obtaining Your Certificate, or if You
-fail to request revocation of Your Certificate immediately following Initial Inspection.
+You warrant to ISRG and the public-at-large, and You agree, that You will immediately inspect the
+contents of Your Certificate (“Initial Inspection”), and to immediately request revocation if you
+become aware of any inaccuracies, errors, defects, or other problems (collectively, “Certificate
+Problems”) with Your Certificate. Your ACME Client Software may perform this task for You. You
+agree that You will have accepted Your Certificate when You first use Your Certificate or the
+corresponding Private Key after obtaining Your Certificate, or if You fail to request revocation of
+Your Certificate immediately following Initial Inspection.
 3.6
 
-Use of Your Certificate
-
-The purpose of Your Certificate is to encrypt Internet communications. ISRG is not responsible for any
-legal or other consequences resulting from or associated with the use of Your Certificate. You agree that
-You will not use Your Certificate for any purpose requiring fail-safe performance, such as the operation of
-public utilities or power facilities, air traffic control or navigation systems, weapons systems, or any other
-systems, the failure of which would reasonably be expected to lead to bodily injury, death or property
-damage.
+Installation and Use of Your Certificate
+
+You may reproduce and distribute Your Certificate on a nonexclusive and royalty-free basis, provided that
+it is reproduced and distributed in full and in compliance with this Agreement. You warrant to ISRG and
+the public-at-large, and You agree, that You will install Your Certificate only on servers that are accessible
+
+Version 1.1.1
+August 1, 2016
+Page 4 of 7
+at the subjectAltName(s) listed in Your Certificate, and that you will use Your Certificate solely in
+compliance with all applicable laws and solely in accordance with this Agreement. Your Certificate will
+remain the property of ISRG, subject to Your right to use it as set forth in this Agreement.
+The purpose of Your Certificate is to authenticate and encrypt Internet communications. ISRG is not
+responsible for any legal or other consequences resulting from or associated with the use of Your
+Certificate. You agree that You will not use Your Certificate for any purpose requiring fail-safe
+performance, such as the operation of public utilities or power facilities, air traffic control or navigation
+systems, weapons systems, or any other systems, the failure of which would reasonably be expected to
+lead to bodily injury, death or property damage.
 3.7.
 
 When to Revoke Your Certificate
 
-You must immediately request that Your Certificate be revoked if: (i) You suspect or discover that
-Your Private Key has been, or is in danger of being, lost, stolen, otherwise compromised, or subjected
-to unauthorized use, or (ii) any information in Your Certificate is no longer accurate, current or
-complete, or any such information becomes misleading. You may make a revocation request to ISRG
-
-Version 1.0.1
-July 27, 2015
-Page 4 of 6
-using ACME Client Software. You should also notify anyone who may have relied upon Your use of
-Your Certificate that Your encrypted communications may have been subject to compromise
+You warrant to ISRG and the public-at-large, and You agree, that You will immediately request that
+Your Certificate be revoked if: (i) there is any actual or suspected misuse or Key Compromise of the
+Private Key associated with the Public Key included in Your Certificate, or (ii) any information in Your
+Certificate is, or becomes, misleading, incorrect or inaccurate. You may make a revocation request to
+ISRG using ACME Client Software. You should also notify anyone who may have relied upon Your
+use of Your Certificate that Your encrypted communications may have been subject to compromise.
 3.8
 
 When to Cease Using Your Certificate
 
-You must immediately cease using Your Certificate if: (i) You suspect or discover that the Private Key
-corresponding to Your Certificate has been or may be stolen, lost, or otherwise compromised or subjected
-to unauthorized use, (ii) any information in Your Certificate is no longer accurate, current or complete, or
-any such information becomes misleading, or (iii) upon the revocation or expiration of Your Certificate.
+You warrant to ISRG and the public-at-large, and You agree, that You will promptly cease using Your
+Certificate (i) if any information in Your Certificate is, or becomes, misleading, incorrect or inaccurate, or
+(ii) upon the revocation or expiration of Your Certificate.
 3.9
 
+When to Cease Using Your Private Key
+
+You warrant to ISRG and the public-at-large, and You agree, that You will promptly cease all use of the
+Private Key corresponding to the Public Key included in Your Certificate upon revocation of Your
+Certificate for reasons of known or suspected Key Compromise.
+3.10
+
 Indemnification
 
 You agree to indemnify and hold harmless ISRG and its directors, officers, employees, agents, and
@@ -207,20 +228,17 @@ ISRG’s Rights and Responsibilities
 
 Privacy
 
-Because others may rely on your use of Your Certificate to encrypt Internet communications, much of the
+Because others may rely on your use of Your Certificates to encrypt Internet communications, much of the
 information You send to ISRG will be published by ISRG and will become a matter of public record.
-However, information used for account-recovery purposes (such as Your email address and telephone
-number) (“Private Recovery Information” or “PRI”) will NOT be published by ISRG. ISRG will not sell
-or share your Private Recovery Information. ISRG may disclose Private Recovery Information, however,
-if compelled to do so by court order or other compulsory legal process. If legally permissible and to the
-extent possible and within ISRG’s control, and if you have provided ISRG with an email address, ISRG
-will send an email to such address notifying You of the potential disclosure. ISRG may also disclose your
-PRI if ISRG believes disclosure is necessary to prevent loss of life, personal injury, damage to property, or
-significant financial harm.
+ISRG’s collection, storage, use and disclosure of such information are governed by the Let’s Encrypt
+Privacy Policy at: https://letsencrypt.org/privacy/.
 4.2
 
 Certificate Repository
 
+Version 1.1.1
+August 1, 2016
+Page 5 of 7
 During the term of the Agreement, ISRG will operate and maintain a secure online Repository that is
 available to authorized relying parties that contains: (i) all past and current Let’s Encrypt Certificates
 (including, as applicable, Your Certificate) and (ii) a CRL or similar online database indicating whether
@@ -231,36 +249,36 @@ public to access this information.
 
 Suspension and Revocation
 
-ISRG may immediately suspend Your Certificate if any party notifies ISRG that Your Certificate is
-invalid or has been compromised. ISRG will determine, in its sole discretion, whether to revoke Your
-Certificate or restore it to valid status. If You or Your agent requests that Your Certificate be revoked,
-ISRG will revoke Your Certificate and update the Repository as soon as practical. If a request for
-revocation is signed by your Private Key, then ISRG will automatically deem the request to be valid.
-
-Version 1.0.1
-July 27, 2015
-Page 5 of 6
-ISRG may also, without advance notice, revoke Your Certificate if ISRG determines, in its sole discretion,
-that: (i) Your Certificate was not properly issued or was obtained through misrepresentation, concealment,
-or fraud; (ii) Your Certificate has become, or appears to have become, unreliable; (iii) the security of the
-Private Key corresponding to Your Certificate has been or may be stolen, lost, or otherwise compromised,
-or subject to unauthorized use; (iv) any information in Your registration with ISRG or Your request for a
-Let’s Encrypt Certificate has changed or has become false or misleading; (v) You have violated any
-applicable law, agreement or other obligation; (vi) You request revocation; (vii) ISRG is legally required to
-revoke Your Certificate pursuant to a valid court order issued by a court of competent jurisdiction; (viii)
-this Agreement has terminated; or (iv) there are other reasonable and lawful grounds for revocation. ISRG
-will provide notice of revocation via email to the email address of record.
+You acknowledge and accept that ISRG may immediately suspend Your Certificate if any party notifies
+ISRG that Your Certificate is invalid or has been compromised. ISRG will determine, in its sole discretion,
+whether to revoke Your Certificate. If You or Your agent requests that Your Certificate be revoked, ISRG
+will revoke Your Certificate and update the Repository as soon as practical. If a request for revocation is
+signed by your Private Key, then ISRG will automatically deem the request to be valid. You also
+acknowledge and accept that ISRG may, without advance notice, immediately revoke Your Certificate if
+ISRG determines, in its sole discretion, that: (i) Your Certificate was not properly issued or was obtained
+through misrepresentation, concealment, or fraud; (ii) Your Certificate has become, or appears to have
+become, unreliable; (iii) the security of the Private Key corresponding to Your Certificate has been or may
+be stolen, lost, or otherwise compromised, or subject to unauthorized use; (iv) any information in Your
+registration with ISRG or Your request for a Let’s Encrypt Certificate has changed or has become false or
+misleading; (v) You have violated any applicable law, agreement (including this Agreement), or other
+obligation; (vi) Your Certificate is being used, or has been used, to enable any criminal activity (such as
+phishing attacks, fraud or the distribution of malware); (vii) Your Certificate is being used, or has been
+used, to intercept the traffic of others; (viii) You request revocation; (ix) ISRG is legally required to revoke
+Your Certificate pursuant to a valid court order issued by a court of competent jurisdiction; (x) this
+Agreement has terminated; or (xi) there are other reasonable and lawful grounds for revocation. ISRG will
+provide notice of revocation via email to the email address of record.
 4.4
 
 IMPORTANT DISCLAIMER OF WARRANTIES AND LIMITATION OF
 LIABILITY
 
-LET’S ENCRYPT CERTIFICATES AND SERVICES ARE PROVIDED “AS-IS.”
-ISRG DISCLAIMS ANY AND ALL WARRANTIES OF ANY TYPE, WHETHER EXPRESS
-OR IMPLIED, INCLUDING AND WITHOUT LIMITATION ANY IMPLIED WARRANTY OF
-TITLE, NON-INFRINGEMENT, MERCHANTABILITY, OR FITNESS FOR A PARTICULAR
-PURPOSE, IN CONNECTION WITH ANY ISRG SERVICE OR LET’S ENCRYPT
-CERTIFICATE.
+EXCEPT AS EXPRESSLY SET FORTH IN ISRG’S CERTIFICATE POLICY AND
+CERTIFICATE PRACTICE STATEMENT, LET’S ENCRYPT CERTIFICATES AND
+SERVICES ARE PROVIDED “AS-IS” AND ISRG DISCLAIMS ANY AND ALL
+WARRANTIES OF ANY TYPE, WHETHER EXPRESS OR IMPLIED, INCLUDING AND
+WITHOUT LIMITATION ANY IMPLIED WARRANTY OF TITLE, NON-INFRINGEMENT,
+MERCHANTABILITY, OR FITNESS FOR A PARTICULAR PURPOSE, IN CONNECTION
+WITH ANY ISRG SERVICE OR LET’S ENCRYPT CERTIFICATE.
 BECAUSE LET’S ENCRYPT CERTIFICATES ARE ISSUED FREE-OF-CHARGE AS A PUBLIC
 SERVICE, ISRG CANNOT ACCEPT ANY LIABILITY FOR ANY LOSS, HARM, CLAIM, OR
 ATTORNEY’S FEES IN CONNECTION WITH SUCH CERTIFICATES. ACCORDINGLY, YOU
@@ -275,6 +293,10 @@ REGULATION, COMMON LAW, OR ANY OTHER SOURCE OF LAW, STANDARD OF CARE,
 CATEGORY OF CLAIM, NOTION OF FAULT OR RESPONSIBILITY, OR THEORY OF
 RECOVERY. THE PARTIES AGREE THAT THIS DISCLAIMER IS INTENDED TO BE
 CONSTRUED TO THE FULLEST EXTENT ALLOWED BY APPLICABLE LAW.
+
+Version 1.1.1
+August 1, 2016
+Page 6 of 7
 BY WAY OF FURTHER EXPLANATION REGARDING THE SCOPE OF THE DISCLAIMER,
 AND WITHOUT WAIVING OR LIMITING THE FOREGOING IN ANY WAY, ISRG DOES NOT
 MAKE, AND ISRG EXPRESSLY DISCLAIMS, ANY WARRANTY REGARDING ITS RIGHT TO
@@ -297,9 +319,6 @@ choice of law and conflicts of law principles.
 
 Choice of Forum
 
-Version 1.0.1
-July 27, 2015
-Page 6 of 6
 Any claim, suit or proceeding arising out of this Agreement must be brought in a state or federal court
 located in San Jose, California.
 5.3
@@ -339,6 +358,10 @@ If any provision of this Agreement is found to be invalid, unenforceable, or con
 Agreement will be deemed amended by modifying such provision to the extent necessary to make it valid
 and enforceable while preserving its intent or, if that is not possible, by striking the provision and enforcing
 the remainder of this Agreement.
+
+Version 1.1.1
+August 1, 2016
+Page 7 of 7
 5.8
 
 Authorization of ISRG to Send Emails
```
