# 资源分组

- cloud：集群资源组
- vm：虚拟服务器组

## 实例ssh支持密码登录

```bash
# 设置root用户密码
sudo passwd root

# 切换用户到root
su root

# 修改sshd配置文件
# 允许密码登录
sed -i 's/^PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
# 允许root用户登录
sed -i 's/^#PermitRootLogin prohibit-password/PermitRootLogin yes/g' /etc/ssh/sshd_config

# 重启ssh服务
systemctl restart sshd
```

### 添加cloud实例（直接挂在内网）

- public网络下的sub-public子网激活DHCP

- 新建实例

- 修改新实例的网络配置
  
  ```bash
  # 输出网络配置
  tee /etc/netplan/50-cloud-init.yaml <<-'EOF'
  network:
    ethernets:
        ens3:
            dhcp4: no
            addresses: [10.0.0.138/24]
            gateway4: 10.0.0.3
            nameservers:
                addresses: [10.0.0.1]
    version: 2
  EOF
  
  # 应用网络配置
  netplan apply
  ```

- public网络下的sub-public子网取消激活DHCP
