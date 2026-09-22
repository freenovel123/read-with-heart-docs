# 自定义TTS

TTS 将文字合成为语音；播放已有章节音频属于听书源，二者使用不同的规则入口。

!!! warning "CENC 配置归属修正"
    CENC 章节音频解密属于听书源，不属于 TTS，请参阅[听书源 CENC 加密音频](制作源/rule-content.md#play-url-cenc)。请勿在 TTS 返回值中配置 `decrypt`；普通 HTTP / WebSocket TTS 的配置方式不变。CENC 功能的可用状态以听书源说明为准。

## 请求处理

### 表达式
| 参数              | 名称   | 说明           |
|:----------------|:-----|:-------------|
| `@{voiceType}`  | 语音标志 | 角色的唯一标识      |
| `@{text}`       | 文本内容 | 需要转换为语音的文字内容 |
| `@{voiceStyle}` | 情感标识 | 角色的情感风格标识 |

### 请求示例

#### GET/POST请求
```javascript
// 请求地址
let url = 'https://www.baidu.com'; 

// 请求参数
let params = {
	type: '@{voiceType}',
	text: '@{text}',
	style: '@{voiceStyle}',
};

// 请求header
let header = {
	token: 'xxx'	
};

// POST请求，具体参考文档 native-to-javascript
let response = await app.post({
	url,
	params,
	header
});

// 如果responseBody是一个json字符串
let json = app.stringToJson(response.responseBody);

// 获取到语音文件的地址
let voiceUrl = json.url;

return {
	url: voiceUrl, // 语音请求地址
	method: 'GET', // 请求方式
	params: {}, // 携带的参数
	header: {}, // 携带的header
	cookie: {} // 携带的cookie
};
```

#### WebSocket

WebSocket和post的最大区别在于WebSocket的返回值是字符串 socket

```javascript
const ws = app.socket('wss://xxxxx');

// socket连接
ws.open(() => {
    console.log('连接socket成功');
    let message = '{"type": "@{voiceType}", "text": "@{text}"}'
    // 发送消息
    ws.send(message);
});

// 获取text文本信息
ws.text((text) => {
    console.log('返回的text数据:', text);
    const json = JSON.parse(text);
    if (json.event === 'TaskFinished') {
        // 告诉app，消息已经接收完毕
        ws.finished();
        
        // 关闭连接
        ws.close();
    }
});

// 获取二进制消息信息
ws.binary((data) => {
    console.log('返回的二进制数据:', data.length);
    // 将获取到的数据处理后重新推送给app，如果不做处理，直接push data数据即可
    ws.push(data)
});

// 必定返回值，告诉app这是一个websocket
return 'socket';
```

#### WebSocket 规则协议表

| 项目 | 协议 | 说明 |
| --- | --- | --- |
| 返回值 | 必须 `return 'socket'` | 告诉 App 这是 WebSocket 规则，不要当作普通 POST。 |
| 接收文本 | `ws.open((text) => { ... })` | 文本消息回调；字符串可能是 JSON，需要自行解析。 |
| 接收二进制 | `ws.binary((data) => { ... })` | 二进制回调；典型用于直接推送音频。 |
| 推送音频 | `ws.push(data)` | 把处理后的音频块推回 App。 |
| 正常结束 | `ws.finished()` | 表示本轮文本已经播完；调用后再决定是否 `ws.close()`。 |
| 中断 / 异常 | `ws.close()` | 关闭连接，不表示正常播完。 |
| 失败回执 | 不要把 `finished()` 当作“关闭连接” | `finished()` 仅说明服务端正常返回完毕，连接仍需 `close()`。 |

## 角色配置

### 参数

| 参数        | 名称   | 类型     | 默认值 | 说明           |
|:----------|:-----|:-------|:----|:-------------|
| voiceStyle | 情感标识 | String | -   | 角色的情感风格标识  |
| voiceType | 语音标识 | String | -   | 角色的唯一标识      |
| sex       | 性别   | Number | 0   | 性别 0未知，1男，2女 |
| name      | 名称   | String | -   | 显示的名称        |

### 示例
```json
[
    {
        "voiceStyle" : "gentle", // 情感风格标识
        "voiceType" : "zh-zhangsan", // 语音标识
        "sex" : 1, // 性别 0未知，1男，2女
        "name" : "张三" // 名字
    },
    {
	    "voiceStyle" : "sad", // 情感风格标识
        "voiceType" : "zh-erya", // 语音标识
        "sex" : 2, // 性别 0未知，1男，2女
        "name" : "二丫" // 名字
    }
]
```

## 本地部署TTS

在没有其它TTS可用的情况下，可以尝试在自己电脑上部署一个TTS

文档采用Docker部署方案，我们从Docker安装开始教学

### 安装Docker

1、下载Docker桌面版，根据不同的电脑系统下载对应的软件，以下是官网地址：

```
https://www.docker.com/products/docker-desktop/
```

2、安装完成后打开软件Docker

3、在您任意电脑位置新建文件夹，命名为 `docker`，即：

- Windows电脑打开 `D` 盘，新建文件夹，命名为 `docker`
- MacOS 可以打开访达 `文稿` 在该目录下新建文件夹，命名为 `docker`
- Linux 用户就不说了，大家都懂

4、在 `docker` 目录下新建一个文件，命名为 `docker-compose.yml`，文件后缀一定是 `.yml`，在该文件中写入以下内容，保存并关闭：

```yaml linenums="1"
services:
  ifreetimeTTS:
    container_name: ifreetimeTTS
    image: yunfinibol/ms-ra-forwarder-for-ifreetime:latest
    restart: unless-stopped
    ports:
      - "3000:3000"
    # 如果可以保持自己的ip或者完整域名不公开的话，可以不用设置环境变量
    environment: []
    #需要的话把上边一行注释，下面两行取消注释
    #environment:
    #  - TOKEN=自定义TOKEN
```

### 安装TTS

打开终端软件使用命令安装TTS

Windows用户打开 `cmd` 软件，mac用户打开 `终端` 软件

cd至新建的 `docker` 目录，运行以下命令，正常情况下可以安装成功：

```
# windows
cd D:\docker

# MacOS OR Linux
cd /路径/docker

# 运行安装命令
docker-compose up --build -d
```

在浏览器打开以下链接，测试是否已经安装成功：

```
http://localhost:3000/api/aiyue?voiceName=zh-CN-XiaoxiaoNeural&text=你好
```

### APP配置TTS

1、打开软件，找到 `语音管理` - `语音制作` ，名称，主地址随便填，切换到 `规则配置` 页面，在 `请求处理` 中输入以下代码：

- 电脑和手机必须在同一个局域网内
- 代码中 `你的ip地址` 需要使用你电脑的ipv4地址（具体可以在百度或者google中输入怎么查看电脑ip），然后将获取到的ip地址替换 `你的ip地址`
- 一般局域网的ip地址是一个 `192.168.x.x` 的样式

```javascript linenums="1"

// 请求地址

let encodeData = encodeURIComponent('@{text}')

let url = `https://你的ip地址:3000/api/aiyue?voiceName=@{voiceType}&text=${encodeData}`

return {
  url,
  method: 'GET'
}

```

2、角色配置，点击添加以下内容，保存即可，然后就可以测试发音了
```
名称：晓晓
角色标识：zh-CN-XiaoxiaoNeural
角色性别：女
```

### 参考

```
https://github.com/yy4382/ms-ra-forwarder-for-ifreetime

https://yfi.moe/post/ifreetime-mstts-selfhost/
```
