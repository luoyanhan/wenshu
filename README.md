# 中国裁判文书网爬虫
如果爬虫失败请多试几次，这个网站连人工操作都很难加载出来，经常400或502或者显示空白页面

## 分析 POST 请求
首先打开控制台正常登录一次，可以很快找到登录的 API 接口，这个就是模拟登录 POST 的链接。

![Image text](https://github.com/luoyanhan/wenshu/blob/master/image/%E6%90%9C%E7%8B%97%E6%88%AA%E5%9B%BE19%E5%B9%B402%E6%9C%8828%E6%97%A51307_1.png)

![Image text](https://github.com/luoyanhan/wenshu/blob/master/image/%E6%90%9C%E7%8B%97%E6%88%AA%E5%9B%BE19%E5%B9%B402%E6%9C%8828%E6%97%A51308_2.png)

![Image text](https://github.com/luoyanhan/wenshu/blob/master/image/%E6%90%9C%E7%8B%97%E6%88%AA%E5%9B%BE19%E5%B9%B402%E6%9C%8828%E6%97%A51308_3.png)

## 构建 formdata
 从上面看出来有三个参数需要构造，分别是guid, number, vl5x
 
 ### 构建guid
 ![Image text](https://github.com/luoyanhan/wenshu/blob/master/image/%E6%90%9C%E7%8B%97%E6%88%AA%E5%9B%BE19%E5%B9%B402%E6%9C%8828%E6%97%A51321_4.png)
 按ctrl+F全局搜索guid,找到上面显示的代码，改写成python即可
 
 ### 构建number
  ![Image text](https://github.com/luoyanhan/wenshu/blob/master/image/%E6%90%9C%E7%8B%97%E6%88%AA%E5%9B%BE19%E5%B9%B402%E6%9C%8828%E6%97%A51330_5.png)
 携带guid向上面的接口post即返回8位number，需要用到前面4位，但是经测试发现最后每次post请求的时候可以用同一个number，一样有列表返回。
 
 ### 构建vl5x
 ![Image text](https://github.com/luoyanhan/wenshu/blob/master/image/%E6%90%9C%E7%8B%97%E6%88%AA%E5%9B%BE19%E5%B9%B402%E6%9C%8828%E6%97%A51334_6.png)
 
  ![Image text](https://github.com/luoyanhan/wenshu/blob/master/image/%E6%90%9C%E7%8B%97%E6%88%AA%E5%9B%BE19%E5%B9%B402%E6%9C%8828%E6%97%A51334_7.png)
 
 
又是全局搜索vl5x发现是用一个getkey函数生成，全局搜索getkey,发现经过简单的混淆，在项目目录下新建一个js文件，将getkey上面的fxxx函数和de函数复制黏贴进去，然后写一个function，这个function的return内容就是上面截图每一行eval里面的内容。在py文件里面用execjs执行这个function,通过这种方式找到正确的js代码，如果生成的js代码还有eval嵌套就继续把eval里面的内容return出去。对每一行重复相同的操作，再将得到的js代码都复制进vl5x.js里面。观察得出的js代码，在最下面发现vlsx是需要用到cookie里面的vjkl5字段的。
再仔细阅读代码发现里面需要用到别的js文件里的函数,例如Base64()全局搜索找到位置
  ![Image text](https://github.com/luoyanhan/wenshu/blob/master/image/%E6%90%9C%E7%8B%97%E6%88%AA%E5%9B%BE19%E5%B9%B402%E6%9C%8828%E6%97%A51335_8.png)
  将这个目录下base64,sha1,md5都黏贴进去vl5x.js。
 

## 获取vjkl5(最难的部分)
以打开url http://wenshu.court.gov.cn/List/List?sorttype=1&conditions=searchWord+2+AJLX++%E6%A1%88%E4%BB%B6%E7%B1%BB%E5%9E%8B:%E6%B0%91%E4%BA%8B%E6%A1%88%E4%BB%B6
为例
 ![Image text](https://github.com/luoyanhan/wenshu/blob/master/image/%E6%90%9C%E7%8B%97%E6%88%AA%E5%9B%BE19%E5%B9%B402%E6%9C%8828%E6%97%A51405_9.png)
 
 可以看到有两个document文件请求，一个202一个200，我们要找的vjkl5字段在200那里，但是这个200请求是带cookie的，cookie字段分别是80T和80S结尾，我们对比202请求里面set的cookie发现80T字段改变了，又发现202和200请求之间有一个js文件，202请求里面本身又包含了混淆很严重的js代码，正常断点调试解密要花很长时间研究，所以这部分js我直接使用了开源的js代码。
 200请求包含结构<meta id="9DhefwqGPrzGxEp9hPaoag" content="大量混淆的js代码">content里面包含字符串映射表、全局方法映射表、加解密算法密钥等  <script type="text/javascript" src="200和202请求中间的js文件">
 将200请求和200和202请求中间的js文件下载下来搭建本地环境可以分析。
 
 成功解析出js代码后可以利用200请求的80T以旧换新获取新的80T，拿到新的80T之后用新的cookie请求网址就会拿到vjkl5,再用vjkl5去生成vl5x，集齐参数之后post可以获得一个列表，里面包含档案名称
 

