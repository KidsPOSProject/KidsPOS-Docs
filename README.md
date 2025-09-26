# KidsPOS-Docs

KidsPOSシステムの現場展開・運用のための包括的ドキュメント

## 概要

このリポジトリは、[KidsPOS-Server](https://github.com/KidsPOSProject/KidsPOS-Server) と [KidsPOS-for-Android](https://github.com/KidsPOSProject/KidsPOS-for-Android) を実際の現場で運用する際に必要な全てのドキュメントを提供します。

## 📁 ドキュメント構成

### 📖 [ドキュメント](docs/)
- **[展開ガイド](docs/deployment/)** - システムの展開と初期設定
- **[ユーザーガイド](docs/user-guide/)** - 現場での操作方法
- **[トラブルシューティング](docs/troubleshooting/)** - 問題解決ガイド

### 📊 [図表](diagrams/)
- **システム構成図** - draw.io形式のSVGファイル
- **ワークフロー図** - 業務フローの可視化

### 🎯 [スライド](slides/)
- **Marp**を使用したプレゼンテーション資料
- **研修用スライド** - スタッフ教育用

### 📝 [テンプレート](templates/)
- **AIとの対話用テンプレート** - 自然言語でのドキュメント作成支援

## 🚀 クイックスタート

### ドキュメントの閲覧
```bash
# リポジトリをクローン
git clone https://github.com/KidsPOSProject/KidsPOS-Docs.git
cd KidsPOS-Docs

# ドキュメントを閲覧
# Markdownファイルはそのまま閲覧可能
# SVGファイルはブラウザまたは対応エディタで開く
```

### スライドの作成・閲覧
```bash
# 依存関係のインストール
npm install

# スライドをHTMLに変換
npm run slides

# ライブサーバーでスライドを表示
npm run serve

# 開発モード（ファイル監視）
npm run slides:watch
```

## 📋 システム要件

### KidsPOS-Server
- Node.js 14.x以上
- MongoDB 4.4以上
- ネットワーク要件: ポート3000番開放

### KidsPOS-for-Android
- Android 7.0 (API level 24)以上
- WiFi接続必須
- カメラ権限

## 🔧 現場展開チェックリスト

- [ ] [サーバー環境の準備](docs/deployment/server-setup.md)
- [ ] [Androidデバイスの設定](docs/deployment/android-setup.md)
- [ ] [ネットワーク設定](docs/deployment/network-configuration.md)
- [ ] [動作確認テスト](docs/deployment/testing-checklist.md)
- [ ] [スタッフ研修](docs/user-guide/training.md)

## 💡 AI活用ガイド

このドキュメントは自然言語でのAI対話による更新・改善を前提として設計されています。

### ドキュメント更新の例
```
「サーバーの初期設定手順を詳しく教えて」
「トラブル時の対応フローを図にして」  
「新人研修用のスライドを作成して」
```

### 図表作成の例
```
「システム全体の構成図を作成」
「データフローを可視化して」
「業務フローチャートが必要」
```

## 🤝 貢献

ドキュメントの改善・追加は随時受け付けています。

1. Issues で改善点を報告
2. Pull Request でドキュメントを更新
3. AI対話ログの共有で知識ベース拡充

## 📞 サポート

現場運用でお困りの際は：
- [Issue](https://github.com/KidsPOSProject/KidsPOS-Docs/issues) として報告
- [Discussion](https://github.com/KidsPOSProject/KidsPOS-Docs/discussions) で相談

## 📄 ライセンス

このドキュメントは実用性を重視し、自由に活用・改変できるライセンスを採用予定です。
