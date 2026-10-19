# 部署与使用

[返回首页](README.md)

## 1. 启动容器

适用于支持 Docker Compose 的 x86-64 NAS 或 Linux 主机。将 [docker-compose.yml](docker-compose.yml) 保存到自己的部署目录：

```yaml
services:
  havencloud:
    image: tang0225/havencloud:latest
    container_name: HavenCloud
    restart: unless-stopped
    ports:
      - "8100:8080"
    environment:
      - TZ=Asia/Shanghai
      - PUID=1000
      - PGID=1000
    volumes:
      - ./data:/data
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"
```

在该目录执行：

```bash
docker compose pull
docker compose up -d
docker logs HavenCloud
```

浏览器打开 `http://你的NAS地址:8100`。初始账号密码见容器首次启动日志。端口冲突时修改映射左侧的 `8100`，`PUID/PGID` 按 NAS 权限调整。

## 2. 申请授权

登录后填写联系方式（QQ／微信／邮箱／TG 任选）提交申请，管理员批准后自动激活。按页面提示修改初始密码，至少 6 位，无需字母数字符号组合。

授权连接失败时，可在申请弹窗的「连接不上？设置代理」中填写容器可访问的 HTTP 代理；后续可在「设置 → 系统 → HTTP 代理」修改，通常不必另加 Compose 授权代理变量。

## 3. 配置需要的功能

| 配置 | 用途 |
| --- | --- |
| 栖影账号 | 按页面流程绑定，搜索、解锁与签到 |
| 网盘 | 绑定 115、123 或光鸭，设置转存目标目录 |
| TMDB | 填写 API Key，获取影视资料与播出日期 |
| Emby | 填写地址和 API 密钥，比对媒体库、扫描缺集与重复版本 |
| 聚影 / TG 频道 | 按需配置聚影账号，添加 TG 公开频道 |
| Telegram | 按需配置机器人和通知 |

修改后保存设置，并使用对应的测试连接按钮。Emby 删除还需在「媒体去重」页登录具备删除权限的 Emby 账号。

## 4. 日常使用

- **找资源**：在资源搜索输入片名，点击来源按钮，按网盘、画质和季集筛选。
- **追剧**：先设置订阅规则与积分预算，再添加作品；日历显示已公布的排期。
- **补缺**：选择 Emby 媒体库扫描，按季查看缺集，预览候选和费用后确认补缺。
- **去重**：扫描后比较文件名和路径，核对删除预览再提交给 Emby。
- **重推**：网盘恢复后，到解锁记录点击失败记录的「重推」。使用原链接，不重复解锁扣分；成功记录不显示按钮。多链接重推可能重复提交其中已成功的文件。

TG 搜索读取公开频道可访问的网页预览，不覆盖全部历史。资源费用由来源站点决定。

## 5. Lucky / nginx 反代

Lucky 上游填 `http://NAS地址:8100`，转发整站 `/`，保留浏览器访问的原始 Host（含外部端口），HTTPS 证书在 Lucky 配置。正常透传 Origin、Referer 等请求头。

nginx 在已配置证书的 HTTPS server 中添加以下片段，将示例地址替换为自己的 NAS：

```nginx
location / {
    proxy_pass http://192.168.1.100:8100;
    proxy_set_header Host $http_host;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-For $remote_addr;
}
```

提示「拒绝跨站写请求」时，先更新镜像，再检查 Host、外部端口和域名是否一致，刷新页面重新登录。不要关闭跨站保护或伪造浏览器站点来源头。

## 6. 更新与备份

更新前备份 Compose 和整个 data 目录；建议停止容器后复制数据，再启动。

在原 Compose 目录更新：

```bash
docker compose pull
docker compose up -d
```

保留原 `./data:/data` 挂载，里面包含配置、数据库和安装身份。不要删除数据目录，也不要让多个实例共用它。出现同名容器冲突时，回到已有容器的原 Compose 项目更新。
