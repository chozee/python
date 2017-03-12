# Python 编码问题

[原文链接](http://blog.csdn.net/haluoluo211/article/details/45055355)

[参考补充](https://github.com/AIMinder/DeepLearning101/wiki/ResPythonEncodeDecode)

## 基本编码知识
    任何字符串都是一串二进制字节的序列。

1. ASCII
    每个字节表示为一个字符，能表示阿拉伯数字、字母共128个字符。很明显汉字无法表示
2. GBK(GB2312的扩展)
    中国人制定的，兼容ASCII的不定长编码，对ASCII码部分用一个字节。对汉字用两个字节
3. UTF-8
    兼容ASCII不定长编码。长度更大能兼容世界所有文字。对ASCII码用一个字节，其余用两个字节
4. Unicode
    定长编码方式。不过它把2字节认为是一个字符，可以映射所有的文字。Unicode字符串体积很大，一般来说Unicode编码只是文字在内存中的内在形式，具体的存储（注入文件、网页）都需要靠外在编码（UTF-8,GBK)诠释。也就理解为Unicode用于内存中表示字符的形式，其他的是具体存储编码和解码的协议。


## Python字符串的本质
    Python有两种字符串，分别是str类型和unicode类型，都是basestring的派生。str本质是一坨二进制串，源文件编码是怎么样，它就跟着是怎样。也就是说Python在指定编码之前不知道str字符串到底是什么编码，它的形式就是一坨二进制，当你pythhon文件开头标定该文件的编码是什么它就按照什么去解码。因此如果你len(str)它返回的是字节数，因为无法确定编码所以只能告诉你字节数，无法告诉你字符数。如果你对一个Unicode字符串len(u_str)它返回的就是字符数。
    |字符串类型|常量子串|内存中表示|len()|len含义|
    |:--------|:--------|:-----------|:--------|:--------|
    |str|S="呵呵"|源文件一致，一坨二进制|若源文件UTF-8则len(S)=6|字节数|
    |unicode|S=u"呵呵"|Unicode编码|len(S)=2|字数|

        在计算机内存中，统一使用Unicode编码；当需要保存到硬盘，或者在网络上传输时，就转为UTF-8编码记事本编辑时，从文件读取的UTF-8字符被转为Unicode字符到内存里；编辑完成保存时，把Unicode转换为UTF-8保存浏览网页时，服务器会把生成的 Unicode 转换为 UTF-8 再传输导浏览器，所以很多网页是用 UTF-8 编码的

## encode() decode()
    他们转换的本质是Unicode和str互相转换。encode将Unicode转换为str，并使用encoding编码。decode将str转换为Unicode，其中str是以coding编码。
    <pre><code>
        #encoding: utf-8
        s = "你好" # 整个文件是UTF-8编码，所以这里的字符串也是UTF-8
        u = s.decode("utf-8") # 将utf-8的str转换为unicode
        g = u.encode('GBK') # 将unicode转换为str，编码为GBK
        print type(s), "len=", len(s)
        # 输出： len= 6，utf-8每个汉字占3字节
        print type(u), "len=", len(u)
        # 输出： len= 6，unicode统计的是字数
        print type(g), "len=", len(g)
        # 输出：g = u.encode('GBK')，GBK每个汉字占2字节
        print s
        # 在GBK/ANSI环境下（如Windows），输出乱码，
        #因为此时屏幕输出会被强制理解为GBK；Linux下显示正常
        print g
        # 在Windows下输出“你好”，
        #Linux(UTF-8环境)下报错，原因同上。
    </pre></code>

1. 循环处理字符串时需要解码
<pre><code>
#-*- coding: utf-8 -*-

token = "，.!?？：:"

# 不解码会乱码
for t in token:
    print t
# 解码
for t in token.decode('utf-8'):
    print t
</code></pre>
2. 字符匹配的时候需要解码
3. jieba分词之后变为Unicode编码

### 判断字符串类型
<pre><code>isinstance(s, basestring)</code></pre>

### 文当读取
1. txt文档
    txt默认采用ascii码，另存为需要指定其他编码才行。
2. 读取非ascii码文件
    如果读取非 ASCII 编码的文本文件，可以用二进制模式打开，再解码；或者给 open 函数传入 encoing 参数

<pre><code>
    # GBK 编码的文件
    f = open('./xx.txt', 'rb') # 二进制读取
    u = f.read().decode('gbk')

    ---

    f = open('/./xx.txt', 'r', encoding='gbk')
    f.read()
</code></pre>
遇到编码不规范的文件，可以给 open 函数一个 errors 的参数
<pre><code>
    f = open('/./xx.txt', 'r', encoding='gbk', errors='ignore') # 忽略错误编码
</code></pre>

3. codecs
python 的 codecs 模块可以帮助我们在读取文件时自动转换编码，直接读出 Unicode
<pre><code>
    # GBK 编码的文件
    import codecs
    with codecs.open('/./xx.txt', 'r', 'gbk') as f:
        f.read()
</code></pre>

4. JSON
    我们可以将一些需要调取的 dict 存储为 JSON。
    另外一个原因，JSON 标准规定编码是 UTF-8，所以 Python 内置的 json 模块让我们很方便地在 Python 的 str 与 JSON 字符串之间转换

<PRE><CODE>
import json
d = dict(name='Bob', age=20, score=88)
json.dumps(d) # dumps 返回一个 str，内容就是标准的 JSON
'{"age": 20, "score": 88, "name": "Bob"}'
json_str = '{"age": 20, "score": 88, "name": "Bob"}'
json.loads(json_str) # 成功读取
{u'age': 20, u'name': u'Bob', u'score': 88}
</CODE></PRE>
实际操作中，可以直接将 json.dumps(d) 写入 txt 文档，读取时 json.loads(f.read()) 即可

5. 直接存储 Unicode
    不将 Unicode 转为实际的文本存储字符集，而是将 Unicode 的内存编码值直接存储，读取文件时再反向转换回来
<PRE><CODE># 存储
# 从 Unicode 到 Unicode-escape
# unicodestr.encode('unicode-escape')
u'中文'.encode('unicode-escape')
'\\u4e2d\\u6587'
# 读取
# 从 Unicode-escape 到 Unicode
# unicodestr.decode('unicode-escape')
'\\u4e2d\\u6587'.decode('unicode-escape')
u'\u4e2d\u6587'
print(u'\u4e2d\u6587')</CODE></PRE>

6. 直接存储 String
<PRE><CODE>
    对于 UTF-8 编码的字符串，在存储时也可以直接存储编码值

# 存储
# 从 UTF-8 到 string-escape
# utf8str.encode('string-escape')
'中文'.encode('string-escape')# 注意 '中文' 是 UTF-8
'\\xe4\\xb8\\xad\\xe6\\x96\\x87'
# 读取
# 从 string-escape 到 UTF-8
# utf8str.decode('string-escape')
'\\xe4\\xb8\\xad\\xe6\\x96\\x87'.decode('string-escape')# 注意 string '中文' 是 UTF-8
'\xe4\xb8\xad\xe6\x96\x87'
</CODE></PRE>
### 总结

1. unicode是支持所有文字的统一编码，但一般只用作文字的内部表示，文件、网页（也是文件）、屏幕输入输出等处均需使用具体的外在编码，如GBK、UTF-8等；
2. encode和decode都是针对unicode进行“编码”和“解码”，所以encode是unicode->str的过程，decode是str->unicode的过程；
3. unicode和str是一对孪生兄弟，来自basestring，所以用isinstance(s, basestring)来判断s是否为字符串。
