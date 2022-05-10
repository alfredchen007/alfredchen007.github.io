---
layout: post

title: "synology上用docker启动gitlab后修改root密码"

date: 2022-5-09 21:47:40 +0300

description:  

cover: 'https://jc-1258611203.cos.ap-beijing.myqcloud.com/blog/2022-05-10-dockerandgitlab.webp'

color: rgb(222,315,333)

tags: [gitlab]
---

 

### 启动 docker

![](https://jc-1258611203.cos.ap-beijing.myqcloud.com/blog/2022-05-10-%E6%88%AA%E5%B1%8F2022-05-10%2022.12.13.png)

用了最新的版本，启动 docker 后发现不再提示创建 root 密码

![](https://jc-1258611203.cos.ap-beijing.myqcloud.com/blog/2022-05-10-%E6%88%AA%E5%B1%8F2022-05-10%2022.07.18.png)

### 进入后台修改

![](https://jc-1258611203.cos.ap-beijing.myqcloud.com/blog/2022-05-10-%E6%88%AA%E5%B1%8F2022-05-10%2022.14.05.png)

新建一个 bash，输入 gitlab-rails console 进入空控制台

![](https://jc-1258611203.cos.ap-beijing.myqcloud.com/blog/2022-05-10-%E6%88%AA%E5%B1%8F2022-05-10%2022.19.32.png)

```shell
user.password = '密码'
user.password_confirmation = '密码'
```

![](https://jc-1258611203.cos.ap-beijing.myqcloud.com/blog/2022-05-10-%E6%88%AA%E5%B1%8F2022-05-10%2022.24.17.png)

保存修改 user.save

![](https://jc-1258611203.cos.ap-beijing.myqcloud.com/blog/2022-05-10-%E6%88%AA%E5%B1%8F2022-05-10%2022.27.34.png)

登陆验证

![](https://jc-1258611203.cos.ap-beijing.myqcloud.com/blog/2022-05-10-%E6%88%AA%E5%B1%8F2022-05-10%2022.29.02.png)



