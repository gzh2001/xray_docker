# 需要修改的部分
1. `./docker-compose.yaml`填上伪装页面的域名(2处)  
2. 在启动容器前，先使用acme.sh或其他工具为伪装域名申请证书，并安装至`./data/certs`，分别为`fullchain.pem`(使用证书链文件可以提高安全性)和`key.pem`。推荐使用`acme.sh --install-cert -d xxxx --fullchain-file xxx --key-file xxx`安装证书，使用dns api申请证书，以实现自动证书配置。

# 使用WARP，参考[通过 Cloudflare Warp 增强代理安全性](https://xtls.github.io/document/level-2/warp.html)
## 申请 Warp 账户
感谢 Cloudflare 推动自由的互联网，现在你可以免费使用 Warp 服务，连接的时候会根据出口自动选择最近的服务器
### 方法 1：
1. 使用一台 vps，下载 wgcf
2. 运行 `wgcf register` 生成 `wgcf-account.toml`
3. 运行 `wgcf generate` 生成 `wgcf-profile.conf` 拷贝内容如下：
```
[Interface]
PrivateKey = 我的私钥
Address = 172.16.0.2/32
Address = 2606:4700:110:8949:fed8:2642:a640:c8e1/128
DNS = 1.1.1.1
MTU = 1280
[Peer]
PublicKey = Warp公钥
AllowedIPs = 0.0.0.0/0
AllowedIPs = ::/0
Endpoint = engage.cloudflareclient.com:2408
```
### 方法 2：
使用 `warp-reg.sh`，运行：
`bash -c "$(curl -L warp-reg.vercel.app)"`
输出
```
{
  "endpoint": {
    "v4": "162.159.192.7",
    "v6": "[2606:4700:d0::a29f:c007]"
  },
  "reserved_dec": [35, 74, 190],
  "reserved_hex": "0x234abe",
  "reserved_str": "I0q+",
  "private_key": "yL0kApRiZW4VFfNkKAQ/nYxnMFT3AH0dfVkj1GAlr1k=",
  "public_key": "bmXOC+F1FxEMF9dyiK2H5/1SUtzH0JuVo51h2wPfgyo=",
  "v4": "172.16.0.2",
  "v6": "2606:4700:110:81f3:2a5b:3cad:9d4:9ea6"
}
```
拷贝输出的内容
### 方法 3：
使用wgcf-cli，运行以下内容进行安装：
`bash -c "$(curl -L wgcf-cli.vercel.app)"`
运行 `wgcf-cli register` 进行注册，输出：
```
❯ wgcf-cli register
{
    "endpoint": {
        "v4": "162.159.192.7:0",
        "v6": "[2606:4700:d0::a29f:c007]:0"
    },
    "reserved_str": "6nT5",
    "reserved_hex": "0xea74f9",
    "reserved_dec": [
        234,
        116,
        249
    ],
    "private_key": "WIAKvgUlq5fBazhttCvjhEGpu8MmGHcb1H0iHSGlU0Q=",
    "public_key": "bmXOC+F1FxEMF9dyiK2H5/1SUtzH0JuVo51h2wPfgyo=",
    "addresses": {
        "v4": "172.16.0.2",
        "v6": "2606:4700:110:8d9c:3c4e:2190:59d1:2d3c"
    }
}
```
完整文件将会保存到工作目录的 `wgcf.json` 内。
运行 `wgcf-cli generate --xray` 来生成一个WireGurad出站，他会将内容保存到 `wgcf.xray.json` 内
示例文件：
```
{
  "protocol": "wireguard",
  "settings": {
    "secretKey": "6CRVRLgFwGajnikoVOPTDNZnDhx3EydhPsMgpxHfBCY=",
    "address": ["172.16.0.2/32", "2606:4700:110:857a:6a95:fe27:1870:2a9d/128"],
    "peers": [
      {
        "publicKey": "bmXOC+F1FxEMF9dyiK2H5/1SUtzH0JuVo51h2wPfgyo=",
        "allowedIPs": ["0.0.0.0/0", "::/0"],
        "endpoint": "162.159.192.1:2408"
      }
    ],
    "reserved": [240, 25, 146],
    "mtu": 1280
  },
  "tag": "wireguard"
}
```