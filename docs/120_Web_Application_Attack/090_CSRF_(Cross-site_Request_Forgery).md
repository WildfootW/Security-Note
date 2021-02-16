# CSRF (Cross-site Request Forgery)
[TOC]
###### tags: `Security` `Web`

當 Request 帶上 Cookie，Server 就沒辦法分辨是不是使用者自主發起的
```
<img src="http://wildfoo.tw/logout">
```

# Preventive measures
## Server Side
CSRF Token

## Client Side
SameSite cookie - 跨域請求帶不帶 cookie