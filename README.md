# 环境变量
- CINDER_DB: cinder数据库ip
- CINDER_DBPASS: cinder数据库密码
- RABBIT_HOST: rabbitmq IP
- RABBIT_USERID: rabbitmq user
- RABBIT_PASSWORD: rabbitmq user 的 password
- KEYSTONE_INTERNAL_ENDPOINT: keystone internal endpoint
- KEYSTONE_ADMIN_ENDPOINT: keystone admin endpoint
- CINDER_PASS: openstack cinder用户密码
- MY_IP: my_ip
- GLANCE_HOST: glance internal endpoint
- VOLUME_BACKEND_NAME: volume_backend_name
- SMBFS_SERVER: smbfs server ip address
- SMB_PASS: samba root 用户的密码

# volumes:
- /opt/openstack/cinder-volume-smbfs: /etc/cinder

# 启动cinder-volume-nfs
```bash
docker run -d --name cinder-volume-smbfs \
    --privileged \
    -v /opt/openstack/cinder-volume-smbfs/:/etc/cinder \
    -e CINDER_DB=10.64.0.52 \
    -e CINDER_DBPASS=cinder_dbpass \
    -e RABBIT_HOST=10.64.0.52 \
    -e RABBIT_USERID=openstack \
    -e RABBIT_PASSWORD=openstack \
    -e KEYSTONE_INTERNAL_ENDPOINT=10.64.0.52 \
    -e KEYSTONE_ADMIN_ENDPOINT=10.64.0.52 \
    -e CINDER_PASS=cinder \
    -e MY_IP=10.64.0.52 \
    -e GLANCE_HOST=10.64.0.52 \
    -e VOLUME_BACKEND_NAME=one \
    -e NFS_SERVER=smbfs_server_ip \
    -e SMB_PASS=123456
    10.64.0.50:5000/lzh/cinder-volume-smbfs:liberty
```

# 在smbfs 节点上安装smbfs server
```bash
yum install samba cifs-utils

mkdir /volume

cat /etc/samba/smb.conf
...
[volume]
        path=/volume
        available = yes
        browseable = yes
        public = yes
        writable = yes

smbpasswd -a root
```

# 配置nova-compute 使用 smbfs
## 在nova-compute节点上安装smbfs client
### centos
```bash
yum install cifs-utils
```
## 配置nova-compute
```bash
cat /etc/nova/nova.conf
[libvirt]
...
smbfs_mount_options = -o username=root,password=123456
```

# 使用多后端cinder
```bash
cat <<EOF>>admin-openrc.sh 
#export OS_TENANT_NAME=admin
export OS_IDENTITY_API_VERSION=3
export OS_USERNAME=admin
export OS_PASSWORD=123456
export OS_USER_DOMAIN_ID=default
export OS_PROJECT_NAME=admin
export OS_PROJECT_DOMAIN_ID=default
#export OS_TENANT_ID=admin
export OS_AUTH_URL=http://10.64.0.52:35357/v3
EOF

source admin-rc.sh

cinder --os-username admin --os-tenant-name admin  --os-volume-api-version 2 type-create one
cinder --os-username admin --os-tenant-name admin  --os-volume-api-version 2 \
       type-key one set volume_backend_name=one
cinder --os-username admin --os-tenant-name admin  --os-volume-api-version 2 extra-specs-list
```