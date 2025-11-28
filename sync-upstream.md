# sync-upstream

フォーク元から最新のコードを同期してください。

## 前提条件

1. **Gitの状態確認**
   - 作業ディレクトリがクリーンであること（未コミットの変更がないこと）
   - 必要に応じて `git status` で確認
   - 未コミットの変更がある場合は、事前にコミットまたはstashする
   - 推奨: `git stash` で一時保存し、同期後に `git stash pop` で復元

2. **リモートリポジトリの設定確認**
   - `upstream` リモートが設定されていること
     - 確認コマンド: `git remote -v`
     - `upstream` が存在しない場合は、以下のコマンドで設定:sh
       git remote add upstream [フォーク元のリポジトリURL]
          - `origin` リモートが設定されていること（プッシュ先）

3. **ブランチ確認**
   - 現在のブランチを確認（`git branch` または `git status`）
   - 必要に応じて `main` ブランチに切り替え

4. **ネットワーク接続**
   - フォーク元リポジトリへのアクセスが可能であること

## 実行手順

### ステップ1: 現在の状態を確認

1. **作業ディレクトリの状態確認**
   git status
      - 未コミットの変更がある場合は、ステップ2で対応

2. **リモートリポジトリの確認**
   git remote -v
      - `upstream` と `origin` が正しく設定されているか確認
   - URLが正しいか確認

3. **現在のブランチ確認**
   git branch --show-current
      - または `git status` で確認

### ステップ2: 未コミットの変更の処理（必要な場合）

1. **変更を一時保存（推奨）**
  
   git stash push -m "sync-upstream前の一時保存"
      - または変更をコミット: `git add . && git commit -m "[メッセージ]"`

2. **stashした内容の確認（任意）**
   
   git stash list
   ### ステップ3: フォーク元から最新を取得

1. **upstreamから最新の情報を取得**ash
   git fetch upstream
      - エラーが発生した場合は「エラーハンドリング」を参照

2. **取得結果の確認**
 
   git log HEAD..upstream/main --oneline
      - 取り込まれる予定のコミットを確認（任意）

### ステップ4: mainブランチに切り替え

1. **mainブランチに切り替え**
   
   git checkout main
      - 既に `main` ブランチにいる場合はスキップ

2. **mainブランチを最新化（推奨）**h
   git pull origin main
      - ローカルの `main` をリモートと同期

### ステップ5: フォーク元をマージ

1. **upstream/mainをマージ**
  
   git merge upstream/main --no-ff -m "Merge upstream/main: フォーク元の最新を取り込み"
      - `--no-ff`: マージコミットを必ず作成（履歴を明確に）
   - マージコミットメッセージは指定の通り

2. **マージコンフリクトが発生した場合**
   - 「エラーハンドリング」の「ケース3: マージコンフリクト」を参照

### ステップ6: リモートにプッシュ

1. **マージ結果をリモートにプッシュ**
   git push origin main
      - エラーが発生した場合は「エラーハンドリング」を参照

2. **プッシュ結果の確認**
   - GitHub等でリモートリポジトリに変更が反映されているか確認（任意）

### ステップ7: 一時保存した変更の復元（ステップ2でstashした場合）

1. **stashした変更を復元**
   git stash pop
      - コンフリクトが発生した場合は手動で解決

2. **復元後の確認**
   
   git status
   ## エラーハンドリング

### ケース1: upstreamリモートが存在しない

**エラーメッセージ例**: `fatal: 'upstream' does not appear to be a git repository`

**対処方法**:
1. フォーク元のリポジトリURLを確認
2. upstreamリモートを追加:
  
   git remote add upstream [フォーク元のリポジトリURL]
   3. 設定を確認: `git remote -v`
4. ステップ3から再開

### ケース2: fetchに失敗

**エラーメッセージ例**: `fatal: Could not read from remote repository`

**対処方法**:
1. ネットワーク接続を確認
2. 認証情報を確認（SSH鍵、アクセストークンなど）
3. フォーク元リポジトリのURLが正しいか確認: `git remote get-url upstream`
4. リポジトリが存在するか、アクセス権限があるか確認

### ケース3: マージコンフリクト

**エラーメッセージ例**: `Automatic merge failed; fix conflicts and then commit the result.`

**対処方法**:
1. **コンフリクトファイルの確認**
   
   git status
      - `both modified` と表示されているファイルがコンフリクトしている

2. **コンフリクトの解決**
   - 各コンフリクトファイルを開き、`<<<<<<<`, `=======`, `>>>>>>>` マーカーを確認
   - 必要な変更を手動でマージ
   - コンフリクトマーカーを削除

3. **解決後のコミット**
   
   git add [解決したファイル]
   git commit -m "Merge upstream/main: フォーク元の最新を取り込み（コンフリクト解決済み）"
      - または `git commit` のみでエディタが開く場合もある

4. **コンフリクト解決を中断する場合（ロールバック）**
   
   git merge --abort
   ### ケース4: プッシュに失敗（リモートが進んでいる）

**エラーメッセージ例**: `Updates were rejected because the remote contains work that you do not have locally`

**対処方法**:
1. **リモートの最新を取得**
   
   git pull origin main
      - 必要に応じてマージまたはリベース

2. **再度プッシュ**
   git push origin main
   ### ケース5: プッシュ権限エラー

**エラーメッセージ例**: `Permission denied` または `Authentication failed`

**対処方法**:
1. 認証情報を確認（SSH鍵、アクセストークンなど）
2. リモートリポジトリへの書き込み権限があるか確認
3. 必要に応じて認証情報を再設定

### ケース6: 現在のブランチに未コミットの変更がある

**エラーメッセージ例**: `error: Your local changes to the following files would be overwritten by checkout`

**対処方法**:
1. **変更を一時保存（推奨）**
   git stash push -m "sync-upstream前の一時保存"
   2. **または変更をコミット**
   
   git add .
   git commit -m "[適切なコミットメッセージ]"
   3. ステップ4から再開

## ロールバック手順（問題が発生した場合）

### マージを取り消す（プッシュ前）

git reset --hard HEAD~1- 注意: ローカルの変更も失われます

### マージを取り消す（プッシュ後）

git revert -m 1 HEAD
git push origin main- `-m 1`: マージコミットの親（通常はmainブランチ側）を指定

### 特定のコミットまで戻す

git log --oneline  # 戻したいコミットのハッシュを確認
git reset --hard [コミットハッシュ]
git push origin main --force  # 注意: force pushは慎重に
## 注意事項

- **force pushは避ける**: 他のメンバーが作業している場合、履歴を書き換えると問題が発生する可能性があります
- **マージコンフリクト**: コンフリクトが発生した場合は、慎重に解決してください
- **バックアップ**: 重要な変更がある場合は、事前にバックアップブランチを作成することを推奨:
  
  git branch backup-before-sync-$(date +%Y%m%d)
  - **テスト**: マージ後は、アプリケーションが正常に動作するか確認してください
- **コミット履歴**: `--no-ff` オプションにより、マージコミットが作成されます。履歴が分岐しますが、これは意図的な動作です

## 実行後の確認事項

- [ ] `git status` でクリーンな状態である
- [ ] `git log --oneline -5` で最新のマージコミットが含まれている
- [ ] フォーク元の最新コミットが取り込まれている
- [ ] リモートに正しくpushされている
- [ ] マージコンフリクトが発生していない（または適切に解決されている）
- [ ] アプリケーションが正常に動作する（ビルド・テストが通る）
- [ ] 一時保存した変更があれば、正しく復元されている

## 補足: upstreamリモートの設定方法（初回のみ）

フォーク元のリポジトリURLが分かっている場合:

# HTTPSの場合
git remote add upstream https://github.com/[ユーザー名]/[リポジトリ名].git

# SSHの場合
git remote add upstream git@github.com:[ユーザー名]/[リポジトリ名].git
設定の確認:
git remote -vupstreamのURLを変更する場合:
git remote set-url upstream [新しいURL]