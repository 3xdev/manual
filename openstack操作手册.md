### Kolla Ansible部署OpenStack（Ubuntu20）

```bash
# 更新软件包
apt update
apt upgrade

# 安装 Python 依赖
apt install python3-dev libffi-dev gcc libssl-dev

# 安装 pip
apt install python3-pip
pip3 install -U pip

# 安装 Ansible
pip install -U 'ansible>=4,<6'

# 安装 Kolla-Ansible
pip3 install git+https://opendev.org/openstack/kolla-ansible@stable/victoria
mkdir -p /etc/kolla
cp -r /usr/local/share/kolla-ansible/etc_examples/kolla/* /etc/kolla
cp /usr/local/share/kolla-ansible/ansible/inventory/* .

# 配置 Ansible
mkdir -p /etc/ansible
vim /etc/ansible/ansible.cfg
# 添加内容
[defaults]
host_key_checking=False
pipelining=True
forks=100

# 生成 kolla 密码
kolla-genpwd

# 修改 Kolla globals.yml
vim /etc/kolla/globals.yml
把
#kolla_base_distro: "centos"
#kolla_install_type: "source"
#network_interface: "eth0"
#neutron_external_interface: "eth1"
#kolla_internal_vip_address: "10.10.10.254"
改为
kolla_base_distro: "ubuntu"
kolla_install_type: "source"
network_interface: "eno1"
neutron_external_interface: "eno2"
kolla_internal_vip_address: "192.168.0.82"

# docker-ce 安装（kolla-ansible bootstrap-servers停住时）
apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
add-apt-repository "deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
apt update
apt install docker-ce

# 部署依赖的引导服务器
kolla-ansible -i ./all-in-one bootstrap-servers
# 部署前检查
kolla-ansible -i ./all-in-one prechecks
# 部署
kolla-ansible -i ./all-in-one deploy -vvv

# 查看admin密码
cat /etc/kolla/passwords.yml | egrep keystone_admin_password

# 空间扩容
vgdisplay 
lvdisplay 
lvextend -l +80%free /dev/ubuntu-vg/ubuntu-lv
resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv
```

### 资源分组

- cloud：集群资源组
- vm：虚拟服务器组

### 实例ssh支持密码登录

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
