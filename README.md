# Ansible 部署 Web 集群

这个项目用 Ansible 自动部署 NFS + rsync + Nginx + sersync。

## 如何运行
1. 修改 inventory 文件中的 IP 地址
2. 运行：ansible-playbook -i inventory site.yml
## 快速使用
```bash
git clone <你的仓库地址>
cd ansible-web-cluster
# 修改 inventory 和 vars/main.yml 中的IP地址
ansible-playbook -i inventory site.yml
