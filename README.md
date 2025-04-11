# Python-Hiding-eMail-Credentials
How to store, but hide, credentials used in sending emails from Python. Alternatives to encryption, decryption and obfuscation. 

## Problem
You are writing a Python script to send an email using the EMAIL and SMTPLIB. You need to provide email address, user name, password and server credentials but don't want others to be able to see them in the PY file. How to hide them?

## Solution Discussion
In the past, I written a 'C' program to create an encrypted value for credentials and a 2nd 'C' program to decrypt these. I only distributed the SO 'C' compiled file so it was quite secure. However, this was cumbersome also not something I could share without exposing my algorithm.

When you compile a Python script from PY to PYC all comments are removed but character strings can seen in, for example, in a HEXDUMP of the PYC. That means the character string credentials can still be discovered. However, numbers, in LISTs are not displayed.

## Final Solution

The solution I came up with was to decompose a string into a LIST of their ASCII numbers representing each character. This LIST can be recomposed back into the original string by coverting the ASCII value into it's specific character. This removes the character string of credentials from the PY/PYC files.

From a documentation point of view, the original credential string value can be stored as a comment ajacent to the LIST value since comments are dropped during Python compilation.

Note that this solution can be applied to MicroPython as well. See below.

## Sample Program With Character String Credentials

This is a simple email program. It works; I tested it before changing the credentials
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
The above is the PY file. If we compile a PYC file, then use HEXDUMP -C to view it, here is what we see. The credentials can easily be read and deciphered. Note the 'Simple Email' and '465' character string values. These will still be seen after we hide the credentials.


![Screenshot 2025-03-28 at 10 12 23 AM](https://github.com/user-attachments/assets/c77af225-6808-4dd3-a990-dfb9972e0bbc)

### Covert String to LIST then Back to String

The following PY script has a function to decompose strings to lists and one to recompose the list back to a character string.

The credentials which need to be hidden are pulled from the above sample program

```# DECOMPOSE_RECOMPOSE.PY - Hide Security Related Data in Scripts
# Done to hide sensitive strings that are still exposed in compiled Python scripts

#------------------------------------------------------------------------
# DECOMPOSE(STRING) - Create string to integer LIST in text form
# The ASCII value of the string is shifted plus 0,1 or 2 values based
# of the modulo(3) of the the position of the character in the string.
# This done since a HEXDUMP of a uPython compiled MPY file will show
# the character of the list value.

def decompose(mystr):
    de_list="["
    pos = 0
    for code in mystr.encode('ascii'):
        pos = pos=+1
        code = code + (pos % 3)				# Shift integer value of ASCII
        de_list = de_list+str(code)+","
        
    de_list=de_list[0:len(de_list)-1]+"]"
    
    return de_list # Return is a LIST type as a string

#------------------------------------------------------------------------
# RECOMPOSE(LIST) - Recreate the string decomposed from DECOMPOSE(STRING)
# Note since we did a positive shift in DECOMPOSE we need to do a negative
# shift of the modulo(3) of the position.	

def recompose(eval_list): # Input must be of type LIST 
    rtn = ""
    pos = 0
    for code in eval_list:
        pos=+1
        code = code - (pos % 3)
        rtn=rtn+chr(code)
    
    return rtn

```

We can test these routines with a simple program

```
# Test_decompose_recompose.py

from decompose_recompose import decompose, recompose

#------------------------------------------------------------------------
def test(str):
    print("Orig     ="+str)
    lt_str = decompose(str)
    print("Text List="+lt_str)
    lt_list = eval(lt_str) 	# Note use of EVAL() to convert from STRING to LIST
                            # In production, just the LIST would be stored
    print("Recomp   ="+recompose(lt_list)) 
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
Running this, and looking at the output, we see they work.

```
Orig     =paul@somedomain.com
Text List=[113,98,118,109,65,116,112,110,102,101,112,110,98,106,111,47,100,112,110]
Recomp   =paul@somedomain.com

Orig     =paul@somedomain.com
Text List=[113,98,118,109,65,116,112,110,102,101,112,110,98,106,111,47,100,112,110]
Recomp   =paul@somedomain.com

Orig     =1234someFancyPassword!@*#
Text List=[50,51,52,53,116,112,110,102,71,98,111,100,122,81,98,116,116,120,112,115,101,34,65,43,36]
Recomp   =1234someFancyPassword!@*#

Orig     =username
Text List=[118,116,102,115,111,98,110,102]
Recomp   =username

Orig     =serveraddress
Text List=[116,102,115,119,102,115,98,101,101,115,102,116,116]
Recomp   =serveraddress

```
## Update Sample Program with Character String Credentials Removed

We now need to update the Sample program with
* Add Import of RECOMPOSE function
* Replace character assignments with RECOMPOSE(LIST) for each credential. Note this needs to be the LIST value and not it's string version.
* I add a comment with the name contained in the list for reference

Here's the updated program

```import smtplib
from email.message import EmailMessage
from decompose_recompose import recompose

# Main Line

# Set Parameters

subject="Simple Email"
# paul@somedomain.com
from_email=recompose([113,98,118,109,65,116,112,110,102,101,112,110,98,106,111,47,100,112,110])
to_email= from_email
email_port = "465"
# 1234someFancyPassword!@*#
email_pass=recompose([50,51,52,53,116,112,110,102,71,98,111,100,122,81,98,116,116,120,112,115,101,34,65,43,36])
# username
email_login=recompose([118,116,102,115,111,98,110,102])
# serveraddress
email_server=recompose([116,102,115,119,102,115,98,101,101,115,102,116,116])

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
If we now compile the PY script, and look at the HEXDUMP -C of the PYC, we can see the "Simple Email" and "465" port value but none of the credentials that we saw in the first compile where character strings where used.

![pyc](https://github.com/user-attachments/assets/280fe26e-08db-4688-9e96-99e3efd448e4)

In the first version of this procedure someone pointed out that compiled version of MicroPython (MPY) stores lists in such a way that the credentials were displayed in a HEXDUMP -C. This led to add the MODULO(3) logic to scramble the ASCII values of the embedded LIST. Here's a HEXDUMP -C of the compiled MPY of the program above. There are ASCII test shown (in blue) but it doesn't suggest credentials.

![mpy](https://github.com/user-attachments/assets/5ea9c8e0-143f-4555-b95b-e24cd853ebcd)

## Conclusion

I hope you found this simple technique useful but I must also provide a warning.

There is a tool called [uncompyle6](https://pypi.org/project/uncompyle6/) that can decompile PYC up to Python 3.8 as of this writing. That means the values of the lists may be able to be exposed.

The above applications were run under Python 3.9 which is now a dated version. When I ran 'uncompyle6' on the last PYC it returned the error `Unsupported Python version, 3.9.0, for decompilation`

There is also a chance that performing a binary read of the bytecode and converting each byte to ASCII could expose the credential. See the [ discussion here 
](https://forums.raspberrypi.com/viewtopic.php?p=2309096#p2309090)

Keeping an eye out for decompile tools is suggested.
