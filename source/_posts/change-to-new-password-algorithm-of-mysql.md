---
title: MySQL中使用新版的密码加密算法
date: 2016-10-10 19:37:59
tags: MySQL
---
忘了是4.1还是多少的版本MySQL的密码加密方法`PASSWORD()`改版了，加长了，从原先的16位加长到了41位，算法好像是两次sha1，加盐。但有时候发现新版的MySQL在grant中还是会提示`The password hash doesn't have the expected format. Check if the correct password algorithm is being used with the PASSWORD() function.`
这时候select password(123)时也会发现输出是16位的，这应该是old_password方法还是默认方法，把old_password方法取消就好。
```powershell
set old_passwords=0;
```
或者直接在my.cnf里配置下
```powershell
old_passwords=0
```
