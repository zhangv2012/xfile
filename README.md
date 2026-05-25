
		⣀⣀⣶⣀⣀⣀⣀⣀⣶⣀⣀⠄⠄⠤⣤⠤⠤⠤⠶⠤⠤⠤⣤⠤⠄⠄⣶⠒⠒⠒⣒⠒⠒⠒⣒⠒⣶⠄⠄⠄⠄⣤⠤⠤⣶⠤⠤⠤⣤⠄⠄⠄
		⣀⣀⣿⣀⣀⣀⣀⣀⣿⣀⣀⠄⠄⠄⠭⠭⠭⠭⠭⠭⣭⣭⠭⠄⠄⠄⣿⠄⠉⣶⠉⠄⠉⣶⠉⠄⣿⠄⠄⠒⣒⠿⠒⠒⠶⣛⠒⣒⣿⠒⠄⠄
		⣀⣀⠤⠒⠄⠄⠄⠒⠤⣀⣀⠄⠄⠒⠒⠒⣒⣒⣿⠛⠒⠒⠒⠒⠄⠄⣿⠄⠛⠄⠛⠄⠛⠄⣛⣀⣿⠄⠄⣀⣶⣒⣒⣶⣒⣶⣒⣒⣶⣀⠄⠄

		


--------------
# 项目简介

基于IPFS开发的分布式存储，旨在构建一个无限空间、不限速的共享存储平台 
用户可以通过共享存储资源来获得共享积分，并通过消费积分托管和访问数据

--------------
# WebDav 服务

- XFile 节点支持通过 WebDav 协议将存储挂载至本地设备或 NAS 服务器，轻松解锁海量无限存储空间。
- 为激励用户贡献本地缓存资源，维护分布式共享网络稳定健康运转，平台设立运行考核机制，指标未达标WebDav服务限制使用。
- 新用户有30天的考核豁免期，在豁免期内不受任何考核指标的限制，用户需在豁免期内完成所有的考核指标达标，否则30天后  webdav服务将不提供数据读取服务
- 具体规则如下：

|考核项目|限制阈值|触发结果|
|---|---|---|
|节点邮箱绑定|未绑定邮箱|WebDav 服务受限|
|本地空间容量|小于 4TiB|WebDav 服务受限|
|分享积分|小于 0|WebDav 服务受限|
|分享率|小于 0\.3|WebDav 服务受限|
|在线率|小于 0\.5|WebDav 服务受限|
|在线时长|小于 1小时|WebDav 服务受限|

### 💡 解锁说明

可通过**捐赠特权**豁免以上部分或全部考核限制，永久解锁WebDav考核限制。


### 积分规则：
1、提供本地存储空间为其他用户缓存数据，每缓存 1GB 数据 48 小时，奖励 (1GB * 奖励倍率) 的共享积分

2、为其他用户提供数据访问服务，每上传 1GB 数据，奖励 1GB 的共享积分

3、本节点从其他用户获取数据，每下载 1GB 数据，消费 1GB 的共享积分

4、本节点托管数据到其他节点，每托管 1GB 数据 48 小时，消费 1GB 的共享积分

💡 上传下载积分实时结算
托管消费 60分钟结算一次，计时前扣除积分
缓存奖励 60分钟结算一次，计时完成后发放


---------------------------------------------------------------------------------
🔔  奖励倍率根据当前节点连续在线天数进行动态调整：

	奖励倍率 = 【100 - 92*math.Exp(-0.01*x)】

	连续在线天数	      奖励倍率

	0	 		8.00
	5			12.49
	10			16.75
	20			24.68
	30			31.84
	50			44.20
	100			66.16
	365			97.61
---------------------------------------------------------------------------------
# Transmission BT下载服务

系统集成了BT下载工具Transmission，下载磁力或种子文件时，如果有其他节点完成了下载，本节点可实现秒下
- 秒下实现是直接将文件对应的根CID保存到本地，实际文件内容在其他分布式节点上
- 秒下完成后Transmission触发自动校验，会同步数据到本地，如网络环境较差可能导致校验失败，但不影响文件的正常使用
- 下载完成后，可能因为本地缓存清理，导致Transmission校验报错，影响继续做种，可手动执行校验重新缓存文件到本地

![预览](https://raw.githubusercontent.com/zhangv2012/xfile/main/4.png)

# 安装方式

### docker
|端口|说明|
|---|---|
|8081|webdav服务端口|
|9081|Transmission web端口|
|7681|Xfile webUI 端口|
```
docker run -d \
  --name xfile \
  --restart always \
  --privileged=true \
  --network bridge \
  -p 8081:8081 \
  -p 9091:9091 \
  -p 7681:7681 \
  --memory 2g \
  --cap-add NET_ADMIN \
  --cap-add SYS_ADMIN \
  --cap-add MKNOD \
  -v /共享网盘/data:/data \
  --device /dev/fuse:/dev/fuse \
  zhangv2012/xfile:latest
```
```

docker run -d \
  --name xfile \
  --restart always \
  --privileged=true \
  --network host \
  --memory 2g \
  --cap-add NET_ADMIN \
  --cap-add SYS_ADMIN \
  --cap-add MKNOD \
  -v /共享网盘/data:/data \
  --device /dev/fuse:/dev/fuse \
  zhangv2012/xfile:latest

```
### docker compose
```

services:
  xfile:
    image: zhangv2012/xfile:latest
    container_name: xfile
    restart: always
    privileged: true
    network_mode: host
    deploy:
      resources:
        limits:
          memory: 2G
    cap_add:
      - NET_ADMIN
      - SYS_ADMIN
      - MKNOD
    volumes:
      - /共享网盘/data:/data
    devices:
      - /dev/fuse:/dev/fuse
```

# 捐赠说明
--------------

🙏 感谢您的捐赠。无论捐赠的数目多少，作者都表示最真诚的感谢！🙏

🔔 **捐赠原则：金额100元起步**

**1\. 累计捐赠 100元 以下**

- 奖励分享积分，奖励额度：`1TiB * 捐赠金额`

**2\. 累计捐赠 100元 及以上**

- 奖励分享积分，奖励额度：`1TiB * 捐赠金额`

- 解锁永久捐赠特权：解除 **本地空间** 考核限制

**3\. 累计捐赠 200元 及以上**

- 奖励分享积分，奖励额度：`1TiB * 捐赠金额`

- 解锁永久捐赠特权：解除 **本地空间、分享率、在线率、在线时长** 全部考核限制

**4\. 累计捐赠 500元 及以上**

- 分享积分奖励：**无上限（+∞）**

- 解锁永久捐赠特权：解除 **全部考核规则**

---

### 💡 特别说明

- 所有奖励权益，自捐赠完成时间起生效

- 积分奖励需在捐赠后30天内完成兑换，逾期失效

- 特权奖励可在特权有效期内随时兑换，节点损坏、重建后可重复兑换

### ⚠️ 重要提示

捐赠时请在留言中填写**有效邮箱地址**，所有奖励兑换凭证、权益激活信息将统一通过邮件发放。


---------------
## 项目预览 

![预览1](https://raw.githubusercontent.com/zhangv2012/xfile/main/1.png)

![预览2](https://raw.githubusercontent.com/zhangv2012/xfile/main/2.png)

![预览3](https://raw.githubusercontent.com/zhangv2012/xfile/main/3.png)
