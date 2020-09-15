# HTTPS

1. HTTPS过程：![tls](tls.png)

	- tcp三次握手
	- 客户端发送clienthello，告知支持的加密算法，随机数
	- 服务端发送serverhello，告知选择的加密算法，随机数
	- 服务端发送certificate，serverhelloDone
	- 客户端验证certificate，然后用certificate上的公钥发送premaster secret（该阶段即为clientkeyexchange）
	- 客户端生成会话密钥，发送changecipherspec，表明开始加密传输，然后发送finished结束握手
	- 服务端用私钥解密，然后生成会话密钥，发送changecipherspec，表明开始加密传输，然后发送finished结束握手

2. 证书验证过程：

	- A向B发送证书，证书上包含域名，CA，公钥，CA的数字签名
	- B使用对应的CA的公钥（浏览器内置）解密数字签名，得到证书的hash，然后自行计算hash，进行对比；如果一致，认为证书没有被篡改，公钥可以使用
	- 证书中包含有效期信息，可以验证证书是否过期
	- 可能需要连接远端CA（OCSP协议），查看证书是否被吊销

3. 和HTTP的区别
	- 端口 ：HTTP的URL由“http://” 起始且默认使用端口80，而HTTPS的URL由“https://” 起始且默认使用端口443
	- 安全性和资源消耗： HTTP协议运行在TCP之上，所有传输的内容都是明文。HTTPS是运行在SSL/TLS之上的HTTP协议，SSL/TLS 运行在TCP之上。所有传输的内容都经过加密，加密采用对称加密，但对称加密的密钥用服务器方的证书进行了非对称加密。所以说，HTTP 安全性没有 HTTPS高，但是 HTTPS 比HTTP耗费更多服务器资源

4. RSA核心原理
	- [原理](https://zh.wikipedia.org/wiki/RSA%E5%8A%A0%E5%AF%86%E6%BC%94%E7%AE%97%E6%B3%95)
	- 安全性保证：破解RSA的核心在于将N分解为p和q，但是将一个大整数分解成两个质因数的乘积目前还没有多项式时间的算法，因此暴力破解RSA非常慢
	- TLS用RSA进行密钥交换时，会话密钥在客户端生成，然后使用RSA算法加密传输给服务端

5. Diffie-Hellman核心原理
	- 选取两个公开的素数，p，g
	- 客户端选取一个随机数a，求A = p^a mod g，然后发送给服务端
	- 服务端选取一个随机数b，求B = p^b mod g，然后发送给客户端
	- 双方各自计算A^b和B^a，这两个值都等于p^(ab) mod g，因此客户端和服务端协商出了一个相同的值，这个值就作为会话密钥
	- 安全性保证：破解DH算法的核心在于获取a或者b，但是通过p^a mod g求a是一个离散对数问题，该问题没有高效的解法，暴力破解非常慢