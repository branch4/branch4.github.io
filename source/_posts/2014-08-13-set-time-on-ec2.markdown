---
layout: post
title: Set the Time of EC2 ~ How to Avoid the Initialization of the Time Setting ~
date: 2014-08-12 01:30:00 +0900
comments: true
published: true
author: risterlab
categories: 
 - AWS
 - EC2
---

Hi, I’m [risterlab](http://diary.risterlab.com), a web infrastructural engineer who love vegetables.   
  
![digital clock](http://blog.branch4.pw/images/2014/07/degital_clock.jpg)  
  
Today, I give some tips about setting the time on EC2 of AWS.  
  
If you set the time from UTC to your local time by following the instruction you usually get from googling,  
the time of EC2 might be reverted to UTC after the server reboots,   
not every time but sometimes...  

<!-- more --> 

### The way which you can get by googling  
----------
  
`cp /usr/share/zoneinfo/Asia/Tokyo /etc/localtime`  
  
You usually could find the instruction like above.  
The time setting will soon become your local time.    
It’s so easy! Done!  
  
But by this way, the time might get back to UTC.  
You need to be careful, because it would happen not every time you reboot the server, but sometimes.  
  
For me, it was after almost a year of service release that the time was reverted to UTC.   
That was so troublesome since cron and application see the time.  
    
### Why the time gets back to UTC  
----------
  
The thing is... when glibc package gets updated,  
/etc/localtime will be set to UTC by the script included in the glibc package.  
  
In the case of Amazon Linux, cloud_init exec the security updates as the server starts.  
The time setting will be initialized at that time.   

So, when you reboot the EC2 server after its glibc package gets updated,  
cloud_init update the package and /etc/localtime will be initialized.  
  
This is why the time gets back to UTC.  

### How to avoid the initialization of the time setting on EC2  
----------
  
There are two ways of dealing with the issue above.  
  
Edit not "/etc/localtime”, but “/etc/sysconfig/clock”  

```
ZONE="Asia/Tokyo"
UTC=False
```
to ZONE, you need to set your local identifier.  
It’s the case for Tokyo, Japan.  
  
Disable the security updates by editing cloud_init config “/etc/cloud/cloud.cfg"
  
It’s not recommended.  
Usually the security updates are necessary though it is troublesome if the time would get back sometimes after the reboots.   

### Summary
----------
  
When you google the way to set the time on EC2,  
you’ll usually get the information like below.  
  
`cp /usr/share/zoneinfo/Asia/Tokyo /etc/localtime`  
  
This is available when you want to set the time to your local time very soon.  
But you need to be careful that this is not that the time is fixed.  
To avoid the time setting gets initialized carelessly,  
Not “/etc/localtime”, but “/etc/sysconfig/clock” is the file you need to edit.  
  
```
ZONE="Asia/Tokyo"
UTC=False
```
  
日本人の方は[日本語の記事](http://blog.branch4.pw/blog/2014/07/12/fix-the-time-on-ec2/
)もあります。

<script type="text/javascript" language="javascript">
  num = Math.floor( Math.random() * 6 );
  document.write( aff[ num ]);
</script>
