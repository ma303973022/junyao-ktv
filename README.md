# 骏耀K歌（junyao-ktv）-工作忙暂停更新 

局域网 K 歌系统 —— 手机扫码点歌，电视/投影仪大屏沉浸演唱。

一台服务器 + 一块大屏，局域网内所有设备扫码即可点歌、切歌、调音，无需额外硬件，无需公网，开箱即用。

---

## ✨ 功能特性

- **手机扫码点歌**：无需安装 App，扫码即可搜索歌曲、点歌、管理已点队列
- **大屏播放主页面**：支持三套视觉风格（简约卡片 / 暗夜霓虹 / 3D 轮播大屏），设置菜单三风格统一交互
- **多端角色管理**：局域网内任意设备可申请「播放端」或「控制端」角色，播放端可上锁防误抢，控制端遥控播放端的暂停/切歌/音量/全屏等操作
- **原唱 / 伴唱切换**：基于双音轨 MV 服务端按需提取，浏览器端无需支持多音轨即可切换
- **均衡器**：标准 / 人声增强 / 低音增强 / 明亮清晰等预设音效
- **曲库管理**：支持本地目录与网盘挂载目录混合曲库，后台可视化管理来源、批量清理孤儿曲目、文件名智能解析、歌手头像自动匹配
- **硬件转码**：自动探测并使用 VAAPI（Intel/AMD 核显）或 NVENC（NVIDIA 显卡）硬件加速转码，无对应硬件时自动回退软件编码
- **管理后台**：曲库来源管理、终端设备管理、转码缓存清理策略配置，全部网页端可视化操作
- **Android 客户端**：面向电视盒子/投影仪场景的原生客户端，功能与网页大屏端对齐

---

## 🖥️ 系统组成

| 端 | 地址 | 说明 |
|---|---|---|
| 大屏播放主页面 | `/tv` | 电视/投影仪打开，K歌主界面 |
| 手机遥控端 | `/m` | 手机扫码打开，点歌与遥控 |
| 管理后台 | `/admin` | 曲库、设备、缓存等后台管理 |
| Android 客户端 | 见 `android-client/` | 电视盒子原生播放端，功能对齐网页 `/tv` |

浏览器打开服务器地址（如 `http://局域网IP:8083`）会看到一个导航首页，三个入口一目了然。

---

## 🚀 快速开始

只需 Docker，无需手动安装 Node.js / ffmpeg 等依赖。

### 1. 准备 `docker-compose.yaml`

```yaml
services:
  junyao-ktv:
    image: ma303973022/junyao-ktv:1.2.0
    container_name: junyao-ktv
    restart: unless-stopped
    ports:
      - "8083:8080"          # 访问端口：http://局域网IP:8083
    environment:
      - TZ=Asia/Shanghai
      - PORT=8080
      - DATA_DIR=/data
      - ADMIN_PASSWORD=admin888          # 管理后台("/admin")登录密码，建议改掉
      - VAAPI_DEVICE=/dev/dri/renderD128 # 核显硬件转码用，没有核显就删掉这行
      - HLS_CACHE_MAX_AGE_DAYS=3         # 转码缓存超过几天没人点就自动清理
      # 用 NVIDIA 显卡转码(NVENC)就取消下面两行注释，同时取消最下面 runtime: nvidia 的注释
      # - NVIDIA_VISIBLE_DEVICES=all
      # - NVIDIA_DRIVER_CAPABILITIES=compute,video,utility
    volumes:
      - /path/to/your/data:/data                 # 应用数据(数据库、封面等)，必须挂载

      # 曲库挂载：按需增删，本地曲库挂到 /mv/<自定义名>，网盘曲库挂到 /mv-net/<自定义名>
      - /path/to/local/library1:/mv/library1
      - /path/to/local/library2:/mv/library2
      - /path/to/netdisk/library:/mv-net/netdisk1
      - /path/to/singer/avatars:/singer           # 歌手头像目录，图片名对应歌手名，如 周杰伦.jpg
      # 挂载完成后，还需要去后台「曲库管理→曲库来源」里把新目录逐个启用，才会真正参与扫描
    devices:
      - /dev/dri:/dev/dri     # 核显硬件转码用，没有核显就删掉这行
    # runtime: nvidia         # 用 NVIDIA 显卡转码时取消注释(同时打开上面两行 NVIDIA 环境变量)
```


### 2. 启动

```bash
docker compose up -d
```

### 3. 访问

浏览器打开 `http://局域网IP:8083`，或直接访问 `/tv`（大屏）、`/m`（手机点歌）、`/admin`（管理后台）。

### 4. 曲库配置

进入「管理后台 → 曲库管理」，将挂载进容器的目录逐个启用为曲库来源，保存后即会自动扫描曲库，无需重建容器。

---

## ⚙️ 硬件转码

- **Intel / AMD 核显**：挂载 `/dev/dri` 设备节点即可自动启用 VAAPI 硬件转码
- **NVIDIA 显卡**：需安装 [nvidia-container-toolkit](https://github.com/NVIDIA/nvidia-container-toolkit)，并在 compose 中启用 `runtime: nvidia`
- 无对应硬件时自动回退到 CPU 软件转码，不影响正常使用

---

## 🛠️ 技术栈

- **服务端**：Node.js + Express + WebSocket + better-sqlite3 + ffmpeg（HLS 转码）
- **前端**：原生 HTML/CSS/JS，无框架依赖，多主题风格通过 CSS 变量 + 状态属性驱动
- **客户端**：Android（Kotlin）
- **部署**：Docker / Docker Compose，支持 x86_64 与 aarch64 多架构镜像

---

## 📄 License

本项目仅供个人学习与局域网自用场景使用。

---

作者：清风渡客
