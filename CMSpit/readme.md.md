# TryHackMe - CMSpit
https://tryhackme.com/room/cmspit
---

**Prerequisite**
- Basic knowledge of Bash or Shell Terminal (Linux knowledge)
- Basic knowledge of Burp
- Basic knowledge of Metasploit
- Basic knowledge of Privilege Escalation

> All exploit is run and tested on platform Kali Linux

---
## Reconnaissance
Run Nmap scan to find possible entries point
![[img/Nmap Scan.png]]
Found port `22` and `80`

Checking the website
![[img/Login Page.png]]
It is login page with `Cockpit` name on it. Since the title of the room is called `CMSpit` maybe this is a CMS (Content Management System). With quick google we can confirm yes this is a `Cockpit CMS Login Page`

Lets see what this page is about, use `View Page Source` on your browser
1. While searching for version number, I found these 3
	- `link href="/assets/app/css/style.css?ver=0.11.1`
	- `script src="/storage/tmp/7a812eebe1eda3162d79b4109b4787d4.js?ver=0.11.1`
	- `/storage/tmp/4cc5a0d2487ec7f4c75b0cc9115bf601.js?ver=0.11.1`
2. Found out how login system worked
	`form class="uk-form" method="post" action="/auth/check" onsubmit="{ submit }"`
	This part basically mean once we complete our login process (We type in username and password) the info will go to `/auth/check` to maybe for checking the *credential* of our login
3. Since we know we have `/auth` directory which sounds very important, lets search what we have for `/auth`
	- `class="uk-button uk-button-link uk-link-muted" href="/auth/forgotpassword"`
	- Another `App.request('/auth/check',`

Lets we make a summary for what we have found so far
- Version number, 90% is `0.11.1`
- There are 2 important info found, function for checking authentication from `/auth/check` and possibly change password function `/auth/forgotpassword`

With this information we have, lets google what we can use it for

---

## Exploit
With quick googling I found there are a vulnerabilities in Cockpit CMS version 0.11.1, ### **CVE-2020-35846** and **CVE-2020-35847** where you can enumerate username and reset thus found username's password. There are a lot of article on how to do the exploits, but I decided to use 2 source. You can use which one, either method will lead you somewhere. We will use **Burpsuite** and **Metasploit**

### Burpsuite
We will use this link[1] as our guidance. Quick read of the room, we know with this we can do
1. Username enumeration
2. Reset password

And now *lets do it*
1. Username enumeration - CVE-2020-35846
	In most cases, there is always a username of `admin` in anything. So lets find out if we have admin user.
	- Start up your Burpsuite
	- Activate proxy on your browser
	- Type in username:password. I did `admin:0`
	- Burp will catch the `post` request to `/auth/check`
	- Send the `post` request to `Repeater`
	Now we have multiple option we can use to try extract username. Play around and see if you can find which one working by yourself. What I use is number 3, using MongoLite variable
	![[img/Burp Result - MongoDB.png]]
	With this, Now we have 4 username. Lets use Admin and reset the password
2. Reset Password - CVE-2020-35847
	Reading through our source [1], the website need a reset password token in order to resetting password. Lets extract token and try to reset `admin` password
	- Go back to login page
	- Login with `admin:0`
	- Catch login `post` request with burp and send it to repeater
	- In line 1, change it to `POST /auth/newpassword HTTP/1.1`
	- On line 14 or 15 where there is our login information, replace all as shown below
		From
		```
		{
			"auth":{
				"user":"admin",
				"password":"0"
			},
			"csfr":"eyJ0eXAiOiJKV1QiLCJhbGciOi..................."
		}
		```
		To
		```
		{
			"token":
			{
				"$func":"var_dump"
			}
		}
		```

	What we did is we dump everything related to `token` belong to the admin. If succeed our extraction, you should have a text start with `string(48)`. Save it somewhere starting from `rp-fe` till the very end
		
	![[img/Burp Result - Extract Token Reset.png]]
		
	Right click on `Request` page and choose `Send to Repeater` and now we send our token to extract every info admin has
		![[img/Burp Result - Token Info Extract.png]]
		With this extraction, we have
		- Username
		- The name of the username
		- Email
		- Status
		- Group
		- Hashed password
		- Reset token
		
	Since we know what variable is called inside the system referring to password, we can now reset admin password
		- Right click on our current `Request` page and choose `Send to Repeater`
		- Change line 1 from `/auth/newpassword` to `/auth/resetpassword`
		- Add comma (,) to the very end of  `token` and type in password with the new password
	![[img/Burp Result - Change Password.png]]
	
	You should be able to login to the website with new password, but for the sake of learning lets see our 2nd method of this exploit.
	
	> Burpsuite version v2021.2.1
	
### Metasploit
Detail on how this exploit work can be found over on Packet Storm website [2]. The fundamental is basically, we exploit using vulnerabilities listed on **CVE-2020-35846** and **CVE-2020-35847**, but we used Metasploit instead Burp which is an easy mode. 

*Lets goo*
1. Start metasploit by typing `msfconsole`
2. Search Cockpit CMS entries `search Cockpit`
3. There is only 1, lets use it
4. We need 3 parameter
	- `RHOST` which is the target IP
	- `RPORT` default to 80, **DO NOT** change this
	- `TARGETURI` vulnerable point, we knew this is `/auth/check` from our research earlier
	- `LHOST` our machine IP, since the target run on TryHackMe VPN we need to set this IP to our VPN IP, you can find this using `ifconfig tun0`
	- After all set, run exploit `run`
	
	There are 2 stages; **1**. Using CVE-2020-35846 to find username **2**. CVE-2020-35847 found info needed to reset the password of thus username. Familiar? Because 2 of this method is using the same basic, just different technique.
	
	> The reason I put metasploit method in second is because it was too easy. No gain no fun, right?
	
	After our 1st stage complete, metasploit will tell us there are 4 username found. If you read from `show options` earlier there is options to set `USER`. We need this to run the 2nd stage. Go ahead set `USER`, I set it as admin
	
	We will find the same result like what we did with Burp when we extract info using admin's token. But this way we automatically reset and create new password. Go ahead login to the website using our new password.

## Reverse Shell
What is reverse shell? In short, we trick the web server to run certain command to give us its shell. Its not a complete shell like SSH, but its useful enough to get more info in our target machine. Go ahead explore what you like. Once you fill satisfied, follow instruction below

1. Click the Cockpit logo on top left corner
2. There are `Dashboard`, `Asstes`, `Finder`, `Settings`, and `Accounts`. We only interested in `Finder` but go ahead exploring this part too if you have not and want to
3. `Finder` part is where any content used in our Cockpit is listed. This is the best place for our reverse shell.
> This is also where `webflag.txt` is.
4. Create new file called `revshell.php` 
5. Go to https://www.revshells.com/
6. On `IP & Port` part, change the IP to your tun0 and port to `5555`
![[img/revshells IP and port setting.png]]
7. Run a new terminal using command on `Listener`
8. On tab called `Reverse`, scroll down until you found `PHP Pentest Monkey`
9. Copy everything using `copy` button to your new file `revshell.php`
10. Upload it to `Finder`
11.  Create new tab in your browser, and access `10.10.194.178/revshell.php` to activate revshell. Remember to change the IP above with your target IP, if done correctly you should have shell in your `Listener` terminal
![[img/revshell succed.png]]

## www-data
If you run command `whoami`, you notice we run this command as `www-data`. What is `www-data`? Quoting from Stackoverflow thread [3]
> `www-data` is the user that web servers on Ubuntu (Apache, nginx, for example) use by default for normal operation. The web server process can access any file that `www-data` can access. It has no other importance.

In short, `www-data` is the user in the web server which the one who will serving the client. It is important to only give `www-data` the most basic permissions for security reason. `www-data` is also can run database, which is an absolute since `www-data` need database to run function like login where it need compare username and password from user input with database to grant thus client and access to website.

We will use `www-data` permissions to:
1. Find another username that can use our target machine
2. Find out how we can access thus username though SSH to get full access

1. Find another username
	- Run `pwd` to check where we are. It will tell us we are in the `root directory` indicated by `/`
	- Run `ls` to see if we have `home` directory. This way we know what other user using this machine
	- With command above, we confirm there is `home` directory. Change our working directory to home `cd home`
	- Running `ls` we have info of username in this machine
	If you run command `ls` inside user directory, there is `user.txt` which we cant access it, since we are `www-data`. We are not allow to access this file. Hence we need to get access to our username
2. SSH using our target username
Lets see what we can use to find out our username password
	- Run command `systemctl`. Scroll through you will find entries about mongodb running, lets see what it has for us.
![[img/systemctl.png]]
	- Run command `mongo`, then `show databases`
	- 1 interesting database, `sudousersbak`. `.bak` file in linux usually used for backup. Maybe our user store the backup of sudo user list in the database with no password. Bad news, we will use this
	- Refer to this source if you want to explore more [4]
	- Run command `use sudousersbak` then `show collections`
	- There are 3 entries, `flag`, `system.indexes`,`user`. 
	- Run command `db.user.find()` to see whats inside, there are password for our username of target machine
	- Save it somewhere to run SSH as our username
> `flag` here is for db flag asked in the room. go ahead use `db.flag.find()`
	
SSH to our target machine using credential we found
![[img/SSH.png]]
Remember to change the IP to your own target machine IP

## Privilege Escalation
Since you are now an actual user in the machine, you can now print `user.txt`

Now we need privilege escalation to root. The final task we have is to get `root.txt`. For that we need to privilege escalation to root.

What is privilege escalation? In short, each user in Linux environment at least has certain permissions they can use. Remember our `www-data` cant access `user.txt` belong to our SSH user? Its because `www-data` is not listed in the permissions of who can access it. And `root.txt` which can be found in `/root/root.txt` is own by `root` which also only allow `root` to access this.

So we need to find a way to access this `root.txt` without even being `root`. There are a lot of way to do this, luckily this challenge has easy method.
- Run `sudo -l` to see what command we can run as `sudo`
![[img/sudo -l.png]]
- What the result meant is we can run `sudo exiftool` as `root` with no password
- Go to https://gtfobins.github.io/ and scroll down until you find an entries related to `exiftool`
- Click on it and you see down below there is entry of running `exitool` as sudo

![[img/Exiftool GTFO Bin.png]]

In here we have to declare 2 bash variable, `LFILE` and `INPUT`
As describe on the image:
- `LFILE=file_to_write` meant file we want to write into
- `INPUT=input_file` meant where the data we want from
Basically we want to copy anything that inside of `root.txt` into a new file, *remember* only `root` allowed to do any activity with `root.txt`. With this method we can copy anything in `root.txt` without even being root.

You may notice below I change directory into `/tmp`, the reason its because its easier and faster to type
```bash
LFILE=/tmp/copy_of_root.txt
INPUT=/root/root.txt

sudo exiftool -filename=$LFILE $INPUT
-> 1 image files updated
ls
-> copy_of_root.txt other_random_file other_random_file other_random_file
cat copy_of_root.txt
-> thm{https://www.youtube.com/watch?v=dQw4w9WgXcQ}
```
And you should have the final flag.

While doing research for the question for PoC and CVE vulnerability affecting the binary assigned to the system user, I found an article from Conviso's blog called "A case study on: CVE-2021-22204 – Exiftool RCE" [5]

In short, the author of the blog explaining the Prove of Concept (PoC) of CVE-2021-22204. With this method, we can also get the same result as above, which not covered in this walkthrough.

---

## Summary
CMSpit covered 2 or 3 CVE:
- CVE-2020-35846
- CVE-2020-35847
- CVE-2021-22204

For CVE-2020-35846 and CVE-2020-35847 we managed to do this with:
- Burpsuite by modifying the `request` method of HTTP/HTTPS where we injecting a mongodb query. This happen because `check` method from Auth controller which responsible for authenticating app user in Cockpit CMS version 0.11.1 does not check user parameter, which allowing attacker to embedding an object with arbitrary MongoDB operators in the query which allowing us to enumerate username. This vulnerabilities found on 14th October 2020 and fixes on 15th October 2020. 
	
	But then on 15th March 2021 by using NoSQL injection effecting on `/auth/newpassword` where it allowing force reset password through `request` method. This vulnerability make it possible for attacker to change password of a user
- Metasploit by using its exploit module. There are not much of an explanation since its the same exploit, just different tools/

As mention on privilege escalation part, CVE-2021-22204 will not be covered due to the length of this walkthrough. I may do it in the future with different test case or you can do it yourself! The blog post form Conviso [5] explained the exploit really well.



---
## Source and Reference
[1. Swarm Ptsecurity - Burpsuite Exploit](https://swarm.ptsecurity.com/rce-cockpit-cms/)
[2. Packet Storm - Metasploit Exploit](https://packetstormsecurity.com/files/162282/Cockpit-CMS-0.11.1-NoSQL-Injection-Remote-Command-Execution.html)
[3. Stackoverflow - What is user www-data](https://askubuntu.com/questions/873839/what-is-the-www-data-user)
[4. Mongodb Manual](https://docs.mongodb.com/manual/reference/mongo-shell/)
[5. Exiftool CVE by Conviso](https://www.google.com/search?client=firefox-b-d&q=exiftool+cve)