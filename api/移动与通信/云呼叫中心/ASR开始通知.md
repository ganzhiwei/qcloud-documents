## 接口说明
仅在开启了智能外呼时会上报该通知。检测到用户说话时，会上报该通知，可以根据该接口来停止正在播放的语音。

## 请求
### 请求包体

| 属性     | 类型     | 必选   | 说明                  |
| ------ | ------ | ---- | ------------------- |
| appId  | String | 是   | 应用 ID               |
| callId | String | 是   | 呼叫 ID               |
| ssrc  |  String | 是  | 对应媒体 rtp 头部 ssrc  |
| event	| String | 是  | 通知事件类型 (vadStart)  |
| timeStamp | String | 否  | 时间戳  |



