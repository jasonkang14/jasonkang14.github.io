---
title: How to Use Google SMTP
date: "2020-03-08T22:53:37.121Z"
template: "post"
draft: false
slug: "/server/how-to-use-google-smtp"
category: "SMTP"
tags:
  - "SMTP"

description: "How to use Google SMTP, which is FREE!!"
---

I had to create a feature where uses report certain things to me when they find a bug in my app. Instead of making a list of reports via Google Sheet, I thought it would be easier to make them email the information to me so that I can deal with it as soon as I receive the email.

Google lets you use their SMTP(Simple Mail Transfer Protocol) for free. If you are using Python, you can do it like this .

```python
import smtplib
import email.mime.text import MIMEText

bug     = "whatever the user has found"
content = f"I have found a bug == {bug}"

message           = MIMEText(content)
message['Subject] = Bug Report
message['From']   = user
message['To']     = service_provider

gmail_address  = 'your_gmail_account@gmail.com'
gmail_password = 'your_gmail_password'

smtp_server = smtplib.SMTP_SSL("smtp.google.com", 465)
smtp_server.login(gmail_address, gmail_password)
smtp_server.sendmail('user_email', ['service_provider_email'], message.as_string())
smtp_server.quit()
```

Make sure that the receipient email is set in the form of a python list. You can send emails to multiple users by adding items to the list.
