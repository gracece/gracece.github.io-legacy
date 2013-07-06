---
author: admin
comments: true
date: 2013-03-30 02:59:41+00:00
layout: post
slug: php-simple-login
title: 使用php实现简单的登录系统
wordpress_id: 816
categories:
- php
- SSP
tags:
- php
---

![](http://i.imgur.com/THAAeWK.jpg)


有些时候，是希望把一些页面加密起来，经过用户名和密码的验证才能查看。没做之前觉得是一件很复杂的事情，真正做起来其实也没那么难。这里用到了PHP 的session来实现用户的登录并记录登录状态。

在这个例子，需要创建两个页面，一个是是index.php，另外一个是protected.php。看了我下面的例子你也可以写出这么个简单的例子来。

<!-- more -->


### 原理


使用session我们可以很简单地实现对用户登录状态的追踪，先看看session的定义：


> 会话机制（Session）在 PHP 中用于保存并发访问中的一些数据。这使可以帮助创建更为人性化的程序，增加站点的吸引力。


session是保存在服务器中的，基于这一点，我们可以在不同的页面中使用同一个session来最终用户是否登录并觉得是否显示被保护的内容。看一个简单的例子：


    <?php
        session_start();
        //使用该函数来创建一个新的session或者调用已经存在的session
        $_SESSION['loggedin'] =1;
        //创建了一个session变量并命名为loggedin
        echo $_SESSION['loggedin'];
        //这样就可以简单地输出loggedin里面的值
    ?>


正如你所见，session使用起来相当简单，下面说说登录系统的逻辑：
对于登录页面

    if （user matched its password） {
      set session(loggedin)=1;
      jump to protected page；
     } else
      echo error message;

对于保护页面


    if (session(loggedin) ==1)
        show the page;
    else
        jump back to the homepage;

逻辑不难懂，现在就可以开始写index.php了。


### index.php


在这个页面里，会包括基本的表单以用于输入用户名跟密码，同时当然还有对session的操作，上代码：

    <?php
    session_start();
    //需要用isset来检测变量，不然php可能会报错。
    if(isset($_SESSION['loggedin'])&&$_SESSION['loggedin']==1)
    {
        header("Location:protected.php");
        exit;
    }

    require("../header.html");
    ?>
        <body style="background:url(./img/<?php echo  mt_rand(1,5);?>.jpg)">
      <div>
        <div>
          <form  method="post">
            <h3>SYSUCS-后台管理</h3>
            <div id="info" style="display:none"></div>
            <input name="login" value="1" type="hidden" />
            <br />
            <input id="username" name="username" placeholder="Username" type="text" />
            <br />
            <input id="password" name="password" placeholder="Password" type="password" />
            <br />
            <span><a href="#" alt="未开放注册！">注册</a></span>
            <input type="submit" style="float:right;" value="登录" />
          </form>
        </div>

    <?php
    if(isset($_POST['login']))
    {
        $user =$_POST['username'];
        $password = $_POST['password'];
        //connect to mysql
        $dbc = new mysqli('localhost','user','password','upload');
        $query = "SELECT count(*) FROM user where name ='".$user."' and
            password = '".sha1($password)."'";
        //使用 sha1（）对密码进行了加密
        $result = mysqli_query($dbc,$query);
        if(!$result)
        {
            echo "数据查询失败！";
            exit;
        }
        $row = mysqli_fetch_row($result);
        $count = $row[0];
        //数据库中不应该使用多个相同的用户，所以这里返回的只能是 0或者1
        if($count >0 )
        {
            $_SESSION['loggedin']=1;
            header("Location:protected.php");
            exit;
        }
        else
        {
            //显示错误信息
            echo" <script > document.getElementById(\"info\").style.display='block';
                  document.getElementById(\"info\").innerHTML='你觉得是用户名错了还是密码错了？ ';
                                             </script > ";
        }
    }
    echo "</div>
        </body>
    </html>";
    ?>

这里可以说是最简单的登录表单了。表单接收用户名跟密码，并post到同个页面。同时还post了一个隐藏的参数login =1，以用于前面判断是否有用户名跟密码传送过来。其实也不一定要这样，也可以用过isset()检测username跟password的状态来实现。如果使用get来传送的话，也未尝不可，只是会在地址后面加参数，看着不是很顺心。
当接收到post过来的用户名跟密码，则进行数据库的查询与判断，这里对密码进行了sha1加密，看起来安全一点，如果匹配成功，则把session置1，跳转到保护页面。另外如果用户名跟密码匹配失败，则调用js输出提示信息（这里感觉应该用其他方式来实现提示的，但是暂时没想到更好的方法）。


###  protected.php


对于保护页面，则简单多了。上代码：

    <?php
    session_start();
    $logout=@$_GET['logout'];
    if($logout ==1)
        $_SESSION['loggedin']=0;

    if($_SESSION['loggedin']!=1)
    {
        header("Location:index.php");
        exit;
    }
    //下面就可以写保护页面的内容了，使用get方法传参数来log out
    ?>
    <a href="?logout=1" class="btn btn-danger">Log out</a>

至此，一个简单的登录验证页面就算完成了，如果还有其他页面需要保护，则只需要在该页面前面加上与保护页面相同的代码，这里可以封装到一个文件中使用require来实现，更加简洁。这里只是用了简单的数据库查找匹配，在实际的应用中还有更多需要注意的问题。在后面的学习中，我会继续看怎么进行改进实现更加安全高效的登录。

Thanks to :[how-to-make-a-login-system-with-php](http://www.webgeekly.com/tutorials/php/how-to-make-a-login-system-with-php/)
