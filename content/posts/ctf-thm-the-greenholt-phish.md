---
title: "CTF writeup: The Greenholt Phish (TryHackMe)"
description: "Description here"
hidedescription: false
date: 2023-08-16T02:24:38+03:00
tags: ["ctf", "writeup"]
category: ["blog"]
author: "linutti"
draft: false
toc: true
disableanchors: false
disablebreadcrumbs: false
disablereadingtime: false
disablewordcount: false
editPost:
    disabled: false
---
Link to challenge: [The Greenholt Phish](https://tryhackme.com/room/phishingemails5fgjlzxc)

## The task

> A Sales Executive at Greenholt PLC received an email that he didn't expect to receive from a customer. He claims that the customer never uses generic greetings such as "Good day" and didn't expect any amount of money to be transferred to his account. The email also contains an attachment that he never requested. He forwarded the email to the SOC (Security Operations Center) department for further investigation. 
>
> Investigate the email sample to determine if it is legitimate. 

Let's launch the machine and dive right in!

## Task 1: What date was the email received? (answer format: M/DD/YY)



## Q2: Who is the email from?



## Q3: What is his email address?



## Q4: What email address will receive a reply to this email? 



## Q5: What is the Originating IP?

*"192.\*\*\*.\*\*.\*\*\*"*

## Q6: Who is the owner of the Originating IP? (Do not include the "." in your answer.)

To get the answer I did a lookup on the IP address. A big hint can be seen on the same line as the answer to the previous question.

The answer is the name of the company, not the URL. You naughty naughty trying to throw me off with hint in the question saying not to include the "."!

![naughty naughty](/pics/you-naughty-naughty)

## Q7: What is the SPF record for the Return-Path domain?



## Q8: What is the DMARC record for the Return-Path domain?



## Q9: What is the name of the attachment?



## Q10: What is the SHA256 hash of the file attachment?



## Q11: What is the attachments file size? (Don't forget to add "KB" to your answer, NUM KB)



## Q12: What is the actual file extension of the attachment?


