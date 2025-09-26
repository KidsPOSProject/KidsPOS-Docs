# KidsPOS ネットワーク設定ガイド

## 概要
KidsPOSシステムを安定して運用するためのネットワーク設定手順です。

## ネットワーク要件

### 基本要件
- **構成**: イントラネット専用（インターネット接続なし）
- **帯域幅**: 最低10Mbps（推奨20Mbps以上）
- **遅延**: 50ms以下（推奨20ms以下）
- **可用性**: 99%以上
- **同時接続**: 最大20台のAndroidタブレット（Nexus 7相当）

### ポート要件
| サービス | ポート | プロトコル | 用途 |
|---------|--------|-----------|------|
| KidsPOS-Server | 8080 | TCP | HTTP API (SpringBoot) |
| SSH | 22 | TCP | Raspberry Pi リモート管理 |
| レシートプリンター | 631/9100 | TCP | IPP/AppSocket印刷 |

## ネットワーク構成例

### 小規模構成（1-5台）
```
[無線LANルーター] ── [Raspberry Pi (KidsPOS-Server)]
    ↓               ── [レシートプリンター (有線LAN)]
[WiFiアクセスポイント]
    ↓
[Nexus 7 タブレット群]
```

### 中規模構成（6-15台）
```
[無線LANルーター/スイッチ] ── [Raspberry Pi (KidsPOS-Server)]
    ↓                      ── [レシートプリンター × 複数 (有線LAN)]
    ↓                      ── [管理用PC]
[WiFiアクセスポイント × 2]
    ↓
[Nexus 7 タブレット群（エリア分割）]
```

### 大規模構成（16-20台）
```
[メインスイッチ] ── [Raspberry Pi (KidsPOS-Server)]
    ↓             ── [レシートプリンター × 複数]
    ↓             ── [管理用PC]
    ↓             ── [予備Raspberry Pi (冗長化)]
[WiFiコントローラー]
    ↓
[複数のアクセスポイント（エリア分割）]
    ↓
[Nexus 7 タブレット群]
```

## IPアドレス設計

### サブネット設計例
```
サーバーセグメント: 192.168.1.0/28
- Raspberry Pi (KidsPOS-Server): 192.168.1.10
- 管理用PC: 192.168.1.11
- 予備Raspberry Pi: 192.168.1.12

デバイスセグメント: 192.168.2.0/24
- Nexus 7 タブレット: 192.168.2.10-192.168.2.29
- WiFiアクセスポイント: 192.168.2.1

プリンターセグメント: 192.168.3.0/28
- レシートプリンター: 192.168.3.10-192.168.3.15
```

### DHCPスコープ設定
```
スコープ名: KidsPOS-Tablets
範囲: 192.168.2.10 - 192.168.2.29
サブネットマスク: 255.255.255.0
デフォルトゲートウェイ: 192.168.2.1
DNSサーバー: 192.168.1.1 (ルーター)
リース期間: 24時間（固定端末のため長期設定）
```

## WiFi設定

### アクセスポイント設定
```
SSID: KidsPOS-Network
セキュリティ: WPA2-PSK (AES)
チャンネル: 自動（DFS対応）
出力: 中程度（干渉を避けるため）
帯域: 2.4GHz + 5GHz デュアルバンド
```

### 推奨WiFi設定
```json
{
  "ssid": "KidsPOS-Network",
  "security": "WPA2-PSK",
  "password": "strong_password_here",
  "channel_2.4ghz": "auto",
  "channel_5ghz": "auto",
  "max_clients": 30,
  "isolation": false
}
```

### 複数AP環境での設定
- **同一SSID**: 端末のローミングを考慮
- **チャンネル計画**: 1,6,11チャンネルで配置
- **電波強度調整**: 隣接APとの干渉を最小化
- **ロードバランシング**: 端末数の均等分散

## ファイアウォール設定

### 基本ルール
```bash
# デフォルトポリシー
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

# ローカルループバック許可
iptables -A INPUT -i lo -j ACCEPT

# 確立された接続の許可
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# SSH（管理用）
iptables -A INPUT -p tcp --dport 22 -s 192.168.10.0/28 -j ACCEPT

# KidsPOS-Server API
iptables -A INPUT -p tcp --dport 8080 -s 192.168.2.0/24 -j ACCEPT

# MongoDB（ローカルのみ）
iptables -A INPUT -p tcp --dport 27017 -s 127.0.0.1 -j ACCEPT
```

### セキュリティ強化設定
```bash
# DDoS対策
iptables -A INPUT -p tcp --dport 8080 -m limit --limit 25/minute --limit-burst 100 -j ACCEPT

# ポートスキャン対策
iptables -A INPUT -m recent --name portscan --rcheck --seconds 86400 -j DROP
iptables -A INPUT -m recent --name portscan --set -j LOG --log-prefix "Portscan:"

# 不正アクセス監視
iptables -A INPUT -m recent --name blacklist --rcheck --seconds 3600 -j DROP
```

## QoS設定

### 帯域制御
```
高優先度: KidsPOS API通信（ポート8080）
中優先度: 管理トラフィック（SSH、SNMP）
低優先度: その他の通信
```

### 設定例（Linux tc）
```bash
# インターフェース設定
tc qdisc add dev eth0 root handle 1: htb default 30

# クラス設定
tc class add dev eth0 parent 1: classid 1:1 htb rate 50mbit
tc class add dev eth0 parent 1:1 classid 1:10 htb rate 30mbit ceil 40mbit # KidsPOS
tc class add dev eth0 parent 1:1 classid 1:20 htb rate 10mbit ceil 20mbit # 管理
tc class add dev eth0 parent 1:1 classid 1:30 htb rate 10mbit ceil 50mbit # その他

# フィルター設定
tc filter add dev eth0 protocol ip parent 1:0 prio 1 u32 match ip dport 8080 0xffff flowid 1:10
```

## 監視設定

### ネットワーク監視項目
- **帯域使用率**: 80%以下を維持
- **パケットロス率**: 1%以下
- **遅延時間**: 50ms以下
- **接続数**: 同時接続数の監視

### SNMP設定
```bash
# SNMP v3設定例
net-snmp-config --create-snmpv3-user -ro -A password123 -X password456 -a SHA -x AES kidspos-monitor
```

### 監視スクリプト例
```bash
#!/bin/bash
# network_monitor.sh

# 遅延測定
LATENCY=$(ping -c 3 8.8.8.8 | tail -1 | awk '{print $4}' | cut -d '/' -f 2)
if (( $(echo "$LATENCY > 100" | bc -l) )); then
    echo "High latency detected: ${LATENCY}ms"
fi

# 帯域使用率
RX_BYTES=$(cat /sys/class/net/eth0/statistics/rx_bytes)
TX_BYTES=$(cat /sys/class/net/eth0/statistics/tx_bytes)
echo "Network usage: RX=${RX_BYTES}, TX=${TX_BYTES}"

# 接続数確認
CONNECTIONS=$(netstat -an | grep :8080 | grep ESTABLISHED | wc -l)
echo "Active connections: $CONNECTIONS"
```

## トラブルシューティング

### 接続問題の診断
```bash
# 基本的な接続確認
ping [サーバーIP]
telnet [サーバーIP] 8080

# ルーティング確認
traceroute [サーバーIP]
ip route show

# DNS確認
nslookup [サーバードメイン]
dig [サーバードメイン]
```

### WiFi問題の診断
```bash
# 電波状況確認
iwconfig wlan0
iwlist wlan0 scan | grep -E "ESSID|Quality"

# 接続ログ確認
dmesg | grep wlan
journalctl -u wpa_supplicant
```

### パフォーマンス測定
```bash
# 帯域幅測定
iperf3 -c [サーバーIP] -t 30

# パケットキャプチャ
tcpdump -i eth0 -w capture.pcap port 8080

# ネットワーク統計
ss -tuln
netstat -i
```

## セキュリティ対策

### 基本セキュリティ
- **WPA2-PSK**: 強固なパスワード設定
- **MAC認証**: 必要に応じて端末制限
- **VLAN分離**: セグメント間の通信制御
- **ファイアウォール**: 不要ポートの閉塞

### 高度なセキュリティ
```bash
# 侵入検知システム（例：fail2ban）
sudo apt install fail2ban

# 設定例
cat > /etc/fail2ban/jail.local << 'EOF'
[DEFAULT]
bantime = 3600
findtime = 600
maxretry = 3

[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log

[kidspos-api]
enabled = true
port = 8080
filter = kidspos-api
logpath = /var/log/kidspos/access.log
maxretry = 10
EOF
```

## 運用手順

### 日次チェック
- [ ] ネットワーク機器の稼働確認
- [ ] 帯域使用率の確認
- [ ] エラーログの確認
- [ ] 接続数の確認

### 週次チェック
- [ ] パフォーマンス測定
- [ ] セキュリティログの確認
- [ ] 設定バックアップ
- [ ] ファームウェア更新確認

### 月次チェック
- [ ] ネットワーク構成の見直し
- [ ] 容量計画の確認
- [ ] セキュリティポリシーの見直し
- [ ] 災害対策の確認

## 緊急時対応

### ネットワーク障害時の対処
1. **問題の切り分け**
   - 物理層の確認
   - 論理層の確認
   - アプリケーション層の確認

2. **一次対応**
   - 機器の再起動
   - ケーブル接続の確認
   - 設定の確認

3. **代替手段**
   - モバイルルーターの活用
   - 有線接続への切り替え
   - オフラインモードでの運用

---

**ネットワーク設定は環境によって大きく異なります。実際の現場に応じて適切に調整してください。**