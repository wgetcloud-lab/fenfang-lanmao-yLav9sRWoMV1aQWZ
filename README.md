
# XPath解析



> XPath(XML Path Language)是一种用于在XML和HTML文档中查找信息的语言,其通过路径表达式来定位节点,属性和文本内容,并支持复杂查询条件,XPath 是许多 Web 抓取工具如 `Scrapy,Selenium` 等的核心技术之一


# XPath 解析的基本步骤


1. **导入lxml.etree**



```
from lxml import etree

```
2. 使用etree.parse(filename, parser\=None)函数返回一个树形结构


	* `etree.parse()`用于解析本地XML或HTML文件,并将其转换为一个树形结构即`ElementTree`对象,可以通过该对象访问文档的各个节点
	* `filename`:要解析的文件路径
	* `parser`(可选):默认情况下,parser()会根据文件扩展名自动选择合适的解析器,如`.xml`文件使用XML解析器,.html使用HTML解析器
3. 使用etree.HTML(html\_string, parser\=None)解析网络html字符串


	* `html_string` :要解析的HTML字符串
	* `parser`:(可选):默认情况下`etree.HTML()`使用`etree.HTMLparser()`进行解析
	* **返回值**:etree.HTML()返回一个`ELement`对象,表示HTML文档的**根元素**,可以通过该对象访问文档各个节点
4. 使用.xpath(xpath\_expression)在已经解析好的HTML文档中执行XPath查询



```
result = html_tree.xpath(xpath_expression)

```

	* `xpath_expression`:XPath表达式,用于在文档中查找节点,XPath表达式可以是绝对路径或相对路径,也可以包含谓词,函数和轴操作,主要的XPath语法下面会展开讲解
	* `html_tree`:可以是 `ElementTree` 对象(由 etree.parse() 返回)或 `Element` 对象(由 etree.HTML() 返回)



```
from lxml import etree

# 使用etree.parser()解析文件路径
parser = etree.HTMLParser(encoding='utf-8')  # 以utf8进行编码
tree = etree.parse('../Learning02/三国演义.html', parser=parser)
print(tree)
#output-> 

# 使用etree.HTML()解析本地文件或网络动态HTML
# 读取文件 解析为字符串
file = open('../Learning02/三国演义.html', 'r', encoding='utf-8')
data = file.read()
root = etree.HTML(data)
print(root)

#整合
root = etree.HTML(open('../Learning02/三国演义.html', 'r', encoding='utf-8').read())
print(root)
#output-> 

```

# XPath语法



> `XPath`语法可以用于在XML与HTML文档中查找信息的语言


## 路径表达式



> XPath使用路径表达式来定位文档中的节点,路径也可以分为绝对路径与相对路径


### 绝对路径


* `/`:表示从根节点开始选择,其用于定义一个绝对路径


从根节点html开始查找到head,再从head下找出title标签



```
root = etree.HTML(open('../Learning02/三国演义.html', 'r', encoding='utf-8').read())
all_titles = root.xpath('/html/head/title')
for title in all_titles:
    print(etree.tostring(title, encoding='utf-8').decode('utf-8'))
#output-> 《三国演义》全集在线阅读_史书典籍_诗词名句网

```

### 相对路径



> 相比与绝对路径,相对路径使用率更好,更好用


* `//`:表示从**当前节点开始,**选择文档中**所有符合条件的节点,**并且不考虑他们的位置



```
root = etree.HTML(open('../Learning02/三国演义.html', 'r', encoding='utf-8').read())
all_a = root.xpath('//a')
for a in all_a:
    print(a.text)
#None
#首页
#分类
#作者
#...

```

### 当前节点


* `./`:表示当前节点,通常用于指明当前节点本身,避免混淆



```
all_a = root.xpath('//a')
print(all_a[1].xpath('./text()')) #./表示当前的a标签
#output-> ['首页']

```

### 选择属性


* `@`:用于选择元素的属性,而不是元素本身



```
# 使用 @ 选择 标签的 href 属性
all_hrefs = root.xpath('//a[@href]')
for hrefs in all_hrefs:
    print(etree.tostring(hrefs, encoding='unicode'))

```

## XPath谓语



> 谓语是`xpath`中用于进一步筛选节点的表达式,通常放在方括号`[]` 内,其可以基于节点的位置,属性值,文本内容或其他条件来**选择特定的节点,谓语可以嵌套使用,也可以与其他谓语组合使用**


* **基本语法**



```
//element[condition]

```

	+ `element`:要选择的元素
	+ `condition`:谓语中的条件,用于进一步筛选符合条件的元素


### 位置谓语



> 位置谓语用于根据节点在兄弟节点中的位置进行选择,可以使用`position()`或直接指定位置编号


* 获取第一个`ul`标签中的第一个`li`标签



```
#//ul获取的是所有ul,[0]选择第一个
lis = root.xpath('//ul')[0].xpath('./li[1]')
for li in lis:
    print(etree.tostring(li, encoding='unicode'))
#output-> * [首页](https://github.com)


```
* 使用`last()`获取最后第一个节点,和导数第二个节点



```
# 倒一个
last_li = root.xpath('//ul')[0].xpath('./li[last()]')
print(etree.tostring(last_li[0], encoding='unicode'))
# 倒二个
last_second_li = root.xpath('//ul')[0].xpath('./li[last()-1]')
print(etree.tostring(last_second_li[0], encoding='unicode'))
#output-> * [安卓下载](https://github.com)

#* [古籍](https://github.com)


```
* 使用`position()`获取位置进行筛选



```
# 获取前两个li标签
last_li = root.xpath('//ul')[0].xpath('./li[position()<3]')
for li in last_li:
    print(etree.tostring(li, encoding='unicode'))
# 获取偶数位标签
lis = root.xpath('//ul')[0].xpath('./li[position() mod 2=0]')
for li in lis:
    print(etree.tostring(li, encoding='unicode'))

```
* 属性谓语



> 属性谓语用于**选择具体特定属性的节点**


	+ 使用`@attribute`来获取属性名称,结合条件进行筛选
```
# 选取所有具有 href 属性的 a 元素
hrefs = root.xpath("//a[@href]")
for href in hrefs:
    print(etree.tostring(href, encoding='unicode'))

```

	+ 查找`class`属性值
```
all_class = root.xpath('//@class')
print(all_class)

```
* 组合谓语



> 将多个条件组合在一起,使用逻辑运算符 `and,or`等来创建更复杂的谓语



```
#选取href属性值为https://example.com且class属性值为link的a元素
//a[@href='https://example.com' and @class='link']

```


```
#选取href属性值为https://example.com或https://another.com的a 元素
//a[@href='https://example.com' or @href='https://another.com']

```
* 函数谓语



> Xpath提供了许多内置函数,来应对更复杂的筛选条件


	+ `contains((string1, string2)`函数:
	
	
		- `string1`:要搜索的字符串
		- `string2`:要查找的字符串
	```
	# 选取class包含"book"的img标签
	images = root.xpath('//img[contains(@src,"book")]')
	for image in images:
	    print(etree.tostring(image, encoding='unicode'))
	
	```
	+ `starts-with(string1, string2)`函数:
	
	
	
	> 检查一个字符串是否以指定字符的前缀开始,是返回`true`,否返回`false`
	
	
		- `string1:`要检查的字符串
		- `string2:`作为前缀的字符串
	```
	# 选取所有href以https://开头的a标签
	all_a = root.xpath('//a[starts-with(@href,"https:")]')
	for a in all_a:
	    print(etree.tostring(a, encoding='unicode'))
	
	```
* 文本内容谓语



> 用于选择包含特定文本内容的节点,可以使用`text()`函数来提取节点的文本内容



```
# 选择使用包含"三国"文本的p标签
paragraphs = root.xpath('//p[contains(text(),"三国")]')
for p in paragraphs:
    print(etree.tostring(p, encoding='unicode'))

```


## 通配符



> xpath提供了多种通配符,用于在路径表达式中匹配未知的元素,属性,或任何节点.这些通配符非常有用,尤其是当不确定具体节点名称和结构的情况下




| **通配符** | **描述** |
| --- | --- |
| \* | 匹配任何元素节点。 一般用于浏览器copy xpath会出现 |
| @\* | 匹配任何属性节点。 |
| node() | 匹配任何类型的节点。 |


### 使用`*`匹配任何元素节点


* `*` 是最常用的通配符之一,其可以匹配任何元素,而不需要具体标签名.这在不确定元素名称或希望选择所有类型的元素时非常有用



```
# 选择所有 div 下的所有子元素
divs = root.xpath("//div/*")
for div in divs:
    print(etree.tostring(div, encoding='unicode'))

```

### 使用`@*` 匹配任何属性节点


* `@*` 用于匹配任何属性节点,而不用指定具体属性名称,在你不确定属性名称或希望选择所有属性时非常有用



```
# 选择所有 a 元素的所有属性
all_a = root.xpath('//a/@*')
for a in all_a:
    print(a)

```

### 使用`node()`匹配任何类型的节点


* `node()` 是一个更通用的通配符,其能匹配任何类型节点,包括元素节点,文本节点,属性节点,注释节点等等,其在需要选择不仅仅是元素节点是十分有用



```
# 选择所有 ul 下的所有子节点(包括文本节点)
nodes = root.xpath('//ul/node()')
print(nodes)
#output-> ['\n ', , '\n,...] 

```

# XPath,re正则,BeautifulSoup对比



> 在之前的学习中我们首先学习了re正则表达式,其次学习了更加便捷的bs4,哪为何还要学习XPath解析呢,接下来我们将它们的优点和适用场景进行对比学习




| **工具** | **优点** | **缺点** | **适用场景** |
| --- | --- | --- | --- |
| **`XPath`** | **强大的路径表达能力，支持层级结构和条件查询** | **学习曲线较陡，对不规范 HTML 容错性较差** | **结构化良好的 XML/HTML，复杂查询** |
| **`re`** | **灵活性高，适合处理纯文本中的模式匹配** | **不适合解析 HTML/XML，可读性差** | **从纯文本中提取特定模式的数据** |
| **`BeautifulSoup`** | **易于使用，容错性强，适合初学者** | **性能稍低，功能有限** | **不规范的 HTML，简单数据提取，网页抓取** |


* **总结**
	+ **若需要处理结构良好的XML或HTML文档,并需要进行复杂查询**,那么XPath解析是最佳选择
	+ **若需要从纯文本中提取特定模式的数据时**,如从日志中提取日期,IP地址的,re正则表达式是最佳选择
	+ **需要解析不规范的 HTML 或者只需要进行简单的数据提取,**BeautifulSoup 是最友好的选择


  * [XPath解析](#xpath%E8%A7%A3%E6%9E%90)
* [XPath 解析的基本步骤](#xpath-%E8%A7%A3%E6%9E%90%E7%9A%84%E5%9F%BA%E6%9C%AC%E6%AD%A5%E9%AA%A4)
* [XPath语法](#xpath%E8%AF%AD%E6%B3%95)
* [路径表达式](#%E8%B7%AF%E5%BE%84%E8%A1%A8%E8%BE%BE%E5%BC%8F)
* [绝对路径](#%E7%BB%9D%E5%AF%B9%E8%B7%AF%E5%BE%84)
* [相对路径](#%E7%9B%B8%E5%AF%B9%E8%B7%AF%E5%BE%84)
* [当前节点](#%E5%BD%93%E5%89%8D%E8%8A%82%E7%82%B9)
* [选择属性](#%E9%80%89%E6%8B%A9%E5%B1%9E%E6%80%A7)
* [XPath谓语](#xpath%E8%B0%93%E8%AF%AD):[悠兔机场官网订阅](https://5tutu.com)
* [位置谓语](#%E4%BD%8D%E7%BD%AE%E8%B0%93%E8%AF%AD)
* [通配符](#%E9%80%9A%E9%85%8D%E7%AC%A6)
* [使用\*匹配任何元素节点](#%E4%BD%BF%E7%94%A8%E5%8C%B9%E9%85%8D%E4%BB%BB%E4%BD%95%E5%85%83%E7%B4%A0%E8%8A%82%E7%82%B9)
* [使用@\* 匹配任何属性节点](#%E4%BD%BF%E7%94%A8-%E5%8C%B9%E9%85%8D%E4%BB%BB%E4%BD%95%E5%B1%9E%E6%80%A7%E8%8A%82%E7%82%B9)
* [使用node()匹配任何类型的节点](#%E4%BD%BF%E7%94%A8node%E5%8C%B9%E9%85%8D%E4%BB%BB%E4%BD%95%E7%B1%BB%E5%9E%8B%E7%9A%84%E8%8A%82%E7%82%B9)
* [XPath,re正则,BeautifulSoup对比](#xpathre%E6%AD%A3%E5%88%99beautifulsoup%E5%AF%B9%E6%AF%94)

   \_\_EOF\_\_

   ![](https://github.com/ihave2carryon)ihave2carryon  - **本文链接：** [https://github.com/ihave2carryon/p/18585329](https://github.com)
 - **关于博主：** 评论和私信会在第一时间回复。或者[直接私信](https://github.com)我。
 - **版权声明：** 本博客所有文章除特别声明外，均采用 [BY\-NC\-SA](https://github.com "BY-NC-SA") 许可协议。转载请注明出处！
 - **声援博主：** 如果您觉得文章对您有帮助，可以点击文章右下角**【[推荐](javascript:void(0);)】**一下。
     
