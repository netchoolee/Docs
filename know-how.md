# K9s 설치
```bash
cd ~
mkdir k9s
cd k9s
wget https://github.com/derailed/k9s/releases/download/v0.32.5/k9s_Linux_amd64.tar.gz
tar zxvf k9s_Linux_amd64.tar.gz
mv k9s /usr/local/bin/k9s
```

# Mgen Tools 설치 방법
## Ubuntu
```bash
apt update
dpkg --configure -a
apt install build-essential
apt install libpcap-dev
apt install libssh-dev

mkdir -p ~/src/mgen_build
cd ~/src/mgen_build
git clone https://github.com/pjhwa/mgen.git
cd ~/src/mgen_build/mgen/makefiles
make -f Makefile.linux
cp ./mgen /usr/local/bin/mgen
chmod +x /usr/local/bin/mgen
```

# Command
Sender:
```bash
mgen input sender.mgn txlog output sender.log
```
Receiver:
```bash
mgen input receiver.mgn output receiver.log
```

# receive.mgn
```bash
# Global Commands
START NOW

# 수신 버퍼 크기 설정
RXBUFFER 65536

# 내장 분석 활성화 (loss, latency 자동 계산)
ANALYTICS
# 분석 윈도우 1초 (지터/손실률 측정 간격)
WINDOW 1

# Reception Events
# 포트 5000 수신
0.0 LISTEN UDP 20003
```

# sender.mgn
```bash
# Global Commands
START NOW

# 송/수신 버퍼 크기 설정 (성능 최적화)
TXBUFFER 65536
RXBUFFER 65536

# Broadcast 활성화 (기본 ON)
BROADCAST ON

# Broadcast는 TTL 1로 충분
TTL 1

# Transmission Events
# 1000 packets/sec, 1024 bytes each (10MBytes)
0.0 ON 1 UDP DST 172.16.3.255/20003 PERIODIC [1 100]

# 10초 후 종료 (테스트 기간 제한)
4.0 OFF 1
```

</xaiArtifact>
