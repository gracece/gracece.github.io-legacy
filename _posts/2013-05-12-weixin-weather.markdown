---
author: admin
comments: true
date: 2013-05-12 08:13:10+00:00
layout: post
slug: weixin-weather
title: 微信天气简单实现
wordpress_id: 847
categories:
- php
- SSP
tags:
- php
---




近日想在班级微信加点新功能，但是暂时没有想到什么好点子，那就先实现个简单的天气查询吧。 google一下天气相关的接口，考虑简单易用选择了国家气象局提供的天气预报接口。 打开[这里](http://m.weather.com.cn/data/101280101.html),可以看到返回的是JSON数据。


使用[JSON格式化工具](http://www.ostools.net/codeformat/json) 格式一下可以更好地分析：

    
    {
        "weatherinfo": {
            "city": "广州", 
            "city_en": "guangzhou", 
            "date_y": "2013年5月12日", 
            "date": "", 
            "week": "星期日", 
            "fchh": "11", 
            "cityid": "101280101", 
            "temp1": "30℃~22℃", 
            "temp2": "31℃~24℃", 
            "temp3": "31℃~25℃", 
            "temp4": "29℃~25℃", 
            "temp5": "29℃~26℃", 
            "temp6": "30℃~25℃", 
            "tempF1": "86℉~71.6℉", 
            "tempF2": "87.8℉~75.2℉", 
            "tempF3": "87.8℉~77℉", 
            "tempF4": "84.2℉~77℉", 
            "tempF5": "84.2℉~78.8℉", 
            "tempF6": "86℉~77℉", 
            "weather1": "阵雨转多云", 
            "weather2": "多云", 
            "weather3": "阴转雷阵雨", 
            "weather4": "大雨转中雨", 
            "weather5": "中雨转大雨", 
            "weather6": "大雨转雷阵雨", 
            "img1": "3", 
            "img2": "1", 
            "img3": "1", 
            "img4": "99", 
            "img5": "2", 
            "img6": "4", 
            "img7": "9", 
            "img8": "8", 
            "img9": "8", 
            "img10": "9", 
            "img11": "9", 
            "img12": "4", 
            "img_single": "3", 
            "img_title1": "阵雨", 
            "img_title2": "多云", 
            "img_title3": "多云", 
            "img_title4": "多云", 
            "img_title5": "阴", 
            "img_title6": "雷阵雨", 
            "img_title7": "大雨", 
            "img_title8": "中雨", 
            "img_title9": "中雨", 
            "img_title10": "大雨", 
            "img_title11": "大雨", 
            "img_title12": "雷阵雨", 
            "img_title_single": "阵雨", 
            "wind1": "微风", 
            "wind2": "微风", 
            "wind3": "微风", 
            "wind4": "微风", 
            "wind5": "微风", 
            "wind6": "微风", 
            "fx1": "微风", 
            "fx2": "微风", 
            "fl1": "小于3级", 
            "fl2": "小于3级", 
            "fl3": "小于3级", 
            "fl4": "小于3级", 
            "fl5": "小于3级", 
            "fl6": "小于3级", 
            "index": "热", 
            "index_d": "天气较热，建议着短裙、短裤、短套装、T恤等夏季服装。年老体弱者宜着长袖衬衫和单裤。", 
            "index48": "热", 
            "index48_d": "天气较热，建议着短裙、短裤、短套装、T恤等夏季服装。年老体弱者宜着长袖衬衫和单裤。", 
            "index_uv": "中等", 
            "index48_uv": "中等", 
            "index_xc": "不宜", 
            "index_tr": "适宜", 
            "index_co": "较不舒适", 
            "st1": "28", 
            "st2": "22", 
            "st3": "31", 
            "st4": "24", 
            "st5": "32", 
            "st6": "23", 
            "index_cl": "较不宜", 
            "index_ls": "不太适宜", 
            "index_ag": "极不易发"
        }
    }
  


`temp1-6`为当前时间6天内的天气温度情况，`tempF1-6`为对应的华氏温度，另外下面还有一些对天气的描述，比如风力风速、今日穿衣指数 、48小时穿衣指数，紫外线、洗车、旅游、晨练、晾晒、过敏等。还有一些关键字没有弄懂代表的意思是什么。

然后接下来就是在微信响应代码中加入一个响应天气的函数咯,这里用到了城市名称和城市代码的对应关系，具体在一个csv文件里面。[下载csv](http://pan.baidu.com/share/link?shareid=460829&uk=2032069257)：

    
    public function weather($keyword)
        {
            $pos =strpos($keyword,"天气");
            $city = substr($keyword,0,$pos);
            if ($pos !=0)
            {
                if($city == "实时")
                {
                    $url ="http://www.7timer.com/eesael/wap/wse.php";
                    $data = file_get_contents($url);
                    $data =trim(strip_tags($data));
                    $data = preg_replace('/((\s)*(\n)+(\s)+)/i',"\n--",$data);
                    $data =trim ($data);
                    $start = strpos($data,"24h图像");
                    $start += 10;
                    $stop = strpos($data,"日照时数");
                    $leng =$stop - $start;
                    return "东校区实时天气:".substr($data,$start,$leng)."信息来源\n".$url;
                }
                else
                {
    
                    $dbc = new mysqli('localhost','xxx','xxx','db');
                    mysqli_query($dbc,"SET NAMES utf8");
                    $query = "select * from weather where name = '".$city."'";
                    $result = mysqli_query($dbc,$query);
                    if(!$result||$result->num_rows == 0)
                        return "未找到相关信息！已经收录大部分县级以上城市，正确的查询格式为“北京天气”，若前面不加城市则默认为广州。目前火星等地方不可查询O(∩_∩)O~";
                    $row = mysqli_fetch_array($result);
                    $code = $row['code'];
                }
            }
            else
                $code = "101280101";
    
            $url = "http://www.weather.com.cn/data/sk/".$code.".html";
            $json =file_get_contents($url);
            $obj = json_decode($json,true);
            $str = $obj['weatherinfo']['city']."实时天气:".$obj['weatherinfo']['time']."\n";
            $str .=$obj['weatherinfo']['temp']."度,";
            $str .=$obj['weatherinfo']['WD'].$obj['weatherinfo']['WS'].",湿度:".$obj['weatherinfo']['SD']."\n";
            $str .= "================\n";
    
            $url ="http://m.weather.com.cn/data/".$code.".html";
            $json =file_get_contents($url);
            $obj = json_decode($json,true);
    
            $str .="5日内:\n今天:";
            $str .= $obj['weatherinfo']['temp1'];
            $str .= $obj['weatherinfo']['weather1']."\n明天:";
            $str .= $obj['weatherinfo']['temp2'];
            $str .= $obj['weatherinfo']['weather2']."\n后天:";
            $str .= $obj['weatherinfo']['temp3'];
            $str .= $obj['weatherinfo']['weather3']."\n大后天:";
            $str .= $obj['weatherinfo']['temp4'];
            $str .= $obj['weatherinfo']['weather4']."\n大大后天：";
            $str .= $obj['weatherinfo']['temp5'];
            $str .= $obj['weatherinfo']['weather5']."\n";
            $str .= $obj['weatherinfo']['index_d'];
    
            return $str;
    
        }




### php中处理json的具体方法以及注意事项：


php中使用`json_encode()`来对 JSON 格式的字符串进行编码，使用`json_decode()`来对变量进行 JSON 编码， 具体的使用说明可以参考[php手册](http://cn2.php.net/manual/zh/book.json.php)中 的相关描述，这里就不再复制过来。需要注意的是，通常情况下，`json_decode()` 总是返回一个PHP对象，而不是数组。如果想要强制生成PHP关联数组，`json_decode()`需要加一个参数true,比如上面给出的函数中所用的

    
    $obj = json_decode($json,true);


另外，[JSON](http://www.json.org/json-zh.html)对格式已经有了严格的规定， 不可随便换用其他分隔符，比如省略双引号，或者是哟给你单引号代替等，都是不允许的。


### 参考


  * [在PHP语言中使用JSON-ruangyifeng](http://www.ruanyifeng.com/blog/2011/01/json_in_php.html)

	
  * [开源免费天气预报接口API以及全国所有地区代码](http://mobile.riaos.com/?p=2006952)


### 题外话

这篇文章是在sublime-text 2 下用markdown写的，感觉markdown写文章还是非常顺手的，相比之下wordpress的编辑器实在是不堪入目。差不多可以考虑逃离wordpress了。
