# Openstack Console 접속
```bash
1. VM이 운영중인 Host 확인 & VM instance name 확인 (openstack server show VM_NAME)
2. 해당 Host에 있는 libvirt Pod에 접속
3. virsh console Instance_name
4. 종료: ^]
```
# cloud-init
```bash
#cloud-config
users:
  - name: root
    lock_passwd: false
    plain_text_passwd: 'cisco123'
ssh_pwauth: True
runcmd:
  - sed -i 's/#PermitRootLogin.*/PermitRootLogin yes/g' /etc/ssh/sshd_config
  - sed -i 's/#UseLogin.*/UseLogin yes/g' /etc/ssh/sshd_config
  - sed -i 's/#UseDNS.*/UseDNS no/g' /etc/ssh/sshd_config
  - systemctl restart ssh
```
# Openstack Command
## VM
```bash
openstack server create --image 'Ubuntu 22.04' --flavor "s1v1m2" --nic net-id="choolee-net" --user-data ~/cloud-init --boot-from-volume 10 choolee-vm
openstack server add security group choolee-vm choolee-sg
# VM Cold migration
openstack server migrate choolee-vm
nova resize-confirm <vm-id> ## 콜드 마이그레이션 후, VM 상태가 VERIFY_RESIZE로 변경 -> 이를 확인하고 승인
```
## Security Group
```bash
openstack security group rule create --protocol "tcp" --remote-ip '0.0.0.0/0' --egress choolee-sg
```
