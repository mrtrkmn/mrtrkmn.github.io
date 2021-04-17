---
title: "fail2ban: block ssh bruteforce attacks "
date: 2021-02-24T12:00:03+00:00
# weight: 1
# aliases: ["/first"]
tags: ["fail2ban", "ssh", "attack","bruteforce"]
author: "mrturkmen"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
description: "fail2ban: ban failed attempts"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
cover:
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/mrtrkmn/mrtrkmn.github.io/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---


# fail2ban 

A while ago, I was checking servers' logs to see any suspicious activities going on from outside. I noticed that the servers both staging/testing and production servers are receiving a lot of brute force SSH attacks from variety of countries which are shown in table below.

--- 

## List of IP Addresses ( who are doing SSH Brute Forcing )

<div class="sticky-table__scroller result-table__table" style="overflow-x: scroll; overflow-y: scroll; border: 1px solid white; height: 600px; border-collapse: collapse;" >
    <table class="table table-bordered table-striped table-condensed table-small sticky-table geoip-demo__table" style="border: 1px solid white;">
      <thead>
         <tr style="border: 1px solid white;" >
           <th >IP Address</th>
           <th >Country Code</th>
           <th >Location</th>
           <th >Network</th>
           <th  class="not-country">Postal Code</th>
           <th  class="not-country">Approximate Coordinates*</th>
           <th  class="not-country">Accuracy Radius (km)</th>
           <th  class="not-country">ISP</th>
           <th  class="not-country">Organization</th>
           <th  class="not-country">Domain</th>
           <th  class="not-country">Metro Code</th>
         </tr>
      </thead>
      <tbody id="geoip-demo-results-tbody" data-use-downloadable-db="1"><tr style="border: 1px solid white;" style="border: 1px solid white;" class="geoip-results"><td>171.239.254.84</td><td>VN</td><td>Ho Chi Minh City,<br>Ho Chi Minh,<br>Vietnam,<br>Asia</td><td>171.239.254.0/23</td><td></td><td>10.8104,<br>106.6444</td><td>1</td><td>Viettel Group</td><td>Viettel Group</td><td>viettel.vn</td><td></td></tr><tr style="border: 1px solid white;" class="geoip-results"><td>184.102.70.222</td><td>US</td><td>Warsaw,<br>Indiana,<br>United States,<br>North America</td><td>184.102.70.0/24</td><td>46582</td><td>41.2817,<br>-85.8541</td><td>100</td><td>CenturyLink</td><td>CenturyLink</td><td>qwest.net</td><td>588</td></tr><tr style="border: 1px solid white;" class="geoip-results"><td>180.251.85.85</td><td>ID</td><td>Surabaya,<br>East Java,<br>Indonesia,<br>Asia</td><td>180.251.85.0/24</td><td></td><td>-7.2484,<br>112.7419</td><td>100</td><td>PT Telkom Indonesia</td><td>PT Telkom Indonesia</td><td></td><td></td></tr><tr style="border: 1px solid white;" class="geoip-results"><td>103.249.240.208</td><td>IN</td><td>Pune,<br>Maharashtra,<br>India,<br>Asia</td><td>103.249.240.0/24</td><td>411001</td><td>18.6161,<br>73.7286</td><td>10</td><td>Gazon Communications India Limited</td><td>Gazon Communications India Limited</td><td></td><td></td></tr><tr style="border: 1px solid white;" class="geoip-results"><td>159.65.194.150</td><td>NL</td><td>Amsterdam,<br>North Holland,<br>Netherlands,<br>Europe</td><td>159.65.192.0/20</td><td>1098</td><td>52.352,<br>4.9392</td><td>1000</td><td>Digital Ocean</td><td>Digital Ocean</td><td></td><td></td></tr><tr style="border: 1px solid white;" class="geoip-results"><td>117.217.35.114</td><td>IN</td><td>Bhopal,<br>Madhya Pradesh,<br>India,<br>Asia</td><td>117.217.35.0/24</td><td>462030</td><td>23.2487,<br>77.4066</td><td>50</td><td>BSNL</td><td>BSNL</td><td></td><td></td></tr><tr style="border: 1px solid white;" class="geoip-results"><td>113.164.79.129</td><td>VN</td><td>Hậu Giang,<br>Vietnam,<br>Asia</td><td>113.164.79.0/24</td><td></td><td>9.7774,<br>105.4592</td><td>50</td><td>VNPT</td><td>VNPT</td><td></td><td></td></tr><tr style="border: 1px solid white;" class="geoip-results"><td>61.14.228.170</td><td>IN</td><td>Madurai,<br>Tamil Nadu,<br>India,<br>Asia</td><td>61.14.228.168/29</td><td>625009</td><td>9.919,<br>78.1195</td><td>500</td><td>World Phone Internet Services Pvt Ltd</td><td>World Phone Internet Services Pvt Ltd</td><td></td><td></td></tr><tr style="border: 1px solid white;" class="geoip-results"><td>116.110.30.245</td><td>VN</td><td>Da Nang,<br>Da Nang,<br>Vietnam,<br>Asia</td><td>116.110.30.0/23</td><td></td><td>16.0685,<br>108.2215</td><td>1</td><td>Viettel Group</td><td>Viettel Group</td><td></td><td></td></tr><tr style="border: 1px solid white;" class="geoip-results"><td>43.239.80.181</td><td>IN</td><td>Kolkata,<br>West Bengal,<br>India,<br>Asia</td><td>43.239.80.0/24</td><td>700006</td><td>22.5602,<br>88.3698</td><td>10</td><td>Meghbela Broadband</td><td>Meghbela Broadband</td><td>PMPL-Broadband.net</td><td></td></tr><tr style="border: 1px solid white;" class="geoip-results"><td>77.222.130.223</td><td>UA</td><td>Kyiv,<br>Kyiv City,<br>Ukraine,<br>Europe</td><td>77.222.130.0/24</td><td>04128</td><td>50.4334,<br>30.5216</td><td>500</td><td>Private Joint Stock Company datagroup</td><td>Private Joint Stock Company datagroup</td><td></td><td></td></tr><tr style="border: 1px solid white;" class="geoip-results"><td>14.255.137.219</td><td>VN</td><td>Thai Binh,<br>Tinh Thai Binh,<br>Vietnam,<br>Asia</td><td>14.255.136.0/23</td><td></td><td>20.4487,<br>106.3343</td><td>100</td><td>VNPT</td><td>VNPT</td><td>vnpt.vn</td><td></td></tr><tr style="border: 1px solid white;" class="geoip-results"><td>184.22.195.230</td><td>TH</td><td>Bangkok,<br>Bangkok,<br>Thailand,<br>Asia</td><td>184.22.195.0/24</td><td>10310</td><td>13.7749,<br>100.5197</td><td>20</td><td>AIS Fibre</td><td>AIS Fibre</td><td>myaisfibre.com</td><td></td></tr><tr style="border: 1px solid white;" class="geoip-results"><td>125.25.82.12</td><td>TH</td><td>Ban Tai,<br>Surat Thani,<br>Thailand,<br>Asia</td><td>125.25.82.0/24</td><td>84280</td><td>9.5694,<br>99.9855</td><td>200</td><td>TOT</td><td>TOT</td><td>totinternet.net</td><td></td></tr><tr style="border: 1px solid white;" class="geoip-results"><td>116.110.109.90</td><td>VN</td><td>Da Nang,<br>Da Nang,<br>Vietnam,<br>Asia</td><td>116.110.109.0/24</td><td></td><td>16.0685,<br>108.2215</td><td>20</td><td>Viettel Group</td><td>Viettel Group</td><td></td><td></td></tr><tr style="border: 1px solid white;" class="geoip-results"><td>115.76.168.231</td><td>VN</td><td>Ho Chi Minh City,<br>Ho Chi Minh,<br>Vietnam,<br>Asia</td><td>115.76.168.0/23</td><td></td><td>10.8104,<br>106.6444</td><td>1</td><td>Viettel Group</td><td>Viettel Group</td><td>viettel.vn</td><td></td></tr></tbody>
    </table>
  </div>

** Information on the table gathered from: [ https://www.maxmind.com/en/geoip-demo ]

--- 

## Ban failed attempts

Although servers have no password login, they are kept brute forcing on SSH port. Well, fail2ban was one of obvious solution to block those IP addresses permanently or temporarily. I prefered to block them all permanently until manual unblocking has been done by me. 

The steps for installing fail2ban is pretty obvious, you are doing same things like,  `apt-get update && apt-get install fail2ban`. After installation completed, configuration is much more important. 

Following steps will guide you to block any ip address who are brute forcing on SSH. 

--- 


- **Copy template file** 
  
```bash 
   $ cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```


- **Set Ban time**

    It is possible to set ban time permanent or temporarily. I preffered to setup permanent, so for this reason I have changed `bantime = -1`. Save and exit from the file when you are done. 

```bash  
$ vim /etc/fail2ban/jail.conf

# Permanent ban 
bantime = -1 

``` 

- **Create custom rules for SSH**

```bash 

 $ vim /etc/fail2ban/jail.d/sshd.local

   [sshd]
   enabled  = true
   port     = ssh
   filter   = sshd
   logpath  = /var/log/auth.log  # place of ssh logs 
   maxretry = 4    # maximum number of attempts that user can do 
```
(*Maxretry value and log file can be changed according to your setup.)

-  **Make the rules persistent**

    In order to make the rules persistent which means, the blocked IPs will not be deleted after restart of fail2ban service or restart of server. It requires to have some tricks to be done inside iptables rules under fail2ban. Add following `cat` and  `echo` commands at the end of actionstart and actionban respectively . 

```bash 
$ vim /etc/fail2ban/action.d/iptables-multiport.conf 

                        .
                        .
                        .

actionstart = iptables -N fail2ban-<name>
              iptables -A fail2ban-<name> -j RETURN
              iptables -I <chain> -p <protocol> -m multiport --dports <port> -j fail2ban-<name>
          cat /etc/fail2ban/persistent.bans | awk '/^fail2ban-<name>/ {print $2}' \
          | while read IP; do iptables -I fail2ban-<name> 1 -s $IP -j <blocktype>; done

                       .
                       .
                       .

actionban = iptables -I fail2ban-<name> 1 -s <ip> -j <blocktype>
        echo "fail2ban-<name> <ip>" >> /etc/fail2ban/persistent.bans
```

- **Save and restart service** 

```bash 
$ systemctl restart fail2ban
```

These are most basic steps to block IP addresses who are actively brute forcing to servers. After some time, I am able to see them with following command :) 

```bash 

$ sudo fail2ban-client status sshd

Status for the jail: sshd
|- Filter
|  |- Currently failed:	12
|  |- Total failed:	107
|  `- File list:	/var/log/auth.log
`- Actions
   |- Currently banned:	16
   |- Total banned:	16
   `- Banned IP list:	171.239.254.84 184.102.70.222 180.251.85.85 103.249.240.208 159.65.194.150 117.217.35.114 113.164.79.129 61.14.228.170 116.110.30.245 43.239.80.181 77.222.130.223 14.255.137.219 184.22.195.230 125.25.82.12 116.110.109.90 115.76.168.231
```

It is growing in time however at least they are not able to brute force the server with same IP addresses. There are plenty of other ways to make SSH port much more secure and effective however I think having updated ssh daemon/client, passwordless login and fail2ban will be enough in most of the cases. Therefore, while I was doing this stuff, although there are plenty of guides over there, I wanted to note down how I did it to come back and check if something happens. 

Take care ! 