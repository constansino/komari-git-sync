# Komari 自动同步到 Git（中文教程）

这个项目用于把 **Komari（komari-monitor）数据库里的节点 IP 信息** 自动同步到 Git 仓库，方便：

- 追踪 GCP 等动态 IP 变化
- 多端查看与审计
- 和团队共享（公开仓库可分享方案，私有仓库存放真实数据）

---

## 一、项目结构

- `komari-git-sync.sh`：主同步脚本
- `komari-git-sync.env.example`：环境变量示例
- `komari-git-sync.service`：systemd 服务
- `komari-git-sync.timer`：systemd 定时器（默认每5分钟）

---

## 二、工作原理

1. 从 Komari 的 sqlite 数据库读取 `clients` 表
2. 提取 `name / ipv4 / ipv6 / updated_at`
3. 生成 `nodes-ip.json`
4. 检测文件变化，有变化才 `git add/commit/push`

---

## 三、前置条件

目标机（部署 Komari 的服务器）需要：

- `sqlite3`
- `git`
- `systemd`
- 该机器有权限访问你的 Git 仓库（PAT/SSH key/gh auth 任一可用）

---

## 四、安装步骤

### 1) 安装脚本和 unit

```bash
sudo install -m 755 komari-git-sync.sh /usr/local/bin/komari-git-sync.sh
sudo install -m 600 komari-git-sync.env.example /etc/komari-git-sync.env
sudo install -m 644 komari-git-sync.service /etc/systemd/system/komari-git-sync.service
sudo install -m 644 komari-git-sync.timer /etc/systemd/system/komari-git-sync.timer
```

### 2) 编辑环境变量

```bash
sudo nano /etc/komari-git-sync.env
```

至少要改：

```bash
REPO_URL=https://github.com/<your-name>/<your-private-repo>.git
BRANCH=main
```

如果你的 Komari 路径不一样，也改：

```bash
KOMARI_DB=/data/komari-monitor/data/komari.db
```

### 3) 启动定时器

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now komari-git-sync.timer
```

---

## 五、手动测试

```bash
sudo systemctl start komari-git-sync.service
sudo systemctl status --no-pager komari-git-sync.service
sudo tail -n 100 /var/log/komari-git-sync.log
```

---

## 六、常见问题

### 1) 日志显示 `REPO_URL is empty, skip`

说明没配置仓库地址：

- 编辑 `/etc/komari-git-sync.env`
- 填写 `REPO_URL`

### 2) push 失败（鉴权问题）

给服务器配置 Git 凭据：

- HTTPS + PAT
- 或 SSH key
- 或 `gh auth login`

### 3) 不想每5分钟

修改 `komari-git-sync.timer` 的 `OnUnitActiveSec`，例如：

```ini
OnUnitActiveSec=15min
```

然后：

```bash
sudo systemctl daemon-reload
sudo systemctl restart komari-git-sync.timer
```

---

## 七、安全建议

- 真实节点 IP 数据建议放 **私有仓库**
- 公共仓库只放脚本与教程
- 避免把 token / 密钥提交进 Git

---

## License

MIT
