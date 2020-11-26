### 服务器配置
`cd /etc/wireguard/`
#### 配置转发功能
`echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf`
#### 创建密钥文件
`umask 077`

`wg genkey | tee privatekey | wg pubkey > publickey
`
#### 查看密钥
`Cat publickey `

```markdown
Na5BMpCXuG0wmyXZH1GE3Uic+hvkq4865lIR+RTJjUU=
```

`Cat privatekey `

```markdown
kH+D4tV+2MJ0r3Pz0ZcfaAKdtW6JGHw1pxcRhWfXGW8=

```
#### 创建服务器wg配置文件
`umask 022`

`vim wg0.conf`
#### 输入如下内容
    [Interface]
    Address = 10.0.1.1/16
    PrivateKey = kH+D4tV+2MJ0r3Pz0ZcfaAKdtW6JGHw1pxcRhWfXGW8=
    ListenPort = 8006
    PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth1 -j MASQUERADE
    PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth1 -j MASQUERADE
    
    [Peer]
    PublicKey = KYhBEfe76T3V2wMPNYqfH67+6KL85WVVMo8NhcFj+xw=
    AllowedIPs = 10.0.1.2/32
    
    [Peer]
    PublicKey = 1MRN8OEUQZ5HSaB0jy907zUjl+Z9zQPyVJQruEg2GCI=
    AllowedIPs = 10.0.1.3/32
#### 启动服务器
`wg-quick up wg0
`
### 客户端的配置
    [Interface]
    PrivateKey = 0OG59gIjuXJzciFFrxBkNDWQzfQoO4p5QkegoxdIv0s=
    Address = 10.0.1.2/16
    DNS = 223.6.6.6
    MTU = 1420
    
    [Peer]
    PublicKey = Na5BMpCXuG0wmyXZH1GE3Uic+hvkq4865lIR+RTJjUU=
    AllowedIPs = 10.0.0.0/22, 172.16.31.0/22
    Endpoint = 116.30.111.111:8006
    PersistentKeepalive = 30
	
	
### 部分配置参考
https://golb.hplar.ch/2019/07/wireguard-windows.html
![配置参照](https://raw.githubusercontent.com/Mircc/K2p_Build_Notes/master/wireguard_config.png "配置参照")
