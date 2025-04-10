# Big questions
1. What does reliability means by "reliability from unreliability"
2. Why we need to achieve reliability?
3.  How to achieve "reliability from unreliability"?
4. What is TCP?
5. How does TCP achieve "reliability from unreliability"?

# Basics
- 互联网提供基础的 `best efforts delivery` [[week1 Logistics and Basic Principals#Three key concepts]]
- 但是很多应用想要的是稳定的, 可靠的双向字节流连接, 例如 `FTP, SMTP, HTTP`
- 这就需要在互联网基础接口之上再抽象出可靠接口, 接口内部做各种可靠性保证, 从而在不可靠之上建立可靠
- `TCP` 就是一个完成这种事情的协议
- ![[Pasted image 20250326051911.png]]


# TCP
- ![[Pasted image 20250326052206.png]]
- 核心就是
	- 确认
	- 超时
	- 重传
	- 消息序列号
- 