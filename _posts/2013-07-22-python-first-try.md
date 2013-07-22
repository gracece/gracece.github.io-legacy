---
layout: post
title: "python练手:高考录取结果抓取"
description: "最近越来越觉得很多时候“用以致学”比“学以致用”要实在一点。在六月份的时候，自己已经把`python简明教程`看了一遍，知道了个python的大概。
真的只是“看了一遍”依然不知道具体怎么用python。需求才是第一生产力，这不，需求来了，自然就知道该怎么办。 "
category: python
tags: 
- python 
---
{% include JB/setup %}

最近越来越觉得很多时候“用以致学”比“学以致用”要实在一点。在六月份的时候，自己已经把`python简明教程`看了一遍，知道了个python的大概。
真的只是“看了一遍”依然不知道具体怎么用python。需求才是第一生产力，这不，需求来了，自然就知道该怎么办。

### 需求
高考录取结果已经出了好几天了，而我也迫切地想知道今年高中母校有多少师弟师妹考来SYSU，但现在毫无消息，除了群里面几个师弟师妹自报录取
到了，其他的没出声的以及没有加群的情况未名。所以得想想办法看怎么搞到录取情况。

### 分析
去年是在`mzedu.com`上抓取的，只需要提供考生号就能查到录取结果，而且还没有验证码，实在是太和谐了！（那时候我连php都还不会，是炳炜大
神帮忙抓下来的）。不过今年，`mzedu.com`到现在都还没有动静，只留有2012年的查询入口。另寻他路，先看看恶心的`5184.com`，需要提供考生号
以及出生年月，而且有验证码! 所以先放着吧。继续搜索，发现一个叫`爱高分`的网站可以查录取结果，需要提供姓名、考生号、出生年月、
手机号（这个根本就是扯淡），最关键的，没有验证码！没有搜索到其他可用网站，那就搞这个吧。

手里有高考成绩表，里面包含有考生的姓名、考生号以及分数，没有出生年月信息，无奈，那只能给出个范围(93.03-96.06)暴力解决。
python脚本读取考生数据csv文件并暴力遍历出生年月，post到`http://120.197.89.132:8800/myExamWeb/wap/school/gaokao/loveHS!dispatcher.action`
，再分析响应的结果判断是否得到正确的信息再决定是否写入到文件中保存起来。

### 关键实现
在python 强大的 `urllib` 以及 `urllib2` 面前，这些需求实现起来都算是比较轻松，定义一个post函数：

    def post(url,data):
    """for score post """
    req = urllib2.Request(url)
    data = urllib.urlencode(data)
    opener = urllib2.build_opener()
    response = opener.open(req,data)
    lnum = 0
    result =""
    for line in response:
        #分析处理line
    return result

就是这么几行，就搞定了最关键的post数据以及分析响应内容。所以不得不感叹python的强大，其实一开始还想用php的curl来写的，我还真的写了...
但是php的curl要依赖于apache才能运行，所以还是算了。写多了python再回去写php其实挺纠结的，尤其是
要写`$`符号。

### 完整代码
    
    #!/usr/bin/python
    # -*- coding:utf-8 -*-

    import urllib
    import urllib2
    import csv

    def post(url,data):
        """for score post """
        req = urllib2.Request(url)
        data = urllib.urlencode(data)
        opener = urllib2.build_opener()
        response = opener.open(req,data)
        lnum = 0
        result =""
        for line in response:
            lnum += 1
            if lnum == 81:
                if(line.find('未能查到') != -1):
                    return "\nnot found!"
            if(lnum >= 80) and (lnum <=83):
                result +=line
        return result

    def main():
        """main"""
        postUrl ="http://120.197.89.132:8800/myExamWeb/wap/school/gaokao/loveHS!dispatcher.action"
        mobileNo = "13800138000"
        birthdayRange = ['9310','9311','9312','9401','9402','9403','9404','9405','9406',\
                '9407','9408','9409','9410','9411','9412','9501','9502','9503','9504','9505',\
                '9506','9507','9508','9509','9510','9511','9512','9601','9602','9603'] 
        reader = csv.reader(open("Book1.csv"))
        for examReferenceNo,userName in reader:
            print examReferenceNo
            print userName
            open("result.txt",'a+').write("\n"+examReferenceNo+userName+"\n")
            for i in range(len(birthdayRange)):
                print birthdayRange[i]
                birthday = birthdayRange[i]
                data = {'id':'14356B43-7F8B-4B00-BB13-F95841F33C46','userName':userName,'examReferenceNo':examReferenceNo,\
                        'birthday':birthday,'mobileNo':mobileNo}
                response = post(postUrl,data)
                print response
                open("result.txt",'a+').write(response)
                open("result.txt",'a+').write('<p>' + birthday + '</p>\n')

        
    if __name__ == '__main__':
        main()

### 不足

 * 太过暴力，没有对频率进行控制。其实脚本抓取，还是得讲究个度的

 * 效率不高，受网络情况影响大，这里可能与对方服务器的响应速度有关，在本地测试（电信4M网络），第一个考生都无法正常抓取完， 放到vps上则可以连续爬取。

 * 承接上一点，没有对超时进行控制，也没有写失败时的重试机制。

### 总结
写完这个脚本，算是入了python的门了吧，life is short ，use python ！
