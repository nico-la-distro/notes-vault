## Sujets couverts :

- [Injection](https://owasp.org/www-project-top-ten/OWASP_Top_Ten_2017/Top_10-2017_A1-Injection)
- [Broken Authentication](https://owasp.org/www-project-top-ten/OWASP_Top_Ten_2017/Top_10-2017_A2-Broken_Authentication)
- [Sensitive Data Exposure](https://owasp.org/www-project-top-ten/OWASP_Top_Ten_2017/Top_10-2017_A3-Sensitive_Data_Exposure)
- [Broken Access Control](https://owasp.org/www-project-top-ten/OWASP_Top_Ten_2017/Top_10-2017_A5-Broken_Access_Control)
- [XSS](https://owasp.org/www-project-top-ten/OWASP_Top_Ten_2017/Top_10-2017_A7-Cross-Site_Scripting_\(XSS\))

---

## Let's go on an adventure!


Before we get into the actual hacking part, it's good to have a look around. In Burp, set the Intercept mode to off and then browse around the site. This allows Burp to log different requests from the server that may be helpful later. 

This is called **walking through** the application, which is also a form of **reconnaissance**!

### Questions
#### Question #1: What's the Administrator's email address?

![[juiceshop_t2q2.png]]

there is a review on the first juice. we can see that's the admin, so we got his email

**Answer** : admin@juice-sh.op

#### Question #2: What parameter is used for searching?

![[juiceshop_t2q1.png]]

when we search for a keyword, we can see /#/search? in the uri, with the parameter q

**Answer** : q

#### Question #3: What show does Jim reference in his review?

![[juiceshop_t2q3.png]]

Jim left a review mentioning a 'replicator'. When we search on Google, we can see that it's a reference to Star Trek

---

## Inject the juice

![](https://assets.tryhackme.com/additional/imgur/uwXqDdH.png)  

This task will be focusing on injection vulnerabilities. Injection vulnerabilities are quite dangerous to a company as they can potentially cause downtime and/or loss of data. Identifying injection points within a web application is usually quite simple, as most of them will return an error. There are many types of injection attacks, some of them are:

| SQL Injection     | SQL Injection is when an attacker enters a malicious or malformed query to either retrieve or tamper data from a database. And in some cases, log into accounts.                                                                                                   |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Command Injection | Command Injection is when web applications take input or user-controlled data and run them as system commands. An attacker may tamper with this data to execute their own system commands. This can be seen in applications that perform misconfigured ping tests. |
| Email Injection   | Email injection is a security vulnerability that allows malicious users to send email messages without prior authorization by the email server. These occur when the attacker adds extra data to fields, which are not interpreted by the server correctly.        |

But in our case, we will be using **SQL Injection**.

For more information: [Injection](https://owasp.org/www-project-top-ten/OWASP_Top_Ten_2017/Top_10-2017_A1-Injection)

### Questions
#### Question #1: Log into the administrator account!

First go on the login page

![[juiceshop_t3q1.png]]

Then type email = test & password = test. Make sure your burp proxy interceptor is on. After that you, can click 'submit'.

![[juiceshop_t3q1_2.png]]


Go to the right request. Change the email form with the SQLi -> ' or 1=1-- 
You are admin !

![[juiceshop_t3q1_3.png]]

**Answer** : 690fa3247a99d651e0b26f947baf0b79b4f404a9

#### Question #2: Log into the Bender account!

First let's check for the email

![[juiceshop_t3q2.png]]

Then return to login page. Put interceptor on. Go to the right request and change de email form with -> bender@juice-sh.op'-- 
Forward the request and your loged as bender !

![[juiceshop_t3q2_2.png]]

**Answer** : 5ff5052e879e6fef64124e64c82c84ebc809c6c4

---

## Who broke my lock?!

In this task, we will look at exploiting authentication through different flaws. When talking about flaws within authentication, we include mechanisms that are vulnerable to manipulation. These mechanisms, listed below, are what we will be exploiting. 

Weak passwords in high privileged accounts

Forgotten password pages

More information: [Broken Authentication](https://owasp.org/www-project-top-ten/OWASP_Top_Ten_2017/Top_10-2017_A2-Broken_Authentication)

### Questions
#### Question #1: Bruteforce the Administrator account's password!

We previously find the admin email : admin@juice-sh.op

![[juiceshop_t4q1.png]]

Set the email of the admin and a random password like 'test'. 
Put interceptor on and go to the right request. 
Then ctrl+I to send the request to intruder tab in burp.
Set the attack on sniper.
We can now put §§ between the password quotes (with add§), so sniper know what field must be bruteforce.
Press the load button and add the _/usr/share/wordlists/SecLists/Passwords/Common-Credentials/best1050.txt_ wordlist

![[juiceshop_t4q1_2.png]]

Now launch the attack and wait for a 200 http code.

![[juiceshop_t4q1_3.png]]

![[juiceshop_t4q1_4.png]]


**Answer** : ff4aebffe31b0ffdea9bdd0207a16a3c01ac6c56

#### Question #2: Reset Jim's password!

Let's OSINT Jim with google.
Searching for Jim Star Trek, we can see his family members. Maybe we can use it to reset the password.

![[juiceshop_t4q2_2.png]]

Go to the login page, click on "Forgot your password ?" and enter his email that we descovered previously.

![[juiceshop_t4q2.png]]

With the informations we gathered, we know that his brother middle name is Samuel.
So we can change the password.

![[juiceshop_t4q2_3.png]]

**Answer** : 3c3e2d6ef99b733b947e92f8e2a9ed08bf57ea63

---

## AH! Don't look!

A web application should store and transmit sensitive data safely and securely. But in some cases, the developer may not correctly protect their sensitive data, making it vulnerable.

Most of the time, data protection is not applied consistently across the web application making certain pages accessible to the public. Other times information is leaked to the public without the knowledge of the developer, making the web application vulnerable to an attack. 

More information: [Sensitive Data Exposure](https://owasp.org/www-project-top-ten/OWASP_Top_Ten_2017/Top_10-2017_A3-Sensitive_Data_Exposure)

### Questions
#### Question #1: Access the Confidential Document!

Go to the About Us page and click the link "Check out our boring terms of use if you are interested in such lame stuff".

We are send to http://10.130.149.134/ftp/legal.md. We can see we are in a file store in ftp directory. so let's check this by going to http://10.130.149.134/ftp.

![[juiceshop_t5q1.png]]

We can see sensitive data, so let's download this and get the flag.

![[juiceshop_t5q1_2.png]]

**Answer** : 8d2072c6b0a455608ca1a293dc0c9579883fc6a5

#### Question #2: Log into MC SafeSearch's account!

![[juiceshop_t5q2.png]]

According to this video https://youtu.be/v59CX2DiX0Y?si=reywGNnT_mV1B4Q7
We have the password of MC SafeSearch that is **Mr. N00dles**

**Answer** : bb105418e73708ceccf1a7b2491f434b8f5230e4

---

## Who's flying this thing?

![](https://assets.tryhackme.com/additional/imgur/r2qq6de.png)

Modern-day systems will allow for multiple users to have access to different pages. Administrators most commonly use an administration page to edit, add and remove different elements of a website. You might use these when you are building a website with programs such as Weebly or Wix.  

When Broken Access Control exploits or bugs are found, it will be categorised into one of **two types**:

| **Horizontal** Privilege Escalation | Occurs when a user can perform an action or access data of another user with the **same** level of permissions. |
| ----------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| **Vertical** Privilege Escalation   | Occurs when a user can perform an action or access data of another user with a higher level of permissions.     |

![](https://assets.tryhackme.com/additional/imgur/bJ9WKY4.png)

_Credits: Packetlabs.net_

More information: [Broken Access Control](https://owasp.org/www-project-top-ten/OWASP_Top_Ten_2017/Top_10-2017_A5-Broken_Access_Control)

### Questions
#### Question #1: Access the administration page!

Open inspecter, go to sources, then main.js and ctrl + f to search admin things

![[juiceshop_t6q1_2.png]]

We can see there is an administration page. we can now login with admin credentials (bruteforce previously) and visit the page.


![[juiceshop_t6q1.png]]

**Answer** : 71aeb3b0bf01cc6e488f0207bb62f79b41454a87

#### Question #2: View another user's shopping basket!

Go to Your Basket with admin account and intercept this request.
Then change the value /rest/basket/1 to /rest/basket/2, this change display the basket of an other user. So it's the basket of the UserID 2.


![[juiceshop_t6q2.png]]

**Answer** : e6982b34b6734ceadd28e5019b251f929a80b815

---

## Where did that come from?

XSS or Cross-site scripting is a vulnerability that allows attackers to run javascript in web applications. These are one of the most found bugs in web applications. Their complexity ranges from easy to extremely hard, as each web application parses the queries in a different way. 

**There are three major types of XSS attacks:**

| DOM (Special)            | DOM XSS _(Document Object Model-based Cross-site Scripting)_ uses the HTML environment to execute malicious javascript. This type of attack commonly uses the _<script></script>_ HTML tag.                                       |
| ------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Persistent (Server-side) | Persistent XSS is javascript that is run when the server loads the page containing it. These can occur when the server does not sanitise the user data when it is **uploaded** to a page. These are commonly found on blog posts. |
| Reflected (Client-side)  | Reflected XSS is javascript that is run on the client-side end of the web application. These are most commonly found when the server doesn't sanitise **search** data.                                                            |

More information: [Cross-Site Scripting XSS](https://owasp.org/www-project-top-ten/OWASP_Top_Ten_2017/Top_10-2017_A7-Cross-Site_Scripting_\(XSS\))

### Questions
#### Question #1: Perform a DOM XSS!

We will be using the iframe element with a javascript alert tag: 

`<iframe src="javascript:alert(`xss`)">`

Inputting this into the **search bar** will trigger the alert.

![[juiceshop_t7q1.png]]

**Answer** : 4a31a4fe0954199566e360a873802bf64d0d0a84

Note that we are using **iframe** which is a common HTML element found in many web applications, there are others which also produce the same result. 

This type of XSS is also called XFS (Cross-Frame Scripting), is one of the most common forms of detecting XSS within web applications.

Websites that allow the user to modify the iframe or other DOM elements will most likely be vulnerable to XSS.   

**Why does this work?**

It is common practice that the search bar will send a request to the server in which it will then send back the related information, but this is where the flaw lies. Without correct input sanitation, we are able to perform an XSS attack against the search bar.

#### Question #2: Perform a persistent XSS!

First, login to the **admin** account.

We are going to navigate to the "**Last Login IP**" page for this attack.

![[juiceshop_t7q2.png]]

It should say the last IP Address is 0.0.0.0 or 10.x.x.x 
As it logs the 'last' login IP we will now logout so that it logs the 'new' IP.
Make sure that Burp **intercept is on**, so it will catch the logout request.
We will then head over to the Headers tab where we will add a new header:

![[juiceshop_t7q2_3.png]]

Then forward the request to the server!  
When **signing back into the admin account** and navigating to the Last Login IP page again, we will see the XSS alert!

![[juiceshop_t7q2_2.png]]

**Answer** : c37da14686b69a220fd9febd09bb9593e7d0539f

**Why do we have to send this Header?**

The _True-Client-IP_  header is similar to the _X-Forwarded-For_ header, both tell the server or proxy what the IP of the client is. Due to there being no sanitation in the header we are able to perform an XSS attack.

#### Question #3: Perform a reflected XSS!

First, we are going to need to be on the right page to perform the reflected XSS!
**Login** into the **admin account** and navigate to the 'Order History' page.

From there you will see a "Truck" icon, clicking on that will bring you to the track result page. You will also see that there is an id paired with the order. 
`http://10.130.149.134/#/track-result?id=5267-d456ef9dd363ea5f`

We will use the iframe XSS, `<iframe src="javascript:alert(`xss`)">`, in the place of the _5267-d456ef9dd363ea5f_
After submitting the URL, refresh the page and you will then get an alert saying XSS!

![[juiceshop_t7q3.png]]

**Answer** : 305021787d3e9cd9cebc057a021c2504550bb3b6

**Why does this work?**

The server will have a lookup table or database (depending on the type of server) for each tracking ID. As the 'id' parameter is not sanitised before it is sent to the server, we are able to perform an XSS attack.

---

## Exploration!

![](https://assets.tryhackme.com/additional/imgur/DGSYlWp.png)  

If you wish to tackle some of the **harder** challenges that were not covered within this room, check out the **/#/score-board/** section on Juice-shop. Here you can see your completed tasks as well as other tasks in varying difficulty.

#### Access the /#/score-board/ page

**Answer** : 2614339936e8282e2f820f023d4d998a1f95e02a