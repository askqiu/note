# 使用token越权修改密码认证
Password reset broken logic

temp-forgot-password-token=5vt2xoz06ylc1hexdaweo9s8hb1ndggn&username=wiener&new-password-1=123&new-password-2=123
wiener改为carlos，就可以重置carlos的密码

# 通过细微不同的响应进行用户名枚举
Username enumeration via subtly different responses
爆破用户名，并且grep错误提示，发现有个结果的错误提示有点不一样。

# 用户名枚举通过响应时间
首先要绕过多次爆破封锁ip，利用xff头
然后枚举用户名，密码的位数写大电，爆破完之后看见个返回时间比较久的