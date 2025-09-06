OSS開発前の練習用 Git/GitHub 手順（PR・FB・マージ体験）
目的
プルリクエスト（PR）を作成し、フィードバック（FB）を受けて修正し、マージされるまでの一連の流れを短時間で体験する
「同一リポジトリ内PR（簡易）」と「フォークを使ったOSS風（本番想定）」の2コースを用意
想定・前提
Git/GitHubの基本操作を一度は触ったことがある
既定ブランチ名は main と仮定（master の場合は読み替え）
OSは任意（macOS/Linux/Windows）
Git 2.30+ 推奨
事前準備
GitHubアカウント
認証（どれか一つ）
HTTPS + Personal Access Token（推奨）
SSH鍵（ssh -T git@github.com で確認）
Git設定
bash

git config --global user.name "Your Name"
git config --global user.email "you@example.com"
目次
コース1: 同一リポジトリ内PR（簡易コース）
コース2: フォークを使ったOSS風（おすすめ）
実戦の小ワザ
よくある詰まりポイント
付録A: 最小のGitHub Actions（CI）
付録B: PRテンプレート最小例
付録C: ブランチ戦略メモ
付録D: このMarkdownをPDFにする方法
コース1: 同一リポジトリ内PR（簡易コース）
このコースではあなたが所有する単一リポジトリでPR→レビュー→マージを体験します。

1) 練習用リポジトリの作成と初期PR
GitHubで新規リポジトリ example-practice を作成（README生成にチェックを入れる）
ローカルへcloneし、初期ファイルを追加
bash

git clone https://github.com/<you>/example-practice.git
cd example-practice
git switch -c setup
mkdir -p docs
printf "Hello OSS\n" > docs/guide.md
git add .
git commit -m "chore: add initial docs"
git push -u origin setup
GitHubで setup → main へPRを作成し、通常マージ（最初のマージを経験）
2) Issueを作成（PRと紐付ける）
Issuesタブ → New issue
タイトル: Fix typo in docs/guide.md
説明: 「ガイドの誤字を修正します」
3) 作業ブランチを切る（main最新化→分岐）
bash

git fetch origin
git switch main
git pull --ff-only origin main
git switch -c fix/guide-typo
4) 変更・コミット
docs/guide.md の誤字修正、1行追記
bash

git add docs/guide.md
git commit -m "docs: fix typo in guide and add a note (Fixes #<issue番号>)"
Fixes #番号 を含めると、PRマージ時にIssueが自動Close
5) Push
bash

git push -u origin fix/guide-typo
6) PR作成
base: main、compare: fix/guide-typo
タイトル例: docs: fix typo in guide
説明に「目的・変更点・テスト・関連Issue（Fixes #）」を記載
ドラフトPRでもOK
7) フィードバック（FB）の擬似体験
自分でPRの Files changed タブから行コメントや「Add a suggestion」を追加してみる
Organizationや別アカウントがあればレビュワーを追加して「Request changes → 修正 → Approve」の流れを体験
可能ならブランチ保護ルールを設定（Settings → Branches → Add rule → Require a pull request before merging）
8) 追加修正（FB反映）
bash

# 指摘対応の修正
git add <files>
git commit -m "docs: apply review suggestions"
git push
PushするとPRに変更が反映される
9) マージ
推奨: Squash and merge（履歴をきれいに保ちやすい）
代替: Merge commit / Rebase and merge
マージ後、GitHubで「Delete branch」をクリック
10) 後片付け（ローカル・リモート）
bash

git switch main
git pull --ff-only origin main
git branch -d fix/guide-typo
所要時間目安: 30〜60分

コース2: フォークを使ったOSS風（おすすめ）
実際のOSS貢献に近い流れ（fork → 変更 → 自分のforkへpush → upstreamへPR → レビュー → マージ → fork同期）

0) リモート構成
upstream: 貢献先（模擬的にあなたの別リポジトリや組織のリポジトリ）
origin: あなたのフォーク
1) フォーク作成・clone・upstream追加
bash

# GitHubの対象リポジトリで Fork ボタンを押す → 自分のアカウントへ作成
git clone https://github.com/<you>/<repo>.git
cd <repo>
git remote add upstream https://github.com/<owner>/<repo>.git
git fetch upstream
2) ベースブランチを最新化して分岐
bash

git switch -c feature/add-checklist upstream/main
3) 変更・コミット
bash

mkdir -p .github
cat > .github/pull_request_template.md <<'EOF'
## 目的
## 変更点
## 動作確認
## 影響範囲/注意点
## 関連Issue
EOF

git add .github/pull_request_template.md
git commit -m "chore: add PR template"
4) 自分のフォークにpush
bash

git push -u origin feature/add-checklist
5) upstreamへPR作成
GitHubで PR 作成
base: <owner>:main（upstream）
compare: <you>:feature/add-checklist（自分のfork）
説明に目的・影響範囲・テスト方法を記載
6) レビュー・FB
upstreamのMaintainer（あなたの別アカウントで代用可）がレビュー
コメント、Suggestion、Request changes などを試す
CI（Actions）を走らせると実戦的（付録A）
7) 最新追従と修正
bash

# 最新のupstream/mainへ追従（rebase派）
git fetch upstream
git rebase upstream/main
# 競合があれば解消 → 続行
git push --force-with-lease
マージ派は git merge upstream/main でも可（rebaseかmergeはチーム方針に従う）
8) マージとクリーンアップ、fork同期
upstream側でマージ（Squash推奨）
forkを同期
bash

git fetch upstream
git switch main
git pull --ff-only upstream main
git push origin main
git branch -d feature/add-checklist
GitHubでリモートの作業ブランチを削除（PR画面から可能）
所要時間目安: 45〜90分（レビュー役を別アカウントで用意する場合 +15分）

実戦の小ワザ
ベースブランチの更新を取り込みたい（作業ブランチ上で）
bash

git fetch origin
git rebase origin/main      # 線形履歴派
# または
git merge origin/main       # マージコミット派
別ブランチの特定ファイルだけ取り込みたい
bash

git fetch origin
git restore --source origin/other-branch -- path/to/file
# 由来を残したい場合はコミット単位で移植
git cherry-pick <commit-hash>
直前のコミットを修正
bash

git commit --amend
未pushのやり直し（注意: 破壊的）
bash

git reset --hard HEAD~1
push済みの履歴書き換えが必要なら（周知の上で）
bash

git push --force-with-lease
よくある詰まりポイント
PRが大きすぎる → 変更は小さく分割し、1PR=1テーマを徹底
コンフリクトが頻発 → こまめに rebase/merge で追従。ドラフトPRで早めに可視化
自分のPRを自分でApproveできない → ブランチ保護/権限設定の影響。練習用にコラボレーターやサブアカウントを用意
認証エラー → HTTPSトークン期限、SSH鍵、2FA設定を確認
PRの宛先ブランチを間違える → どのブランチに入れたいか（main, release系など）を最初に決めてから分岐
付録A: 最小のGitHub Actions（Nodeの例）
.github/workflows/ci.yml

yaml

name: CI
on:
  pull_request:
  push:
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      - run: npm test --if-present
付録B: PR説明テンプレート（最小例）
.github/pull_request_template.md

md

## 目的

## 変更点

## 動作確認

## 影響範囲/注意点

## 関連Issue
Fixes #<番号>
付録C: ブランチ戦略メモ
ベースにするブランチは「PRの宛先」と一致させる（例: release/1.2 宛てならそこから分岐）
命名例
feature/*（機能追加）
fix/*（バグ修正）
chore/*（雑務・設定）
docs/*（ドキュメント）
追従方法
線形履歴を重視 → rebase
マージ履歴を残す → merge
付録D: このMarkdownをPDFにする方法
方法A（手軽）
このMarkdownをファイルに保存（例: practice-oss-git.md）
ブラウザやエディタで開く
印刷 → PDFに保存（余白は適宜調整）
方法B（Pandoc）
Pandocをインストール
次を実行
bash

pandoc -V papersize:a4 -V geometry:margin=20mm -o practice-oss-git.pdf practice-oss-git.md
方法C（GitHub経由）
リポジトリに practice-oss-git.md を追加
GitHubでファイルを開く → ブラウザの印刷 → PDFに保存

