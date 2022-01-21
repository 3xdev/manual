### 项目分配
- cloud：集群资源组
- vm：虚拟服务器组

### 添加cloud项目实例
- 激活sub-public子网的DHCP
- 新建实例
- 修改实例的网络配置和sshd配置
```bash
# 修改root用户密码
sudo passwd root

# 切换到root用户
su root

# 修改网络配置
tee /etc/netplan/50-cloud-init.yaml <<-'EOF'
network:
    ethernets:
        ens3:
            dhcp4: no
            addresses: [10.0.0.103/24]
            gateway4: 10.0.0.3
            nameservers:
                addresses: [10.0.0.1]
    version: 2
EOF

# 应用网络配置
netplan apply

# 修改sshd配置
sed -i 's/^PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
sed -i 's/^#PermitRootLogin prohibit-password/PermitRootLogin yes/g' /etc/ssh/sshd_config

# 重启sshd
systemctl restart sshd
```

- 禁止sub-public子网的DHCP









