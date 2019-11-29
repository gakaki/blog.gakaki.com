---
title: Python3.4 asyncio 实战爬取网站 (二) 无废话实战代码解说
date: 2017-05-13 15:45:01
categories:
- 爬虫
tags:
- python
- 爬虫
- 抓取
---
## 方案

使用Python3.6 asyncio+uvloop+beautifulsoap

推荐安装
 [jetbrain toolbox](https://www.jetbrains.com/toolbox/) -> 再安装 pycharm 
 [python官网](https://www.python.org/downloads/)->py 3系最新版下载
 
[先上代码](https://coding.net/u/gakaki/p/finnishdesignshop_py/git)

Python3.4 asyncio 直接利用了linux的epoll, [有调查显示](http://www.dongwm.com/archives/%E4%BD%BF%E7%94%A8Python%E8%BF%9B%E8%A1%8C%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B-asyncio%E7%AF%87/)比多线程的py还要更快,最大的特点是使用async语法,
可以将java中的callback语法变成同步代码的顺序写法,
符合程序员的思维,并且又保证了异步的高性能和非阻塞.
  
uvloop是基于nodejs v8 engine的libuv库的python桥接调用.
beautifulsoap是py家的html解析库了,支持css3选择符,使用率太高.

运行效果:
    ![20170613-py-fetch-06](/media/20170613-py-fetch-06.png)

### 代码解说
    final_res = []          //存放最后导出的json数据
    final_res_excel = []    //存放最后导出的excel数据
    sema = asyncio.Semaphore(10)
    //类似java中的信号量 Semaphore 可以控制同时请求的http数量,这里设置为同时请求10个,不容很容被要爬的网站封杀哦
    
    // 异步的抓取 url 并解析 并返回单个商品的数据
    async def fetch_async(url): 
        async with aiohttp.ClientSession() as session: //创建client session
            async with session.get(url) as resp: //获取url数据
                print(resp.status)          // http response的状态 这里其实应该判断下http状态
                data = await resp.text()    //异步 获取其中的数据
                p    = Product()            
                p.parse(data)               //product类解析 并获得product数据
                print(p.__dict__)       
                return p
    async def print_result(url):        //主要的抓取类
        with (await sema):              //调用上面的信号量限制
            r = await fetch_async(url)      //抓取单个商品数据
            print('fetch({}) = {}'.format(url, r)) //拼接url
            final_res.append(r.__dict__)        //
            r_excel = copy.deepcopy(r)
            r_excel.deal_for_excel()
            final_res_excel.append(r_excel.__dict__)
    //asyncio 使用 uvloop 库启动
    loop = asyncio.get_event_loop()
    asyncio.set_event_loop(loop)
    //抓取的商品具体列表
    URLS = [
        "https://www.finnishdesignshop.com/outdoor-outdoor-furniture-mira-armchair-hunter-green-p-14127.html"
        , "https://www.finnishdesignshop.com/-mira-chair-anthracite-black-p-14126.html"
        , "https://www.finnishdesignshop.com/outdoor-outdoor-furniture-mira-chair-hunter-green,
        .....    //假设这里有很多的商品详情页链接
    ]
    
    // 这里的asyncio wait 会将所有操作事项推入到 event loop中 ,并且是异步单线程运行的
    // 具体的每个异步的执行函数为 print_result 函数
    f = asyncio.wait([print_result(num) for num in URLS])
    loop.run_until_complete(f) //等待所有的异步 搞定
    loop.close()        //关闭时间循环
    write_to_json(final_res)    //所有数据写入json
    write_to_xls(final_res_excel) //所有数据写入excel xls文件
    print(final_res) //打印结果


