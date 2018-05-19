---
layout: post
title:  "Spamming using SSH Tunneling"
date:   2016-09-25 10:51:47 +0530
categories: Spamming
image: "/images/spamming.png"
categories: [one, two]
---


We have seen different methods of spamming like spamming using a compromised email account, spamming using a php script etc. All this type of spamming techniques leaves some logs or evidence in the server which can be used to track down the origin of the spam mails. Here we discuss about a different type of spamming that can be done using ssh tunneling or ssh port forwarding.

An attacker can use ssh tunneling to send out spam mails from your server. This type of attack can be difficult to detect as it leaves only little log evidences.

<span style="color: red"> This type of spamming method need access to a user account in the server.</span>
Suppose if a user uses a weak password and it gets compromised by the attacker.

Port forwarding via SSH (SSH tunneling) creates a secure connection between a local computer and a remote machine through which services can be relayed.

<span style="color: red">This type of attack even works if the user has /bin/false or /sbin/nologin shell</span>, provided a password has been assigned to this user. As ssh tunneling do not need a valid shell to work.


**Following is what happens at the end of attacker:**

{% highlight ruby %}
$ ssh -fN -L 2500:localhost:25 testuser@server.com
{% endhighlight %}


Here testuser is the compromised account in our server and server.com is the hostname of our server.

What happens here is that the above command forwards the local port 2500 of the attacker to the remote port 25 of the server.
Attacker can now connect to the smtp port of the server via his local port 2500.

{% highlight ruby %}
$ telnet localhost 2500
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
220 server.com ESMTP Postfix
{% endhighlight %}

Now the attacker can send thousands of emails using this method.
The emails appears to be coming from 127.0.0.1 (localhost) when you check the email headers. This makes it difficult to trace the spam back to the account from were it originated.

**Following is what happens in the server side:**

In most of the spam cases that involve explotied password, the attacker connects directly to the mail server. As a result, your mail logs will be filled with SMTP authentication attempts usually from many IP addresses. This makes it easy to identiy the compromised account.

With SSH tunnelling method, SMTP authentication is not required. Which results in little evidence regarding the attack in the logs. We could see a high volume of bounces or emails that are being sent via localhost.
Most of the servers trust email from localhost and the emails sent via localhost host connection to the SMTP server do not require an SMTP authentication. Without SMTP authentication there is no log evidence to identify the compromised account. We could only see lot of email coming from localhost.

There are two clue that we can find in the server:
- Email logs showing SMTP connections from localhost
- Netstat showing SSH connection to SMTP

Using the ssh tunnel attack the mail logs looks like this:
(/var/log/maillog)


	Nov 1 22:56:44 server postfix/smtpd[24316]: connect from localhost[::1]
	Nov 1 22:57:16 server postfix/smtpd[24316]: 967B0658CB: client=localhost[::1]
	Nov 1 22:57:41 server postfix/cleanup[24397]: 967B0658CB: message-id=<20151101225716.967B0658CB@server.serversolace.com>
	Nov 1 22:57:41 server postfix/qmgr[1434]: 967B0658CB: from=<test@abc.com>, size=391, nrcpt=1 (queue active)
	Nov 1 22:57:41 server postfix/smtp[24404]: connect to gmail-smtp-in.l.google.com[2607:f8b0:400d:c06::1a]:25: Network is unreachable
	Nov 1 22:57:41 server postfix/smtp[24404]: 967B0658CB: to=<mailtest@gmail.com>, relay=gmail-smtp-in.l.google.com[74.125.29.26]:25, delay=36, delays=36/0.01/0.1/0.15, dsn=2.0.0, status=sent (250 2.0.0 OK 1446418679 d7si15751454qhc.16 - gsmtp)
	Nov 1 22:57:41 server postfix/qmgr[1434]: 967B0658CB: removed
	Nov 1 22:57:46 server postfix/smtpd[24316]: disconnect from localhost[::1]


**Following is the output of netstat:**

{% highlight ruby %}
[root@server ~]# netstat -ntup | grep :22
tcp 0 48 172.31.55.215:22 118.102.223.130:41416 ESTABLISHED 1297/sshd
tcp 0 0 172.31.55.215:22 43.229.53.85:64020 ESTABLISHED 4343/sshd
tcp 0 0 172.31.55.215:22 118.102.223.130:50768 ESTABLISHED 4230/sshd

[root@server ~]# netstat -ntup | grep :25
tcp 0 0 ::1:25 ::1:45877 ESTABLISHED 4336/smtpd
tcp 0 0 ::1:45877 ::1:25 ESTABLISHED 4234/sshd
{% endhighlight %}

We could also use lsof command to check this:

{% highlight ruby %}
[root@server ~]# lsof -i :22
COMMAND PID USER FD TYPE DEVICE SIZE/OFF NODE NAME
sshd 1019 root 3u IPv4 8089 0t0 TCP *:ssh (LISTEN)
sshd 1019 root 4u IPv6 8091 0t0 TCP *:ssh (LISTEN)
sshd 1297 root 3u IPv4 9928264 0t0 TCP server.com:ssh->local.attacker.com:41416 (ESTABLISHED)
sshd 4230 root 3r IPv4 9956271 0t0 TCP server.com:ssh->local.attacker.com:50768 (ESTABLISHED)
sshd 4234 testuser 3u IPv4 9956271 0t0 TCP server.com:ssh->local.attacker.com:50768 (ESTABLISHED)
sshd 4349 root 3r IPv4 9957655 0t0 TCP server.com:ssh->43.229.53.85:25374 (ESTABLISHED)
sshd 4350 sshd 3u IPv4 9957655 0t0 TCP server.com:ssh->43.229.53.85:25374 (ESTABLISHED)

[root@server ~]# lsof -i :25
COMMAND PID USER FD TYPE DEVICE SIZE/OFF NODE NAME
master 1422 root 12u IPv4 9256 0t0 TCP *:smtp (LISTEN)
master 1422 root 13u IPv6 9258 0t0 TCP *:smtp (LISTEN)
sshd 4234 testuser 8u IPv6 9957374 0t0 TCP localhost:45877->localhost:smtp (ESTABLISHED)
smtpd 4336 postfix 6u IPv4 9256 0t0 TCP *:smtp (LISTEN)
smtpd 4336 postfix 7u IPv6 9258 0t0 TCP *:smtp (LISTEN)
smtpd 4336 postfix 12u IPv6 9957399 0t0 TCP localhost:smtp->localhost:45877 (ESTABLISHED)
{% endhighlight %}

In the above output you can see that there is an ssh connection to smtp

This type of attack can be tracked down only when the connections from the attacker is active. When spamming has occurred and the connections from the attacker is inactive it will be difficult to track it down.

"w" command in linux shows who is logged on and what they are doing. w command will not show this compromised user (testuser) even when the attacker is connected to the server using ssh tunnel.

"last" command also will not show any evidence of the testuser was logged into the server.
(last command show listing of last logged in users)

The only log that we can find is in the /var/log/secure file:


	Nov 1 23:24:17 server sshd[25622]: Accepted password for testuser from 122.x.x.11 port 53349 ssh2
	Nov 1 23:24:17 server sshd[25622]: pam_unix(sshd:session): session opened for user testuser by (uid=0)


**How can we prevent this type of attack:**

The simplest way to prevent this type of attack is by disabling TCP port forwarding in your sshd configuration file.
In /etc/ssh/sshd_config



	AllowAgentForwarding no
	AllowTcpForwarding no
	X11Forwarding no



This will not deny the actual ssh connection to be established; it will only deny the ability to exploit the forwarding.
So if we now try to do ssh tunneling we will get the following error:




	channel 2: open failed: administratively prohibited: open failed


Also make sure to use a strong and complex password for all the users.
