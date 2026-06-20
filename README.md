# Ansible 自动化部署 Web 集群

[![Ansible](https://img.shields.io/badge/Ansible-2.9+-EE0000?logo=ansible&logoColor=white)](https://www.ansible.com/)
[![License](https://img.shields.io/badge/license-MIT-blue)](LICENSE)
[![Platform](https://img.shields.io/badge/platform-CentOS%207%2B-brightgreen)]()

使用 Ansible 一键部署 NFS + rsync + Nginx + sersync 的完整 Web 服务集群。

## 架构概览

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   NFS Server │     │  Web Server  │     │ Backup Server│
│  (nfs 组)    │     │  (web 组)    │     │ (backup 组)  │
├──────────────┤     ├──────────────┤     ├──────────────┤
│ • NFS 共享   │────▶│ • Nginx      │────▶│ • rsyncd     │
│ • sersync    │     │ • NFS 挂载   │     │ • 备份存储   │
│   (实时同步)  │     │ • rsync 客户端│     │              │
└──────────────┘     └──────────────┘     └──────────────┘
```

**数据流向：** NFS Server 共享目录 → sersync 实时监控 → rsync 同步到 Backup Server。Web Server 挂载 NFS 并通过 Nginx 对外提供服务。

## 技术栈

| 技术 | 用途 |
|------|------|
| Ansible | 自动化编排，一键部署全部节点 |
| NFS | 文件共享存储 |
| Nginx | Web 服务 |
| rsync | 增量文件同步 |
| sersync | 基于 inotify 的实时同步触发 |

## 目录结构

```
ansible-project/
├── site.yml              # 主入口 playbook
├── nfs.yml               # NFS 服务端 + 客户端部署
├── nginx.yml             # Nginx 安装和配置
├── rsync.yml             # rsync 服务端 + 客户端部署
├── sersync.yml           # sersync 实时同步配置
├── inventory             # 主机清单
├── vars/
│   └── main.yml          # 可配置变量
└── files/
    └── web.html          # 示例 Web 页面
```

## 快速开始

### 1. 环境要求

- **控制端：** 任意 Linux/macOS，安装 Ansible ≥ 2.9
- **被控端：** CentOS 7+，配置好 SSH 密钥认证
- 至少需要 3 台服务器（NFS、Web、Backup），也可以部署在同一台机器上测试

### 2. 克隆项目

```bash
git clone https://github.com/psy-psy329/ansible-project.git
cd ansible-project
```

### 3. 修改配置

编辑 `inventory` 文件，替换为你的服务器 IP：

```ini
[nfs]
192.168.100.20

[web]
192.168.100.30

[backup]
192.168.100.40
```

编辑 `vars/main.yml`，根据实际情况修改：

```yaml
nfs_server_ip: 192.168.100.20
rsync_server_ip: 192.168.100.20
web_subnet: 192.168.100.0/24
rsync_auth_user: rsync_backup
rsync_password: "请修改为复杂密码"
```

### 4. 执行部署

```bash
ansible-playbook -i inventory site.yml
```

## 验证部署

| 组件 | 验证方法 |
|------|----------|
| Nginx | 浏览器访问 `http://<web服务器IP>` |
| NFS | 在 Web 服务器上执行 `df -h \| grep nfs` |
| rsync | `rsync rsync_backup@<backup服务器IP>::backup` |
| sersync | 在 NFS 共享目录创建文件，检查 Backup 是否同步 |

## 常用命令

```bash
# 仅部署 Nginx
ansible-playbook -i inventory nginx.yml

# 仅部署 NFS
ansible-playbook -i inventory nfs.yml

# 检查所有节点连通性
ansible all -i inventory -m ping

# 查看 nginx 服务状态
ansible web -i inventory -m systemd -a "name=nginx state=started"
```

## 排错指南

| 问题 | 可能原因 | 解决方法 |
|------|----------|----------|
| SSH 连接失败 | 密钥未配置 | `ssh-copy-id root@<目标IP>` |
| NFS 挂载失败 | 防火墙未关闭或 rpcbind 未启动 | `systemctl status rpcbind` |
| sersync 不同步 | confxml.xml 中 IP 配置错误 | 检查 `vars/main.yml` 中的 IP |
| Nginx 404 | web.html 未正确拷贝 | 检查 `/web/web.html` 是否存在 |

## License

MIT
