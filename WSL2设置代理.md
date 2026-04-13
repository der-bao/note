# WSL2设置代理笔记.md
## 一、环境说明
- 宿主机：Windows 11
- 子系统：WSL2 Ubuntu
- 代理软件：Clash Verge
- 代理端口：HTTP/SOCKS `7897`
- Windows WSL网卡IP：`172.17.16.1`
- 报错前置：WSL2 解析到错误网关`10.255.255.254`、SSL握手失败

## 二、Windows 宿主机配置（必做）
### 1. 代理软件端设置
1. Clash Verge 开启：**允许局域网 / Allow LAN**
2. 监听端口确认：`7897`
3. 开启系统代理

### 2. 管理员PowerShell放行防火墙端口
以**管理员身份**打开PowerShell执行：
```powershell
# 放行TCP 7897
New-NetFirewallRule -DisplayName "WSL2 Proxy 7897" -Direction Inbound -Action Allow -Protocol TCP -LocalPort 7897
# 放行UDP 7897（SOCKS5用）
New-NetFirewallRule -DisplayName "WSL2 Proxy 7897 UDP" -Direction Inbound -Action Allow -Protocol UDP -LocalPort 7897
```

### 3. 查看本机WSL网卡IP
CMD执行`ipconfig`，取如下地址：
> vEthernet (WSL (Hyper-V firewall)) IPv4：`172.17.16.1`

## 三、WSL2 终端配置代理
### 1. 编辑环境变量配置文件
```bash
nano ~/.bashrc
```

### 2. 写入最终固定配置（直接全量粘贴）
```bash
# WSL2 固定代理（适配本机172.17.16.1 + Clash 7897）
export hostip=172.17.16.1
export proxy_port=7897

export http_proxy="http://${hostip}:${proxy_port}"
export https_proxy="http://${hostip}:${proxy_port}"
export all_proxy="socks5://${hostip}:${proxy_port}"

# 内网不走代理
export no_proxy="localhost,127.0.0.1,::1,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16"

# 快捷命令
proxy_on(){
  export http_proxy="http://${hostip}:${proxy_port}"
  export https_proxy="http://${hostip}:${proxy_port}"
  export all_proxy="socks5://${hostip}:${proxy_port}"
  echo "✅ 代理已开启"
}

proxy_off(){
  unset http_proxy https_proxy all_proxy no_proxy
  echo "❌ 代理已关闭"
}
```

保存退出：
`Ctrl+O` → 回车 → `Ctrl+X`

生效配置：
```bash
source ~/.bashrc
```

## 四、修复SSL报错（SSL_ERROR_SYSCALL）
出现：`curl: (35) OpenSSL SSL_connect: SSL_ERROR_SYSCALL`
执行修复：
```bash
git config --global http.sslVerify false
git config --global https.sslVerify false
```

## 五、验证代理是否生效
```bash
# 查看出口IP（显示代理IPv6/国外IP即为成功）
curl ip.sb

# 测试HTTPS
curl -I https://www.google.com
```
判断标准：
1. `curl ip.sb` 返回代理节点IP（非本机内网IP）
2. Google 返回`200`相关响应即为连通

## 六、Git 配套代理（可选）
### 开启Git代理
```bash
git config --global http.proxy http://172.17.16.1:7897
git config --global https.proxy http://172.17.16.1:7897
```
### 关闭Git代理
```bash
git config --global --unset http.proxy
git config --global --unset https.proxy
```

## 七、本次踩坑总结
1. ❌ 普通PowerShell无法改防火墙，必须**管理员身份**
2. ❌ Windows终端不能用`nano`，`nano`只能在WSL2内执行
3. ❌ 自动获取网关拿到`10.255.255.254`无效，改用**WSL网卡固定IP 172.17.16.1**
4. ❌ 连通后HTTPS报SSL错误，关闭Git证书校验即可临时解决
5. ✅ 核心三要素：Clash开LAN + 防火墙放行端口 + WSL填对宿主机WSL网卡IP

## 八、日常使用快捷命令
```bash
proxy_on   # 开代理
proxy_off  # 关代理
```