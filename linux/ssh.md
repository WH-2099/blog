---
description: SSH 使用小技巧
---

# SSH 技高一筹

> Linux 不能失去 SSH，就像西方不能失去耶路撒冷。  ——WH-2099

Linux 用户免不了接触 SSH，但大多数人还停留在**“会用，能用”**的水平，本文从三个方面入手

## 配置提升

### 服务端

### 客户端

## 身份验证

### 部署证书体系

SSH 常见的身份验证方式有 3 种，根据其安全性自弱向强排列：

1. 密码验证
2. 公钥验证
3. 证书验证

选用证书验证，能够更加

### 公钥加密算法选用 ED25519

#### 原因

个人对加密算法了解很浅薄，所以只能参考网络上的各类资料，例如 [Comparing SSH Keys - RSA, DSA, ECDSA, or EdDSA?](https://goteleport.com/blog/comparing-ssh-keys/) 。

在安全应用领域里（如 tls 证书），用 ED25519 替换 RSA 正成为主流。

就实际使用而言，ED25519 生成的密钥很短，没条件复制粘贴的时候，手敲也不会太痛苦。

#### 方法

```
ssh-keygen -t ed25519
```



## 隧道转发

### 本地转发

### 动态转发

## 参考源



1. [ssh(1)](https://man.archlinux.org/man/ssh.1.en)
2. sshd(8)
3. ssh-agent(1)
