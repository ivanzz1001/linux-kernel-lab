# Ubuntu22.04 安装openssh-server

>ps: 参考https://blog.csdn.net/DT_FlagshipStore/article/details/126051811

1) **安装openssh-server**
```bash
# apt install openssh-server
```

2）**修改配置: /etc/ssh/sshd_config**
```text
PermitRootLogin yes
PasswordAuthentication yes

# 配置文件底部增加如下
HostKeyAlgorithms=+ssh-rsa,ssh-dss
KexAlgorithms=+diffie-hellman-group-exchange-sha1,diffie-hellman-group14-sha1,diffie-hellman-group1-sha1
```

3）**重启openssh服务**
```bash
# service sshd restart
```
