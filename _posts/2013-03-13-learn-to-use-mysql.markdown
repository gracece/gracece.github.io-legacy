---
author: admin
comments: true
date: 2013-03-13 05:34:21+00:00
layout: post
slug: learn-to-use-mysql
title: 数据库简单操作，实现内容发布管理
wordpress_id: 803
categories:
- SSP
tags:
- mysql
- php
- ssp
---

终日忙忙碌碌，却似乎没干成什么事情，博客也好久没更新了。原本的计划是每一章各写一篇文章的，现在看来在有需求的时候可以连续翻好几章的内容，虽是走马观花但也算真正在操作数据库了。

下面是我尝试的一个简单的使用php对数据库进行操作的例子。index.php里面是实现内容的发布以及列出已经发布的内容。index.php将内容post到inseert_info.php进行插入数据库的操作，而delete.php则是实现删除操作。这里的数据库的结构是这样的，我在一个名为upload的数据库里面建立了一个表info，结构为[ content ，date，ip],分别记录内容，发布日期和发布者ip。前端用了bootstrap，所以下面的代码的css类都是与之相关的。

上代码：


`index.php`

    <?php
    require('../header.html');
    ?>

     <script type="text/javascript">
           function reloadPage(){
               window.location.reload();
           }
           function checkForm(){
               if(window.confirm("删除后不可恢复，确定删除？")){
                   return true;
               }
            return false;
           }
        </script>

    <body>
      <div>
        <h1><a href="http://gracece.net">gracece.net</a><small>  消息管理后台</small></h1>
        <hr />
        <form action="insert_info.php" method="post">
          <p>输入通知内容:</p>
          <textarea id="content" name="content" style="width:98% ;" rows="6"  />
          </textarea>
          <br />
          <input type="submit" class ="btn btn-large" style="margin-right:3px;" value="发布" />
          </form>
    <?php
            error_reporting(E_ALL);
              $dbc = new mysqli('localhost','user','password','upload');
              if(mysqli_connect_error()){
              echo "failed";
              }
              else{
              echo "<p>数据库连接成功，下面是历史消息：</p>";
              }
              mysqli_query($dbc,"set names utf-8");
              date_default_timezone_set('PRC');

              $query = "SELECT * FROM info ORDER BY date DESC;";
              $result =mysqli_query($dbc,$query);
              $num_results = $result->num_rows;

              echo"
                  <table class='table table-striped table-bordered'>
                  <thead>
                      <tr>
                          <th>#</th>
                          <th>date</th>
                          <th width='70%'>content</th>
                          <th>option</th>
                      </tr>
                  </thead>
                  <tbody> ";

              for($i=0;$i<$num_results;$i++)
              {
                  $row = mysqli_fetch_array($result);
                  echo "<tr>
                      <td>No.".($i+1)."</td>  ";
                  echo "<td>".date("Y/m/d H:i",$row['date'])."</td>";
                  echo "<td>".$row['content']."</td>";
                  echo "
                      <td>
                      <form style='margin:0 0 0; ' onsubmit='return checkForm()' target='_blank' action='delete.php' method='post'>
                      <input name='deleteInfo' type='hidden' value='".$row['date']."' />
                      <input type='submit'class='btn btn-danger' value='delete' />
                      </form>

                      </td> ";
              }
    ?>
        </div>
    </body>
    </html>


`insert_info.php`

    <?php

      function get_user_ip() {
          if(isset($_SERVER['HTTP_CLIENT_IP']) && $_SERVER['HTTP_CLIENT_IP'] !='unknown')
          { $ip = $_SERVER['HTTP_CLIENT_IP'];}
          elseif (isset($_SERVER['HTTP_X_FORWARDED_FOR']) && $_SERVER['HTTP_X_FORWARDED_FOR']!='unknown')
          {$ip = $_SERVER['HTTP_X_FORWARDED_FOR'];}
          else{
          $ip = $_SERVER['REMOTE_ADDR'];
          }
          return $ip;
      }

      $ip =get_user_ip();
      $dbc = new mysqli('localhost','user','password','upload');
              date_default_timezone_set('PRC');
               $date =date("U");
              @$content=$_POST['content'];
              if(!get_magic_quotes_gpc())
              {
                  $content = addslashes($content);
              }
              echo $content;
              echo "<br />";

              if ($content == NULL)
                  exit;
                         $query = "INSERT INTO info values ('".$content."','".$date."','".$ip."')";
              $sub = mysqli_query($dbc,$query);
              if($sub){
                  echo "加入数据库<br />";
              }else{
                  echo "失败";
              }
    ?>

`delete.php`

    <?php
    require("../header.html");
    ?>
    <body>
    <div>

    <?php
        $dbc = new mysqli('localhost','user','password','upload');
        mysqli_query($dbc,"set names UTF-8");
        if(mysqli_connect_error()){
            echo "failed";
        }
        date_default_timezone_set('PRC');
        $deleteInfo = $_POST['deleteInfo'];
        echo "<div class='well'><h2>deleting</h2><div class='alert alert-info'> ".$deleteInfo." </div>";
        $query = "DELETE FROM info WHERE date = '".$deleteInfo."'";
        if(mysqli_query($dbc,$query)) echo "已从数据库删除";
    ?>

    </body>
    </html>
