---
title: git常用命令
type: categories
date: 2019-02-27 12:52:46
---
### git常用命令
1.如何配置你的身份
```
git config --global user.name "Albert"

git config --global user.email "albert@gmal.com"
```
验证是否配置成功(只需要将最后的名字和邮箱地址去掉即可)
```
git config --global user.name

git config --global user.email
```
2.如何创建代码仓库

先进入到项目目录下面,执行命令:
`git init`

仓库创建完成后,会在项目的根目录下生成一个隐藏的.git文件夹,可以通过ls -al命令来查看一下

3.如何提交本地代码

add是用于把想要提交的代码先添加进来,而commit则是真正地去执行提交操作,比如我们想要添加AndroidManifest.xml,可以输入一下命令:
`git add AndroidManifest.xml`

添加目录也是这样,只需将文件名改成目录名即可.
`git add src`

一次性把所有文件都添加好的命令如下:
`git add .`

现在项目所有文件已经添加好了,我们来提交一下,输入如下命令:
`git commit -m "First commit."`

注意-m参数用来加上提交的描述信息.

4.如何把远程版本库克隆到本地
`git clone https://github.com/albert567/mobilesafe.git`

5.如何将提交的内容同步到远程版本库
`git push https://github.com/albert567/mobilesafe.git master`

同步时GitHub需要输入用户名和密码来进行身份校验.

注意,如果本地第一次同步远程版本库,需要先将远程版本库clone到本地,然后将.git和README.md拷贝到本地项目根目录,再执行add,commit命令

6.如何把文件从版本控制中删掉
`git rm res/layout/activity_main.xml`

7.如何查看状态
`git status`

8.如何查看所有分支
`git branch -a`

9.如何添加分支
`git branch test    //如果 test分支不存在,添加该分支`

10.如何切换分支
`git checkout test  //切换到test分支`

11.如何删除分支
`git branch -d test   //删除test分支`

12.如何查看提交修改记录
`git log`