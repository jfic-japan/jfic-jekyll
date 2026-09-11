# JFIC Web

日本室内自転車競技連盟の公式サイトを管理するリポジトリです。

- 公開サイト: <http://jfic-japan.com>
- GitHub: <https://github.com/jfic-japan/jfic-jekyll>

## このサイトの仕組み

このサイトは [Jekyll] で作られています。Jekyll は、記事ファイル（Markdown または HTML）を、公開用の HTML ファイルに変換する仕組みです。

普段編集するのは `jfic-web/` 配下のファイルです。記事を追加すると、公開環境で Jekyll がサイトを生成します。生成結果の `jfic-web/_site/` は編集しません。

## 公開までの流れ

このサイトは、記事を `main` ブランチへ取り込むと自動的に公開されます。全体の流れは次のとおりです。

```text
Pull Requestをmainへ取り込む
			  |
			  v
GitHubのmainへのpushを検知
			  |
			  v
AWS CodeBuildが起動
			  |
			  v
Jekyllでサイトをビルド
			  |
			  v
生成されたjfic-web/_site/を成果物としてS3へ配置
			  |
			  v
CloudFrontを経由して公開サイトへ配信
```

### CodeBuildで行われること

リポジトリ直下の `buildspec.yml` に、CodeBuildの処理内容が定義されています。主な処理は次のとおりです。

1. Ruby 3.4 と Bundlerを準備する。
2. `jfic-web/` に移動し、`bundle install` でJekyllなどの依存パッケージをインストールする。
3. `bundle exec jekyll build` を実行し、記事や各ページをHTMLへ変換する。
4. `jfic-web/_site/` に生成されたファイルをビルド成果物として渡す。

CodeBuildはサイトを閲覧するためのサーバーではなく、公開用ファイルを生成する処理です。ビルドされたファイルはS3バケットに配置され、その後CloudFrontがS3のファイルを配信します。

### 編集者が意識すること

- 作業用ブランチでは、ローカルの `bundle exec jekyll serve` で表示を確認する。
- Pull Requestを作成し、確認が終わってから `main` に取り込む。
- `main` への取り込み後は自動でビルド・配置・公開されるため、公開してよい内容だけを取り込む。
- 公開直後に古い表示が残る場合は、CloudFrontのキャッシュが更新されるまで少し時間がかかることがある。

自動ビルドが失敗した場合は公開ファイルが更新されません。CodeBuildのログを確認し、エラーを修正してから再度Pull Requestを更新してください。

主な場所は次のとおりです。

| 場所 | 用途 |
| --- | --- |
| `jfic-web/_posts/topics/` | お知らせ、ニュースなどの Topics |
| `jfic-web/_posts/competition_info/` | 大会の詳細情報 |
| `jfic-web/_posts/info/` | 募集案内などの一般情報 |
| `jfic-web/img/` | 画像 |
| `jfic-web/doc/` | PDF、Excelなどの配布資料 |
| `jfic-web/_site/` | Jekyllが生成するファイル。直接編集しない |

## 記事を追加する基本的な流れ

1. リポジトリをローカルに clone する。
2. `main` から作業用のブランチを作る。
3. ローカルで記事を追加または編集する。
4. Jekyllを起動し、ブラウザで表示を確認する。
5. 問題がなければ commit して GitHub に push する。
6. Pull Request（変更を取り込んでもらう依頼）を作る。
7. 内容の確認後、`main` ブランチに取り込まれると公開される。

`main` に直接変更を加えず、Pull Request で確認してもらう運用にしてください。特に、表示確認をしないまま commit や Pull Request を作らないようにしてください。

## ローカルで記事を追加する方法

### 1. リポジトリと作業用ブランチを準備する

初回は次のコマンドでリポジトリを取得します。すでに取得済みの場合は、`cd` から実行してください。

```bash
git clone https://github.com/jfic-japan/jfic-jekyll.git
cd jfic-jekyll
git switch -c update-記事の内容
```

2回目以降は、作業を始める前に `main` の最新状態を取得してから、作業用ブランチを作ります。
```bash
git switch main
git pull origin main
git switch -c update-記事の内容
```

その後、`jfic-web/` 配下のファイルを編集します。記事ファイルの保存場所や書き方は、以下の「記事のファイル名」「Front Matter」「本文」の説明を参照してください。

### 2. 記事を表示して確認する

初回のみ、依存パッケージをインストールします。

```bash
cd jfic-web
bundle install
```

記事を編集したら、同じ `jfic-web` ディレクトリでJekyllを起動します。

```bash
bundle exec jekyll serve
```

ブラウザで <http://localhost:4000> を開き、トップページ、Topics一覧、追加した記事、記事内のリンクや画像を確認します。終了するときはターミナルで `Ctrl` + `C` を押します。

表示確認後、生成だけを確認したい場合は次を実行します。

```bash
bundle exec jekyll build
```

### 3. commitしてPull Requestを作る

表示に問題がなければ、リポジトリのルートで変更を commit し、GitHubへ push します。

```bash
cd ..
git status
git diff
git add jfic-web/_posts/ jfic-web/img/ jfic-web/doc/
git commit -m "全日本選手権2026の記事を追加"
git push -u origin update-記事の内容
```

GitHubで `Compare & pull request` を選び、変更内容とローカルで確認した内容を書いてPull Requestを作成します。

### 4. 記事のファイル名を決める

ファイル名は基本的に次の形式にします。

```text
YYYY-MM-DD-記事を表す英数字.md
```

例:

```text
2026-09-20-japan-championship-results.md
```

日付は記事を公開する日、または記事の内容に対応する日を使います。ファイル名には、日本語・空白・記号をできるだけ使わず、英小文字、数字、ハイフンを使ってください。

Topicsの記事は年ごとのディレクトリに入れます。大会情報も同様に、その大会の年のディレクトリに入れます。

```text
jfic-web/_posts/topics/2026/
jfic-web/_posts/competition_info/2026/
```

### 5. Front Matterを書く

記事ファイルの先頭には、2本の `---` で囲んだ YAML Front Matter が必要です。ここにタイトル、日付、記事の種類などを記入します。

Topicsの例:

```markdown
---
layout: topic_post
title: "全日本選手権の結果を掲載しました"
date: 2026-09-20
tags: topics 2026
categories: topics 2026
---

ここから本文を書きます。
```

大会情報の例:

```html
---
layout: competition_post
title: "全日本選手権 2026"
term: "2026 年 11 月 21 日、22 日"
site: "開催会場名"
tags: competition_info 2026
categories: competition_info 2026
---

ここから大会の詳細を書きます。
```

Front Matter の項目は次の意味です。

| 項目 | 内容 |
| --- | --- |
| `layout` | 表示に使うテンプレート。Topicsは `topic_post`、大会情報は `competition_post`、一般情報は `info_post` |
| `title` | ページのタイトル |
| `date` | 記事の日付。`YYYY-MM-DD` 形式 |
| `tags` | 記事一覧に表示する分類。Topicsは必ず `topics` を含める |
| `categories` | URLの分類。Topicsは `topics 年`、大会情報は `competition_info 年` |
| `term` | 大会情報に表示する開催日程 |
| `site` | 大会情報に表示する会場 |

Front Matter の `---` より前に空行や文章を置かないでください。YAMLの値に `:` や記号を含める場合は、既存記事と同じように引用符で囲みます。

### 6. 本文を書く

Topicsなどの短い記事は Markdown で書けます。

```markdown
## お知らせ

2026年9月20日に開催された大会の結果を掲載しました。

[大会結果の詳細]({{ site.baseurl }}{% post_url /competition_info/2026/2026-09-12-japancup-2026 %})
```

見出しは `#`、箇条書きは `-`、リンクは `[表示する文字](URL)` です。既存記事の書き方をコピーして、タイトル・日付・本文・リンク先を置き換えると安全です。

大会情報のように、レイアウトや表の細かい指定が必要な記事は HTML で書かれている既存記事を参考にしてください。HTMLとMarkdownは同じ記事ファイル内でも使用できます。

### 7. 画像・資料を追加する

画像は `jfic-web/img/`、PDFやExcelなどは `jfic-web/doc/` に追加します。記事本文からは、サイト内のパスを使って参照します。

```html
<img src="{{ site.baseurl }}/img/2026/example.jpg" alt="大会の様子" width="100%">
```

```markdown
[大会要項（PDF）]({{ site.baseurl }}/doc/2026/example.pdf)
```

ファイル名は英数字とハイフンを使い、空白や日本語は避けてください。画像には内容を説明する `alt` を付けてください。大きすぎる画像はページの読み込みが遅くなるため、掲載前に適切なサイズへ縮小してください。

## エラーが出た場合

Jekyllの起動やビルドでエラーが出た場合は、次を確認してください。

- コマンドを `jfic-web` ディレクトリで実行しているか
- 初回の `bundle install` が完了しているか
- Front Matter の `---` が2本あり、インデントやコロンが正しいか
- 画像や資料のファイル名・パスが正しいか
- ファイル名の日付が `YYYY-MM-DD` になっているか

macOSでRubyが入っていない場合は、Homebrewを使って `brew install ruby` でインストールできます。

## GitHub上で直接編集する場合

GitHubの `Edit this file` や `Create new file` から記事を編集することもできます。ただし、この方法ではローカルでJekyllを起動して表示確認できません。リンク切れ、Front Matterの記述ミス、レイアウト崩れなどに気づきにくいため、緊急時を除いて利用しないでください。

やむを得ずGitHub上で編集する場合も、`main`へ直接保存せず、作業用ブランチを作ってPull Requestを作成してください。Pull Requestの本文には、ローカルで確認できていないことを明記してください。

## Pull Requestを作る方法

GitHubでブランチへの変更を保存すると、`Compare & pull request` ボタンが表示されます。表示されない場合は、リポジトリの `Pull requests` → `New pull request` から作成します。

Pull Requestには、次の内容を書いてください。

- 何を追加・変更したか
- 関係する大会名や公開希望日
- ローカルで確認したか、確認したURL
- 確認してほしい点

例:

```text
全日本選手権2026の大会結果を追加しました。
Topicsから大会詳細へリンクしています。
ローカルで表示とリンクを確認済みです。
```

内容に修正が必要な場合は、Pull Requestを閉じずに同じブランチのファイルを修正してください。修正を保存すると、同じPull Requestに自動的に反映されます。

## よくある注意点

- `jfic-web/_site/` は生成物なので、記事の追加先にしない。
- Topics記事と大会詳細記事は別ファイルにする。
- Topicsに掲載する記事には `tags: topics 年` を付ける。
- `date` やファイル名の日付を間違えると、一覧の順番や表示日が意図しないものになる。
- ファイル名やリンクの大文字・小文字は区別されることがある。
- 既存記事をコピーした場合は、タイトル、日付、分類、リンク、画像の差し替え漏れがないか確認する。

## 技術情報

- [Jekyll](https://jekyllrb.com/)
- [Bootstrap](https://getbootstrap.com/)
- [Start Bootstrap - Portfolio Item](https://startbootstrap.com/template-overviews/portfolio-item/)
