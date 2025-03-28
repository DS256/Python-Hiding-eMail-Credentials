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
### Covert String to LIST then Back to String

The following script has a function to decompose strings to lists and one to recompose the list back to a character string.

The credentials which need to be hidden are pulled from the above sample program

```
# DECOMPOSE_RECOMPOSE.PY - Hide Security Related Data in Scripts
# Done to hide sensitive strings that are still exposed in compiled Python scripts

#------------------------------------------------------------------------
# DECOMPOSE(STRING) - Create string to integer LIST in text form

def decompose(mystr):
    de_list="["

    for code in mystr.encode('ascii'):
        de_list = de_list+str(code)+","
        
    de_list=de_list[0:len(de_list)-1]+"]"
    
    return de_list # Return is a LIST type as a string

#------------------------------------------------------------------------
# RECOMPOSE(LIST) - Recreate the string decomposed from DECOMPOSE(STRING)

def recompose(eval_list): # Input must be of type LIST 
    rtn=""
    for c in eval_list:
        rtn=rtn+chr(c)
    
    return rtn

#------------------------------------------------------------------------
def test(str):
    print("Orig     ="+str)
    lt = decompose(str)
    print("Text List="+lt)
    print("Recomp   ="+recompose(eval(lt))) # Note use of EVAL() to convert from STRING to LIST
    print()
    
    return

#------------------------------------------------------------------------
# Mainline

from_email="paul@somedomain.com"
test(from_email)
to_email= "paul@somedomain.com"
test(to_email)
email_pass="1234someFancyPassword!@*#"
test(email_pass)
email_login="username"
test(email_login)
email_server="serveraddress"
test(email_server)
    

```

Running the program produces the needed list and confirmation that the list can be recomposed back to the original string

```
Orig     =paul@somedomain.com
Text List=[112,97,117,108,64,115,111,109,101,100,111,109,97,105,110,46,99,111,109]
Recomp   =paul@somedomain.com

Orig     =paul@somedomain.com
Text List=[112,97,117,108,64,115,111,109,101,100,111,109,97,105,110,46,99,111,109]
Recomp   =paul@somedomain.com

Orig     =1234someFancyPassword!@*#
Text List=[49,50,51,52,115,111,109,101,70,97,110,99,121,80,97,115,115,119,111,114,100,33,64,42,35]
Recomp   =1234someFancyPassword!@*#

Orig     =username
Text List=[117,115,101,114,110,97,109,101]
Recomp   =username

Orig     =serveraddress
Text List=[115,101,114,118,101,114,97,100,100,114,101,115,115]
Recomp   =serveraddress

```
## Update Sample Program 

We now need to update the Sample program with
* Add RECOMPOSE() Function
* Replace character assignments with RECOMPOSE(LIST) for each credential. Note this needs to be the LIST value and not it's string version.
* I add a comment with the name contained in the list for reference

```
import smtplib
from email.message import EmailMessage

#------------------------------------------------------------------------
# RECOMPOSE(LIST) - Recreate the string decomposed from DECOMPOSE(STRING)

def recompose(eval_list): # Input must be of type LIST 
    rtn=""
    for c in eval_list:
        rtn=rtn+chr(c)
    
    return rtn

#------------------------------------------------------------------------
# Main Line

# Set Parameters

subject="Simple Email"
# paul@somedomain.com
from_email=recompose([112,97,117,108,64,115,111,109,101,100,111,109,97,105,110,46,99,111,109])
to_email= from_email
email_port = "465"
# 1234someFancyPassword!@*#
email_pass=recompose([49,50,51,52,115,111,109,101,70,97,110,99,121,80,97,115,115,119,111,114,100,33,64,42,35])
# username
email_login=recompose([117,115,101,114,110,97,109,101])
# serveraddress
email_server=recompose([115,101,114,118,101,114,97,100,100,114,101,115,115])

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
If we now compile and look at the HEXDUMP of the PYC we can see the "465" port value but none of the credentials that we saw in the first compile where character strings where used.

```
00000140  5f 6c 69 73 74 5a 03 72  74 6e da 01 63 a9 00 72  |_listZ.rtn..c..r|
00000150  06 00 00 00 fa 0f 2e 2f  73 65 6e 64 5f 65 6d 61  |......./send_ema|
00000160  69 6c 2e 70 79 da 09 72  65 63 6f 6d 70 6f 73 65  |il.py..recompose|
00000170  07 00 00 00 73 08 00 00  00 00 01 04 01 08 01 0e  |....s...........|
00000180  02 72 08 00 00 00 7a 0c  53 69 6d 70 6c 65 20 45  |.r....z.Simple E|
00000190  6d 61 69 6c 29 13 e9 70  00 00 00 e9 61 00 00 00  |mail)..p....a...|
000001a0  e9 75 00 00 00 e9 6c 00  00 00 e9 40 00 00 00 e9  |.u....l....@....|
000001b0  73 00 00 00 e9 6f 00 00  00 e9 6d 00 00 00 e9 65  |s....o....m....e|
000001c0  00 00 00 e9 64 00 00 00  72 0f 00 00 00 72 10 00  |....d...r....r..|
000001d0  00 00 72 0a 00 00 00 e9  69 00 00 00 e9 6e 00 00  |..r.....i....n..|
000001e0  00 e9 2e 00 00 00 e9 63  00 00 00 72 0f 00 00 00  |.......c...r....|
000001f0  72 10 00 00 00 5a 03 34  36 35 29 19 e9 31 00 00  |r....Z.465)..1..|
00000200  00 e9 32 00 00 00 e9 33  00 00 00 e9 34 00 00 00  |..2....3....4...|
00000210  72 0e 00 00 00 72 0f 00  00 00 72 10 00 00 00 72  |r....r....r....r|
00000220  11 00 00 00 e9 46 00 00  00 72 0a 00 00 00 72 14  |.....F...r....r.|
00000230  00 00 00 72 16 00 00 00  e9 79 00 00 00 e9 50 00  |...r.....y....P.|
00000240  00 00 72 0a 00 00 00 72  0e 00 00 00 72 0e 00 00  |..r....r....r...|
00000250  00 e9 77 00 00 00 72 0f  00 00 00 e9 72 00 00 00  |..w...r.....r...|
00000260  72 12 00 00 00 e9 21 00  00 00 72 0d 00 00 00 e9  |r.....!...r.....|
00000270  2a 00 00 00 e9 23 00 00  00 29 08 72 0b 00 00 00  |*....#...).r....|
00000280  72 0e 00 00 00 72 11 00  00 00 72 1f 00 00 00 72  |r....r....r....r|
00000290  14 00 00 00 72 0a 00 00  00 72 10 00 00 00 72 11  |....r....r....r.|
000002a0  00 00 00 29 0d 72 0e 00  00 00 72 11 00 00 00 72  |...).r....r....r|
```

## Conclusion

I hope you found this simple technique useful but I must also provide a warning.

There is a tool called [uncompyle6](https://pypi.org/project/uncompyle6/) that can decompile PYC up to Python 3.8 as of this writing. That means the values of the lists may be able to be exposed.

The above applications were run under Python 3.9 which is now a dated version. When I ran 'uncompyle6' on the last PYC it returned the error `Unsupported Python version, 3.9.0, for decompilation`

Keeping an eye out for decompile tools is suggested.
