## WebRTC 通讯

WebRTC（Web Real-Time Communication）是一项实时通信开放标准，由 W3C 和 IETF 联合制定，核心目标是让浏览器 / 应用无需插件，通过 JavaScript API 直接实现端到端（P2P）的实时音视频通话、数据传输和屏幕共享，广泛应用于视频会议、在线教育、直播互动等场景。

一、核心定位与核心价值

1. 核心定位
   无需依赖第三方插件（如 Flash、Java Applet）或专有软件，仅通过浏览器原生 API 即可实现实时通信；
   底层基于 P2P 架构，减少服务器中转压力，降低延迟；
   支持跨平台（浏览器、移动应用、桌面应用）、跨设备互通。
2. 核心价值
   低延迟：P2P 直连 + 优化的传输协议，延迟通常在 100-500ms，满足实时交互需求；
   低成本：无需搭建复杂的中转服务器（仅需辅助服务），开发和部署成本低；
   开放性：开源标准，无专利壁垒，支持自定义扩展；
   易用性：高层 API 简洁，开发者无需关注底层传输细节。
   二、核心技术组件
   WebRTC 的功能由三大核心 API + 底层协议栈支撑，开发者通过高层 API 即可调用，无需深入底层：
3. 三大核心 API（开发者直接使用）
   API 名称 核心功能
   getUserMedia() 访问设备摄像头、麦克风，获取音视频流（MediaStream），用于 “采集本地媒体”。
   RTCPeerConnection 核心组件，负责端到端连接建立、音视频编解码协商、数据传输、网络自适应（NAT 穿透、丢包重传）。
   RTCDataChannel 基于 P2P 的双向数据通道，支持传输文本、二进制数据（如文件、游戏数据），延迟低（类似 WebSocket 但无需服务器中转）。
4. 底层关键技术（支撑 API 运行）
   媒体处理：内置音视频编解码器（如 VP8/VP9（视频）、OPUS（音频）），支持回声消除、噪声抑制、自动增益控制（AGC）等优化；
   网络传输：基于 UDP 的 RTP/RTCP 协议（实时传输 / 控制协议），负责音视频数据传输和质量监控；
   NAT 穿透：通过 STUN（获取公网地址）、TURN（中继 fallback）、ICE（整合 STUN/TURN 的连接候选算法）解决内网设备互通问题；
   安全保障：强制加密（DTLS 协议加密媒体数据，SRTP 协议保护传输），防止数据窃听和篡改。
   三、典型工作流程（以双端音视频通话为例）
   媒体采集：双方通过 getUserMedia()获取本地音视频流（MediaStream）；
   信令交换：通过第三方信令服务器（如 WebSocket、HTTP）传递 “连接元数据”（如设备能力、ICE 候选地址、SDP 会话描述）—— 核心是让双方知道 “如何连接”；
   连接建立：双方创建 RTCPeerConnection 实例，交换 SDP（协商编解码、媒体格式）和 ICE 候选（协商网络路径），通过 ICE 算法筛选最优连接（优先 P2P 直连，失败则用 TURN 中继）；
   实时通信：连接建立后，通过 RTCPeerConnection 传输音视频流，通过 RTCDataChannel 传输额外数据；RTCP 实时监控传输质量，动态调整码率、分辨率以适应网络波动；
   结束通话：调用 RTCPeerConnection.close()关闭连接，释放资源。
   四、应用场景
   实时音视频：视频会议（Zoom、腾讯会议的 Web 端）、在线教育（一对一辅导、大班课）、社交聊天（微信网页版视频通话）；
   数据传输：P2P 文件共享、实时协作工具（多人编辑文档）、在线游戏（低延迟数据同步）；
   屏幕 / 媒体共享：远程协助、直播带货（主播共享屏幕）、在线培训（共享 PPT）；
   物联网（IoT）：智能设备实时视频流传输（如监控摄像头、智能家居屏幕）。
   五、优势与局限性
5. 优势
   原生支持：浏览器（Chrome、Firefox、Edge、Safari）和主流移动平台（Android、iOS）均原生支持，无需额外安装；
   低延迟 + 高可靠性：P2P 架构 + 网络自适应算法，兼顾延迟和稳定性；
   安全无插件：无需插件，且强制加密，用户体验和安全性双优；
   开源免费：核心代码开源（Chromium WebRTC 项目），无商业授权成本。
6. 局限性
   浏览器兼容性：部分旧浏览器（如 IE）不支持，不同浏览器对 API 的细节实现有差异（需兼容处理）；
   信令服务器依赖：WebRTC 本身不提供信令机制，需自行开发或使用第三方信令服务（如 Firebase、Socket.io）；
   NAT 穿透限制：复杂内网环境（如多层 NAT）下 P2P 直连成功率低，需依赖 TURN 中继服务器（增加带宽成本）；
   带宽压力：高清音视频传输对双方网络带宽要求较高（尤其是多人通话时，需考虑 MCU/SFU 服务器进行媒体混合或转发，降低客户端压力）。
   六、常用工具与框架（简化开发）
   原生 API：适合需高度自定义的场景（如复杂音视频处理、特殊网络优化）；
   开源框架：WebRTC.js（轻量封装）、SimpleWebRTC（快速搭建基础通话）、Kurento（支持媒体服务器增强，如多人通话混音、录制）、Janus（多功能 WebRTC 网关）；
   云服务：Agora、Twilio、Amazon IVS（提供封装好的 WebRTC SDK + 信令 / 媒体服务器，无需关注底层细节，快速集成）。
   总结
   WebRTC 的核心是 “让实时通信在 Web 端原生可达”，通过标准化的 API 和底层技术，解决了传统实时通信 “依赖插件、延迟高、成本高” 的痛点。其优势在于低延迟、开放性和易用性，短板集中在兼容性、信令依赖和复杂网络适配 —— 但通过框架封装和云服务支持，这些问题可有效缓解。如今 WebRTC 已成为 Web 端实时通信的事实标准，广泛渗透到办公、教育、社交、IoT 等多个领域。

### 一个简单的例子

```js
import { useRef, useState, useEffect } from "preact/hooks";
import "./style.css";
import Button from "../../components/Button";

export default function WebRTCClient() {
  const config = {
    iceServers: [
      { urls: "stun:stun.l.google.com:19302" },
      {
        urls: "turn:a.relay.metered.ca:80",
        username: "free",
        credential: "free",
      },
    ],
  };

  // 基础引用和状态
  const local = useRef(null);
  const ws = useRef(null);
  const localStream = useRef(null);
  const messageInput = useRef(null);
  const messagesEnd = useRef(null); // 用于自动滚动到最新消息

  const peers = useRef({});

  const [roomId, setRoomId] = useState("");
  const [isAudioEnabled, setIsAudioEnabled] = useState(true);
  const [isVideoEnabled, setIsVideoEnabled] = useState(true);
  const [messages, setMessages] = useState([]); // 聊天消息列表
  const [isInRoom, setIsInRoom] = useState(false); // 是否在房间内
  const [userId, setUserId] = useState([]);

  const [remoteStreams, setRemoteStreams] = useState([]);

  const clientId = Math.random().toString(36).substring(2, 8); // 简化clientId

  // 自动滚动到最新消息
  useEffect(() => {
    messagesEnd.current?.scrollIntoView({ behavior: "smooth" });
  }, [messages]);

  // 打开本地视频
  const startLocalStream = async () => {
    try {
      localStream.current = await navigator.mediaDevices.getUserMedia({
        video: true,
        audio: {
          echoCancellation: true,
          noiseSuppression: true,
        },
      });
      local.current.srcObject = localStream.current;
    } catch (err) {
      console.error("Media error:", err);
    }
  };

  // 创建/加入房间
  const joinRoom = async (roomId) => {
    if (roomId && !isInRoom) {
      await startLocalStream();
      connectSignaling(roomId);
      setIsInRoom(true);
      // 添加系统消息
      setMessages((prev) => [
        ...prev,
        {
          type: "system",
          content: `Joined room: ${roomId}`,
        },
      ]);
    }
  };

  // 连接信令服务器
  const connectSignaling = (roomId) => {
    let wsLink = `public/ws?room=${roomId}&clientId=${clientId}`;
    if (import.meta.env.MODE === "development") {
      wsLink = `ws://localhost:9001/` + wsLink;
    } else {
      wsLink = `wss://yyl202248.ch.eu.org/serverApi/` + wsLink;
    }
    ws.current = new WebSocket(wsLink);

    ws.current.onopen = () => {
      console.log("WebSocket connected");
    };

    ws.current.onmessage = async ({ data }) => {
      console.log("Received message:", data);
      const message = JSON.parse(data);

      switch (message.type) {
        case "existing-users":
          console.log("avdd");
          setUserId(message.payload);

          break;
        case "user-joined":
          if (message.from !== clientId) {
            setMessages((prev) => [
              ...prev,
              {
                type: "system",
                content: `User ${message.from} joined`,
              },
            ]);
            await createOffer(message);
          }
          break;
        case "offer":
          await handleOffer(message);
          break;
        case "answer":
          await handleAnswer(message);
          break;
        case "ice-candidate":
          await handleIceCandidate(message);
          break;
        case "user-left":
          handleUserLeft(message);
          setMessages((prev) => [
            ...prev,
            {
              type: "system",
              content: `User ${message.from} left`,
            },
          ]);
          break;
        case "user-message":
          handleMessage(message);
          break;
      }
    };

    ws.current.onclose = () => {
      console.log("WebSocket disconnected");
      if (isInRoom) {
        setMessages((prev) => [
          ...prev,
          {
            type: "system",
            content: "Connection lost",
          },
        ]);
      }
    };
  };

  // 处理聊天消息
  const handleMessage = (message) => {
    setMessages((prev) => [
      ...prev,
      {
        id: Date.now(),
        sender: message.from === clientId ? "me" : message.from,
        content: message.payload,
        time: new Date().toLocaleTimeString(),
      },
    ]);
  };

  // 发送聊天消息
  const sendChatMessage = () => {
    if (ws.current && messageInput.current?.value.trim() && isInRoom) {
      const content = messageInput.current.value.trim();
      // 发送到服务器
      ws.current.send(
        JSON.stringify({
          type: "user-message",
          from: clientId,
          payload: content,
        })
      );
      // 同时添加到本地消息列表
      setMessages((prev) => [
        ...prev,
        {
          id: Date.now(),
          sender: "me",
          content,
          time: new Date().toLocaleTimeString(),
        },
      ]);
      // 清空输入框
      messageInput.current.value = "";
    }
  };

  // 创建P2P连接
  const createPeerConnection = async (targetId) => {
    const pc = new RTCPeerConnection(config);

    localStream.current.getTracks().forEach((track) => {
      pc.addTrack(track, localStream.current);
    });
    const newStream = new MediaStream();
    pc.ontrack = (event) => {
      console.log("✅ Received track from", targetId);
      event.streams[0].getTracks().forEach((t) => newStream.addTrack(t));
      setRemoteStreams((prev) => ({ ...prev, [targetId]: newStream }));
    };

    pc.onicecandidate = (event) => {
      if (event.candidate) {
        ws.current.send(
          JSON.stringify({
            type: "ice-candidate",
            from: clientId,
            to: targetId,
            payload: event.candidate,
          })
        );
      }
    };
    peers.current[targetId] = pc;
    return pc;
  };

  // 创建offer
  const createOffer = async (message) => {
    const pcs = await createPeerConnection(message.from);
    const offer = await pcs.createOffer();
    await pcs.setLocalDescription(offer);
    console.log("handleOffer", offer);

    ws.current.send(
      JSON.stringify({
        type: "offer",
        from: clientId,
        to: message.from,
        payload: offer,
      })
    );
  };

  // 处理offer
  const handleOffer = async (message) => {
    const pcs = await createPeerConnection(message.from);
    await pcs.setRemoteDescription(new RTCSessionDescription(message.payload));
    console.log("plaload", message.payload);

    const answer = await pcs.createAnswer();
    await pcs.setLocalDescription(answer);

    ws.current.send(
      JSON.stringify({
        type: "answer",
        from: clientId,
        to: message.from,
        payload: answer,
      })
    );
  };

  // 处理answer
  const handleAnswer = async (message) => {
    const pcs = peers.current[message.from];
    await pcs.setRemoteDescription(new RTCSessionDescription(message.payload));
  };

  // 处理ICE候选
  const handleIceCandidate = async (message) => {
    const pcs = peers.current[message.from];
    if (pcs) {
      await pcs.addIceCandidate(new RTCIceCandidate(message.payload));
    }
  };

  // 处理用户离开
  const handleUserLeft = (message) => {
    console.log("User left:", message.from);
    setRemoteStreams((prev) => {
      const updated = { ...prev };
      delete updated[message.from];
      return updated;
    });
    Object.values(peers.current).forEach((pc) => pc.close());
    peers.current = {};
  };

  // 离开房间
  const leaveRoom = () => {
    Object.values(peers.current).forEach((pc) => pc.close());
    peers.current = {};
    if (ws.current) {
      ws.current.close();
    }
    if (localStream.current) {
      localStream.current.getTracks().forEach((track) => track.stop());
    }
    local.current.srcObject = null;
    localStream.current = null;
    setRemoteStreams({});
    setIsAudioEnabled(true);
    setIsVideoEnabled(true);
    setIsInRoom(false);
    setMessages([]); // 清空消息
  };

  // 切换音频
  const toggleAudio = () => {
    if (localStream.current) {
      const enabled = !localStream.current.getAudioTracks()[0].enabled;
      localStream.current.getAudioTracks().forEach((track) => {
        track.enabled = enabled;
      });
      setIsAudioEnabled(enabled);
    }
  };

  // 切换视频
  const toggleVideo = () => {
    if (localStream.current) {
      const enabled = !localStream.current.getVideoTracks()[0].enabled;
      localStream.current.getVideoTracks().forEach((track) => {
        track.enabled = enabled;
      });
      setIsVideoEnabled(enabled);
    }
  };

  return (
    <div className="rtc-container">
      <h1>Private Space</h1>
      <div className="webrtc-app">
        {/* 房间输入区域 */}
        <div className="room">
          <input
            type="text"
            className="room-id"
            placeholder="Enter Room ID"
            value={roomId}
            onChange={(e) => setRoomId(e.target.value)}
            disabled={isInRoom}
            required
          />
          <Button
            type="submit"
            onClick={() => joinRoom(roomId)}
            disabled={isInRoom || !roomId.trim()}
          >
            {isInRoom ? "In Room" : "Join Room"}
          </Button>
        </div>

        <div className="main-content">
          {/* 视频区域 */}
          <div className="video-section">
            <div className="video-container-view">
              {/* 远程视频（大窗口） */}
              <div className="video-grid">
                {Object.entries(remoteStreams).map(([id, stream]) => (
                  <div className="remote-video video-container" key={id}>
                    <video
                      key={id}
                      autoPlay
                      playsInline
                      ref={(el) => {
                        if (el && stream && el.srcObject !== stream)
                          el.srcObject = stream;
                      }}
                      className="responsive-video"
                    ></video>
                    <div className="video-status">Remote UserId {id}</div>
                    <div className="remote-controls">
                      <button
                        onClick={() => {
                          stream._muted = !stream._muted;
                          setRemoteStreams((prev) => ({ ...prev }));
                        }}
                      >
                        {stream._muted ? "🔈 打开声音" : "🔇 关闭声音"}
                      </button>
                      <button
                        onClick={() => {
                          stream._hidden = !stream._hidden;
                          setRemoteStreams((prev) => ({ ...prev }));
                        }}
                      >
                        {stream._hidden ? "👁️ 显示视频" : "🙈 隐藏视频"}
                      </button>
                    </div>
                  </div>
                ))}
              </div>

              {/* 本地视频（小窗口） */}
              <div className="local-video video-container">
                <video
                  ref={local}
                  autoPlay
                  muted
                  className="responsive-video"
                ></video>
                <div className="video-status">
                  Local {!isAudioEnabled && "(Muted)"}{" "}
                  {!isVideoEnabled && "(Video Off)"}
                </div>
              </div>
            </div>

            {/* 视频控制按钮 */}
            <div className="control-buttons video-controls">
              <button
                className={`toggle-audio ${!isAudioEnabled ? "active" : ""}`}
                onClick={toggleAudio}
                disabled={!isInRoom}
              >
                {isAudioEnabled ? "Mute Audio" : "Unmute Audio"}
              </button>
              <button
                className={`toggle-video ${!isVideoEnabled ? "active" : ""}`}
                onClick={toggleVideo}
                disabled={!isInRoom}
              >
                {isVideoEnabled ? "Turn Off Video" : "Turn On Video"}
              </button>
              <button
                className="leave-btn"
                onClick={leaveRoom}
                disabled={!isInRoom}
              >
                Leave Room
              </button>
            </div>
          </div>

          {/* 聊天区域 */}
          <div className="chat-section" disabled={!isInRoom}>
            <h3>Chat</h3>
            <div className="messages-container">
              {messages.map((msg, index) => (
                <div
                  key={msg.id || index}
                  className={`message ${msg.type || msg.sender}`}
                >
                  {msg.type === "system" ? (
                    <span className="system-message">{msg.content}</span>
                  ) : (
                    <>
                      <span className="sender">{msg.sender}:</span>
                      <span className="content">{msg.content}</span>
                      <span className="time">{msg.time}</span>
                    </>
                  )}
                </div>
              ))}
              <div ref={messagesEnd} />
            </div>
            <div className="message-input">
              <input
                type="text"
                ref={messageInput}
                placeholder="Type a message..."
                disabled={!isInRoom}
                onKeyPress={(e) => e.key === "Enter" && sendChatMessage()}
              />
              <button onClick={sendChatMessage} disabled={!isInRoom}>
                Send
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  );
}
```
