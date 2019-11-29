---
title: Python3.4 asyncio 实战爬取网站 (一) 爬取思路
date: 2017-05-13 12:45:01
categories:
- 爬虫
tags:
- python
- 爬虫
- 抓取
---

## 目标
运营需要抓取的国外网站为 Finnishdesignshop 站内所有商品
我的想法是: 从分类列表抓取,具体的商品列表页 
再从商品列表页分页抓取到,抓取 finnishdesignshop.com的商品明细 product detail 存入JSON

## 流程

### 1  抓取具体的分类列表
网站首页最下部有一排,导航分页链接,注意头部的高亮的文字是总分类 ,这样就有了一级分类名.
二级分类就是那些非高亮的链接,其实点开一级分类名(那些高亮的链接)就是这些二级分类了,所以为了简单起见,
直接抓取这里的非高亮链接就可以了.

[finnishdesignshop.com](http://www.finnishdesignshop.com)

![](https://midu.studio515.cn/2017/2017-06-13-20170613-py-fetch-01.png)
### 2  抓取具体分类里的产品列表 

[举例某个具体分类地址](http://www.finnishdesignshop.com/furniture-hay-can-c-899_1496.html)

![](https://midu.studio515.cn/2017/2017-06-13-20170613-py-fetch-02-1.png)


文字很长的那行,可以看做是具体分类的具体描述吧,
所以分类描述和图片就拿到了,可用作具体分类的数据.
### 3 抓取具体的商品详情页面(表头)
如上图,剩下的每个项可以看做是具体商品详情的缩减版了.在这里抓取的每个项的url就是商品详情页面了.可以看做是商品数据的表头吧....
    
### 4 抓取具体的商品详情页面(表身)

[举例某个具体商品详情地址](https://www.finnishdesignshop.com/furniture-hay-can-can-1seater-blackarmy-frame-surface-120-p-11802.html)
![](https://midu.studio515.cn/2017/2017-06-13-20170613-py-fetch-03.png)

![20170613-py-fetch-04](https://midu.studio515.cn/2017-06-13-20170613-py-fetch-04.png)

就下来就是抓取 商品主图,商品具体图n张,商品库存,商品价格,商品名,商品url,商品送货日期,商品订单日,快递费用,商品描述,制造商,设计者.
### FAQ:
#### sku的问题
url可以获得到具体的商品id,注意该网站一个url算作一个sku.见下图,所以之后还需要将类似标题的商品作为一个产品.可以通过关键词筛选.(例如Can 1-seater, black-army frame, 2个逗号之前的算作一起的)
![](https://midu.studio515.cn/2017/2017-06-13-20170613-py-fetch-05.png)


