一闪而过的后台
会话存储，可能有logintype这样的参数判断你是否登陆，改成true，
f12--存储--会话存储--key-value

挖小程序的时候，api的测试很重要
/comment/praise?...
改成/comment/delete?
删除别人评论

中间商二次退钱思路，飞猪买票，12306退票拿钱，飞猪再次退票拿钱，
类似的思路也可以在做活动的时候挖

oauth2.0，code劫持，问题出在redirect_uri没有校验
1.看code回调到哪一个参数的url
2.如何绕过限制（常规绕过，二次跳转）