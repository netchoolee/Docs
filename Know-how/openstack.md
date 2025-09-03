# Openstack Console 접속
```bash
1. VM이 운영중인 Host 확인 & VM instance name 확인 (openstack server show VM_NAME)
2. 해당 Host에 있는 libvirt Pod에 접속
3. virsh list --all
4. virsh console Instance_name
5. 종료: ^]
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
# Openstack Octavia & Amphora
## Amphora Failover 단계
**openstack loadbalancer amphora failover <amphora_id>** -> 여기서 <amphora_id>는 ERROR 상태인 Amphora의 ID
```bash
Octavia Health Manager가 Failover 작업을 조율
1. Backup Amphora가 즉시 Active 역활을 인계받아 트래픽을 처리 (VRRP를 통해 VIP 전환, 초단위 중단 발생)
2. 실패한 Master Amphora 삭제
3. 새로운 Amphora 생성 및 구성
4. 새로 생성된 Amphora는 Standby(Backup) 역활로 배치
```
## Amphora failover와 lb failover의 차이점
**openstack loadbalancer amphora failover <amphora_id>:**
```bash
목적: 특정 Amphora 인스턴스(VM 또는 컨테이너)를 강제로 failover합니다. 이는 개별 Amphora의 문제를 해결하거나, 특정 호스트에서 Amphora를 이전하는 데 초점을 맞춥니다.
범위: 단일 Amphora에 한정됩니다. 로드밸런서 전체에 영향을 주지 않고, 대상 Amphora만 대체합니다.
주요 용도: Amphora 수준의 유지보수나 복구(예: 호스트 실패 시 Amphora 이전).
```
**openstack loadbalancer failover <loadbalancer_id>:**
```bash
목적: 전체 로드밸런서를 강제로 failover합니다. 이는 로드밸런서와 연관된 모든 Amphora를 재구성하여 서비스 연속성을 보장합니다.
범위: 로드밸런서 전체(listeners, pools, members, Amphora 등)를 대상으로 합니다. ACTIVE/STANDBY 토폴로지에서 여러 Amphora가 관련된 경우 모두 처리할 수 있습니다.
주요 용도: 로드밸런서 수준의 업데이트나 복구(예: Amphora 이미지 로테이션, 인증서 변경).
```
| 항목                  | openstack loadbalancer amphora failover <amphora_id> | openstack loadbalancer failover <loadbalancer_id> |
|-----------------------|-----------------------------------------------------|--------------------------------------------------|
| **대상 범위**        | 단일 Amphora                                      | 전체 로드밸런서 (여러 Amphora 포함)               |
| **인수**             | Amphora UUID                                      | 로드밸런서 UUID 또는 이름                        |
| **프로세스 초점**    | 개별 Amphora 삭제 및 새 생성                      | 전체 구성 재적용 및 Amphora 재생성               |
| **서비스 영향**      | 최소 중단 (초 단위)                               | Standalone: 1-2분 중단; ACTIVE/STANDBY: 최소    |
| **시스템 부하**      | 낮음                                              | 높음 (새 VM/포트 생성)                           |
| **주요 사용 사례**   | 호스트 유지보수, 단일 실패 복구                   | 이미지 로테이션, 인증서 업데이트, 다중 실패 복구 |
| **주의사항**         | Stale Amphora 수동 관리                           | 대량 failover 시 throttling 필수                 |
```
