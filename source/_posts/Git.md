---
title: Git
date: 2019.05.14 11:49:30
tags: IDE
categories: IDE
---


**上传单个超过100M文件可使用Git LFS**：
在将要push的仓库里重新打开一个bash命令行：
只需设置1次 LFS : git lfs install
然后 跟踪一下你要push的大文件的文件或指定文件类型 git lfs track "*.pdf" ， 当然还可以直接编辑.gitattributes文件
以上已经设置完毕， 其余的工作就是按照正常的 add , commit , push 流程就可以了 :
```
git add yourLargeFile.pdf
git commit -m "Add Large file"
git push -u origin master
```

**只删除远程仓库，不删除本地仓库**
把xxx.iml加到`.gitignore`里面忽略掉，然后提交使.gitignore生效
```
git rm -r --cached xxx.iml　　  //-r 是递归的意思   当最后面是文件夹的时候有用
（git add xxx.iml）　　　　　 //若.gitignore文件中已经忽略了xxx.iml则可以不用执行此句
git commit -m "ignore xxx.xml"
git push
```

**上传项目**
第一步：本地git与github之间需要通过ssh密钥来连接，需先生成一个密钥
ssh-keygen -t rsa -C github邮箱
将.ssh中的内容复制到setting-new  SSH keys
第二步：
git init
git add .
git commit -m 'descriptionXXX'
(git config --global user.email    若首次登录需要验证邮箱)
(git config --global user.name    若首次登录需要验证用户名)
git remote add origin XXX.git    若remote origin already exits错误，可先执行git remote add origin再执行
git push -u origin master           需要输入用户名及密码

```
git add .                                   提交被修改的和新建的文件，但不包括被删除的文件                            
git add -u     --update              更新所有改变的文件，即提交所有变化的文件
git add -A    --all                       提交已被修改和已被删除文件，但是不包括新的文件
git add *                                   同步所有本地仓库和远程仓库（可用于删除远程仓库）
git pull (–rebase) origin master  意为先取消commit记录，并且把它们临时 保存为补丁(patch)(这些补丁放到”.git/rebase”目录中)，之后同步远程库到本地，下次push合并补丁到本地库之中。 处理 error：fail to push some refs
```
