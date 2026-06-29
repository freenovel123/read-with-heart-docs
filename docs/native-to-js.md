# Native to Javascript
App暴露的Javascript方法，可以使用这些方法调用原生的组件！

!!! note "命名规范说明"
    文档中的方法支持多种命名格式，以下三种写法等价：
    
    - `App.xxx()` - 推荐使用
    - `app.xxx()` - 小写形式
    - `APP.xxx()` - 大写形式
    
    您可以根据个人习惯选择任意一种写法。

## Document
`v2.6.0 以上版本支持`

HTML解析和操作，轻松实现数据提取和转换，遵循WHATWG HTML5 规范。

=== "示例"

    ```javascript
    let html = "<html><head><title>Example</title></head>
        <body><p>Hello!</p><p>Hi!</p></body></html>";

    // 解析html
    let document = app.doc(html, false);

    // 获取标题
    let title = document.title();
    app.log(title);
    
    // 用 CSS 查询选择元素
    let list = document.select("p")
    list.forEach(element => {
        // 获取内容
        app.log(element.text());
    });
    ```
=== "API"

    | 参数 | 名称 | 类型 | 默认值 | 说明 |
    |:---|:---|:---|:---|:---|
    | html | 参数 | String | - | html内容 |
    | clean | 参数 | Boolean | - | 清理HTML |

## log
打印调试日志，等同于`console.log()`

=== "示例"

    ```javascript
    // 方法名称内置了三种写法，可以根据喜好选择
    App.log('你好!', 'hello!')
    app.log('你好!', 'hello!')
    APP.log('你好!', 'hello!')
    ```
=== "API"
    
    | 参数 | 名称 | 类型 | 默认值 | 说明 |
    |:---|:---|:---|:---|:---|
    | args | 参数 | Any | - | 支持多个参数 |
    
## toast

提示用于需要用户注意的信息。

=== "示例"

    ```javascript
    App.toast('这是一个提示！', 'info')
    ```
=== "API"

    | 参数 | 名称 | 类型 | 默认值 | 说明 |
    |:---|:---|:---|:---|:---|
    | info | 信息 | String | - | 需要提示的信息 |
    | type | 提示类型 | String | `info` | info/success/error |

## showDialog
在APP中打开一个对话框承载相应的信息及操作。
=== "示例"

    ```javascript linenums="1"
    const params = {
        title: '标题',
        body: '提示的内容',
        confirmBtn: '确认',
        cancelBtn: false // (1)!
    }

    App.showDialog(params, (bool) => {
        console.log(bool)
    })
    ```

    1. 取消按钮如果传入false,则会隐藏
=== "API"

    | 参数 | 名称 | 类型 | 默认值 | 说明 |
    |:---|:---|:---|:---|:---|
    | params | 参数 | Object | `{}` | 参数 |
    | - `title` | 标题名称 | String | `提示` | - |
    | - `body` | 提示内容 | String | - | - |
    | - `confirmBtn` | 确认按钮名称 | String | `确认` | - |
    | - `cancelBtn` | 取消按钮名称 | String/Boolean | `取消` | 如果传入false，则会隐藏该按钮. |
    | onCompletion | 回调方法 | function | - | 点击确认按钮返回true,取消按钮返回false |

## isJson

判断一个字符串是否为json格式。

=== "示例"

    ```javascript
    let bool = App.isJson('{"a":"b"}') // return true
    let bool = App.isJson('你好') // return false
    ```
=== "API"

    - 请求

    | 参数 | 名称 | 类型 | 默认值 | 说明 |
    |:---|:---|:---|:---|:---|
    | jsonString | 字符串 | String | - | 需要判断的字符串 |

    - 响应

    | 参数 | 名称 | 类型 | 说明 |
    |:---|:---|:---|:---|
    | bool | 状态 | Boolean | true/false |

## jsonToString

将一个json对象转换成一个json字符串

=== "示例"

    ```javascript
    let jsonObject = {
        a: '1',
        b: '2'
    }
    let jsonString = App.jsonToString(jsonObject)
    ```
=== "API"

    - 请求

    | 参数 | 名称 | 类型 | 默认值 | 说明 |
    |:---|:---|:---|:---|:---|
    | json | json | Object | - | - |

    - 响应

    | 参数 | 名称 | 类型 | 说明 |
    |:---|:---|:---|:---|
    | jsonString | - | string | 转换失败返回`null` |

## stringToJson

将一个json字符串转换成一个json对象

=== "示例"

    ```javascript
    let jsonString = '{"a":"1","b":"2"}'
    let jsonObject = App.stringToJson(jsonString)
    ```
=== "API"

    - 请求

    | 参数 | 名称 | 类型 | 默认值 | 说明 |
    |:---|:---|:---|:---|:---|
    | jsonString | json字符串 | String | - | - |

    - 响应

    | 参数 | 名称 | 类型 | 说明 |
    |:---|:---|:---|:---|
    | jsonObject | - | Object | 转换失败返回`null` |

## strToBytes
`v2.6.0 以上版本支持`

将字符串转化成字节序列

=== "示例"

    ```javascript
    let str = 'Hello!'
    let bytes = App.strToBytes(str)
    // [72,101,108,108,111,33]
    ```
=== "API"

    - 请求

    | 参数 | 名称 | 类型 | 默认值 | 说明 |
    |:---|:---|:---|:---|:---|
    | str | 字符串 | String | - | - |

    - 响应

    | 参数 | 名称 | 类型 | 说明 |
    |:---|:---|:---|:---|
    | bytes | - | Array | 转换失败返回`null` |

## bytesToStr
`v2.6.0 以上版本支持`

将字节序列转化成字符串

=== "示例"

    ```javascript
    let bytes = [72,101,108,108,111,33]
    let str = App.bytesToStr(bytes, 'utf-8')
    // Hello!
    ```
=== "API"

    - 请求

    | 参数 | 名称 | 类型 | 默认值 | 说明 |
    |:---|:---|:---|:---|:---|
    | bytes | 字节数列 | Array | - | - |
    | code | 编码类型 | String | utf-8 | utf-8、gbk |

    - 响应

    | 参数 | 名称 | 类型 | 说明 |
    |:---|:---|:---|:---|
    | str | - | String | 转换失败返回`null` |

## time

将一个UNIX时间戳转换为日期格式.

=== "示例"

    ```javascript
    let unix = '1732340412'
    let formatDate = App.time(unix, 'yyyy-MM-dd HH:mm:ss')
    App.log(formatDate) // 2024-11-23 13:40:12
    ```
=== "API"

    - 请求

    | 参数 | 名称 | 类型 | 默认值 | 说明 |
    |:---|:---|:---|:---|:---|
    | unix | 时间戳 | String/Number | - | - |

    - 响应

    | 参数 | 名称 | 类型 | 说明 |
    |:---|:---|:---|:---|
    | string | - | String | 转换失败返回`null` |

## post/get

进行一个HTTP的POST/GET请求，采用原生交互写法，不存在普通js的ajax跨域问题.

=== "示例"

    ```javascript linenums="1"
    // 请求地址
    let url = 'your url'

    // 请求参数
    let params = {
        type: 1
    }
    
    // header信息
    let header = {
        token: 'xxxx'    
    }
    
    // 开始请求，采用 async 方式，所以必须带修饰符 await
    let data = await App.post({
        url,
        params,
        header
    });
    
    // GET请求，params中的参数会自动和url拼接，即http://www.a.com/?type=1
    let data = await App.get({
        url,
        params,
        header
    });

    App.log(data)
    ```
=== "API"

    - 请求

    | 参数 | 名称 | 类型 | 默认值 | 说明 |
    |:---|:---|:---|:---|:---|
    | params | 参数 | Object | - | - |
    | - `url` | 请求地址 | String | - | - |
    | - `params` | post请求参数 | Object | `{}` | - |
    | - `header` | header信息 | Object | `{}` | - |

    - 响应

    | 参数 | 名称 | 类型 | 说明 |
    |:---|:---|:---|:---|
    | responseBody | - | String | 响应的内容 |
    | requestHeader | - | Object | 请求标头 |
    | responseHeader | - | Object | 响应标头 |
    | cookie | - | Object | Cookie |

## WebSocket

=== "示例"

    ```javascript linenums="1"
    const ws = app.socket('wss://xxxxx');
    
    // 连接socket成功
    ws.open(() => {
        // 发送消息
        ws.send('hello');
    })
    
    // 获取文本信息
    ws.text((text) => {
        console.log(text);

        // 消息接收完毕，主要用于自定义tts中
        ws.finished();
    })

    // 获取二进制数据
    ws.binary((data) => {
        console.log(data);
        
        // 消息推送给app，主要用于自定义tts中
        ws.push(data);
    })

    // 关闭连接
    ws.close();
    ```

=== "API"
    - 请求

    | 参数 | 名称 | 类型 | 默认值 | 说明 |
    |:---|:---|:---|:---|:---|
    | path | 地址 | String | - | - |
    | protocols | 协议 | Array | [] | - |

    - 方法

    | 参数 | 名称 | 类型 | 说明 |
    |:---|:---|:---|:---|
    | open(() => {}) | 连接成功 | - | - |
    | text((text) => {}) | 字符串消息回调 | String | - |
    | binary((data) => {}) | 二进制消息回调 | Uint8Array | - |
    | close(() => {}) | 关闭连接 | - | - |
    | push | 数据推送 | - | - |
    | finished | 数据接收完成推送 | - | - |

## 简易存储

将一个或多个参数本地化保存,获取和删除.

### put
持久化数据

=== "示例"

    ```javascript
    let bool = App.sp.put('name', 'zhangsan');
    ```
=== "API"

    - 请求

    | 参数 | 名称 | 类型 | 默认值 | 说明 |
    |:---|:---|:---|:---|:---|
    | key | 键 | String | - | - |
    | value | 值 | String | - | - |

    - 响应

    | 参数 | 名称 | 类型 | 说明 |
    |:---|:---|:---|:---|
    | bool | - | Boolean | 写入成功返回true |

### get
获取本地持久化的数据

=== "示例"

    ```javascript
    let data = App.sp.get('name');
    ```
=== "API"

    - 请求

    | 参数 | 名称 | 类型 | 默认值 | 说明 |
    |:---|:---|:---|:---|:---|
    | key | 键 | String | - | - |

    - 响应

    | 参数 | 名称 | 类型 | 说明 |
    |:---|:---|:---|:---|
    | data | - | String | 返回存入的数据 |

### delete
删除一个本地持久化的数据

=== "示例"

    ```javascript
    let bool = App.sp.delete('name');
    ```
=== "API"

    - 请求

    | 参数 | 名称 | 类型 | 默认值 | 说明 |
    |:---|:---|:---|:---|:---|
    | key | 键 | String | - | - |

    - 响应

    | 参数 | 名称 | 类型 | 说明 |
    |:---|:---|:---|:---|
    | bool | - | Boolean | 删除本地的数据 |

## Base64 编码解码

=== "示例"

    ```javascript
    let data = 'Hello, World!';
    
    // 编码
    let encodeData = App.base64.encode(data);
    App.log(encryptData); // SGVsbG8sIFdvcmxkIQ==
    
    // 解码
    let decodeData = App.base64.decode(encodeData);    
    App.log(encryptData); // Hello, World!

    // 解码成Bytes v2.6.0以上支持
    let decodeData = App.base64.decodeToBytes(encodeData);    
    App.log(encryptData); // [72, 101, 108, 108, 111, 44, 32, 87, 111, 114, 108, 100, 33]
    ```

=== "API"

    - 请求

    | 参数 | 名称 | 类型 | 默认值 | 说明 |
    |:---|:---|:---|:---|:---|
    | data | 编码解码内容 | String | - | - |

    - 响应

    | 参数 | 名称 | 类型 | 说明 |
    |:---|:---|:---|:---|
    | data | - | String | 编码解码内容 |

## GBK转UTF8转GBK
`v2.6.0 以上版本支持`

=== "示例"

    ```javascript
    let text = '你好！';
    
    // utf8转gbk
    let gbkText = App.string.toGBK('utf8字符');
    
    // gbk转utf8
    let utf8Text = App.string.toUTF8('gbk字符');
    ```

=== "API"

    - 请求

    | 参数 | 名称 | 类型 | 默认值 | 说明 |
    |:---|:---|:---|:---|:---|
    | text | 字符串 | String | - | - |

    - 响应

    | 参数 | 名称 | 类型 | 说明 |
    |:---|:---|:---|:---|
    | text | - | String | 对应的编码字符串 |

## Unicode 编码解码

=== "示例"

    ```javascript
    let data = '你好！';
    
    // 编码
    let encodeData = App.unicode.encode(data);
    App.log(encryptData); // \u4f60\u597d\uff01
    
    // 解码
    let decodeData = App.unicode.decode(encodeData);    
    App.log(encryptData); // 你好！
    ```

=== "API"

    - 请求

    | 参数 | 名称 | 类型 | 默认值 | 说明 |
    |:---|:---|:---|:---|:---|
    | data | 编码解码内容 | String | - | - |

    - 响应

    | 参数 | 名称 | 类型 | 说明 |
    |:---|:---|:---|:---|
    | data | - | String | 编码解码内容 |


## MD5 哈希

一种被广泛使用的密码散列函数，用于确保信息传输完整一致。

=== "示例"

    ```javascript
    let data = 'Hello, World!';
    
    // 计算哈希
    let hashData = App.md5(data);
    App.log(hashData); // 65a8e27d8879283831b664bd8b7f0ad4
    ```

=== "API"

    - 请求

    | 参数 | 名称 | 类型 | 默认值 | 说明 |
    |:---|:---|:---|:---|:---|
    | data | 哈希内容 | String | - | - |

    - 响应

    | 参数 | 名称 | 类型 | 说明 |
    |:---|:---|:---|:---|
    | data | - | String | 小写十六进制哈希值 |


## SHA 哈希

SHA 哈希常用于接口签名、摘要校验和防篡改校验。它和 MD5 一样是单向摘要算法，不需要密钥，也不能反向解密；返回值统一为小写十六进制字符串。

=== "示例"

    ```javascript
    let data = 'abc';

    let sha1Data = App.sha1(data);
    App.log(sha1Data); // a9993e364706816aba3e25717850c26c9cd0d89d

    let sha224Data = App.sha224(data);
    App.log(sha224Data); // 23097d223405d8228642a477bda255b32aadbce4bda0b3f7e36c9da7

    let sha256Data = App.sha256(data);
    App.log(sha256Data); // ba7816bf8f01cfea414140de5dae2223b00361a396177a9cb410ff61f20015ad

    let sha384Data = App.sha384(data);
    App.log(sha384Data); // cb00753f45a35e8bb5a03d699ac65007272c32ab0eded1631a8b605a43ff5bed8086072ba1e7cc2358baeca134c825a7

    let sha512Data = App.sha512(data);
    App.log(sha512Data); // ddaf35a193617abacc417349ae20413112e6fa4e89a97ea20a9eeee64b55d39a2192992a274fc1a836ba3c23a3feebbd454d4423643ce80e2a9ac94fa54ca49f
    ```

=== "API"

    - 请求

    | 方法 | 名称 | 类型 | 默认值 | 说明 |
    |:---|:---|:---|:---|:---|
    | App.sha1(data) | SHA1 哈希 | String | - | 传入待哈希内容 |
    | App.sha224(data) | SHA224 哈希 | String | - | 传入待哈希内容 |
    | App.sha256(data) | SHA256 哈希 | String | - | 传入待哈希内容 |
    | App.sha384(data) | SHA384 哈希 | String | - | 传入待哈希内容 |
    | App.sha512(data) | SHA512 哈希 | String | - | 传入待哈希内容 |

    - 响应

    | 参数 | 名称 | 类型 | 说明 |
    |:---|:---|:---|:---|
    | data | - | String | 小写十六进制哈希值 |


## AES 加解密

AES（高级加密标准） 是一种对称加密算法，广泛应用于数据加密和保护数据隐私。AES 的加密和解密使用相同的密钥，因此加密和解密过程对发送方和接收方来说是对称的。AES 算法支持多种密钥长度，包括 128 位、192 位和 256 位。

=== "示例"

    ```javascript
    let data = "Hello, World!"
    let key = "1234567890123456" // 16 字节的密钥，128 位
    let iv = "1234567890123456" // 初始化向量（IV）
    
    // 加密
    let encryptData = App.aes.encrypt(data, key, iv);

    // 解密
    let decryptData = App.aes.decrypt(encryptData, key, iv);
    ```
=== "API"

    - 请求

    | 参数 | 名称 | 类型 | 默认值 | 说明 |
    |:---|:---|:---|:---|:---|
    | data | 加解密内容 | String | - | - |
    | key | 密钥 | String | - | - |
    | iv | 初始化向量 | String | - | - |

    - 响应

    | 参数 | 名称 | 类型 | 说明 |
    |:---|:---|:---|:---|
    | data | - | String | Base64后的密文/解密内容 |


## RSA 加解密

RSA（Rivest-Shamir-Adleman）是一种广泛使用的 非对称加密算法，它使用一对密钥：公钥和私钥。公钥用来加密数据，私钥用来解密数据。与对称加密（如 AES）不同，非对称加密使用不同的密钥进行加密和解密，因此更适合用于安全的通信和数据交换。

=== "示例"

    ```javascript
    let data = "Hello, World!"
    let publicKey = "公钥密钥串"
    let privateKey = "私钥密钥串"
    
    // 加密
    let encryptData = App.rsa.encrypt(data, publicKey);

    // 解密
    let decryptData = App.rsa.decrypt(encryptData, privateKey);
    ```
=== "API"

    - 请求

    | 参数 | 名称 | 类型 | 默认值 | 说明 |
    |:---|:---|:---|:---|:---|
    | data | 加解密内容 | String | - | - |
    | publicKey/privateKey | 密钥 | String | - | 加密用公钥，解密用私钥 |

    - 响应

    | 参数 | 名称 | 类型 | 说明 |
    |:---|:---|:---|:---|
    | data | - | String | 密文/解密内容 |

## 自然语言处理

### 简繁体转换

=== "示例"

    ```javascript
    let data = '繁体字';
    
    // 转繁体
    let cht = App.nlp.cht(data);
    App.log(cht); // 繁體字

    // 转简体
    let chs = App.nlp.chs(cht);
    App.log(chs); // 繁体字
    ```

=== "API"

    - 请求

    | 参数 | 名称 | 类型 | 默认值 | 说明 |
    |:---|:---|:---|:---|:---|
    | data | 需转换的文字 | String | - | - |

    - 响应

    | 参数 | 名称 | 类型 | 说明 |
    |:---|:---|:---|:---|
    | data | - | String | 转换后的文字 |
