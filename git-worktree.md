# Git ワークツリー（worktree）使い方ガイド

## 概要

Gitのワークツリーは、1つのリポジトリで複数の作業ディレクトリを同時に持てる機能です。
異なるブランチでの作業を並行して行いたい場合に便利です。

## 基本コマンド

### ワークツリーを作成

```bash
# 新しいブランチを作成してワークツリーを追加
git worktree add -b <新ブランチ名> <パス> [<元ブランチ>]

# 既存のブランチでワークツリーを追加
git worktree add <パス> <既存ブランチ名>
```

### ワークツリーの一覧を表示

```bash
git worktree list
```

### ワークツリーを削除

```bash
# 通常の削除
git worktree remove <パス>

# 強制削除
git worktree remove --force <パス>
```

### 不要なワークツリー情報をクリーンアップ

```bash
git worktree prune
```

## 実用例

### 1. 新規ブランチでワークツリーを作成

```bash
# main ブランチから feature-login ブランチを作成
git worktree add -b feature-login ../feature-login main

# 作成されたディレクトリに移動
cd ../feature-login

# 通常通り作業
git status
git add .
git commit -m "Add login feature"
```

### 2. リモートブランチからワークツリーを作成

```bash
# リモートブランチを確認
git branch -a

# リモートブランチからワークツリーを作成
git worktree add ../bugfix origin/bugfix-123
```

### 3. 並行作業のシナリオ

```bash
# メインの作業は main ブランチで継続
cd /path/to/main/repo

# 緊急のバグ修正用にワークツリーを作成
git worktree add -b hotfix-urgent ../hotfix main

# hotfix ディレクトリで修正作業
cd ../hotfix
# ... 修正作業 ...
git commit -am "Fix critical bug"
git push origin hotfix-urgent

# 元のディレクトリに戻って開発を継続
cd -

# hotfix が不要になったら削除
git worktree remove ../hotfix
```

### 4. レビュー用にワークツリーを作成

```bash
# PR のレビュー用に別ブランチをチェックアウト
git worktree add ../review-pr-123 origin/feature-new-api

# レビュー用ディレクトリでビルド・テスト
cd ../review-pr-123
npm install
npm test

# レビュー完了後に削除
cd -
git worktree remove ../review-pr-123
```

## よくあるエラーと対処法

### エラー: `fatal: invalid reference: <ブランチ名>`

**原因**: 指定したブランチが存在しない

**対処法**:
```bash
# -b オプションで新規ブランチを作成
git worktree add -b feature-new ../feature-new

# または既存のブランチ名を確認
git branch -a
```

### エラー: `fatal: '<ブランチ名>' is already checked out at '<パス>'`

**原因**: 同じブランチが既に他のワークツリーでチェックアウトされている

**対処法**:
```bash
# 既存のワークツリーを確認
git worktree list

# 不要なら削除
git worktree remove <パス>

# または別のブランチ名を使用
```

## メリット

- ✅ ブランチ切り替え（checkout）なしで複数ブランチで同時作業できる
- ✅ ビルドやテストを並行実行できる
- ✅ コンテキストスイッチが少なくて済む
- ✅ 作業中の変更を stash する必要がない

## 注意点

- ⚠️ 同じブランチを複数のワークツリーでチェックアウトできない
- ⚠️ ディスク容量は複数分必要（ただし `.git` は共有）
- ⚠️ 各ワークツリーに `.git` ファイル（実体へのリンク）が作られる

## ベストプラクティス

### ディレクトリ構成の例

```
project/
├── main-repo/          # メインのワークツリー
├── worktrees/
│   ├── feature-a/      # feature-a ブランチ
│   ├── feature-b/      # feature-b ブランチ
│   └── hotfix/         # hotfix ブランチ
```

### クリーンアップの習慣

```bash
# 定期的に不要なワークツリーを削除
git worktree list
git worktree remove <不要なパス>
git worktree prune
```

### ワークツリーの移動

```bash
# ワークツリーを移動したい場合は、一度削除して再作成
git worktree remove <古いパス>
git worktree add <新しいパス> <ブランチ名>

# または move コマンド（Git 2.17以降）
git worktree move <古いパス> <新しいパス>
```

## まとめ

ワークツリーは以下のような場面で特に有効です:

- 複数の機能を並行開発している時
- レビュー中にも開発を継続したい時
- 緊急対応と通常開発を切り替えたい時
- 異なるバージョンでテストを実行したい時

`git worktree` を活用して、より効率的な開発ワークフローを実現しましょう！
