---
title: Appointment
date: 2025-04-26 00:00:00 +/-TTTT
categories:
  - Practice
  - Hackthebox - Linux
tags:
  - nmap
  - sqli
image: preview-image.png
media_subpath: /assets/img/Practice/Hackthebox - Linux/2025-05-31-appointment/
---

This box is from starting point tier 1 from HTB and will help us to practice performing an SQL injection against an SQL database enabled web application. SQL Injection is a common way of exploiting web pages that use SQL Statements to retrieve and store user input data.

**NOTE:** this writeup is not for solving task answers foe this machine but is written to simple explain SQL injection in context to the box.

## Enumeration <a href="#enumeration" id="enumeration"></a>

### Nmap <a href="#nmap" id="nmap"></a>

![](2025-05-31-appointment-2.png)

### HTTP - 80 <a href="#http---80" id="http---80"></a>

* From the nmap scan, we can see that port 80 is open and it is running Apache httpd server with version 2.4.38
* Apache httpd server is used for running web pages on either physical or virtual web servers.
* If we type the IP address of the device into the address bar of our browser, we can see a website with a login form

![](2025-05-31-appointment-3.png)

## Explotiation <a href="#explotiation" id="explotiation"></a>

* Since Gobuster did not find anything useful, we need to check for any default credentials or bypass the log-in page somehow. To check for default credentials, we could type the most common combinations in the username and password fields, such as:

```bash
admin:admin
guest:guest
user:user
root:root
administrator:password
```

* After attempting all of those combinations, we have still failed to log in. We could, hypothetically, use a tool to attempt brute-forcing the log-in page.
* However, that would take much time and might trigger a security measure.
* The next sensible tactic would be to test the log-in form for a possible **SQL Injection vulnerability**.
* Below is an example SQL query to verify username and password and log the person in
* `SELECT * FROM users WHERE username='$username' AND password='$password’`

![](2025-05-31-appointment-4.png)

* `$username` will be replaced by whatever we type in the username field of the form and similarly for `$password`
* In PHP, we use `#`to comment lines of code.
* With this much knowledge we can try to do SQL injection in the login form provided to us.

![](2025-05-31-appointment-5.png)

* In the username field, we can type `admin'#` .
* This line will go inside the code, and it will put the username as admin and because of `#` the rest of the line is commented, so the code only checks for the usernames with admin and ignores the password validation. So, we can put any value in password field and login.
* After filling the form and clicking on the login button, this is how the SQL query will look like

![](2025-05-31-appointment-6.png)

* After submitting the form, we get a page like:

![](2025-05-31-appointment-7.png)

* This shows that the login form on the website was vulnerable to SQL injection, and we have successfully exploited it.
