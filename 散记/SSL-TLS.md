SSL（Secure Sockets Layer）是一种用于加密网络通信的加密协议。它最初由网景公司开发，现已被其继任者传输层安全性协议（TLS，Transport Layer Security）所取代。然而，术语"SSL"仍然广泛用于描述 TLS 协议的工作。

SSL/TLS 协议的主要目标是确保在客户端和服务器之间传输的数据的机密性、完整性和身份验证。它通过使用加密算法将通信内容加密，并使用数字证书对服务器进行身份验证。

SSL/TLS 的工作原理如下：

1. 加密：在 SSL/TLS 握手过程中，客户端和服务器协商加密算法和密钥长度，然后通过对称密钥加密算法加密传输的数据。对称密钥在握手过程中协商生成，并且在通信会话期间用于加密和解密数据。

2. 身份验证：服务器使用数字证书来证明其身份的真实性。数字证书由受信任的证书颁发机构（Certificate Authority，CA）签发，其中包含服务器的公钥和其他相关信息。客户端在握手过程中验证服务器的证书，并确认其合法性和可信度。

3. 完整性：SSL/TLS 使用消息认证码（Message Authentication Code，MAC）来验证数据的完整性，以防止数据在传输过程中被篡改。

SSL/TLS 广泛应用于保护敏感信息的网络通信，如网上银行、电子商务和隐私数据传输。它可以确保用户与网站之间的通信是加密的，防止中间人攻击和窃听。

需要注意的是，SSL/TLS 协议只负责在客户端和服务器之间提供安全的通信通道，而不涉及应用程序本身的安全性。因此，开发人员仍然需要采取其他安全措施来保护应用程序的安全性，如输入验证、访问控制和安全编码实践。

![[Pasted image 20240225100247.png]]

TLS（Transport Layer Security）是一种加密协议，用于保护网络通信的安全性和隐私性。它是 SSL（Secure Sockets Layer）的继任者，旨在提供更强大、更安全的加密通信。

TLS 协议的主要目标是确保在客户端和服务器之间传输的数据的机密性、完整性和身份验证。它通过使用加密算法将通信内容加密，并使用数字证书对服务器进行身份验证。

以下是 TLS 协议的基本工作流程：

1. 握手阶段（Handshake）：TLS 握手是在客户端和服务器之间建立安全连接的过程。握手包括以下步骤：

   a. 客户端向服务器发送 `ClientHello` 消息，其中包含支持的 TLS 版本、加密套件列表以及随机生成的数据。

   b. 服务器接收到 `ClientHello` 消息后，向客户端发送 `ServerHello` 消息，其中包含确定使用的 TLS 版本、加密套件以及随机生成的数据。

   c. 客户端和服务器彼此交换密钥信息，包括公钥、数字证书以及协商的加密算法和密钥长度。

   d. 双方生成会话密钥，用于对称加密算法进行后续的数据加密和解密。

2. 加密通信阶段：在完成握手阶段后，客户端和服务器之间建立了安全的通信通道。在这个阶段，TLS 使用协商的加密算法和密钥对传输的数据进行加密和解密，确保数据的机密性。

3. 完整性校验阶段：TLS 使用消息认证码（MAC）或哈希函数，对传输的数据进行完整性校验，以检测数据是否在传输过程中被篡改。

TLS 协议支持多种加密算法和密钥长度，以满足不同安全要求的需求。常见的加密套件包括 RSA、DSA、Diffie-Hellman、AES、3 DES 等。

TLS 还提供了一种称为"中间人攻击"的保护机制。它使用数字证书来验证服务器的身份，确保客户端与真正的服务器进行通信，而不是一个恶意的第三方。

TLS 广泛应用于保护敏感信息的网络通信，如网上银行、电子商务和隐私数据传输。它在互联网通信中扮演着重要的角色，确保数据的安全性和隐私性。

需要注意的是，TLS 协议的安全性也取决于其配置和实现的正确性。服务器和客户端都需要采取适当的安全措施来保护私钥、更新证书和应用最新的安全修补程序，以确保 TLS 连接的安全性。

![[Pasted image 20240225100637.png]]

### 对称加密
客户端和服务器都知道同一个密钥
![[Pasted image 20240225100854.png]]
![[Pasted image 20240225101019.png]]
这样不能保证数据是否被篡改

### 身份验证加密
![[Pasted image 20240225101311.png]]
