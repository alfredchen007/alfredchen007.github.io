---

layout: post

title: "Use 'expect' tool to change passwd in Linux"

date: 2017-12-24 10:23:20 +0300

description:  

img: passwd.jpg 

tags: [linux]

---



### change_passwd

A little demo for the usage of **'expect'** in shell.<!-- more -->

Normally, distributed system always contain certain nodes, we come across such a situation that each node owns a same user we call it 'pop', we use this script to change password of user 'pop'.

#### install & run

copy the whole folder to one of the nodes and install expect

use 'admin' user run "run.sh" `sh run.sh 2>&1 | tee passwd.log`( you can also use root as you like, just change the script ) .

#### find source code 

[click me](https://github.com/alfredchen007/change_passwd)

