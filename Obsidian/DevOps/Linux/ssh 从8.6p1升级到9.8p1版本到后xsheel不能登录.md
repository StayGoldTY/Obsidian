```
## 修改状态为yes
PermitRootLogin yes
PubkeyAuthentication yes
## 配置文件中在最后一行补充一行算法
HostKeyAlgorithms ssh-rsa,ssh-dss

```
最开始用的这个，但是因为9.8p1  不支持HostKeyAlgorithms ssh-rsa,ssh-dss，所以配置后ssh不能正常启动。所以最后的配置改成
```
KexAlgorithms curve25519-sha256,curve25519-sha256@libssh.org,ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp521,diffie-hellman-group-exchange-sha256,diffie-hellman-group14-sha256,diffie-hellman-group16-sha512,diffie-hellman-group18-sha512

Ciphers aes256-ctr,aes192-ctr,aes128-ctr,aes256-cbc,aes192-cbc,aes128-cbc,chacha20-poly1305@openssh.com

MACs hmac-sha2-512,hmac-sha2-256,hmac-sha1
```
配置后还是不行，而且发现报错如下：
![[Pasted image 20250528141701.png]]

进一步需要配置文件/etc/pam.d/sshd修改成为如下：
```
auth     include    system-auth
account  required   pam_nologin.so
account  include    system-auth
password include    system-auth
session  include    system-auth
```

重启后可以了
```
sudo systemctl restart sshd
```