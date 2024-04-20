---
title: 家庭云(ipv4&&ipv6)
categories:
  - 网站运维
tags:
  - 1Panel
  - OpenResty
halo:
  site: https://mengplus.top
  name: 3bef86be-f2c4-4d4d-b97c-93433eebc63b
  publish: true
---
# 家庭云的公网访问
## 前言
笔者拥有一个家庭云(仅ipv6)和一个阿里云(ipv4&ipv6),想在家庭云部署主要业务，用阿里云弥补ipv4访问的缺失，在此处记录配置方式，为大家在相似环境需求下提供一个解决方案。
最终实现效果:
1. ipv4环境访问，通过ipv4设备请求到阿里云服务器,阿里云再反代通过ipv6访问到家里云；
2. ipv6环境访问,通过ipv6网络请求到家里云服务器，直接获得数据。
3. 像我这样的网络环境，由于家庭宽带免费，我就有了不限流量的上行30M的服务器，而ipv4请求的 通过阿里云，阿里云每月有免费的20G下行流量。
## 前置条件
1. **家庭云**：ipv6已经正常访问，可以使用ddns，在ipv6环境下已经可以正常使用，但是缺失ipv4访问
2. **阿里云**：采用99/年的ECS,支持ipv4固定公网，ipv6手动开通
3. 最起码你要有个域名比如`mengplus.top`，国内没备案的就无法采用阿里云服务器
4. ipv6测试工具：[IPV6版_在线ping_多地ping_多线路ping_持续ping_网络延迟测试_服务器延迟测试 (itdog.cn)](https://www.itdog.cn/ping_ipv6/)
5. [IP查询(ipw.cn) | IPv6测试 | IPv6在线Ping测试 | IPv6网站检测 | IPv6网站测速 | IPv6地址查询 | IP查询(ipw.cn)](https://ipw.cn/)
## 家庭云
### 获得IPv6方式
家庭宽带的ipv6已经实现了全覆盖，如果您wifi没有获取到ipv6请检查路由器配置，老旧路由器or未配置均有可能无ipv6。
如果您已经看到获取到了ipv6 可以使用ipv6测试工具看是否能访问到。
如果不能访问到，这里给出少量建议，更多需要您自行了解
1. 中国移动：家庭宽带改桥接后使用家庭路由器拨号(联系装机运维，如果他们不改，你就咸鱼搜搜吧)
2. 中国联通：进入后台，关闭一个防火墙就可以了
3. 中国电信没用过
4. 建议根据自己的情况上网搜寻方案（动手能力）
## 阿里云
1. 如果没有可以考虑在此购买[阿里云99元1年](https://www.aliyun.com/daily-act/ecs/activity_selection?userCode=9ikm4mm1) ECS服务器(**2**核(vCPU)**2** GiB)
2. 在控制台找到ECS实例，可以找到专有网络相关的配置，自己找到ipv6默认是没有开通的
3. 登录服务器可以使用 **curl 6.ipw.cn** 看是否地址确认开通情况
4. 由于固定带宽不够用，这里建议改为按量计费（每月免费20G国内 180G国外流量）
**注意**：必须是阿里云有ipv6网络，否则与家里云根本无法互通


##  笔者环境介绍
1. 家里云
	1. 华为rh1288服务器，24c32g, 2T 100W/h
	2. Ubuntu 22.04.4 LTS
	3. 1Panel运维面板
	4. Halo博客服务
	5. ipv6(移动，动态)
 2. 阿里云
	1. **2**核(vCPU)**2** GiB 40G
	2. Ubuntu 22.04.4 LTS
	3. 1Panel运维面板
	5. ipv4/ipv6
## 实操演示

## DNS域名服务商处配置
| 备注   | 类型    | 名称           | 内容                                     |
| ---- | ----- | ------------ | -------------------------------------- |
| 阿里云  | A     | aliyun       | 101.132.AAA.AAA                        |
| 家里云  | AAAA  | home         | 2409:8a44:130:60b8:BBBB:BBBB:BBBB:BBBB |
|      | AAAA  | mengplus.top | 101.132.AAA.AAA                        |
|      | A     | mengplus.top | 2409:8a44:130:60b8:BBBB:BBBB:BBBB:BBBB |
| 博客地址 | CNAME | blog         | mengplus.top                           |
### 家里云
1. 安装打开1Panel面板，应用商店安装Halo博客网站，可能您还需要ddns-go用于动态更新您的ipv6地址；
2.  在网站>证书处安装申请证书，或者你导入证书，网站需要https；
3. 在网站>网站处安装反代工具==OpenResty==,点击创建网站，一键部署，根据向导完成Halo博客网站（blog.mengplus.top）的反代配置，监听IPv6一定要选上，保存后，再次打开配置https；
4. 此时请测试你的家里云部署的halo博客网站能否通过手里流量正常访问到，wifi环境下可能直接走您的路由器访问了。
此时家里云就配置完毕，能够实现公网ipv6环境下对家里云的直接访问。
OpenResty将halo的访问端口都反代到了80/443端口
下面截图展示的是通过反向代理配置的方式，效果是一样的
![家里云blog配置](https://mengplus.top/upload/homeCloud_blog.png)
## 阿里云
1. 同上安装打开1Panel面板，但是不需要安装halo，只需要一个OpenResty
2. 证书同样也需要；
3. 在网站>网站>创建网站,选反向代理，配置您的博客域名还是blog.mengplus.top，监听ipv6可以不选上，代理地址输入https://home.mengplus.top(如果家里云没有添加证书就用http://home.mengplus.top)
见截图所示
![阿里云blog配置](https://mengplus.top/upload/aliyun_blog.png)
## 效果测试
通过命令行可以测试是否获得了双IP，截图只做示例，我这个是套了cdn
```cmd
C:\Users\cheng>nslookup blog.mengplus.top
服务器:  UnKnown
Address:  fe80::af4:58ff:fea2:e4ac

非权威应答:
名称:    blog.mengplus.top.w.kunlunpi.com
Addresses:  2409:8c44:0:ff03:3::3fd
          183.205.177.108
Aliases:  blog.mengplus.top
```
## 其它问题
1. 阿里云的20G免费流量获得？
	1. 搜索cdt去看进行相关配置升级下
2. 家里云通过云服务器转发会增加流量消耗么？
	1. 阿里云的计费规则来说，他们只统计下行流量不统计上行流量
3. 家里云套CDN是否可以避免云服务的代理？
	1. 可以的，而且CDN流量更加便宜0.2/GB ,但是我还部署了一个代码仓库tcp转发CDN做不了
	2. 单转IPV6需要提交工单申请（群友说的）
	3. cloudflare提供免费的CDN减速器，也能为你提供ipv4场景访问
4. 既然有云服务为什么不选择frp方案?
	1. frp适合无公网IP确实可以使用
	2. 我当前选择这个方案是为了部分流量走ipv6直连，缩减云服务器的使用成本，毕竟我是按量付费的
5. 说到按量付费，为什么选择这个方案?
	1. 阿里云国内机器带宽太贵，自带的3M带宽也就刷个博客玩玩，可是我还部署了网盘服务，git仓库，根本不够用
	2. 群友说可以使用香港机更加便宜
6. 为什么不选择香港机?
	1. 我主要是国内使用
	2. 阿里云的上行不计算费用而且带宽大
	3. 99一年比较便宜，另外我切换成按量计费又退30块钱，一年才70又没啥访问量先用着吧
	4. 不过确实备案稍微麻烦点，我就是为了备案才选择购买国内机器
7. 运维面板1Panel怎么获取
	1. 见官方文档与介绍[1Panel 文档](https://1panel.cn/docs/)
