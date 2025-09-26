# KidsPOS トラブルシューティングガイド

## 概要
KidsPOSシステムで発生する一般的な問題とその解決方法をまとめています。

## 🔍 問題診断フローチャート

```
問題発生
    ↓
サーバー側の問題？
    ↓ Yes
サーバー関連対応へ
    ↓ No
Android端末の問題？
    ↓ Yes  
Android関連対応へ
    ↓ No
ネットワーク関連対応へ
```

## 🖥️ サーバー関連の問題

### 1. サーバーが起動しない

#### 症状
- `http://[サーバーIP]:3000` にアクセスできない
- PM2で「stopped」状態

#### 診断コマンド
```bash
# プロセス状態確認
pm2 status

# システム状態確認
sudo systemctl status mongod
sudo netstat -tlnp | grep :3000

# ログ確認
pm2 logs kidspos-server
sudo journalctl -u mongod -f
```

#### 解決方法
1. **MongoDB起動確認**
   ```bash
   sudo systemctl start mongod
   sudo systemctl enable mongod
   ```

2. **アプリケーション再起動**
   ```bash
   pm2 restart kidspos-server
   ```

3. **ポート競合確認**
   ```bash
   sudo lsof -i :3000
   # 競合プロセスがあれば停止
   sudo kill -9 [PID]
   ```

### 2. データベース接続エラー

#### 症状
- "MongoDB connection failed" エラー
- データの読み書きができない

#### 診断コマンド
```bash
# MongoDB接続テスト
mongo --eval "db.adminCommand('ismaster')"

# 設定ファイル確認
cat .env | grep MONGODB

# MongoDB状態確認
sudo systemctl status mongod
```

#### 解決方法
1. **MongoDB再起動**
   ```bash
   sudo systemctl restart mongod
   ```

2. **接続設定確認**
   ```env
   MONGODB_URI=mongodb://localhost:27017/kidspos
   ```

3. **ディスク容量確認**
   ```bash
   df -h
   # /var/lib/mongodb のディスク使用量確認
   ```

### 3. 高負荷・レスポンス遅延

#### 症状
- API応答時間が5秒以上
- CPU使用率が常に80%以上

#### 診断コマンド
```bash
# システムリソース確認
htop
free -h
iostat 1 5

# アプリケーション監視
pm2 monit
```

#### 解決方法
1. **メモリ増設の検討**
   ```bash
   # スワップファイル作成（一時的対策）
   sudo fallocate -l 2G /swapfile
   sudo chmod 600 /swapfile
   sudo mkswap /swapfile
   sudo swapon /swapfile
   ```

2. **PM2クラスター設定**
   ```javascript
   // ecosystem.config.js
   module.exports = {
     apps: [{
       name: 'kidspos-server',
       script: 'app.js',
       instances: 'max', // CPUコア数に応じて自動調整
       exec_mode: 'cluster'
     }]
   }
   ```

## 📱 Android端末関連の問題

### 1. アプリがインストールできない

#### 症状
- APKインストール時にエラー
- "アプリがインストールされていません"

#### 診断コマンド
```bash
# デバイス接続確認
adb devices

# ストレージ容量確認
adb shell df -h

# 署名確認
keytool -printcert -jarfile app-release.apk
```

#### 解決方法
1. **ストレージ容量確保**
   ```bash
   # 不要ファイル削除
   adb shell pm clear com.android.browser
   adb shell rm -rf /sdcard/Download/*
   ```

2. **セキュリティ設定確認**
   - 設定 → セキュリティ → 提供元不明のアプリを有効化

3. **古いバージョン削除**
   ```bash
   adb uninstall com.kidspos.android
   adb install app-release.apk
   ```

### 2. QRコード読み取りができない

#### 症状
- カメラが起動しない
- QRコードを認識しない

#### 診断手順
1. **カメラ権限確認**
   ```bash
   adb shell dumpsys package com.kidspos.android | grep CAMERA
   ```

2. **カメラテスト**
   - 標準カメラアプリで動作確認

#### 解決方法
1. **権限の再設定**
   - アプリ情報 → 権限 → カメラを有効化

2. **アプリデータクリア**
   ```bash
   adb shell pm clear com.kidspos.android
   ```

3. **端末再起動**
   ```bash
   adb reboot
   ```

### 3. サーバー接続エラー

#### 症状
- "サーバーに接続できません"エラー
- データ送信が失敗する

#### 診断コマンド
```bash
# ネットワーク接続確認
adb shell ping 8.8.8.8
adb shell ping [サーバーIP]

# ポート接続確認
adb shell nc -zv [サーバーIP] 3000

# DNS確認
adb shell nslookup [サーバードメイン]
```

#### 解決方法
1. **WiFi再接続**
   ```bash
   adb shell svc wifi disable
   adb shell svc wifi enable
   ```

2. **アプリ設定確認**
   - サーバーURL設定の確認
   - HTTPSではなくHTTPで接続確認

3. **ファイアウォール確認**
   ```bash
   # サーバー側でファイアウォール確認
   sudo ufw status
   sudo iptables -L
   ```

## 🌐 ネットワーク関連の問題

### 1. WiFi接続が不安定

#### 症状
- 接続が頻繁に切れる
- 通信速度が極端に遅い

#### 診断手順
```bash
# 電波強度確認
adb shell dumpsys wifi | grep RSSI

# 接続履歴確認
adb shell logcat | grep WifiManager
```

#### 解決方法
1. **WiFiチャンネル変更**
   - ルーター設定で混雑していないチャンネルに変更

2. **省電力設定無効化**
   - WiFi詳細設定で省電力モードを無効化

3. **DNS設定変更**
   ```
   プライマリDNS: 8.8.8.8
   セカンダリDNS: 8.8.4.4
   ```

### 2. 複数端末での接続問題

#### 症状
- 端末数が多いと接続が不安定
- 一部端末だけ接続エラー

#### 解決方法
1. **ルーター設定見直し**
   ```
   • 最大接続数設定の確認
   • QoS設定の最適化
   • 帯域制限の調整
   ```

2. **負荷分散**
   - 複数アクセスポイントの使用
   - ネットワーク分離の検討

## 🔧 パフォーマンス改善

### システム最適化チェックリスト

#### サーバー側
- [ ] CPUとメモリ使用率が80%未満
- [ ] ディスク使用率が70%未満
- [ ] ログローテーション設定済み
- [ ] データベースインデックス最適化
- [ ] 不要なプロセス停止

#### Android端末側
- [ ] 不要なアプリの削除
- [ ] バックグラウンドアプリの制限
- [ ] 自動更新の無効化
- [ ] キャッシュクリア実行
- [ ] 再起動スケジュール設定

## 📊 監視とログ

### 重要なログファイル

#### サーバー側
```bash
# アプリケーションログ
~/.pm2/logs/kidspos-server-out.log
~/.pm2/logs/kidspos-server-error.log

# システムログ
/var/log/mongodb/mongod.log
/var/log/nginx/access.log (使用時)
```

#### Android端末側
```bash
# アプリケーションログ
adb logcat -s KidsPOS

# システムログ
adb logcat -b system
```

### 監視スクリプト例

```bash
#!/bin/bash
# health_check.sh

# サーバー死活監視
curl -f http://localhost:3000/health || echo "Server down!"

# メモリ使用量監視
MEMORY_USAGE=$(free | grep Mem | awk '{print ($3/$2) * 100.0}')
if (( $(echo "$MEMORY_USAGE > 80" | bc -l) )); then
    echo "High memory usage: $MEMORY_USAGE%"
fi

# ディスク使用量監視
DISK_USAGE=$(df / | awk 'NR==2 {print $5}' | sed 's/%//')
if [ $DISK_USAGE -gt 80 ]; then
    echo "High disk usage: $DISK_USAGE%"
fi
```

## 🚨 緊急時対応

### エスカレーション基準
- **レベル1**: 再起動で解決可能
- **レベル2**: 設定変更が必要
- **レベル3**: システム管理者が必要
- **レベル4**: 外部サポートが必要

### 緊急時チェックリスト
- [ ] 問題の症状を詳細に記録
- [ ] エラーログの保存
- [ ] 影響範囲の確認
- [ ] 代替手順への切り替え
- [ ] 関係者への連絡

## 📞 サポート情報

### 問題報告時に必要な情報
1. **環境情報**
   - OS、ソフトウェアバージョン
   - ハードウェア構成

2. **問題詳細**
   - 発生日時
   - 症状の詳細
   - 再現手順

3. **ログファイル**
   - エラーログ
   - システムログ

### 連絡先
- **緊急時**: [緊急連絡先]
- **技術サポート**: [サポート窓口]
- **システム管理者**: [管理者連絡先]

---

**このガイドで解決しない問題については、遠慮なく技術サポートにお問い合わせください。**