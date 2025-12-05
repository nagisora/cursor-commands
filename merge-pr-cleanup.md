# プルリクマージ・ブランチ削除・main最新化（一括実行）

## 概要

GitHub CLIを使ってプルリクをマージし、マージされたブランチを削除したあと、ローカルのmainブランチを最新化するための一括実行コマンドです。  
`commit-push-pr.md` で作成したプルリクをマージする際に使用します。

## 前提条件

- GitHub CLI (`gh`) がインストール済みであること
- `gh auth status` で認証済みであること
- 現在のブランチが作業ブランチ（feature/_, fix/_ など）であること
- マージ対象のプルリクが存在し、マージ可能な状態であること
- リモート `origin` が設定済みであること

## 実行手順（対話なし）

1. 現在のブランチからプルリク番号を取得（または引数で指定）
2. プルリクの状態確認（マージ可能かチェック）
3. プルリクをマージ（`gh pr merge`）
4. mainブランチに切り替え
5. mainブランチを最新化（`git pull origin main`）
6. マージされたブランチをローカルから削除（`git branch -d`）
7. リモートブランチを削除（`git push origin --delete` または `gh pr delete`）

## 使い方

### A) 現在のブランチから自動検出（推奨）

現在のブランチに対応するプルリクを自動的に検出してマージします。

```bash
# 現在のブランチ名を取得
CURRENT_BRANCH=$(git branch --show-current)

# 現在のブランチに対応するプルリクを検索
PR_NUMBER=$(gh pr list --head "$CURRENT_BRANCH" --json number -q '.[0].number')

if [ -z "$PR_NUMBER" ]; then
  echo "⚠️ 現在のブランチ ($CURRENT_BRANCH) に対応するプルリクが見つかりません"
  exit 1
fi

echo "プルリク #$PR_NUMBER をマージします..."

# プルリクをマージ（--merge は通常のマージ、--squash はスカッシュマージ、--rebase はリベースマージ）
gh pr merge "$PR_NUMBER" --merge --delete-branch

# mainブランチに切り替え
git checkout main

# mainブランチを最新化
git pull origin main

# ローカルのマージ済みブランチを削除（存在する場合のみ）
if git show-ref --verify --quiet refs/heads/"$CURRENT_BRANCH"; then
  git branch -d "$CURRENT_BRANCH"
  echo "✅ ローカルブランチ $CURRENT_BRANCH を削除しました"
else
  echo "ℹ️ ローカルブランチ $CURRENT_BRANCH は既に削除されています"
fi
```

### B) プルリク番号を明示的に指定

プルリク番号を直接指定する場合。

```bash
# プルリク番号を指定
PR_NUMBER=123

# プルリク情報を取得
PR_INFO=$(gh pr view "$PR_NUMBER" --json headRefName,state,merged)

# プルリクの状態を確認
STATE=$(echo "$PR_INFO" | jq -r '.state')
if [ "$STATE" != "OPEN" ] && [ "$STATE" != "MERGED" ]; then
  echo "⚠️ プルリク #$PR_NUMBER はマージ可能な状態ではありません（状態: $STATE）"
  exit 1
fi

# 既にマージ済みかチェック
MERGED=$(echo "$PR_INFO" | jq -r '.merged')
if [ "$MERGED" = "true" ]; then
  echo "ℹ️ プルリク #$PR_NUMBER は既にマージ済みです"
else
  # プルリクをマージ
  echo "プルリク #$PR_NUMBER をマージします..."
  gh pr merge "$PR_NUMBER" --merge --delete-branch
fi

# ブランチ名を取得
BRANCH_NAME=$(echo "$PR_INFO" | jq -r '.headRefName')

# mainブランチに切り替え
git checkout main

# mainブランチを最新化
git pull origin main

# ローカルのマージ済みブランチを削除（存在する場合のみ）
if git show-ref --verify --quiet refs/heads/"$BRANCH_NAME"; then
  git branch -d "$BRANCH_NAME"
  echo "✅ ローカルブランチ $BRANCH_NAME を削除しました"
else
  echo "ℹ️ ローカルブランチ $BRANCH_NAME は既に削除されています"
fi
```

### C) スカッシュマージを使用する場合

スカッシュマージを使用する場合の例。

```bash
CURRENT_BRANCH=$(git branch --show-current)
PR_NUMBER=$(gh pr list --head "$CURRENT_BRANCH" --json number -q '.[0].number')

if [ -z "$PR_NUMBER" ]; then
  echo "⚠️ 現在のブランチ ($CURRENT_BRANCH) に対応するプルリクが見つかりません"
  exit 1
fi

echo "プルリク #$PR_NUMBER をスカッシュマージします..."

# スカッシュマージを実行
gh pr merge "$PR_NUMBER" --squash --delete-branch

# mainブランチに切り替え
git checkout main

# mainブランチを最新化
git pull origin main

# ローカルのマージ済みブランチを削除
if git show-ref --verify --quiet refs/heads/"$CURRENT_BRANCH"; then
  git branch -d "$CURRENT_BRANCH"
  echo "✅ ローカルブランチ $CURRENT_BRANCH を削除しました"
fi
```

### D) ステップ実行（デバッグ用）

各ステップを個別に実行して確認する場合。

```bash
# 1) 現在のブランチ確認
CURRENT_BRANCH=$(git branch --show-current)
echo "現在のブランチ: $CURRENT_BRANCH"

# 2) プルリク番号を取得
PR_NUMBER=$(gh pr list --head "$CURRENT_BRANCH" --json number -q '.[0].number')
if [ -z "$PR_NUMBER" ]; then
  echo "⚠️ 現在のブランチ ($CURRENT_BRANCH) に対応するプルリクが見つかりません"
  exit 1
fi
echo "プルリク番号: #$PR_NUMBER"

# 3) プルリクの状態確認
gh pr view "$PR_NUMBER"

# 4) プルリクをマージ
echo "プルリク #$PR_NUMBER をマージします..."
gh pr merge "$PR_NUMBER" --merge --delete-branch

# 5) mainブランチに切り替え
echo "mainブランチに切り替えます..."
git checkout main

# 6) mainブランチを最新化
echo "mainブランチを最新化します..."
git pull origin main

# 7) ローカルブランチを削除
if git show-ref --verify --quiet refs/heads/"$CURRENT_BRANCH"; then
  echo "ローカルブランチ $CURRENT_BRANCH を削除します..."
  git branch -d "$CURRENT_BRANCH"
  echo "✅ ローカルブランチ $CURRENT_BRANCH を削除しました"
else
  echo "ℹ️ ローカルブランチ $CURRENT_BRANCH は既に削除されています"
fi
```

## マージ方法の選択

`gh pr merge` コマンドでは、以下のマージ方法を選択できます：

- `--merge`: 通常のマージコミットを作成（デフォルト）
- `--squash`: スカッシュマージ（すべてのコミットを1つにまとめる）
- `--rebase`: リベースマージ（コミット履歴を線形に保つ）

プロジェクトのポリシーに応じて選択してください。

## 注意事項

- **マージ前の確認**: プルリクがマージ可能な状態か、レビューが完了しているかを事前に確認してください
- **未保存の変更**: ブランチを切り替える前に、未保存の変更がないことを確認してください（`git status` で確認）
- **リモートブランチの削除**: `--delete-branch` オプションにより、リモートブランチは自動的に削除されます
- **ローカルブランチの削除**: マージ済みのブランチのみ削除されます（`-d` オプション）。未マージの場合は `-D` が必要ですが、本コマンドでは安全のため `-d` を使用しています
- **mainブランチの保護**: 本コマンドは main ブランチへの直接マージを防ぐものではありません。GitHub のブランチ保護ルールを設定することを推奨します

## トラブルシューティング

### プルリクが見つからない場合

```bash
# すべてのプルリクを確認
gh pr list

# 特定のブランチのプルリクを確認
gh pr list --head feature/your-branch
```

### マージに失敗した場合

```bash
# プルリクの状態を確認
gh pr view <PR_NUMBER>

# マージできない理由を確認（コンフリクト、レビュー未完了など）
gh pr checks <PR_NUMBER>
```

### ローカルブランチの削除に失敗した場合

マージされていない変更がある場合、`git branch -d` は失敗します。強制削除が必要な場合は手動で実行してください：

```bash
# 強制削除（注意: 未マージの変更も失われます）
git branch -D <BRANCH_NAME>
```

### mainブランチの更新に失敗した場合

```bash
# コンフリクトがある場合は手動で解決
git pull origin main

# または、リセットが必要な場合（注意: ローカルの変更が失われます）
git fetch origin
git reset --hard origin/main
```

## 実行例

```bash
# 例1: 現在のブランチから自動検出してマージ
CURRENT_BRANCH=$(git branch --show-current) && \
PR_NUMBER=$(gh pr list --head "$CURRENT_BRANCH" --json number -q '.[0].number') && \
gh pr merge "$PR_NUMBER" --merge --delete-branch && \
git checkout main && \
git pull origin main && \
git branch -d "$CURRENT_BRANCH" 2>/dev/null || echo "ブランチは既に削除済み"
```

## 関連ドキュメント

- コミット・プッシュ・PR作成: `.cursor/commands/commit-push-pr.md`
- GitHub CLI ドキュメント: <https://cli.github.com/manual/gh_pr_merge>
- 開発フロー: プロジェクト固有の README / CONTRIBUTING / 開発ガイド等
