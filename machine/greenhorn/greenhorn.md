Let's start with nmap:

```
└──╼ $nmap -A -sC -sV 10.10.11.25
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-06 21:11 CEST
Nmap scan report for 10.10.11.25
Host is up (0.047s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 57:d6:92:8a:72:44:84:17:29:eb:5c:c9:63:6a:fe:fd (ECDSA)
|_  256 40:ea:17:b1:b6:c5:3f:42:56:67:4a:3c:ee:75:23:2f (ED25519)
80/tcp   open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://greenhorn.htb/
3000/tcp open  ppp?
| fingerprint-strings: 
|   GenericLines, Help, RTSPRequest: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.0 200 OK
|     Cache-Control: max-age=0, private, must-revalidate, no-transform
|     Content-Type: text/html; charset=utf-8
|     Set-Cookie: i_like_gitea=9ea7a78e96364b81; Path=/; HttpOnly; SameSite=Lax
|     Set-Cookie: _csrf=0fFj1eiGL8TmG2GtuctdYM8ooOM6MTcyMjk3MTQ4OTc5ODIyNTkzNA; Path=/; Max-Age=86400; HttpOnly; SameSite=Lax
|     X-Frame-Options: SAMEORIGIN
|     Date: Tue, 06 Aug 2024 19:11:29 GMT
|     <!DOCTYPE html>
|     <html lang="en-US" class="theme-auto">
|     <head>
|     <meta name="viewport" content="width=device-width, initial-scale=1">
|     <title>GreenHorn</title>
|     <link rel="manifest" href="data:application/json;base64,eyJuYW1lIjoiR3JlZW5Ib3JuIiwic2hvcnRfbmFtZSI6IkdyZWVuSG9ybiIsInN0YXJ0X3VybCI6Imh0dHA6Ly9ncmVlbmhvcm4uaHRiOjMwMDAvIiwiaWNvbnMiOlt7InNyYyI6Imh0dHA6Ly9ncmVlbmhvcm4uaHRiOjMwMDAvYXNzZXRzL2ltZy9sb2dvLnBuZyIsInR5cGUiOiJpbWFnZS9wbmciLCJzaXplcyI6IjUxMng1MTIifSx7InNyYyI6Imh0dHA6Ly9ncmVlbmhvcm4uaHRiOjMwMDAvYX
|   HTTPOptions: 
|     HTTP/1.0 405 Method Not Allowed
|     Allow: HEAD
|     Allow: HEAD
|     Allow: GET
|     Cache-Control: max-age=0, private, must-revalidate, no-transform
|     Set-Cookie: i_like_gitea=447959683ed31f2a; Path=/; HttpOnly; SameSite=Lax
|     Set-Cookie: _csrf=3HzZaAV_8fQOuQi-54kHSxPRsYo6MTcyMjk3MTQ5NTA5NzE3MDM3NQ; Path=/; Max-Age=86400; HttpOnly; SameSite=Lax
|     X-Frame-Options: SAMEORIGIN
|     Date: Tue, 06 Aug 2024 19:11:35 GMT
|_    Content-Length: 0
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3000-TCP:V=7.94SVN%I=7%D=8/6%Time=66B27561%P=x86_64-pc-linux-gnu%r(
SF:GenericLines,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x2
SF:0text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad
SF:\x20Request")%r(GetRequest,2A60,"HTTP/1\.0\x20200\x20OK\r\nCache-Contro
SF:l:\x20max-age=0,\x20private,\x20must-revalidate,\x20no-transform\r\nCon
SF:tent-Type:\x20text/html;\x20charset=utf-8\r\nSet-Cookie:\x20i_like_gite
SF:a=9ea7a78e96364b81;\x20Path=/;\x20HttpOnly;\x20SameSite=Lax\r\nSet-Cook
SF:ie:\x20_csrf=0fFj1eiGL8TmG2GtuctdYM8ooOM6MTcyMjk3MTQ4OTc5ODIyNTkzNA;\x2
SF:0Path=/;\x20Max-Age=86400;\x20HttpOnly;\x20SameSite=Lax\r\nX-Frame-Opti
SF:ons:\x20SAMEORIGIN\r\nDate:\x20Tue,\x2006\x20Aug\x202024\x2019:11:29\x2
SF:0GMT\r\n\r\n<!DOCTYPE\x20html>\n<html\x20lang=\"en-US\"\x20class=\"them
SF:e-auto\">\n<head>\n\t<meta\x20name=\"viewport\"\x20content=\"width=devi
SF:ce-width,\x20initial-scale=1\">\n\t<title>GreenHorn</title>\n\t<link\x2
SF:0rel=\"manifest\"\x20href=\"data:application/json;base64,eyJuYW1lIjoiR3
SF:JlZW5Ib3JuIiwic2hvcnRfbmFtZSI6IkdyZWVuSG9ybiIsInN0YXJ0X3VybCI6Imh0dHA6L
SF:y9ncmVlbmhvcm4uaHRiOjMwMDAvIiwiaWNvbnMiOlt7InNyYyI6Imh0dHA6Ly9ncmVlbmhv
SF:cm4uaHRiOjMwMDAvYXNzZXRzL2ltZy9sb2dvLnBuZyIsInR5cGUiOiJpbWFnZS9wbmciLCJ
SF:zaXplcyI6IjUxMng1MTIifSx7InNyYyI6Imh0dHA6Ly9ncmVlbmhvcm4uaHRiOjMwMDAvYX
SF:")%r(Help,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20te
SF:xt/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x2
SF:0Request")%r(HTTPOptions,1A4,"HTTP/1\.0\x20405\x20Method\x20Not\x20Allo
SF:wed\r\nAllow:\x20HEAD\r\nAllow:\x20HEAD\r\nAllow:\x20GET\r\nCache-Contr
SF:ol:\x20max-age=0,\x20private,\x20must-revalidate,\x20no-transform\r\nSe
SF:t-Cookie:\x20i_like_gitea=447959683ed31f2a;\x20Path=/;\x20HttpOnly;\x20
SF:SameSite=Lax\r\nSet-Cookie:\x20_csrf=3HzZaAV_8fQOuQi-54kHSxPRsYo6MTcyMj
SF:k3MTQ5NTA5NzE3MDM3NQ;\x20Path=/;\x20Max-Age=86400;\x20HttpOnly;\x20Same
SF:Site=Lax\r\nX-Frame-Options:\x20SAMEORIGIN\r\nDate:\x20Tue,\x2006\x20Au
SF:g\x202024\x2019:11:35\x20GMT\r\nContent-Length:\x200\r\n\r\n")%r(RTSPRe
SF:quest,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20text/p
SF:lain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x20Req
SF:uest");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

We notice a web server nginx on port 80, so we check the site (after adding greenhorn.htb to hosts file).
The website is really simple without anything interesting except for a link to an admin page referred to a CMS: plunk version 4.7.18.
Searching online I found a CVE for file upload used for RCE: https://github.com/Rai2en/CVE-2023-50564_Pluck-v4.7.18_PoC?tab=readme-ov-file

Following the poc we managed to upload and use the rev shell and gaining a shell on the www-data user.

Inside var/www/html/pluck/setting we found reference to some token and password (written in files) used into login page. 
In general, in pluck docs it's specified that each setting is written to file and there is no db used.

I tried to use the token to use an existing session and automatically log in but it seems not working.
Talking about the password we found, instead, we notice is not in plaintext and is cyphered in some way.
Pasting it in cyberchef had no results, so I started to searching around on the machine, expecially on how password is set.

Searching online and on pluck's docs it turned out that in installation phase the password is set. In the pluck folder we find under /data/inc folder
the file changepass.php in which is defined the change passw function. In it we see that the passw is being encrypted in SHA512:

	//Include old password.
	require_once ('data/settings/pass.php');

	//SHA512-encrypt posted passwords.
	if (!empty($cont1))
		$cont1 = hash('sha512', $cont1);


Now we know that the password find before is a SHA512 hash we can use hashcat specifing 1700 (SHA512 mode)

```
hashcat -m 1700 hashtobecracked /home/francesco/Downloads/rockyou.txt
```

Aaaand, here we get our password! Let's log in the pluck admin page. Keeping in mind that password are often reused we try to log in ssh with
the user on the machine (junior), we can search for it in /etc/passwd.
We are not allowed to ssh to junior user so we try to change user from the rev shell created before and we use the password found and we are in.

In the junior's home directory we notice a pdf written from the admin, asking junior to keep in mind that he will be enabled to use a sw with an explicited passw.
The passw in the pdf is obfuscated so we need to recover it.

We use pdfimages to extract it to a ppm file. We now use the image extracted with a tool called Depix to recover the pixel effect:

```
python3 depix.py -p /home/francesco/Downloads/gg-000.ppm -s images/searchimages/debruinseq_notepad_Windows10_closeAndSpaced.png
```

Trying different searchimages, the previous one seams to have the best result. 
The output shows the recovered passw, we prompt it to root login and we are in!







