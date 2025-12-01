# Cursor Rules Initializer
Cursor v2.1.42で動作確認済み

## 概要
- リポジトリのアーキテクチャに沿ったcursor rulesの初期版を作成するためのツールです
- 運用しているリポジトリにまだrulesがないときにご利用ください
- リポジトリ内のコードからアーキテクチャを読み取ってrulesを作成するため、新規のリポジトリには向いていません
- 新規リポジトリでは[awesome-cursorrules](https://github.com/PatrickJS/awesome-cursorrules)のような、公開されているrulesを参考にしてください

## Cursor rules利用のポイント
- このツールで作成したruleを完全版とせず、AIの出力結果に合わせてruleファイルをチューニングしながら活用してください

## 使用手順

### インストール
```bash
git clone https://github.com/showcase-gig-platform/cursor-rules-initializer.git
cp -r cursor-rules-initializer/.cursor ~/your-repository/
```

### 実行
推論能力の高いモデルを使用しての実行を推奨します
1. Planモードで計画を立てる
   - `/cursor-rules-initializer/init`
2. AIからの質問があれば回答する
3. Planを実行させる
   - build
4. 作成されたruleファイルを確認し、明らかにおかしい箇所があればCursorに指摘し直してもらう

## ライセンス
このプロジェクトは [MIT License](LICENSE) の下で公開されています。
