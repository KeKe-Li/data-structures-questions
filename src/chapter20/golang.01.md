#### QUIC网络传输协议 

QUIC 全称 quick udp internet connection，“快速 UDP 互联网连接”，（和英文 quick 谐音，简称“快”）是由 Google 提出的基于 UDP 进行可靠传输的协议。QUIC 在应用层实现了丢包恢复、拥塞控制、滑窗机制等保证数据传输的可靠性，同时对传输的数据具备前向安全的加密能力。HTTP3 则是 IETF(互联网工程任务组)基于 QUIC 协议基础进行设计的新一代 HTTP 协议。

QUIC/HTTP3 分层模型及与 HTTP2 对比：

<p align="center">
<img width="700" align="center" src="../images/185.jpg" />
</p>

