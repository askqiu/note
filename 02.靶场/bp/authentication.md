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

# 破坏暴力破解保护、IP 封锁
Lab: Broken brute-force protection, IP block
行 Burp 后，调查登录页面。注意，如果您连续提交 3 次错误的登录信息，您的 IP 将被临时阻止。然而，请注意，在达到此限制之前，您可以通过登录自己的账户来重置失败登录尝试的计数器。