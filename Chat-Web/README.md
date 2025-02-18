# 发送和接收点对点消息

本页面介绍了如何快速集成 Agora Chat SDK 来实现单聊。

## 消息发送与接收流程
登录 Agora Chat 流程如下：

使用帐号和密码在 App Server 上注册。
注册成功后，使用账号和密码从 App Server 中获取 Token 。
使用账号和 Token 登录到 Chat 服务器。

![登录流程](https://web-cdn.agora.io/docs-files/1636443945728)

发送和接收点对点消息包括以下流程：

客户端 A 发送点对点消息到 Chat 服务器。
Chat 服务器将消息发送到客户端 B。客户端 B 收到点对点消息。

## 前提条件

- 有效的 Agora Chat 开发者账号。
- [创建 Agora Chat 项目并获取 AppKey](https://docs-preprod.agora.io/en/test/enable_agora_chat) 。
- [npm](https://www.npmjs.com/get-npm)
- SDK 支持 IE9+、FireFox10+、Chrome54+、Safari6+ 之间文本、表情、图片、音频、地址消息相互发送。
- SDK 本身已支持 IE9+、FireFox10+、Chrome54+、Safari6+。


## 操作步骤

### 1. 准备开发环境

本节介绍如何创建项目，将 Agora Chat SDK 集成进你的项目中。

#### 新建 Web 项目

新建一个目录 Agora_quickstart。在目录下运行 npm init 创建一个 package.json 文件，然后创建以下文件:

index.html
index.js
此时你的目录中包含以下文件：

Agora_quickstart
├─ index.html
├─ index.js
└─ package.json

### 2. 集成 SDK

- 在 `package.json` 中的 `dependencies` 字段中加入 `agora-chat` 及对应版本：

    ```json
   {
     "name": "web",
     "version": "1.0.0",
     "description": "",
     "main": "index.js",
     "scripts": {
       "test": "echo \"Error: no test specified\" && exit 1"
     },
     "dependencies": {
       "agora-chat": "latest"
     },
     "author": "",
     "license": "ISC"
   }
   ```

- 在你的 JS 文件中导入 `agora-chat` 模块：

```JavaScript
import WebIM from 'agora-chat'
```

- 如果是 Typescript 这样引入类型声明：

```JavaScript
import WebIM, { AgoraChat } from 'agora-chat'
```

### 3. 实现用户界面

index.html 的内容如下。<script src="./dist/bundle.js"></script> 用来引用 webpack 打包之后的bundle.js 文件。webpack 的配置会在后续步骤提及。

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Agora Chat Examples</title>
</head>

<body>
    <h2 class="left-align">Agora Chat Examples</h5>
        <form id="loginForm">
            <div class="col" style="min-width: 433px; max-width: 443px">
                <div class="card" style="margin-top: 0px; margin-bottom: 0px;">
                    <div class="row card-content" style="margin-bottom: 0px; margin-top: 10px;">
                        <div class="input-field">
                            <label>Username</label>
                            <input type="text" placeholder="Username" id="userID">
                        </div>
                        <div class="input-field">
                            <label>Password</label>
                            <input type="passward" placeholder="Password" id="password">
                        </div>
                        <div class="row">
                            <div>
                                <button type="button" id="register">register</button>
                                <button type="button" id="login">login</button>
                                <button type="button" id="logout">logout</button>
                            </div>
                        </div>
                        <div class="input-field">
                            <label>Peer username</label>
                            <input type="text" placeholder="Peer username" id="peerId">
                        </div>
                        <div class="input-field">
                            <label>Peer Message</label>
                            <input type="text" placeholder="Peer message" id="peerMessage">
                            <button type="button" id="send_peer_message">send</button>
                        </div>
                        <div class="input-field" style="position:relative">
                            <button type="button" id="send_audio_message">SendAudioMessage</button>
                            <div id="recordBox" style="display:none;position: absolute;left:0;bottom:0;width: 140px; height: 50px; text-align:center;line-height:50px;background:#ccc;cursor:pointer;font-weight:bold;">start recording</div>
                        </div>
                        <div class="row">
                            <div>
                                <button type="button" id="conversationList">ConversationList</button>
                            </div>
                        </div>
                        <div class="row">
                            <div>
                                <input type="text" placeholder="converationId" id="converationId">
                                <button type="button" id="roamingMessage">RoamingMessage</button>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </form>
        <hr>
        <div id="log"></div>
</body>
<script src="./dist/bundle.js"></script>
</html>
```


### 4. 实现消息发送与接收

index.js 的内容如下。本文使用 import 的方法导入 SDK，并使用 webpack 对 JS 文件进行打包，以避免浏览器兼容性问题。你需要分别将代码中的 "<Your app key>" 替换为你之前获取的 AppKey。

```Javascript
import WebIM from 'agora-chat'
const appKey = "<Your app key>"

let username, password

// 初始化客户端
WebIM.conn = new WebIM.connection({
    appKey: appKey,
})

// 添加回调函数
WebIM.conn.addEventHandler('connection&message', {
    onConnected: () => {
        document.getElementById("log").appendChild(document.createElement('div')).append("Connect success !")
    },
    onDisconnected: () => {
        document.getElementById("log").appendChild(document.createElement('div')).append("Logout success !")
    },
    onTextMessage: (message) => {
        console.log(message)
        document.getElementById("log").appendChild(document.createElement('div')).append("Message from: " + message.from + " Message: " + message.msg)
    },
    onTokenWillExpire: (params) => {
        document.getElementById("log").appendChild(document.createElement('div')).append("Token is about to expire")
        refreshToken(username, password)
    },
    onTokenExpired: (params) => {
        document.getElementById("log").appendChild(document.createElement('div')).append("The token has expired")
        refreshToken(username, password)
    },
    onError: (error) => {
        console.log('on error', error)
    }
})

// 从 app server 获取token
function refreshToken(username, password) {
    postData('https://a41.easemob.com/app/chat/user/login', { "userAccount": username, "userPassword": password })
        .then((res) => {
            let agoraToken = res.accessToken
            WebIM.conn.resetToken(agoraToken)
        })
}

// 发送请求
function postData(url, data) {
    return fetch(url, {
        body: JSON.stringify(data),
        cache: 'no-cache',
        headers: {
            'content-type': 'application/json'
        },
        method: 'POST',
        mode: 'cors',
        redirect: 'follow',
        referrer: 'no-referrer',
    })
        .then(response => response.json())
}

// 按钮行为定义
// 注册
    document.getElementById("register").onclick = function(){
        username = document.getElementById("userID").value.toString()
        password = document.getElementById("password").value.toString()
        postData('https://a41.easemob.com/app/chat/user/register', { "userAccount": username, "userPassword": password })
            .then((res) => {
                if (res.errorInfo && res.errorInfo.indexOf('already exists') !== -1) {
                    document.getElementById("log").appendChild(document.createElement('div')).append(`${username} already exists`)
                    return
                }
                document.getElementById("log").appendChild(document.createElement('div')).append(`${username} regist success`)
            })
    }
    // 登录
    document.getElementById("login").onclick = function () {
        username = document.getElementById("userID").value.toString()
        password = document.getElementById("password").value.toString()
        postData('https://a41.easemob.com/app/chat/user/login', { "userAccount": username, "userPassword": password })
            .then((res) => {
                let agoraToken = res.accessToken
                let easemobUserName = res.easemobUserName
                WebIM.conn.open({
                    user: easemobUserName,
                    agoraToken: agoraToken
                });
            })
    }

    // 登出
    document.getElementById("logout").onclick = function () {
        WebIM.conn.close();
    }

    // 发送一条单聊消息
    document.getElementById("send_peer_message").onclick = function () {
        let peerId = document.getElementById("peerId").value.toString()
        let peerMessage = document.getElementById("peerMessage").value.toString()
        let option = {
            chatType: 'singleChat',    // 设置为单聊
            type: 'txt',               // 消息类型
            to: peerId,                // 接收消息对象（用户 ID)
            msg: peerMessage           // 消息
        }
        let msg = WebIM.message.create(option); 
        WebIM.conn.send(msg).then((res) => {
            console.log('send private text success');
            document.getElementById("log").appendChild(document.createElement('div')).append("Message send to: " + peerId + " Message: " + peerMessage)
        }).catch(() => {
            console.log('send private text fail');
        })
    }
```

### 5. 运行项目

本文使用 webpack 对项目进行打包，并使用 webpack-dev-server 运行项目。

1.在 package.json 的 dependencies 字段中添加 webpack，webpack-cli，webpack-dev-server。并在 scripts 字段中增加 build 和 start:dev 命令。

```json
{
    "name": "web",
    "version": "1.0.0",
    "description": "",
    "main": "index.js",
    "scripts": {
        "build": "webpack --config webpack.config.js",
        "start:dev": "webpack serve --open --config webpack.config.js"
    },
    "dependencies": {
        "agora-chat": "latest",
        "webpack": "^5.50.0",
        "webpack-dev-server": "^3.11.2",
        "webpack-cli": "^4.8.0"
    },
    "author": "",
    "license": "ISC"
}
```

2.在项目根目录添加 webpack.config.js 文件，用于配置 webpack。文件内容如下：

```Javascript
const path = require('path');

module.exports = {
    entry: ['./src/index.js','./src/conversationList.js','./src/sendAudioMessage.js'],
    mode: 'production',
    output: {
        filename: 'bundle.js',
        path: path.resolve(__dirname, './dist'),
    },
    devServer: {
        compress: true,
        port: 9000,
        https: true
    }
};
```

此时你的目录中包含以下文件：

Agora_quickstart
├── index.html
├── package.json
├── src
│   ├── index.js
│   ├── sendAudioMessage.js
│   └── conversationList.js
├── utils
│   └── recordAudio.js
└── webpack.config.js


3.在项目根目录运行以下命令，安装依赖项。

```bash
$ npm install
```

4.运行以下命令使用 webpack 构建并运行项目。

```bash
# 使用 webpack 打包
$ npm run build

# 使用 webpack-dev-server 运行项目
$ npm run start:dev
```

项目启动之后，在页面输入用户名、密码进行注册，然后用注册的用户名密码登录，在登录成功之后，输入对方的用户名和要发送的消息，点击“发送”按钮进行发送，可以同时再打开一个页面相互收发消息。

### 6. 参考信息

集成 SDK 有两种方式：

#### 方法一：通过 npm 安装并导入 SDK

1. 在 `package.json` 中的 `dependencies` 字段中加入 `agora-chat` 及对应版本：

    ```json
   {
     "name": "web",
     "version": "1.0.0",
     "description": "",
     "main": "index.js",
     "scripts": {
       "test": "echo \"Error: no test specified\" && exit 1"
     },
     "dependencies": {
       "agora-chat": "latest"
     },
     "author": "",
     "license": "ISC"
   }
   ```

2. 在你的 JS 文件中导入 `agora-chat` 模块：

```JavaScript
import WebIM from 'agora-chat'
```

#### 方法二：从官网获取并导入 SDK

1. 下载 [Agora Chat SDK for Web](https://docs....)。将 `libs` 中的 JS 文件保存到你的项目下。（下载地址需添加，现在没有单独下载 SDK 的地址）

2. 在 HTML 文件中，对 JS 文件进行引用。

```JavaScript
   <script src="path to the JS file"></script>
```

### 7. 实现视频消息的录制与发送
sendAudioMessage.js 的内容如下。本文使用 import 的方法导入 SDK，并使用 webpack 对 JS 文件进行打包，以避免浏览器兼容性问题。


```Javascript
import WebIM from 'agora-chat'
//录制音频所需文件
import recorder from '../utils/recordAudio'
var _startTime, _endTime, recorderObj, time = 60, timer = null, MediaStream;

// 添加回调函数
WebIM.conn.addEventHandler('audioMessage', {
    onAudioMessage: (message) => {
        document.getElementById("log").appendChild(document.createElement('div')).append("Message from: " + message.from + " Type: " + message.type)
    }
})

// 按钮行为定义

//发送一条音频消息
document.getElementById("send_audio_message").onclick = function () {
    document.getElementById("recordBox").style.display = 'block';
}
//录制音频并发送音频消息
document.getElementById("recordBox").onclick = function () {
    let step = document.getElementById("recordBox").textContent.toString();
    if (step === 'start recording') {
        //开始录音
        document.getElementById("recordBox").textContent = 'recording';
        _startTime = new Date().getTime();
        window.clearInterval(timer)

        recorder.get((rec, val) => {
            recorderObj = rec;
            MediaStream = val
            if (rec) {
                timer = setInterval(() => {
                    if (time <= 0) {
                        rec.stop();
                        time = 60
                        timer = null;
                        window.clearInterval(timer)
                    } else {
                        time--;
                        rec.start();
                    }
                }, 1000);
            }
        });
    } else if (step === 'recording') {
        //停止录音，发消息
        window.clearInterval(timer)
        let targetId = document.getElementById("peerId").value.toString()
        _endTime = new Date().getTime();
        let duration = (_endTime - _startTime) / 1000;
        if (recorderObj) {
            recorderObj.stop();
            // 重置说话时间
            time = 60;
            // 获取语音二进制文件
            let blob = recorderObj.getBlob();
            // 发送语音功能
            const uri = {
                url: WebIM.utils.parseDownloadResponse.call(WebIM.conn, blob),
                filename: "audio-message.wav",
                filetype: "audio",
                data: blob,
                length: duration,
                duration: duration,
            };
            MediaStream.getTracks()[0].stop()
            let option = {
                chatType: 'singleChat',             // 会话类型，设置为单聊。
                type: 'audio',                      // 消息类型，设置为音频。
                to: targetId,                       // 消息接收方。
                file: uri,
                filename: uri.filename,
                onFileUploadError: function () {
                    // 消息上传失败。      
                    console.log('onFileUploadError');
                },
                onFileUploadProgress: function (e) {
                    // 上传进度的回调。
                    console.log('onFileUploadProgress', e)
                },
                onFileUploadComplete: function () {
                    // 消息上传成功。
                    console.log('onFileUploadComplete');
                }
            };
            let msg = WebIM.message.create(option);
            WebIM.conn.send(msg).then((res) => {
                console.log('send private audio success');
                document.getElementById("recordBox").style.display = 'none';
                document.getElementById("recordBox").textContent = 'start recording';
                document.getElementById("log").appendChild(document.createElement('div')).append("Message send to: " + targetId + " Type: " + msg.type)
            }).catch((err) => {
                console.log('send private audio fail', err);
            })
        }

    }
}
```

### 8. 实现获取会话列表和漫游消息

conversationList.js 的内容如下。本文使用 import 的方法导入 SDK，并使用 webpack 对 JS 文件进行打包，以避免浏览器兼容性问题。

```Javascript
import WebIM from 'agora-chat'

// 按钮行为定义
//获取会话列表
document.getElementById("conversationList").onclick = function () {
    document.getElementById("log").appendChild(document.createElement('div')).append("getConversationList...")
    WebIM.conn.getConversationList().then((res) => {
        console.log('getConversationList success')
        document.getElementById("log").appendChild(document.createElement('div')).append("getConversationList success")
        let str='';
        res.data.channel_infos.map((item) => {
            const chanelId = item.channel_id;
            let reg = /(?<=_).*?(?=@)/;
            const username = chanelId.match(reg)[0];
            str += '\n'+ JSON.stringify({
                conversationId:username,
                conversationType:chanelId.indexOf('@conference.easemob.com')>=0 ? 'groupChat':'singleChat'
            })
        })
        var odIV = document.createElement("div");
        odIV.style.whiteSpace = "pre";
        document.getElementById("log").appendChild(odIV).append('getConversationList:', str)
    }).catch(() => {
        document.getElementById("log").appendChild(document.createElement('div')).append("getConversationList failed")
    })
}

//获取漫游消息
document.getElementById("roamingMessage").onclick = function () {
    document.getElementById("log").appendChild(document.createElement('div')).append("getRoamingMessage...")
    let converationId = document.getElementById("converationId").value.toString()
    WebIM.conn.getHistoryMessages({ targetId: converationId }).then((res) => {
        console.log('getRoamingMessage success')
        document.getElementById("log").appendChild(document.createElement('div')).append("getRoamingMessage success")
        let str='';
        res.messages.map((item) => {
            str += '\n'+ JSON.stringify({
                messageId:item.id,
                messageType:item.type,
                from: item.from,
                to: item.to,
            }) 
        })
        var odIV = document.createElement("div");
        odIV.style.whiteSpace = "pre";
        document.getElementById("log").appendChild(odIV).append('roamingMessage:', str)
    }).catch(() => {
        document.getElementById("log").appendChild(document.createElement('div')).append("getRoamingMessage failed")
    })
}
```