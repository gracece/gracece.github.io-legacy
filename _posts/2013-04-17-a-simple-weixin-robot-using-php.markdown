---
author: admin
comments: true
date: 2013-04-17 12:47:21+00:00
layout: post
slug: a-simple-weixin-robot-using-php
title: 一个简单的微信机器人
wordpress_id: 836
categories:
- php
- SSP
tags:
- php
- ssp
- 微信
---

闲来无事（明明是还有好多作业没有做T_T),想把前一阵子给班里面注册的微信公共帐号给搞起来。具体的目的是，通过微信简单的指令，能够查询m.sysucs.org 上面的通知或者作业，方便其他同学使用。进一步的想法是能增加点趣味性，实现一些简单的关键词回复，并且用户可以按照规定的方式增加新的关键词与回答。
微信公共平台自带了简单的自动回复，不过直接忽略了，因为不能够和服务器交互，只能是自己手动输入。选择开发模式，按照文档验证好token之后，以后就不需要再验证了，代码只需调用一次。整个代码是在腾讯给的示例代码上更改增加得来。所以其实算是自己写的只有 sysucs这个类，类里面有两个函数，分别是reply()和talk()，其中reply()用于查询通知和作业，talk()用于回复简单的关键词。

<!-- more -->

在代码里面还有几个自己比较生疏的东西，在这里标记一下：

1、整个消息的接收与传送都是用xml来完成的。对于php，可以用simplexml_load_string 将一个xml转换为一个对象，使得数据更容易操作。具体说明可以看[simplexml-load-string](http://cn2.php.net/manual/zh/function.simplexml-load-string.php)说明文档。

2、如需使用 PHP 在服务器上生成 XML 响应，请使用下面的代码：

    
    <?php
    header("Content-type:text/xml");
    echo "<?xml version='1.0' encoding='ISO-8859-1'?>";
    echo "<note>";
    echo "<from>John</from>";
    echo "<to>George</to>";
    echo "<message>Don't forget the meeting!</message>";
    echo "</note>";
    ?>


> 请注意，响应头部的内容类型必须设置为 "text/xml"。来自[w3c xml_server](http://www.w3school.com.cn/xml/xml_server.asp)

在此次示例中，使用了sprintf进行模式替换生存xml进行响应。

3、CDATA （[参考xml_cdata.asp](http://www.w3school.com.cn/xml/xml_cdata.asp)）

术语 `CDATA` 指的是不应由 XML 解析器进行解析的文本数据（Unparsed Character Data）。
在 XML 元素中，"<" 和 "&" 是非法的。
"<" 会产生错误，因为解析器会把该字符解释为新元素的开始。
"&" 也会产生错误，因为解析器会把该字符解释为字符实体的开始。
某些文本，比如 JavaScript 代码，包含大量 "<" 或 "&" 字符。为了避免错误，可以将脚本代码定义为 CDATA。
CDATA 部分中的所有内容都会被解析器忽略。
CDATA 部分由 `<![CDATA[` 开始，由 `]]>` 结束：
CDATA 部分不能包含字符串 "]]>"。也不允许嵌套的 CDATA 部分。
标记 CDATA 部分结尾的 "]]>" 不能包含空格或折行。

4、在插入数据的过程中要对数据进行转义，以减去特殊符号带来的错误。比如表情 “哭”的代码就是 /::’( ，含有单引号会引起错误。另外更多表情代码关系在[这里](http://www.54575.com/blog/2013/04/11/%E5%BE%AE%E4%BF%A1%E8%87%AA%E5%B8%A6%E8%A1%A8%E6%83%85%E7%AC%A6%E5%8F%B7%E7%9A%84%E9%BB%98%E8%AE%A4%E4%BB%A3%E7%A0%81/)。

然后上代码吧，整个代码的逻辑还算是比较简单。

	<?php

	//define your token
	define("TOKEN", "sysucs");


	$wechatObj = new wechatCallbackapiTest();
	//$wechatObj->valid(); //第一次验证token时候才使用
	$wechatObj->responseMsg();

	class wechatCallbackapiTest
	{
	    public function valid()
	    {
	        $echoStr = $_GET["echostr"];

	        //valid signature , option
	        if($this->checkSignature()){
	            echo $echoStr;
	            exit;
	        }
	    }

	    public function responseMsg()
	    {
	        //get post data, May be due to the different environments
	        $postStr = $GLOBALS["HTTP_RAW_POST_DATA"];

	        //extract post data
	        if (!empty($postStr)){

	            $postObj = simplexml_load_string($postStr, 'SimpleXMLElement', LIBXML_NOCDATA);
	            $fromUsername = $postObj->FromUserName;
	            $toUsername = $postObj->ToUserName;
	            $keyword = trim($postObj->Content);
	            $time = time();
	            $textTpl = "<xml>
	                <ToUserName><![CDATA[%s]]></ToUserName>
	                <FromUserName><![CDATA[%s]]></FromUserName>
	                <CreateTime>%s</CreateTime>
	                <MsgType><![CDATA[%s]]></MsgType>
	                <Content><![CDATA[%s]]></Content>
	                <FuncFlag>0</FuncFlag>
	                </xml>";             
	            if(!empty( $keyword ))
	            {
	                $msgType = "text";
	                $sysucs =new sysucs();
	                if(preg_match('#通知|作业#i',$keyword))
	                {
	                    $contentStr=$sysucs->reply($keyword);
	                }
	                else if(preg_match('#subscribe|Hello2BizUser#i',$keyword))
	                    $contentStr ="欢迎关注计科一班微信！";
	                else
	                    $contentStr = $sysucs->talk($keyword);

	                $resultStr = sprintf($textTpl, $fromUsername, $toUsername, $time, $msgType, $contentStr);
	                echo $resultStr;
	            }else{
	                echo "空消息..";
	            }
	        }else {
	            echo "欢迎使用计科一班微信公共帐号 （sysucs)!";
	            exit;
	        }
	    }

	    private function checkSignature()
	    {
	        $signature = $_GET["signature"];
	        $timestamp = $_GET["timestamp"];
	        $nonce = $_GET["nonce"];  

	        $token = TOKEN;
	        $tmpArr = array($token, $timestamp, $nonce);
	        sort($tmpArr);
	        $tmpStr = implode( $tmpArr );
	        $tmpStr = sha1( $tmpStr );

	        if( $tmpStr == $signature ){
	            return true;
	        }else{
	            return false;
	        }
	    }
	}

	class sysucs
	{
	    public   function reply($keyword){
	        error_reporting(E_ALL);
	        $dbc = new mysqli('localhost','user','password','sysucs');
	        mysqli_query($dbc,"SET NAMES utf8");
	        date_default_timezone_set('PRC');
	        $resultStr =" ";

	        if(preg_match('#通知#i',$keyword))
	        {
	            $query = "SELECT * FROM info where 'content' not regexp '本周作业' ORDER BY date DESC;";
	            $result =mysqli_query($dbc,$query);
	            $num_results = $result -> num_rows;

	            if($num_results >3)
	                $num_results =3;
	            for($i=0;$i < $num_results;$i++)
	            {
	                $row = mysqli_fetch_array($result);
	                $resultStr .=  "【".($i+1)."】".date("Y-m-d H:i ",$row['date'])."\n";
	                $resultStr .=$row['content']."\n";
	            }
	            $resultStr .= "更多通知请访问 http://m.sysucs.org/more.php";
	        }
	        else if(preg_match('#作业#i',$keyword))
	        {
	            $query = "SELECT * FROM info where content like '%本周作业%' ORDER BY date DESC;";
	            $result =mysqli_query($dbc,$query);
	            $num_results = $result -> num_rows;

	            if($num_results >1)
	                $num_results =1;
	            for($i=0;$i < $num_results;$i++)
	            {
	                $row = mysqli_fetch_array($result);
	                echo "<p>".date("Y-m-d H:i ",$row['date'])."</p>";
	                echo $row['content'];
	                $resultStr .= "【".($i+1)."】".$row['content']."\n";
	            }
	        }
	        $resultStr=html_entity_decode($resultStr);
	        return strip_tags($resultStr);
	    }

	    function talk($keyword)
	    {
	        $dbc = new mysqli('localhost','user','password','sysucs');
	        mysqli_query($dbc,"SET NAMES utf8");
	        date_default_timezone_set('PRC');
	        if($keyword =="帮助")
	        {
	            return "回复“通知”查看最近通知,回复“作业”查看本周作业。\n   要教我回答，请英文下划线开头，接着问题，接着#，接着回答,例如“_计科一班#威武”。\n   其他功能还在完善中，欢迎提建议！http://m.sysucs.org";
	        }
	        if (substr($keyword, 0,1) == '_')
	        {
	            $pos = strpos($keyword, '#');
	            if ($pos > -1)
	            {
	                $q = substr($keyword, 1,$pos - 1);
	                if(preg_match('#帮助|作业|通知#i',$q))
	                    return "去去去，瞎捣乱，关键词你也来改";
	                $a = substr($keyword, $pos + 1);
	                if($q==""||$a=="")
	                    return "问题答案未填完整，换个姿势再来一次！";
	                $orginQ = $q;
	                $orgina = $a;
	                $q =addslashes($q);
	                $a =addslashes($a);
	                $query ="select * from talk where question = '".$q."' and answer = '".$a."'";
	                $exist = mysqli_query($dbc,$query);
	                if($exist->num_rows >0)
	                {
	                    return "讨厌，已经有一模一样的就不要再插了嘛 /::~";
	                }
	                else
	                {
	                    $query ="insert into talk values('".$q."','".$a."')";
	                    $result = mysqli_query($dbc,$query);
	                    if($result)
	                        return "/:8-)已记录：" . $orginQ . '/' . $orgina ;
	                    else 
	                        return "插入失败！";
	                }
	            }
	            else
	                return "分隔符改成#了(在英文标点里找)，可以输入表情 /::B";
	        }
	        else 
	        {
	            $orginKeyword =$keyword;
	            $keyword=addslashes($keyword);
	            $query ="select * from talk where question like '%".$keyword."%' order by rand() limit 1";
	            $result = mysqli_query($dbc,$query);
	            if($result)
	            {
	                $num_results = $result -> num_rows;
	                if($num_results == 0)
	                    return "我不知道【".$orginKeyword."】是神马东东/:&-(，教教我吧，下次我就能回答你了/:8*~可以按照 “_问题#回答” 的格式使我变得更聪(xié)明(è),可以加表情的~。输入“帮助”看看更多功能~";
	                else
	                {
	                    $row = mysqli_fetch_array($result);
	                    return $row['answer'];
	                }
	            }
	            else
	            {
	                return "数据库查询失败";
	            }
	        }
	    }
	}
	?>

最后，上一下二维码吧~

![](http://ww4.sinaimg.cn/large/a74ecc4cjw1e3sw67uyesj20by0byq3s.jpg)
