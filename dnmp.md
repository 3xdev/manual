# dnmp

## 安装git、docker和docker-compose

```bash
yum install git









# 创建 docker-compose-dnmp 服务
cd /etc/systemd/system

cat <<EOF > docker-compose-dnmp.service
[Unit]
Description=Docker Compose DNMP Service
Requires=docker.service
After=docker.service

[Service]
Type=oneshot
RemainAfterExit=yes
WorkingDirectory=/home/dnmp
ExecStart=/usr/bin/docker compose up -d
ExecStop=/usr/bin/docker compose down

[Install]
WantedBy=multi-user.target
EOF






```





