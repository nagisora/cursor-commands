# プルリクマージ・クリーンアップ

現在のブランチのPRをマージし、ブランチ削除・main最新化を行う。

## 実行条件

- `gh auth status` で認証済みであること
- 現在のブランチがmain以外の作業ブランチであること

## 手順

以下を順に実行する。各ステップでエラーが発生した場合は停止し、原因をユーザーに報告する。

### 1. 現在のブランチとPR番号を取得

```bash
CURRENT_BRANCH=$(git branch --show-current)
PR_NUMBER=$(gh pr list --head "$CURRENT_BRANCH" --json number -q '.[0].number')
```

PR番号が取得できない場合は `gh pr list` で一覧を表示し、ユーザーに確認を求める。

### 2. PRの状態を確認

```bash
gh pr view "$PR_NUMBER" --json state,mergeable,reviewDecision
```

マージ不可の場合は `gh pr checks "$PR_NUMBER"` で原因を確認し、ユーザーに報告する。

### 3. PRをマージ

```bash
gh pr merge "$PR_NUMBER" --merge --delete-branch
```

### 4. mainブランチを最新化

```bash
git checkout main
git pull origin main
```

### 5. ローカルブランチを削除

```bash
git branch -d "$CURRENT_BRANCH" 2>/dev/null || echo "ブランチは既に削除済み"
```

削除失敗時は未マージ変更の可能性を警告する。強制削除（`-D`）はユーザーの明示的な許可なく実行しない。

## マージ方法

ユーザーの指定がなければ `--merge` を使用。指定があれば従う：

- `--merge`: 通常マージ（デフォルト）
- `--squash`: スカッシュマージ
- `--rebase`: リベースマージ
