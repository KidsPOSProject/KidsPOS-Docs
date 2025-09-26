# KidsPOS-for-Android 現場展開ガイド

## 概要
KidsPOS-for-Androidアプリを実際の現場で展開・運用するための詳細手順です。

## 事前準備

### デバイス要件
- **推奨デバイス**: Nexus 7 相当のAndroidタブレット
- **OS**: Android 7.0 (API level 24) 以上
- **RAM**: 2GB以上（Nexus 7の場合）
- **ストレージ**: 1GB以上の空き容量
- **カメラ**: リアカメラ必須（QRコード読み取り用）
- **USB**: USBバーコードリーダー接続用
- **ネットワーク**: WiFi接続必須（イントラネット環境）

### 推奨デバイス仕様
- **Nexus 7 (2013)**: 7インチタブレット、2GB RAM
- **または同等スペックのタブレット**: 7-10インチ画面推奨
- **USB Host機能**: バーコードリーダー接続のため必須

### 周辺機器
- **USBバーコードリーダー**: USB HID対応の汎用バーコードスキャナー
- **USB OTGケーブル**: タブレットとバーコードリーダーの接続用

## APKファイルの準備

### 1. ビルド済みAPKの取得
```bash
# GitHub Releasesからダウンロード
wget https://github.com/KidsPOSProject/KidsPOS-for-Android/releases/latest/download/app-release.apk
```

### 2. 自身でビルドする場合
```bash
# リポジトリクローン
git clone https://github.com/KidsPOSProject/KidsPOS-for-Android.git
cd KidsPOS-for-Android

# Android Studioでビルド、または
./gradlew assembleRelease
```

## デバイス設定

### 1. 開発者オプションの有効化
1. **設定** → **デバイス情報** → **ビルド番号**を7回タップ
2. 開発者オプションが有効になります

### 2. 提供元不明のアプリの許可
1. **設定** → **セキュリティ**
2. **提供元不明のアプリ**を有効化
3. または**不明なアプリのインストール**でブラウザまたはファイルマネージャーを許可

### 3. USBデバッグの有効化（開発時のみ）
1. **設定** → **開発者オプション**
2. **USBデバッグ**を有効化

## アプリのインストール

### 方法1: ADB経由でインストール
```bash
# デバイス接続確認
adb devices

# APKインストール
adb install app-release.apk

# 複数デバイスがある場合
adb -s [device_id] install app-release.apk
```

### 方法2: デバイス直接インストール
1. APKファイルをデバイスに転送
2. ファイルマネージャーでAPKファイルをタップ
3. インストール画面でインストール実行

### 方法3: ワイヤレス配布
```bash
# 簡易HTTPサーバーでAPK配布
python3 -m http.server 8080

# デバイスのブラウザで以下にアクセス
# http://[サーバーIP]:8080/app-release.apk
```

## 初期設定

### 1. 権限の許可
アプリ初回起動時に以下の権限を許可：
- **カメラ**: QRコード読み取り
- **ストレージ**: データ保存
- **ネットワーク**: サーバー通信

### 2. サーバー接続設定
1. アプリ起動後、設定画面を開く
2. **サーバーURL**を入力: `http://[RaspberryPi IP]:8080`
3. **接続テスト**を実行して通信確認

### 3. USBバーコードリーダー設定
1. **USB OTGケーブルの接続**
   - タブレットのmicroUSBポートにOTGケーブル接続
   - バーコードリーダーをOTGケーブルに接続

2. **デバイス認識確認**
   ```bash
   # adb経由で確認（開発時）
   adb shell lsusb
   # または
   adb shell cat /proc/bus/input/devices
   ```

3. **アプリでの設定**
   - 設定画面で「USBバーコードリーダー」を有効化
   - テストスキャンで動作確認

### 4. 端末識別設定
```json
{
  "device_id": "tablet-01", 
  "location": "store-front",
  "barcode_reader": "usb_hid"
}
```

## 現場運用設定

### 1. キオスクモード設定（推奨）
```bash
# Samsung Knox等のMDMソリューション使用
# または専用ランチャーアプリ使用

# 設定例：
# - ホームボタン無効化
# - 通知パネル無効化
# - 他アプリ起動制限
```

### 2. 画面設定
- **画面の向き**: 固定（portrait）
- **画面の明るさ**: 80%以上
- **画面タイムアウト**: 無効または長時間

### 3. 音量設定
- **メディア音量**: 適切なレベル（フィードバック音用）
- **通知音**: 必要に応じてオフ

## ネットワーク設定

### WiFi設定の確認
```bash
# WiFi接続テスト
ping 8.8.8.8
ping [サーバーIP]

# ネットワーク情報確認
adb shell ip addr show wlan0
```

### プロキシ設定（必要に応じて）
1. **設定** → **WiFi**
2. 接続中のネットワークを長押し
3. **ネットワークを変更**
4. **詳細設定** → **プロキシ**で設定

## 動作確認テスト

### 1. 基本機能テスト
- [ ] アプリ起動確認
- [ ] サーバー接続確認
- [ ] QRコード読み取り確認
- [ ] データ送受信確認

### 2. 操作性テスト
- [ ] タッチ操作の反応
- [ ] 画面遷移の動作
- [ ] エラーハンドリング

### 3. パフォーマンステスト
```bash
# メモリ使用量確認
adb shell dumpsys meminfo [package_name]

# CPU使用率確認
adb shell top | grep [package_name]
```

## トラブルシューティング

### よくある問題

#### 1. アプリがインストールできない
```bash
# 署名確認
keytool -printcert -jarfile app-release.apk

# デバイス容量確認
adb shell df -h

# アプリ一覧確認
adb shell pm list packages | grep kidspos
```

#### 2. サーバーに接続できない
```bash
# ネットワーク疎通確認
adb shell ping [サーバーIP]

# ポート確認
adb shell nc -zv [サーバーIP] 8080

# DNS設定確認
adb shell nslookup [サーバードメイン]
```

#### 3. カメラが動作しない
```bash
# カメラ権限確認
adb shell dumpsys package [package_name] | grep permission

# カメラデバイス確認
adb shell ls /dev/video*
```

### ログ確認
```bash
# アプリログ確認
adb logcat | grep [package_name]

# システムログ
adb logcat -b system

# クラッシュログ
adb logcat -b crash
```

## 一括展開手順

### 1. スクリプト化
```bash
#!/bin/bash
# bulk_deploy.sh

DEVICES=$(adb devices | grep -v "List" | grep "device" | cut -f1)

for device in $DEVICES; do
    echo "Deploying to device: $device"
    adb -s $device install -r app-release.apk
    adb -s $device shell am start -n com.kidspos.android/.MainActivity
done
```

### 2. 設定ファイル配布
```bash
# 設定ファイルをデバイスに配布
adb push config.json /sdcard/kidspos/
```

### 3. 動作確認自動化
```bash
#!/bin/bash
# health_check.sh

adb shell am start -n com.kidspos.android/.MainActivity
sleep 5
adb shell input tap 500 300  # 設定ボタンタップ例
adb shell screencap /sdcard/screenshot.png
adb pull /sdcard/screenshot.png ./screenshots/
```

## 運用監視

### 1. デバイス状態監視
```bash
# バッテリー状態
adb shell dumpsys battery

# ストレージ使用量
adb shell du -sh /data/data/[package_name]

# メモリ使用量
adb shell cat /proc/meminfo
```

### 2. アプリ状態監視
```bash
# プロセス確認
adb shell ps | grep kidspos

# アプリバージョン確認
adb shell dumpsys package [package_name] | grep versionCode
```

## 緊急時対応

### 1. アプリ強制停止
```bash
adb shell am force-stop [package_name]
```

### 2. アプリ再起動
```bash
adb shell am start -n com.kidspos.android/.MainActivity
```

### 3. 工場リセット（最終手段）
```bash
adb shell recovery --wipe_data
```

## チェックリスト

現場展開時の確認事項：

### デバイス準備
- [ ] デバイス要件確認
- [ ] 開発者オプション有効化
- [ ] 提供元不明のアプリ許可
- [ ] WiFi接続設定

### アプリ展開
- [ ] APKファイル準備
- [ ] アプリインストール
- [ ] 権限許可設定
- [ ] サーバー接続設定

### 動作確認
- [ ] 基本機能テスト完了
- [ ] QRコード読み取り確認
- [ ] データ同期確認
- [ ] エラーハンドリング確認

### 運用設定
- [ ] キオスクモード設定
- [ ] 画面・音量設定
- [ ] 監視設定完了
- [ ] 緊急時手順書準備

---

**注意**: デバイスメーカーやAndroidバージョンにより手順が異なる場合があります。実際の環境に応じて調整してください。