# KidsPOS システムドキュメント

## 🚀 現場展開ガイド

KidsPOSシステムを実際の現場で安全かつ効率的に展開するための包括的なドキュメント集です。

### 📋 展開フェーズ別ガイド

#### 1. [サーバー環境構築](deployment/server-setup.md)
- システム要件と事前準備
- Java + SpringBoot環境の構築
- 基本設定とプロセス管理
- 運用監視とログ管理

#### 2. [Android端末設定](deployment/android-setup.md)
- デバイス要件と推奨機種
- APKインストール手順（複数方法）
- 権限設定と初期設定
- 一括展開のためのスクリプト化
- キオスクモード設定

#### 3. [ネットワーク設定](deployment/network-configuration.md)
- ネットワーク要件と構成例
- IPアドレス設計とDHCP設定
- WiFi設定とベーシック構成
- 基本的な監視とトラブルシューティング

## 📖 運用ガイド

### [現場運用マニュアル](user-guide/field-operations.md)
- 日常の操作手順
- QRコード読み取り操作
- データ管理と同期
- 定期チェックリスト
- パフォーマンス最適化

## 🔧 トラブルシューティング

### [一般的な問題と解決方法](troubleshooting/common-issues.md)
- サーバー関連の問題
- Android端末の問題  
- ネットワーク関連の問題
- パフォーマンス問題の診断
- 緊急時対応手順

## 🎯 プレゼンテーション資料

### [現場展開研修スライド](../slides/deployment-training.md)
- Marp形式のスライド資料
- 研修用プレゼンテーション
- 実践的な展開手順
- Q&A セッション用資料

**スライドのHTMLビルド:**
```bash
npm run slides
```

## 📊 システム構成図

### [アーキテクチャ図](../diagrams/system-architecture.svg)
- システム全体の構成
- コンポーネント間の関係
- データフローの可視化

### [展開ワークフロー図](../diagrams/deployment-workflow.svg)
- 段階的な展開プロセス
- 各フェーズでの確認事項
- 失敗時の対処フロー

## 🤖 AI活用ガイド

### [AI対話テンプレート](../templates/ai-conversation-templates.md)
- ドキュメント作成のためのプロンプト集
- 図表作成の依頼方法
- スライド作成のテンプレート
- 品質チェック項目

## 📋 現場展開チェックリスト

### 事前準備
- [ ] 要件確認（ハードウェア、ネットワーク、人員）
- [ ] 機器調達と検収
- [ ] ネットワーク設計と機器設定

### サーバー構築
- [ ] OS・ミドルウェアインストール
- [ ] KidsPOS-Serverの展開
- [ ] データベース設定と初期化
- [ ] 基本設定と動作確認
- [ ] 監視・バックアップ設定

### Android端末設定
- [ ] デバイス設定（開発者オプション等）
- [ ] APKインストール
- [ ] 初期設定とサーバー接続確認
- [ ] 権限設定と基本設定
- [ ] キオスクモード設定（必要に応じて）

### 統合テスト
- [ ] サーバー・端末間の通信確認
- [ ] QRコード読み取り機能テスト
- [ ] データ同期機能テスト
- [ ] 負荷テスト（複数端末同時使用）
- [ ] エラーハンドリングテスト

### 運用開始
- [ ] スタッフ研修実施
- [ ] 運用マニュアルの配布
- [ ] 監視体制の確立
- [ ] 緊急時連絡体制の確認
- [ ] バックアップ・復旧手順の確認

## 🚨 緊急時対応

### 連絡体制
1. **現場スタッフ** → 基本的なトラブルシューティング
2. **システム管理者** → 技術的な問題の解決
3. **技術サポート** → 複雑な問題のエスカレーション

### よくある問題への対処
- **サーバー応答なし** → [サーバー問題解決手順](troubleshooting/common-issues.md#サーバー関連の問題)
- **端末接続エラー** → [Android問題解決手順](troubleshooting/common-issues.md#android端末関連の問題)
- **ネットワーク不調** → [ネットワーク問題解決手順](troubleshooting/common-issues.md#ネットワーク関連の問題)

## 📞 サポート情報

### 技術サポート
- **ドキュメント**: この GitHub リポジトリ
- **Issue報告**: [GitHub Issues](https://github.com/KidsPOSProject/KidsPOS-Docs/issues)
- **ディスカッション**: [GitHub Discussions](https://github.com/KidsPOSProject/KidsPOS-Docs/discussions)

### 関連リポジトリ
- **サーバー**: [KidsPOS-Server](https://github.com/KidsPOSProject/KidsPOS-Server)
- **Android**: [KidsPOS-for-Android](https://github.com/KidsPOSProject/KidsPOS-for-Android)

---

**このドキュメントは現場での実用性を重視して作成されています。**
**不明な点や改善提案がありましたら、お気軽にIssueやDiscussionでお知らせください。**