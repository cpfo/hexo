---
title: python电子邮件
categories: [技术]
tags: [python]
date: 2024-01-09 15:56:43
---

使用SMTP发送邮件，使用POP3接收邮件。

<!-- more -->

# SMTP发送邮件

SMTP是发送邮件的协议，Python内置对SMTP的支持，可以发送纯文本邮件、HTML邮件以及带附件的邮件。

Python对SMTP支持有`smtplib`和`email`两个模块，`email`负责构造邮件，`smtplib`负责发送邮件。

```python
from email.mime.text import MIMEText
msg = MIMEText('测试下邮件发送', 'plain', 'utf-8')
```
注意到构造`MIMEText`对象时，第一个参数就是邮件正文，第二个参数是MIME的subtype，传入`'plain'`表示纯文本，最终的MIME就是`'text/plain'`，最后一定要用`utf-8`编码保证多语言兼容性。

然后，通过SMTP发出去：

```python
from email.header import Header
from email.utils import formataddr, parseaddr
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart

def _format_addr(s):
    name, addr = parseaddr(s)
    return formataddr((Header(name, 'utf-8').encode(), addr))


# 输入Email地址和口令:
from_addr = 'xxxxxx'
password = 'xxxx'
# 输入收件人地址:
to_addr = 'xxx@qq.com'
# 输入SMTP服务器地址:
smtp_server = 'xxx.xxx.com'

msg = MIMEText('哈哈哈哈哈', 'plain', 'utf-8')

msg['From'] = _format_addr('测试下邮件发送 <%s>' % from_addr)
msg['To'] = _format_addr('管理员 <%s>' % to_addr)
msg['Subject'] = Header('来自SMTP的问候……', 'utf-8').encode()

import smtplib
server = smtplib.SMTP(smtp_server, 25) # SMTP协议默认端口是25
server.set_debuglevel(1)
server.login(from_addr, password)
server.sendmail(from_addr, [to_addr], msg.as_string())
server.quit()
```
我们用`set_debuglevel(1)`就可以打印出和SMTP服务器交互的所有信息。SMTP协议就是简单的文本命令和响应。`login()`方法用来登录SMTP服务器，`sendmail()`方法就是发邮件，由于可以一次发给多个人，所以传入一个`list`，邮件正文是一个`str`，`as_string()`把`MIMEText`对象变成`str`。

必须把`From`、`To`和`Subject`添加到`MIMEText`中，才是一封完整的邮件， 否则无法显示主题，收件人等信息。

## 发送HTML邮件

如果我们要发送HTML邮件，方法很简单，在构造`MIMEText`对象时，把`HTML`字符串传进去，再把第二个参数由`plain`变为`html`就可以了

```python
msg = MIMEText('<h1>哈哈哈哈哈</h1>', 'html', 'utf-8')
```

## 发送附件

可以构造一个`MIMEMultipart`对象代表邮件本身，然后往里面加上一个`MIMEText`作为邮件正文，再继续往里面加上表示附件的`MIMEBase`对象即可

```python
msg = MIMEMultipart()
msg['From'] = _format_addr('Python爱好者 <%s>' % from_addr)
msg['To'] = _format_addr('管理员 <%s>' % to_addr)
msg['Subject'] = Header('来自SMTP的问候……', 'utf-8').encode()

# 邮件正文是MIMEText:
msg.attach(MIMEText('send with file...', 'plain', 'utf-8'))

# 添加附件就是加上一个MIMEBase，从本地读取一个图片:
with open('D:/oldimg.jpg', 'rb') as f:
    # 设置附件的MIME和文件名，这里是png类型:
    mime = MIMEBase('image', 'jpg', filename='oldimg.jpg')
    # 加上必要的头信息:
    mime.add_header('Content-Disposition', 'attachment', filename='oldimg.jpg')
    mime.add_header('Content-ID', '<0>')
    mime.add_header('X-Attachment-Id', '0')
    # 把附件的内容读进来:
    mime.set_payload(f.read())
    # 用Base64编码:
    encoders.encode_base64(mime)
    # 添加到MIMEMultipart:
    msg.attach(mime)
```
然后，按正常发送流程把msg（注意类型已变为`MIMEMultipart`）发送出去，就可以收到如下带附件的邮件

## 发送图片

如果要把一个图片嵌入到邮件正文中怎么做？直接在HTML邮件中链接图片地址行不行？答案是，大部分邮件服务商都会自动屏蔽带有外链的图片，因为不知道这些链接是否指向恶意网站。

要把图片嵌入到邮件正文中，我们只需按照发送附件的方式，先把邮件作为附件添加进去，然后，在HTML中通过引用`src="cid:0"`就可以把附件作为图片嵌入了。如果有多个图片，给它们依次编号，然后引用不同的`cid:x`即可。

把上面代码加入`MIMEMultipart`的`MIMEText`从`plain`改为`html`，然后在适当的位置引用图片

```python
msg.attach(MIMEText('<html><body><h1>下面是一个图片</h1><br/><img src="cid:0"></body></html>', 'html', 'utf-8'))
```


## 同时支持HTML和Plain格式
为了兼容旧设备，在发送HTML的同时再附加一个纯文本，如果收件人无法查看HTML格式的邮件，就可以自动降级查看纯文本邮件。

```python
msg = MIMEMultipart('alternative')
msg['From'] = ...
msg['To'] = ...
msg['Subject'] = ...

msg.attach(MIMEText('hello', 'plain', 'utf-8'))
msg.attach(MIMEText('<html><body><h1>Hello</h1></body></html>', 'html', 'utf-8'))
# 正常发送msg对象...
```

## 加密SMTP

使用标准的25端口连接SMTP服务器时，使用的是明文传输，发送邮件的整个过程可能会被窃听。要更安全地发送邮件，可以加密SMTP会话，实际上就是先创建SSL安全连接，然后再使用SMTP协议发送邮件。
```python
server = smtplib.SMTP(smtp_server, 465) 
server.starttls()
```
只需要在创建SMTP对象后，立刻调用starttls()方法，就创建了安全连接。后面的代码和前面的发送邮件代码完全一样。

构造一个邮件对象就是一个`Messag`对象，如果构造一个`MIMEText`对象，就表示一个文本邮件对象，如果构造一个`MIMEImage`对象，就表示一个作为附件的图片，要把多个对象组合起来，就用`MIMEMultipart`对象，而`MIMEBase`可以表示任何对象。它们的继承关系如下：

```shell
Message
+- MIMEBase
   +- MIMEMultipart
   +- MIMENonMultipart
      +- MIMEMessage
      +- MIMEText
      +- MIMEImage
```
这种嵌套关系就可以构造出任意复杂的邮件。


# POP3收取邮件

Python内置一个`poplib`模块，实现了POP3协议，可以直接用来收邮件。

注意到POP3协议收取的不是一个已经可以阅读的邮件本身，而是邮件的原始文本，这和SMTP协议很像，SMTP发送的也是经过编码后的一大段文本。

要把POP3收取的文本变成可以阅读的邮件，还需要用email模块提供的各种类来解析原始文本，变成可阅读的邮件对象。

所以，收取邮件分两步：

第一步：用`poplib`把邮件的原始文本下载到本地；

第二步：用`email`解析原始文本，还原为邮件对象。

## 通过POP3下载邮件

```python
import poplib

# 输入邮件地址, 口令和POP3服务器地址:
email = input('Email: ')
password = input('Password: ')
pop3_server = input('POP3 server: ')

# 连接到POP3服务器:
server = poplib.POP3(pop3_server)
# 可以打开或关闭调试信息:
server.set_debuglevel(1)
# 可选:打印POP3服务器的欢迎文字:
print(server.getwelcome().decode('utf-8'))

# 身份认证:
server.user(email)
server.pass_(password)

# stat()返回邮件数量和占用空间:
print('Messages: %s. Size: %s' % server.stat())
# list()返回所有邮件的编号:
resp, mails, octets = server.list()
# 可以查看返回的列表类似[b'1 82923', b'2 2184', ...]
print(mails)

# 获取最新一封邮件, 注意索引号从1开始:
index = len(mails)
resp, lines, octets = server.retr(index)

# lines存储了邮件的原始文本的每一行,
# 可以获得整个邮件的原始文本:
msg_content = b'\r\n'.join(lines).decode('utf-8')
# 稍后解析出邮件:
msg = Parser().parsestr(msg_content)

# 可以根据邮件索引号直接从服务器删除邮件:
# server.dele(index)
# 关闭连接:
server.quit()
```

用POP3获取邮件其实很简单，要获取所有邮件，只需要循环使用`retr()`把每一封邮件内容拿到即可。真正麻烦的是把邮件的原始内容解析为可以阅读的邮件对象。

## 解析邮件

解析邮件的过程和上一节构造邮件正好相反，因此，先导入必要的模块：

```shell
from email.parser import Parser
from email.header import decode_header
from email.utils import parseaddr

import poplib
```
只需要一行代码就可以把邮件内容解析为`Message`对象：

但是这个`Message`对象本身可能是一个`MIMEMultipart`对象，即包含嵌套的其他`MIMEBase`对象，嵌套可能还不止一层。

所以我们要递归地打印出`Message`对象的层次结构：

