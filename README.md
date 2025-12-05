# Cursor カスタムコマンド集

このリポジトリは、Cursor IDEで使用するカスタムコマンドのコレクションです。開発効率を向上させるための便利なコマンドを提供します。

## 📋 概要

Cursorのカスタムコマンド機能を使用して、よく使う開発タスクを自動化・簡略化するためのコマンド集です。WindowsとUbuntuの両方で動作します。

## 🚀 インストール方法

1. このリポジトリをクローンまたはダウンロードします
2. Cursorのカスタムコマンドフォルダに配置します：
   - **Windows**: `%APPDATA%\Cursor\User\globalStorage\cursor.commands\`
   - **macOS**: `~/Library/Application Support/Cursor/User/globalStorage/cursor.commands/`
   - **Linux**: `~/.config/Cursor/User/globalStorage/cursor.commands/`

3. または、Cursorの設定からカスタムコマンドフォルダのパスを確認し、そこにファイルを配置してください

## 📝 利用可能なコマンド

### 1. `apply-cursorrules`

[きのぴーさんのGitHubリポジトリ](https://github.com/kinopeee/cursorrules)から最新のCursorrulesを取得し、現在のプロジェクトに適用します。

### 2. `go-on-agent`

エージェントモードに切り替えた際に、先ほどの議論（設計）に基づいて実装を進めるためのコマンドです。

### 3. `remove-ai-code-slop`

現在の作業ブランチで `main` との差分を確認し、AI生成と思われるスロップ（不要生成物）を取り除くためのコマンドです。
CursorのEricさんより提案されたベストプラクティス（[引用元](https://x.com/ericzakariasson/status/1995671800643297542?s=20)）。

### 4. `refactor-plan`

現在の作業ブランチでの変更差分を対象に、リファクタリングの提案を行います。

### 5. `pr-review-response`

プルリクエストに対してレビューコメントがあった場合に、その解説と対応方針を提案するコマンドです。

### 6. `pr-merge-cleanup`

現在のブランチのプルリクエストをマージし、ブランチ削除・main最新化を行うコマンドです。

## 🔧 前提条件

- **Cursor IDE** がインストールされていること
- **Git** がインストール・設定されていること

## 📖 使い方

1. Cursorでプロジェクトを開きます
2. コマンドパレット（`Ctrl+Shift+P` / `Cmd+Shift+P`）を開きます
3. スラッシュ（`/`）を入力すると、カスタムコマンドが表示されます
4. 使用したいコマンドを選択します

または、チャット画面で直接コマンド名を入力することもできます：
```
/apply-cursorrules
```

## ⚠️ 注意事項

- 各コマンドは、適切な前提条件（Gitの状態、ブランチ、リモート設定など）が満たされている必要があります
- コマンドによっては、既存のファイルを上書きする可能性があります
- 重要な変更を行う前に、バックアップを取ることを推奨します

## 🤝 コントリビューション

改善提案やバグ報告は、IssueやPull Requestでお願いします。

## 📄 ライセンス

このプロジェクトは MIT ライセンスのもとで公開されています。詳細は `LICENSE` ファイルを参照してください。

## 🔗 関連リンク

- [Cursor IDE](https://cursor.sh/)
- [Cursor カスタムコマンドのドキュメント](https://docs.cursor.com/)

---

**作成者**: nagisora  
**リポジトリ**: https://github.com/nagisora/cursor-commands

