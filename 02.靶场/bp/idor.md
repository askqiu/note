# Lab: Unprotected admin functionality
这个lab有一个不受保护的管理员面板，请进去删除carlos

一开始我直接根路径拼接admin路径，显然不行，又去登陆页面看了下功能点，不行
访问robots.txt，发现管理员路径，直接访问可以进去

这里的难点是robos.txt

# Lab: Unprotected admin functionality with unpredictable URL
这个实验室有一个未受保护的管理员面板。它的位置不可预测，但该位置信息隐藏在应用程序的某个地方，解决实验室问题，请访问管理面板，并使用它来删除用户 carlos 。

![[Pasted image 20241103115034.png]]
findsomething直接拼接路径

# Lab: User role controlled by request parameter
这个实验室有一个管理员面板位于 /admin ，它通过一个可伪造的 cookie 来识别管理员。
解决实验室问题，请访问管理面板，并使用它来删除用户 carlos 。
您可以使用以下凭证登录您的账户： wiener:peter

访问admin路径，更改cookie参数，这里判断是不是admin只根据这个参数来，改为true
![[Pasted image 20241103115721.png]]
进行删除的时候也要改
![[Pasted image 20241103115854.png]]

# User role can be modified in user profile
找到对应功能点，加入我们插入roleid会怎么样
![[Pasted image 20241103170726.png]]
