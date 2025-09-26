# KidsPOS-Server 現場展開ガイド

## 概要
KidsPOS-Serverを実際の現場環境で展開するための詳細手順です。

## 事前準備

### システム要件
- **ハードウェア**: Raspberry Pi 4 (4GB RAM推奨)
- **OS**: Raspberry Pi OS (64-bit推奨) または Ubuntu 20.04 LTS for ARM
- **CPU**: ARM Cortex-A72 (Raspberry Pi 4)
- **メモリ**: 4GB以上推奨
- **ストレージ**: 32GB以上のmicroSDカード (Class 10以上)
- **ネットワーク**: 無線LAN（イントラネット構成・インターネット接続不要）

### 必要なソフトウェア
- Java 11以上 (OpenJDK推奨)
- Spring Boot アプリケーション
- データベース (組み込みH2またはMySQL)
- Git

## インストール手順

### 1. Raspberry Pi のセットアップ
```bash
# システムの更新
sudo apt update && sudo apt upgrade -y

# 必要なパッケージのインストール
sudo apt install -y git vim curl wget
```

### 2. Java のインストール
```bash
# OpenJDK 11のインストール
sudo apt install -y openjdk-11-jdk

# バージョン確認
java -version
javac -version

# JAVA_HOME環境変数の設定
echo 'export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-arm64' >> ~/.bashrc
source ~/.bashrc
```

### 3. KidsPOS-Server のクローンとビルド
```bash
cd /opt
sudo git clone https://github.com/KidsPOSProject/KidsPOS-Server.git
sudo chown -R $USER:$USER KidsPOS-Server
cd KidsPOS-Server

# Gradleでビルド（Gradleが含まれている場合）
./gradlew build

# または、Mavenでビルド（pom.xmlがある場合）
mvn clean package
```

### 4. 設定ファイルの設定
```bash
# application.propertiesまたはapplication.ymlを編集
cp src/main/resources/application.properties.example src/main/resources/application.properties
```

`application.properties` ファイルを編集：
```properties
# サーバー設定
server.port=8080
server.address=0.0.0.0

# データベース設定（H2組み込みデータベースの場合）
spring.datasource.url=jdbc:h2:file:./data/kidspos
spring.datasource.driver-class-name=org.h2.Driver
spring.jpa.hibernate.ddl-auto=update

# または MySQL使用の場合
# spring.datasource.url=jdbc:mysql://localhost:3306/kidspos
# spring.datasource.username=kidspos
# spring.datasource.password=your_password

# ログ設定
logging.level.root=INFO
logging.file.name=logs/kidspos.log
```

### 5. サービス化設定
```bash
# systemdサービスファイルを作成
sudo tee /etc/systemd/system/kidspos-server.service << 'EOF'
[Unit]
Description=KidsPOS Server
After=network.target

[Service]
Type=simple
User=pi
WorkingDirectory=/opt/KidsPOS-Server
ExecStart=/usr/bin/java -jar target/kidspos-server-*.jar
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF

# サービス有効化
sudo systemctl daemon-reload
sudo systemctl enable kidspos-server
```

## 現場展開時の確認事項

### ネットワーク設定
```bash
# 無線LAN設定確認
iwconfig wlan0
ip addr show wlan0

# ファイアウォール設定（UFWを使用）
sudo ufw allow 8080/tcp
sudo ufw allow ssh
sudo ufw --force enable

# ポート確認
sudo netstat -tlnp | grep :8080
```

### 周辺機器設定
```bash
# レシートプリンター設定（有線LAN接続）
# プリンターのIPアドレスを確認
nmap -sn 192.168.1.0/24

# CUPS設定（必要に応じて）
sudo apt install -y cups
sudo systemctl enable cups
sudo systemctl start cups
```

### データベース初期化
```bash
# H2データベースの場合は自動で作成される
# MySQLを使用する場合
sudo apt install -y mysql-server
sudo mysql_secure_installation

# データベースとユーザーの作成
sudo mysql -u root -p << 'EOF'
CREATE DATABASE kidspos;
CREATE USER 'kidspos'@'localhost' IDENTIFIED BY 'your_password';
GRANT ALL PRIVILEGES ON kidspos.* TO 'kidspos'@'localhost';
FLUSH PRIVILEGES;
EOF
```

## 起動と動作確認

### 1. サーバー起動
```bash
# systemdサービスで起動
sudo systemctl start kidspos-server

# 自動起動設定
sudo systemctl enable kidspos-server

# 状態確認
sudo systemctl status kidspos-server
```

### 2. 動作確認
```bash
# プロセス確認
ps aux | grep java

# ログ確認
sudo journalctl -u kidspos-server -f

# ヘルスチェック
curl http://localhost:8080/actuator/health
```

### 3. API動作テスト
```bash
# 基本APIテスト
curl -X GET http://localhost:8080/api/status
curl -X GET http://localhost:8080/api/version
```

## 運用監視

### ログ管理
```bash
# systemdログの確認
sudo journalctl -u kidspos-server --since "1 hour ago"

# アプリケーションログの確認
tail -f /opt/KidsPOS-Server/logs/kidspos.log

# ログローテーション設定
sudo tee /etc/logrotate.d/kidspos << 'EOF'
/opt/KidsPOS-Server/logs/*.log {
    daily
    rotate 30
    compress
    delaycompress
    missingok
    notifempty
    copytruncate
}
EOF
```

### システム監視
```bash
# リソース使用量確認
htop
df -h
free -h

# Raspberry Pi固有の監視
vcgencmd measure_temp  # CPU温度
vcgencmd get_throttled # スロットリング状態
```

## トラブルシューティング

### よくある問題

1. **ポート8080が使用中**
   ```bash
   sudo lsof -i :8080
   # プロセスを確認して必要に応じて停止
   ```

2. **Java OutOfMemoryError**
   ```bash
   # JVMメモリ設定を調整
   # /etc/systemd/system/kidspos-server.service の ExecStart に追加:
   # -Xmx1g -Xms512m
   ```

3. **データベース接続エラー**
   ```bash
   # H2データベースファイルの権限確認
   ls -la /opt/KidsPOS-Server/data/
   sudo chown -R pi:pi /opt/KidsPOS-Server/data/
   ```

4. **無線LAN接続不安定**
   ```bash
   # WiFi省電力モード無効化
   sudo tee /etc/systemd/system/wifi-powersave-off.service << 'EOF'
[Unit]
Description=Turn off WiFi power saving
After=multi-user.target

[Service]
Type=oneshot
ExecStart=/sbin/iw dev wlan0 set power_save off

[Install]
WantedBy=multi-user.target
EOF
   sudo systemctl enable wifi-powersave-off
   ```

### ログ確認コマンド
```bash
# アプリケーションログ
tail -f /opt/KidsPOS-Server/logs/kidspos.log

# システムログ
sudo journalctl -u kidspos-server -f
sudo dmesg | tail
```

## 緊急時対応

### サービス再起動
```bash
sudo systemctl restart kidspos-server
sudo systemctl restart networking
```

### バックアップ
```bash
# データベースバックアップ（H2の場合）
cp -R /opt/KidsPOS-Server/data/ /backup/$(date +%Y%m%d)/

# MySQLの場合
mysqldump -u kidspos -p kidspos > /backup/kidspos-$(date +%Y%m%d).sql

# アプリケーションコードバックアップ
tar -czf /backup/kidspos-server-$(date +%Y%m%d).tar.gz /opt/KidsPOS-Server
```

## チェックリスト

現場展開時の確認事項：

- [ ] Raspberry Pi 4の動作確認
- [ ] Java 11インストール完了
- [ ] KidsPOS-Serverビルド成功
- [ ] 設定ファイル設定完了
- [ ] 無線LAN接続確認
- [ ] レシートプリンター接続確認
- [ ] ファイアウォール設定
- [ ] サーバー起動確認
- [ ] API動作確認
- [ ] ログ出力確認
- [ ] 監視設定完了
- [ ] バックアップ設定完了
- [ ] 緊急時手順書準備

---

**注意**: 本手順はRaspberry Pi + SpringBootを前提としています。実際の現場環境に応じて適宜調整してください。