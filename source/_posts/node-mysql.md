---
title: node连接mysql数据库实例
date: 2016-11-18 15:07:33
tags: [node,mysql]
---


## Express生成应用
首先假定你已经安装了[Node.js](https://nodejs.org/en/)，接下来通过应用生成器工具 express 可以快速创建一个应用的骨架。
通过如下命令安装express和express-generator：
```
$ npm install express -g
$ npm install express-generator -g
```
当前工作目录下创建一个命名为 myapp 的应用:
```
$ express myapp

   create : myapp
   create : myapp/package.json
   create : myapp/app.js
   create : myapp/public
   create : myapp/public/javascripts
   create : myapp/public/images
   create : myapp/routes
   create : myapp/routes/index.js
   create : myapp/routes/users.js
   create : myapp/public/stylesheets
   create : myapp/public/stylesheets/style.css
   create : myapp/views
   create : myapp/views/index.jade
   create : myapp/views/layout.jade
   create : myapp/views/error.jade
   create : myapp/bin
   create : myapp/bin/www
```
然后安装所有依赖包：

```
$ cd myapp 
$ npm install
```
启动这个应用（MacOS 或 Linux 平台）：

```
$ DEBUG=myapp npm start
```
Windows 平台使用如下命令：

```
> set DEBUG=myapp & npm start
```
然后在浏览器中打开 http://localhost:3000/ 网址就可以看到这个应用了。
通过 Express 应用生成器创建的应用一般都有如下目录结构：
```
.
├── app.js
├── bin
│   └── www
├── package.json
├── public
│   ├── images
│   ├── javascripts
│   └── stylesheets
│       └── style.css
├── routes
│   ├── index.js
│   └── users.js
└── views
    ├── error.jade
    ├── index.jade
    └── layout.jade

7 directories, 9 files
```


## MySQL数据库

### 连接本地数据库
安装navicat和MySQL。
进入【系统偏好设置】启动本地mysql服务:
![run mysql](http://7xw4c0.com1.z0.glb.clouddn.com/mysqlrun.png?imageView/2/w/619/q/100)
在navicat中连接本地数据库:
![connect mysql](http://7xw4c0.com1.z0.glb.clouddn.com/newsql.png?imageView/2/w/500/q/100)

### 创建数据库
创建一个测试数据库nodesample，在数据库中建一个userinfo表
```sql
CREATE DATABASE IF NOT EXISTS nodesample CHARACTER SET UTF8;

USE nodesample;
 
SET FOREIGN_KEY_CHECKS=0;

DROP TABLE IF EXISTS `userinfo`;
CREATE TABLE `userinfo` (
`Id` int(11) NOT NULL AUTO_INCREMENT COMMENT '主键',
`UserName` varchar(64) NOT NULL COMMENT '用户名',
`UserPass` varchar(64) NOT NULL COMMENT '用户密码',
PRIMARY KEY (`Id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='用户信息表';

```


## node连接数据库

Node.js与MySQL交互操作有很多库,这里选择[felixge/node-mysql](https://github.com/felixge/node-mysql)。
安装mysql模块
```
$ npm install mysql --save
```
打开`routes/users.js`,引入mysql模块
```javascript
var mysql = require('mysql');
```
建立连接
```javascript
var connection = mysql.createConnection({
  host: '127.0.0.1',
  user: 'root',
  password: '123',
  port: '3306',
  database: 'nodesample'
});

connection.connect(function(err){
  if(!err) {
    console.log("Database is connected ... nn");
  } else {
    console.log("Error connecting database ... nn");
  }
});
```
## 实现RESTFUL接口
1. 增
```javascript
/*
 * PUT to updateuser.
 */
router.put('/updateuser/:id', function(req, res) {
  var userUpSql = 'UPDATE userinfo SET ? WHERE Id='+req.params.id;
  connection.query(userUpSql, req.body, function(err, result) {
    res.send(
        (err === null) ? { msg: '' } : { msg: err }
    );
  });
});
```
2. 删
```javascript
/*
 * DELETE to deleteuser.
 */
router.delete('/deleteuser/:id', function(req, res) {
  var userDelSql = 'DELETE FROM userinfo  where Id='+req.params.id;
  connection.query(userDelSql, function (err, result) {
    res.send((err === null) ? { msg: '' } : { msg:'error: ' + err });
  });
});
```
3. 改
```javascript
/*
 * POST to adduser.
 */
router.post('/adduser', function(req, res) {
  connection.query('INSERT INTO userinfo SET ?', req.body, function(err, result) {
    res.send(
        (err === null) ? { msg: '' } : { msg: err }
    );
  });
});
```
4. 查
```javascript
/* GET users listing. */
router.get("/userlist",function(req,res){
  connection.query('SELECT * from userinfo', function(err, rows, fields) {
    if (!err)
      res.json(rows);
    else
      console.log('Error while performing Query.');
  });
});
```

## 画页面
在public/javascripts中新建global.js,并打开views/layout.jade,
在末尾引入jquery和global.js。
```
    script(src='http://cdn.bootcss.com/jquery/1.11.1/jquery.min.js')
    script(src='/javascripts/global.js')
```
打开views/index.jade,写好页面结构:
```
extends layout
block content
  // Wrapper
  #wrapper
    // USER INFO
    #userInfo
      h2 User Info
      p
        strong Name:
        |  <span id='UserName'></span>
        br
        strong Pass:
        |  <span id='UserPass'></span>
    // /USER INFO

    // USER LIST
    h2 User List
    #userList
      table
        thead
          th UserName
          th UserPass
          th Delete?
          th Edit?
        tbody
    // /USER LIST

    // ADD USER
    h2 Add User
    #addUser
      fieldset
        input#inputUserName(type='text', placeholder='Username')
        input#inputUserPass(type='text', placeholder='UserPass')
        br
        button#btnAddUser Add User
    // /ADD USER

    .mask
    #upUser.alert
      fieldset
        input#modId(type='hidden')
        input#modUserName(type='text', placeholder='Username')
        input#modUserPass(type='text', placeholder='UserPass')
        br
        button#btnModUser Modify User
```
样式写在public/stylesheets/style.css里,可以看我放到[github](https://github.com/YehanZhou/node-mysql-tutorial)上的源码。

## 调用接口
在上面新建的global.js里编写调用接口的代码
1. 查数据并画表格
```javascript
function populateTable() {
    var tableContent = '';
    $.getJSON( '/users/userlist', function( data ) {
        userListData = data;
        $.each(data, function(){
            tableContent += '<tr>';
            tableContent += '<td><a href="#" class="linkshowuser" rel="' + this.UserName + '" title="Show Details">' + this.UserName + '</a></td>';
            tableContent += '<td>' + this.UserPass + '</td>';
            tableContent += '<td><a href="#" class="linkdeleteuser" rel="' + this.Id + '">delete</a></td>';
            tableContent += '<td><a href="#" class="linkupdateuser" rel="' + this.Id + '">edit</a></td>';
            tableContent += '</tr>';
        });
        $('#userList table tbody').html(tableContent);
    });
};
```
2. 增
```javascript
function addUser(event) {
    event.preventDefault();
    // 每个输入框必填
    var errorCount = 0;
    $('#addUser input').each(function(index, val) {
        if($(this).val() === '') { errorCount++; }
    });
    if(errorCount === 0) {
        var newUser = {
            'UserName': $('#addUser fieldset input#inputUserName').val(),
            'UserPass': $('#addUser fieldset input#inputUserPass').val()
        };
        $.ajax({
            type: 'POST',
            url: '/users/adduser',
            data: newUser,
            dataType: 'JSON'
        }).done(function( response ) {
            if (response.msg === '') {
                // Clear the form inputs
                $('#addUser fieldset input').val('');
                // Update the table
                populateTable();
            }
            else {
                alert('Error: ' + response.msg);
            }
        });
    }
    else {
        alert('Please fill in all fields');
        return false;
    }
};
```
3. 删
```javascript
function deleteUser(event) {
    event.preventDefault();
    // 确定删除吗
    var confirmation = confirm('Are you sure you want to delete this user?');
    if (confirmation === true) {
        $.ajax({
            type: 'DELETE',
            url: '/users/deleteuser/' + $(this).attr('rel')
        }).done(function( response ) {
            if (response.msg === '') {
            }
            else {
                alert('Error: ' + response.msg);
            }
            populateTable();

        });

    }
    else {
        return false;
    }
};
```
4. 改
```javascript
function updateUser(event) {
    event.preventDefault();
    // 每个输入框必填
    var errorCount = 0;
    $('#upUser input').each(function(index, val) {
        if($(this).val() === '') { errorCount++; }
    });
    if(errorCount === 0) {
        var upUser = {
            'UserName': $('#upUser fieldset input#modUserName').val(),
            'UserPass': $('#upUser fieldset input#modUserPass').val()
        };
        $.ajax({
            type: 'PUT',
            url: '/users/updateuser/' + $('#upUser fieldset input#modId').val(),
            data: upUser,
            dataType: 'JSON'
        }).done(function( response ) {
            if (response.msg === '') {
                $('.mask').hide();
                $('.alert').hide();
                populateTable();
            }
            else {
                alert('Error: ' + response.msg);
            }
        });
    }
    else {
        return false;
    }
};
```
完整的代码可以去[github](https://github.com/YehanZhou/node-mysql-tutorial)上查看,以下是最终效果:
![final view](http://7xw4c0.com1.z0.glb.clouddn.com/view.png?imageView/2/w/619/q/100)

## 参考文章

### express

[用NODE.JS,EXPRESS,MONGODB创建一个简单的RESTFUL WEB APP](http://cwbuecheler.com/web/tutorials/2014/restful-web-app-node-express-mongodb/)

[express中文网](http://www.expressjs.com.cn/)

### mysql与node

[Nodejs学习笔记(四)与MySQL交互(felixge/node-mysql)](http://blog.csdn.net/gebitan505/article/details/46346917)

[Node.js and MySQL tutorial](https://codeforgeek.com/2015/01/nodejs-mysql-tutorial/)

