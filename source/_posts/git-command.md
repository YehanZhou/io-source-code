---
title: 将本地仓库提交到github上
date: 2016-09-19 10:32:10
tags: git 
---

1. 先在github上新建一个仓库,取名,例test

2. 本地新建文件夹

3. 打开命令行工具cd进这个文件夹

4. 执行`git init`,执行后会在该文件夹下生成一个名为.git的文件夹,里面有有关git的配置

5. 关联到远程仓库
`git remote add origin https://YehanZhou@github.com/YehanZhou/test.git`

6. 添加所有文件到暂存区
`git add .`

7. 提交
`git commit -m "description"`

8. 推送
`git push -u origin master`


