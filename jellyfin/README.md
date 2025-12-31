# Jellyfin 媒体服务器应用
## 使用 Docker Compose 部署

项目结构：
```
.
├── compose.yaml
├── config/
├── cache/
└── media/
```

[_compose.yaml_](compose.yaml)
```yaml
services:
  jellyfin:
    image: jellyfin/jellyfin:latest
    container_name: jellyfin
    restart: unless-stopped
    ports:
      - '8096:8096'      # Web 界面
      - '8920:8920'      # HTTPS（可选）
      - '7359:7359/udp'  # 客户端自动发现
      - '1900:1900/udp'  # DLNA（可选）
    volumes:
      - ./config:/config  # 配置文件
      - ./cache:/cache    # 缓存数据
      - ./media:/media    # 媒体文件
    environment:
      - PUID=1000        # 用户ID（按需修改）
      - PGID=1000        # 用户组ID（按需修改）
      - TZ=Asia/Shanghai # 时区设置
    devices:
      - /dev/dri/renderD128:/dev/dri/renderD128  # Intel GPU 硬件加速（可选）
      - /dev/dri/card0:/dev/dri/card0            # Intel GPU（可选）
    group_add:
      - '44'    # video 组（硬件加速需要）
      - '107'   # render 组（硬件加速需要）
```

## 使用 Docker Compose 部署

```bash
$ docker compose up -d
Creating network "jellyfin_default" with the default driver
Pulling jellyfin (jellyfin/jellyfin:latest)...
latest: Pulling from jellyfin/jellyfin
...
Status: Downloaded newer image for jellyfin/jellyfin:latest
Creating jellyfin ... done
```

## 预期结果

运行容器列表应显示一个正在运行的容器及端口映射如下：
```bash
$ docker ps
CONTAINER ID   IMAGE                    COMMAND                  CREATED         STATUS         PORTS                                                                                  NAMES
a1b2c3d4e5f6   jellyfin/jellyfin:latest "/jellyfin/jellyfin"     2 minutes ago   Up 2 minutes   0.0.0.0:8096->8096/tcp, 0.0.0.0:8920->8920/tcp, 1900/udp, 0.0.0.0:7359->7359/udp   jellyfin
```

## 访问应用

应用启动后，在浏览器中访问 `http://localhost:8096`：

1. **首次设置**：
   - 选择语言（简体中文）
   - 创建管理员账户
   - 配置媒体库

2. **硬件加速配置**（如有需要）：
   - 进入控制台 → 播放
   - 选择相应的硬件加速选项

3. **媒体库管理**：
   - 添加媒体文件夹（映射到 `/media` 目录）
   - 配置元数据下载器

## 停止并移除容器

```bash
$ docker compose down
```

## 注意事项

1. **权限设置**：
   - 确保本地 `config`、`cache`、`media` 目录存在且权限正确
   - 如遇权限问题，可修改 compose.yaml 中的 `PUID` 和 `PGID`

2. **硬件加速**：
   - 仅当使用 Intel/AMD/NVIDIA GPU 时需要配置 devices 部分
   - NVIDIA GPU 需要额外安装 nvidia-container-runtime

3. **反向代理**（可选）：
   - 可通过 Nginx/Caddy 配置域名和 SSL 证书
   - 建议启用 HTTPS 访问

4. **数据备份**：
   - 定期备份 `config` 目录以保存设置和元数据
   - `cache` 目录可定期清理或重建

## 故障排除

1. **无法访问 Web 界面**：
   ```bash
   # 检查容器状态
   docker logs jellyfin
   
   # 检查端口占用
   netstat -tlnp | grep 8096
   ```

2. **硬件加速问题**：
   ```bash
   # 检查设备权限
   ls -la /dev/dri/
   
   # 测试转码
   docker exec jellyfin vainfo
   ```

3. **媒体文件无法识别**：
   - 检查目录映射权限
   - 确认媒体文件格式支持
   - 查看日志获取具体错误信息

更多配置选项请参考 [Jellyfin 官方文档](https://jellyfin.org/docs)。
