---
title: Notes for Computer Networking 6e
tags:
  - Computer Networking
  - DNS
  - TCP
categories:
  - Notes
mathjax: true
sitemap: false
date: 2019-04-30 18:22:24
---

# Chapter 2 - Application Layer
## DNS
- Goal: To resolve a fully qualified domain name (FQDN) to an IP address. This process is called name resolution.
- Runs over **UDP** and uses **Port 53**  
- DNS other important services: Host Aliasing and Load Balance 
- A DNS __recursive query__ is between a DNS client and its local DNS server. When local DNS server can't resolve a new name from its own database, it would make an __iterative query__ to other DNS servers.
- DNS server 可以設定fowarder，等於是將別人的query丟給另一台DNS server。若Client搜尋時，一直有Non-authoritative answer，那可能就是local DNS server只是forward給其他DNS server

---
<!--more-->
### BIND (Berkeley Internet Name Domain)
互聯網上最常使用的DNS軟件，占所有DNS服務器的九成  

---


### [Zone file](https://en.wikipedia.org/wiki/Zone_file)
A zone file is a sequence of entries for resource records. Each line is a text description that defines a single resource record  
	`| name | ttl | record class | record type | record data |`  
	
- record class: namespace of the record information,  most commonly used namespace is Internet (__IN__)
- [record type](https://adon988.logdown.com/posts/7811375-dns-resource-record-rr)	
	- Type=[SOA](http://eservice.seed.net.tw/class/class45.html) (Start of Authority): 定義於Zone file的開頭，描述關於zone的基本資訊
	- Type=A: (hostname, IP)
	- Type=NS: (Domain Name, Authoritative DNS Server)
	- Type=CNAME: (Domain Name, Canonical hostname)  
	- Type=PTR: 反查IP的Domain Name
	- Type=AXFR: Zone Transfer
		- __[Zone Transfer](https://en.wikipedia.org/wiki/DNS_zone_transfer)__: Available for administrators to replicate DNS databases across a set of DNS servers for backup
		- [Security Concerns](https://devco.re/blog/2014/05/05/zone-transfer-CVE-1999-0532-an-old-dns-security-issue/) : 利用zone transfer去找所有可以攻擊的切入點 


### nslookup
- Get manual page: `man nslookup`   

```  
> server 8.8.8.8  (set default server)
Default server: 8.8.8.8
Address: 8.8.8.8#53
-------------------------
> set all  (show all settings)
> set debug  (顯示DNS Message)
-------------------------
> set type=any (將全部各類型的RR都顯示出來)
> google.com
Server:		8.8.8.8
Address:	8.8.8.8#53

Non-authoritative answer:    (存在於local DNS Server cache中)
Name:	google.com
Address: 172.217.27.142
-------------------------
> set norecurse (若local dns server沒有cache就回傳Nothing)
> www.hs.fi
Server:		8.8.8.8
Address:	8.8.8.8#53

** server can't find www.hs.fi: REFUSED
> set recurse
> www.hs.fi
Server:		8.8.8.8
Address:	8.8.8.8#53

Non-authoritative answer:
Name:	www.hs.fi
Address: 54.192.146.52
```


### host
`host [-al] FQDN`  

- -a : 等於nslookup `set debug`
- -l : 等於nslookup `set type=axfr`


### dig
```
> dig +trace google.com @168.95.192.1 (追蹤經過哪些節點，並利用168.95.192.1作為server)  
> dig -x 168.95.192.1  (等於nslookup type=PTR)
> dig -t [type] FQDN (指定RR type)
```

### whois
顯示該domain name的管理者資訊  
`> whois FQDN`



### [/etc/hosts file](https://debian-handbook.info/browse/zh-TW/stable/sect.hostname-name-service.html)
- 作用與DNS類似。例如瀏覽器在搜尋網址時會先找該檔案的設定，如果沒有才問DNS server。
- 格式: ` [IP]  [hostname]  [hostname別名]`  
e.g. `127.0.0.1  localhost `

### hostname
用於顯示或修改system的host name，儲存於`/etc/hostname` or `/proc/sys/kernel/hostname`

```
> hostname   (顯示host name)  
mac.local  
> hostname -i   (顯示hostname的IP)  
127.0.1.1  
> hostname [name]  (設置臨時hostname)  
```




### gethostbyname()

# Chapter 3 - Transport Layer
## UDP (User Datagram Protocol)
### Headers (8 bytes) 
1. Source Port
2. Destination Port
3. Length (header plus data)
4. Checksum
	- At the sender side performs the 1s complement of the sum of all the 16-bit words(2 bytes) in the segment, with any overflow encountered during the sum being wrapped around。也就是  
	 `checksum =  將segment裡全部的16-bit words總合後，再 1's complement  `
	- At the receiver, all four 16-bit words are added, including the checksum. 若總合後每一位不是都1，那就表示有錯誤發生。因為一個數字加上其1's complement的值，一定是都是111111...

> wrap around： 將overflow的值拿到最低位再相加
>1101  
>1100  
>\--------  
>11001  
>^ &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(overflow)  
>\--------  
>1101  
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1 &nbsp;(wrapped around)  
>\--------  
>1110   (answer)  


##Reliable Data Transfer  
### GBN (Go Back to N)
- 解決 Stop-and-wait  s

```
0        Base       Nextseqnum      Base+N   
|---------|-------------|---------------|--------------->  
    (1)         (2)            (3)             (4)  
    
          |-----------------------------|  
                  Window Size  N

(1)  Already sent and ACKed
(2)  Sent, not yet ACKed
(3)  Usable, not yet sent
(4)  Not usable (不在sliding window中，所以還不能傳送)
```
1. Receiver不需要buffer順序不對的packet
2. Cummulative Acknowledgment （不是base的packet都不予理會）
3. 使用一個Timer，該timer是綁定最早傳送但未ACKed的packet(Base Packet)。當Timeout Event發生時，需要重送全部已送過但未ACKed的封包 `(Base ~ Nextseqnum-1)`  


### Selective Repeat

- Sender
	- 允許ACKed但不是base的packet留在window裡
	- 每個packet有自己獨立的timer

- Receiver 
	- Out-of-order packets會被buffer
	- 當base packet到某一個packet都是in order且acked時，就會被delievered to upper layer  


- Window size must be less than or equal to half the size of the sequence number space


## TCP
- Maximum Segment Size (MSS)  



