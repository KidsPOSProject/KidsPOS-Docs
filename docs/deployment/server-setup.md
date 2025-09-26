# KidsPOS-Server 現場展開ガイド

## 概要
KidsPOS-Serverを実際の現場環境で展開するための詳細手順です。

## 事前準備

### システム要件
- **OS**: Ubuntu 20.04 LTS またはCentOS 8以上推奨
- **CPU**: 2コア以上
- **メモリ**: 4GB以上推奨
- **ストレージ**: 50GB以上の空き容量
- **ネットワーク**: インターネット接続必須

### 必要なソフトウェア
- Node.js 14.x以上
- MongoDB 4.4以上
- Git
- PM2 (プロセス管理)

## インストール手順

### 1. システムの更新
```bash
# Ubuntu/Debian
sudo apt update && sudo apt upgrade -y

# CentOS/RHEL
sudo yum update -y
# または
sudo dnf update -y
```

### 2. Node.js のインストール
```bash
# NodeSourceからインストール（推奨）
curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
sudo apt-get install -y nodejs

# バージョン確認
node --version
npm --version
```

### 3. MongoDB のインストール
```bash
# MongoDB公式リポジトリを追加
wget -qO - https://www.mongodb.org/static/pgp/server-5.0.asc | sudo apt-key add -
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/5.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-5.0.list

# インストール
sudo apt-get update
sudo apt-get install -y mongodb-org

# 自動起動設定
sudo systemctl enable mongod
sudo systemctl start mongod
```

### 4. KidsPOS-Server のクローン
```bash
cd /opt
sudo git clone https://github.com/KidsPOSProject/KidsPOS-Server.git
sudo chown -R $USER:$USER KidsPOS-Server
cd KidsPOS-Server
```

### 5. 依存関係のインストール
```bash
npm install
```

### 6. 環境設定ファイルの作成
```bash
cp .env.example .env
```

`.env` ファイルを編集：
```env
# データベース設定
MONGODB_URI=mongodb://localhost:27017/kidspos
DB_NAME=kidspos

# サーバー設定
PORT=3000
NODE_ENV=production

# セキュリティ
JWT_SECRET=your-super-secret-jwt-key-change-this
SESSION_SECRET=your-super-secret-session-key-change-this

# その他の設定
LOG_LEVEL=info
```

### 7. PM2 の設定
```bash
# PM2をグローバルインストール
sudo npm install -g pm2

# ecosystem.config.js を作成
cat > ecosystem.config.js << 'EOF'
module.exports = {
  apps: [{
    name: 'kidspos-server',
    script: 'app.js',
    instances: 1,
    autorestart: true,
    watch: false,
    max_memory_restart: '1G',
    env: {
      NODE_ENV: 'production'
    }
  }]
}
EOF
```

## 現場展開時の確認事項

### ネットワーク設定
```bash
# ファイアウォール設定（Ubuntu）
sudo ufw allow 3000/tcp
sudo ufw allow ssh
sudo ufw --force enable

# ポート確認
sudo netstat -tlnp | grep :3000
```

### セキュリティ設定
```bash
# SSL証明書の設置（Let's Encrypt使用例）
sudo apt install certbot
sudo certbot certonly --standalone -d your-domain.com

# Nginxリバースプロキシ設定（オプション）
sudo apt install nginx
```

### データベース初期化
```bash
# MongoDB接続確認
mongo --eval "db.adminCommand('ismaster')"

# 初期データの投入（必要に応じて）
npm run db:seed
```

## 起動と動作確認

### 1. サーバー起動
```bash
# PM2で起動
pm2 start ecosystem.config.js

# 自動起動設定
pm2 startup
pm2 save
```

### 2. 動作確認
```bash
# プロセス確認
pm2 status

# ログ確認
pm2 logs kidspos-server

# ヘルスチェック
curl http://localhost:3000/health
```

### 3. API動作テスト
```bash
# 基本APIテスト
curl -X GET http://localhost:3000/api/status
curl -X GET http://localhost:3000/api/version
```

## 運用監視

### ログ管理
```bash
# PM2ログローテーション
pm2 install pm2-logrotate
pm2 set pm2-logrotate:max_size 10M
pm2 set pm2-logrotate:retain 30
```

### システム監視
```bash
# リソース使用量確認
pm2 monit

# システム負荷確認
htop
df -h
free -h
```

## トラブルシューティング

### よくある問題

1. **ポート3000が使用中**
   ```bash
   sudo lsof -i :3000
   # プロセスを確認して必要に応じて停止
   ```

2. **MongoDB接続エラー**
   ```bash
   sudo systemctl status mongod
   sudo systemctl restart mongod
   ```

3. **メモリ不足**
   ```bash
   # スワップファイル作成
   sudo fallocate -l 2G /swapfile
   sudo chmod 600 /swapfile
   sudo mkswap /swapfile
   sudo swapon /swapfile
   ```

### ログ確認コマンド
```bash
# アプリケーションログ
pm2 logs kidspos-server --lines 100

# システムログ
sudo journalctl -u mongod -f
tail -f /var/log/nginx/error.log
```

## 緊急時対応

### サービス再起動
```bash
pm2 restart kidspos-server
sudo systemctl restart mongod
sudo systemctl restart nginx
```

### バックアップ
```bash
# データベースバックアップ
mongodump --db kidspos --out /backup/$(date +%Y%m%d)

# アプリケーションコードバックアップ
tar -czf /backup/kidspos-server-$(date +%Y%m%d).tar.gz /opt/KidsPOS-Server
```

## チェックリスト

現場展開時の確認事項：

- [ ] システム要件確認
- [ ] ソフトウェアインストール完了
- [ ] 環境設定ファイル設定
- [ ] データベース接続確認
- [ ] ファイアウォール設定
- [ ] サーバー起動確認
- [ ] API動作確認
- [ ] ログ出力確認
- [ ] 監視設定完了
- [ ] バックアップ設定完了
- [ ] 緊急時手順書準備

---

**注意**: 本手順は一般的な環境を想定しています。実際の現場環境に応じて適宜調整してください。