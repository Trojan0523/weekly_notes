# **Personal.WebRTC 浅水区的业务场景**

> WebRTC 可以说是 Web 近几年促进大前端发展的一个重要利器，目前谈谈具体的使用方式和在业务中的作用，给前端业务赋能（不具体介绍其架构，只会聊聊在浏览器上的使用）



- WebRTC 架构

  ![](https://github.com/bovinphang/WebRTC/blob/master/images/webrtc-structure.png?raw=true)

- WebRTC 音视频实现应用 API 架构

  ![](https://github.com/bovinphang/WebRTC/blob/master/images/WebRTC-internal-structure.jpg?raw=true)

- WebRTC (Web Real-Time Communications) web 即时通讯手段，在浏览器中的实现使得浏览器可以调用 IO 设备，实现一些通讯的能力，极大的丰富了多媒体在 Web 浏览器上面的功能，使得现在大前端音视频流上面大展身手
- 需要注意的是，WebRTC 在应用层浏览器端调用是以 HTTPS 进行传输的，非 HTTPS 下的网站无法使用 WebRTC 功能，浏览器需要获取用户权限，获取用户权限成功之后会返回一个 resolve 的 promise 回调函数，失败的话需要抛出一个错误对象提示 rtc 开启失败



#### WebRTC 浏览器 API

- MediaStream 通过媒体流的 API 可以获取摄像头，话筒等 IO 设备获取视频、音频流同步流。
- RTCPeerConnection 是 WebRTC 用于点对点连接获取 RTC 稳定高效流传输的组件
- RTCDataChannel 是 RTC 数据流传输渠道(频道)，用于点对点在浏览器之间建立一个高吞吐量、低时延的信道



#### MediaStream 调用方式

1. 通过 navigator.getUserMedia() 调用，其接受三个参数，一个是约束对象(constraints object) ，第二个是调用成功的 promise resolve，第三个是调用失败的 promise reject

   1. 约束对象属性
      - video 视频流
      - audio 音频流
      - MinWidth/MaxWidth 视频流最小最大宽
      - MinHeight/MaxHeight 视频流最小最大宽
      - MinAspectRatio/MaxAspectRatio 视频流最大最小宽高比
      - MinFramerate/MaxFramerate 视频流最小最大帧速率

2. 对于不同浏览器厂商的实现方式，同样要做兼容，这里的话是对几个浏览器内核实现的方式进行兼容：

   ```js
   let getUserMedia = (navigator.getUserMedia || navigator.webkitGetUserMedia 
   || navigator.mozGetUserMedia || navigator.msGetUserMedia)
   ```

   ```vue
   // 实现demo
   <template>
   <video ref="video" autoplay muted playsinline>
   maybe your browser doesn't support this, please check your browser version or upgrade to a modern browser  
   </video>
   </template>
   <script>
   	methods: {
       initWebRTC() {
         // video标签获取
         this.video = this.$refs.video
         // getUserMedia 第一个约束参数,音频、视频参数
         const constrains = { audio: false, video: {width: 640, height: 480} }
   			navigator.mediaDevices.getUserMedia = (constrains) => {
           // 浏览器getUserMedia简单兼容
           const getUserMedia = navigator.webkitGetUserMedia || navigator.mozGetUserMedia
           if(!getUserMedia) {
             return Promise.reject('getUserMedia兼容处理不了，换浏览器把')
           }
           return new Promise((resolve, reject) => {
             // 媒体流设备对象调用
             getUserMedia.call(navigator, constrains, resolve, reject)
           })
         }
         // resolve后获取流对象
         navigator.mediaDevices.getUserMedia(constrains).then((stream) => {
           if ('srcObject' in this.video) {
             // 流对象存在就将流对象赋给视频区域的src流
             thie.video.srcObject = stream
           } else {
             this.video.onloadmetadata = () => {
               // 视频按默认流处理播放
               this.video.play()
             }
           }
         }).catch((e) => {
           // 权限获取失败
           console.log(e)
         })
       }
     }
   </script>
   ```

- demo 执行后，浏览器会出现弹窗询问是否允许访问摄像头或话筒权限，确认之后 getUserMedia 获得一个 resolve 回调就会进行调用，音视频 RTC 流通常会与视频标签中的视频流对象绑定在一起，其实也可以使用`window.URL.createObjectURL(localMediaStream)`创造能在 video 使用 src 属性播放的 Blob url，同时在 video 标签中记得加上 autoplay 属性，不然标签上只会捕获到一张图片。



#### RTCPeerConnection 

- 获取流之后，就要将流内容发送出去，这是 client-client 之间传输的话，就要在协议层解决 NAT 穿透的问题, 即时通讯对可靠性的要求不高，所以也是采用 UDP 作为传输层协议，低延迟和及时性关注度高高一点，虽然是 UDP，但是对于用户加密数据，拥塞和流量控制同样是有的。协议栈如下图：

  ![](https://github.com/bovinphang/WebRTC/blob/master/images/WebRTC-protocol-stack-cn.jpg?raw=true)

- ICE（Interactive Connectivity Establishment，交互连接建立） STUN TURN 是通过 UDP 建立并维护端到端链接必须的。
- SDP 是一种数据格式，用于端到端连接时协商参数
- SCTP SRTP 属于应用层协议，用于 UDP 上提供不同流的多路复用、拥塞、和流量控制
- WebRTC 使用了 SDP（session description Protocol，会话描述协议），描述端对端链接参数，其本身不包含媒体的任何信息， 只描述会话情况: 交换媒体类型，网络传输协议，使用的编解码器，带宽及其他元数据
- DTLS 对 TLS 进行了拓展，为每条握手记录明确添加偏移字段和序号，这样才满足有序交付的条件，保证顺序正确，丢包问题通过计时器重传握手记录，签名证书两方也需要认证。
- SRTP 通过 IP 网络交付音频和视频定义标准的分组格式，本身不对数据及时可靠重传等等提供保证，只负责把数字化的音频采样和视频帧通过元数据封装起来，以辅助接受放的形式处理流。
- SCTP 是一个传输层协议，在 IP 协议上运行，不过他在一个安全的 DTLS 信道中运行，这个信道又建立在 UDP 上由于 WebRTC 支持通过 DataChannel api 在端对端质传检输任意应用数据，其就依赖 SCTP 的`可靠性可配置`，`交付次序可配置`, `面向消息传输方式` `支持流量、拥塞控制`等特性
- 完整的生命周期如下：
  - RTCPeerConnection 管理穿越 NAT 的完整 ICE 工作流
  - RTCPeerConnection 发送自动（STUN）持久化信号
  - RTCPeerConnection 跟踪本地流
  - RTCPeerConnection 跟踪远程流
  - RTCPeerConnection 按需触发自动流协商
  - RTCPeerConnection 提供必要的 API，以生成连接提议，接收应答，允许我们查询连接的当前状态，等等

```javascript
// 使用demo
//使用Google的stun服务器
var iceServer = {
    "iceServers": [{
        "url": "stun:stun.l.google.com:19302"
    }]
};
//兼容浏览器的getUserMedia写法
var getUserMedia = (navigator.getUserMedia ||
                    navigator.webkitGetUserMedia || 
                    navigator.mozGetUserMedia || 
                    navigator.msGetUserMedia);
//兼容浏览器的PeerConnection写法
var PeerConnection = (window.PeerConnection ||
                    window.webkitPeerConnection00 || 
                    window.webkitRTCPeerConnection || 
                    window.mozRTCPeerConnection);
//与后台服务器的WebSocket连接
var socket = __createWebSocketChannel();
//创建PeerConnection实例
var pc = new PeerConnection(iceServer);
//发送ICE候选到其他客户端
pc.onicecandidate = function(event){
    socket.send(JSON.stringify({
        "event": "__ice_candidate",
        "data": {
            "candidate": event.candidate
        }
    }));
};
//如果检测到媒体流连接到本地，将其绑定到一个video标签上输出
pc.onaddstream = function(event){
    someVideoElement.src = URL.createObjectURL(event.stream);
};
//获取本地的媒体流，并绑定到一个video标签上输出，并且发送这个媒体流给其他客户端
getUserMedia.call(navigator, {
    "audio": true,
    "video": true
}, function(stream){
    //发送offer和answer的函数，发送本地session描述
    var sendOfferFn = function(desc){
            pc.setLocalDescription(desc);
            socket.send(JSON.stringify({ 
                "event": "__offer",
                "data": {
                    "sdp": desc
                }
            }));
        },
        sendAnswerFn = function(desc){
            pc.setLocalDescription(desc);
            socket.send(JSON.stringify({ 
                "event": "__answer",
                "data": {
                    "sdp": desc
                }
            }));
        };
    //绑定本地媒体流到video标签用于输出
    myselfVideoElement.src = URL.createObjectURL(stream);
    //向PeerConnection中加入需要发送的流
    pc.addStream(stream);
    //如果是发送方则发送一个offer信令，否则发送一个answer信令
    if(isCaller){
        pc.createOffer(sendOfferFn);
    } else {
        pc.createAnswer(sendAnswerFn);
    }
}, function(error){
    //处理媒体流创建失败错误
});
//处理到来的信令
socket.onmessage = function(event){
    var json = JSON.parse(event.data);
    //如果是一个ICE的候选，则将其加入到PeerConnection中，否则设定对方的session描述为传递过来的描述
    if( json.event === "__ice_candidate" ){
        pc.addIceCandidate(new RTCIceCandidate(json.data.candidate));
    } else {
         pc.setRemoteDescription(new RTCSessionDescription(json.data.sdp));
    }
};
```



#### RTCDataChannel

- 此信道就是专门用来传输实时音视频流或者其他数据的，其建立在 PeerConnection 上，不能单独使用，建立`RTCPeerConnection`后，两端可以开启一个或者多个信道交换文本或二进制数据
- DataChannel 和 WebSocket 区别如下：

![](https://github.com/bovinphang/WebRTC/blob/master/images/DataChannel-WebSocket.jpg?raw=true)

- DataChannel 同样有四个事件，语义化就可以知道用来干嘛的了：`onopen` `onclose` `onmessage` `onerror`
- 四个状态通过 readyState 获取
  1. connecting： 浏览器之间试图建立 channel
  2. open: 建立成功，可以使用 send 方法发送数据
  3. closing：浏览器在关闭 channel
  4. closed：channel 已经被关闭了
- 暴露两个方法：`close()` 用于关闭 channel `send()` 用于发送数据

```javascript
// 实现demo
var ice = {
    'iceServers': [
        {'url': 'stun:stun.l.google.com:19302'},   // google公共测试服务器
        // {"url": "turn:user@turnservera.com", "credential": "pass"}
    ]
};

// var signalingChannel =  new SignalingChannel();

var pc = new RTCPeerConnection(ice);

navigator.getUserMedia({'audio': true}, gotStream, logError);

function gotStream(stram) {
    pc.addStream(stram);

    pc.createOffer().then(function(offer){
        pc.setLocalDescription(offer);
    });
}

pc.onicecandidate = function(evt) {
    // console.log(evt);
    if(evt.target.iceGatheringState == 'complete') {
        pc.createOffer().then(function(offer){
            // console.log(offer.sdp);
            // signalingChannel.send(sdp);
        })
    }
}

function handleChannel(chan) {
    console.log(chan);
    chan.onerror = function(err) {}
    chan.onclose = function() {}
    chan.onopen = function(evt) {
        console.log('established');
        chan.send('DataChannel connection established.');
    }

    chan.onmessage = function(msg){
        // do something
    }
}

// 以合适的交付语义初始化新的DataChannel
var dc = pc.createDataChannel('namedChannel', {reliable: false});

handleChannel(dc);
pc.onDataChannel = handleChannel;

function logError(){
    console.log('error');
}
```

- 通过 DataChannel 发送文件大致思路：JS 提供 File API 从 input[type = 'file']元素提取文件，通过 FileReader 将文件转换成 DataURL，同时我们可以将 DataURL 分成多个碎片通过 Channel 进行文件传输



#### 浏览器层面增强

- 底层上，浏览器会通过音视频引擎对捕获到的原始音频、视频流处理，除了增强画质和音质以外，还会尽量保持传输流之间的同步，对于非直播流对时延和网络波动没有那么大要求的业务来说知道以上的东西已经足够了，不过在音视频质量要求较高的业务(直播等)，丢包，时延，发送方要适应不断变化的带宽和客户端质检的网络延迟从而及时提醒客户端调整倍率或者自动调整输出比特率。



## summary

- 今天只是简单的介绍了 WebRTC 的三个 API 和业务中使用到的拍照截帧的场景 demo，从整个 RTC 实现原理可以从链接中去看看从应用层到协议层到传输层对 RTC 的实现： [WebRTC](https://github.com/bovinphang/WebRTC) ，写下本文的时候我也只看到了 17 章[WebRTC 信令交换](https://github.com/bovinphang/WebRTC#signaling-server)就没往下看了，不过作为大知识的补充或者以后音视频方向的精进，还是有必要去了解一下底层实现方面的东西的，无论是音视频协议还是 RTC 协议，对个人来说帮助都很大。





