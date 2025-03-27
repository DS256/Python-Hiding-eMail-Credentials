# Python-Hiding-eMail-Credentials
How to store, but hide, credentials used in sending emails from Python

## Problem
You are writing a Python script to send an email using the EMAIL and SMTPLIB. You need to provide email address, user name, password and server but don't want others to be able to see them. How to hide them?

## Solution Discussion
In the past, I written a 'C' program to create an encrypted value for credentials and a 2nd 'C' program to decrypt these. I only distributed the SO 'C' compiled file so it was quite secure. However, this was cumbersome also not something I could share without exposing my algorithm.

When you compile a Python script from PY to PYC all comments are lost but character strings can seen in, for example, in a HEXDUMP of the PYC. That means the credentials can still be discovered. However, numbers are not displayed.

## Final Solution

The solution I came up with was to decompose a string into a LIST of ASCII numbers representing each character. This LIST can be recomposed back into the original string by coverting the ASCII value into it's specific character.

From a documentation point of view, the original credential string can be stored as a comment ajacent to the LIST value since comments are dropped during Python compilation.

## Sample Program

This is a simple email program. I works (I tested it) before changing the credentials
```
import smtplib
from email.message import EmailMessage

# Set Parameters

subject="Simple Email"
from_email="paul@somedomain.com"
to_email= "paul@somedomain.com"
email_port = "465"
email_pass="1234someFancyPassword!@*#"
email_login="username"
email_server="serveraddress"

# Create email body (simple; no body or attachments)

msg = EmailMessage()
msg["From"] = from_email
msg["Subject"] = subject
msg["To"] = to_email

# Send email

s = smtplib.SMTP_SSL(email_server,email_port)
s.login(email_login,email_pass)
s.send_message(msg)
s.quit()

exit()

```
This is the PY file. If we create a PYC file then use HEXDUMP here is what we see the credentials can easily be read and deciphered.

```
00000090  a1 02 01 00 65 0c a0 0e  65 0a a1 01 01 00 65 0c  |....e...e.....e.|
000000a0  a0 0f a1 00 01 00 65 10  83 00 01 00 64 01 53 00  |......e.....d.S.|
000000b0  29 0c e9 00 00 00 00 4e  29 01 da 0c 45 6d 61 69  |)......N)...Emai|
000000c0  6c 4d 65 73 73 61 67 65  7a 0c 53 69 6d 70 6c 65  |lMessagez.Simple|
000000d0  20 45 6d 61 69 6c 7a 13  70 61 75 6c 40 73 6f 6d  | Emailz.paul@som|
000000e0  65 64 6f 6d 61 69 6e 2e  63 6f 6d 5a 03 34 36 35  |edomain.comZ.465|
000000f0  7a 19 31 32 33 34 73 6f  6d 65 46 61 6e 63 79 50  |z.1234someFancyP|
00000100  61 73 73 77 6f 72 64 21  40 2a 23 5a 08 75 73 65  |assword!@*#Z.use|
00000110  72 6e 61 6d 65 5a 0d 73  65 72 76 65 72 61 64 64  |rnameZ.serveradd|
00000120  72 65 73 73 5a 04 46 72  6f 6d 5a 07 53 75 62 6a  |ressZ.FromZ.Subj|
00000130  65 63 74 5a 02 54 6f 29  11 5a 07 73 6d 74 70 6c  |ectZ.To).Z.smtpl|
00000140  69 62 5a 0d 65 6d 61 69  6c 2e 6d 65 73 73 61 67  |ibZ.email.messag|
00000150  65 72 02 00 00 00 5a 07  73 75 62 6a 65 63 74 5a  |er....Z.subjectZ|
00000160  0a 66 72 6f 6d 5f 65 6d  61 69 6c 5a 08 74 6f 5f  |.from_emailZ.to_|
00000170  65 6d 61 69 6c 5a 0a 65  6d 61 69 6c 5f 70 6f 72  |emailZ.email_por|
00000180  74 5a 0a 65 6d 61 69 6c  5f 70 61 73 73 5a 0b 65  |tZ.email_passZ.e|
00000190  6d 61 69 6c 5f 6c 6f 67  69 6e 5a 0c 65 6d 61 69  |mail_loginZ.emai|
000001a0  6c 5f 73 65 72 76 65 72  da 03 6d 73 67 5a 08 53  |l_server..msgZ.S|
000001b0  4d 54 50 5f 53 53 4c da  01 73 5a 05 6c 6f 67 69  |MTP_SSL..sZ.logi|
000001c0  6e 5a 0c 73 65 6e 64 5f  6d 65 73 73 61 67 65 da  |nZ.send_message.|
000001d0  04 71 75 69 74 da 04 65  78 69 74 a9 00 72 07 00  |.quit..exit..r..|
000001e0  00 00 72 07 00 00 00 fa  0f 2e 2f 73 65 6e 64 5f  |..r......./send_|
000001f0  65 6d 61 69 6c 2e 70 79  da 08 3c 6d 6f 64 75 6c  |email.py
```
