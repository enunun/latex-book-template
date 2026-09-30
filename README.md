# latex-book-template

LuaLaTeXで日本語の数学書を書くためのテンプレート．
Claude Code for VSCode + Docker(mise) + rtkの開発環境に，TeX Liveの公式イメージを組み合わせている．
原稿の整形にはlatexindent，ビルドにはllmkを使う．

## 構成

``` text
.devcontainer/
  devcontainer.json  VSCode Dev Containersの設定．texlabの拡張機能を入れ，
                      保存時にlatexindentで整形する．claude-home/rtk-homeを
                      ホストにバインドマウントし，資格情報や履歴をコンテナの
                      再作成後も保つ．
  Dockerfile          TeX Liveの公式イメージ(texlive/texlive)をベースに，
                      miseの公式イメージからmiseを取り込み，rtk/lefthookなどをmiseで入れる．
  compose.yml         コンテナを起動したままにする(sleep infinity)だけの設定．
.github/workflows/
  build.yml           開発用コンテナと同じイメージで整形の検査とビルドを行う．
                      Actionsタブから手動で実行し，タグを指定するとPDFをReleaseに添付する．
src/
  main.tex            原稿の入口．書誌情報(書名，著者など)と，章の読み込み順を書く．
  preamble/           プリアンブル．本の中身によらない設定をまとめる．
    packages.tex      パッケージの読み込み．
    fonts.tex         欧文・数式・和文のフォント．
    layout.tex        見出し(§付き)，式番号，キャプションの体裁．
    theorems.tex      定理環境(定義，定理，補題，系，例，注意，演習)．
    macros.tex        括弧類や冪集合など，汎用の数式マクロ．
    index.tex         記号索引と用語索引．
    references.tex    hyperref，cleveref(和文の参照名)，biblatex．
  contents/           本文．1章を1ファイルにする．
    intro.tex         はじめに(前付け)．
    chapter01.tex     本文の見本．定理環境，相互参照，索引，引用の書き方を示す．
    answer.tex        演習の解答(後付け)．
  colophon.tex        奥付．main.texの書誌情報を使う．
  reference/book.bib  参考文献．
  fig/                図．\includegraphicsはここから探す．
  llmk.toml           ビルドの手順(lualatex → biber → upmendex → lualatex × 2)．
  index.ist           索引の体裁(upmendexのスタイル)．
.latexindent.yaml     latexindentの設定．
.claude/
  settings.json        Bashツール呼び出しをrtk経由に書き換えるフック．
                        enunun/system-development-skillsを参照するプラグイン設定も含む．
.rtk/
  filters.toml          プロジェクト固有のrtkフィルタ(雛形のみ)．
mise.toml               ツールの版とタスク(build/fmt/lint/checkなど)．
lefthook.yml             コミット時の整形と検査．
CLAUDE.md                プロジェクト向けのClaude Code指示の雛形．
.gitignore
```

## 使い方

1. このリポジトリの中身を，新しい本のリポジトリのルートにコピーする．
2. `PROJECT_NAME`という文字列を，プロジェクト名に置き換える(`devcontainer.json`，`compose.yml`，`CLAUDE.md`)．
3. `src/main.tex`の書誌情報(`\booktitle`，`\bookauthor`など)を書き換える．
4. `src/contents/chapter01.tex`を見本にして章のファイルを作り，`src/main.tex`から`\include`する．
5. VSCodeで「Reopen in Container」を実行する．初回は`mise run setup`が走る．
6. `.gitignore`から`pnpm-lock.yaml`を削除し，lockファイルがコミットされるようにする．
   あわせて，`.github/workflows/build.yml`の検査を`mise run check`に置き換える．
7. `mise.toml`の`[settings]`と`lockfile = true`の行のコメントを解除し，lockファイルを使用するようにする．

## タスク

コンテナの中で，次のタスクを使う．

| コマンド | 内容 |
| --- | --- |
| `mise run build` | 原稿をビルドし，`src/main.pdf`を作る． |
| `mise run clean` | ビルドで生成したファイルを削除する． |
| `mise run fmt` | TeXのソースをlatexindentで整形する． |
| `mise run lint` | TeXのソースの整形と，Markdownの文書を検査する． |
| `mise run lint:nag` | ビルドし，時代遅れの命令や環境があれば失敗する． |
| `mise run check` | リント，ビルド，`lint:nag`の検査をまとめて実行する． |

コミット時には，lefthookがステージしたTeXのソースをlatexindentで整形し，ステージし直す．
VSCodeでは，TeXのソースを保存したときにtexlab経由でlatexindentが整形する．

## 原稿の書き方

`src/contents/chapter01.tex`に見本がある．

- 定理環境は`Def`，`Thm`，`Lemma`，`Corollary`，`Ex`，`Note`，`Que`(演習)を使う．
  演習以外は節ごとに共通の番号になる．
- 参照は`\cref`，`\Cref`を使う．ラベルは`環境名:識別子`の形にする(例：`Def:group`)．
- 用語は`\term[読み]{用語}`で太字にし，用語索引に載せる．
  記号は`\index[sidx]{読み@記号}`で記号索引に載せる．
- 数の集合は，numbersetsパッケージの`\NaturalNumbers`，`\Integers`，`\RationalNumbers`，`\RealNumbers`，`\ComplexNumbers`で書く．
- 演習の解答は`src/contents/answer.tex`に書く．
- 定理環境を追加するときは，`src/preamble/theorems.tex`と`src/preamble/references.tex`の両方に書く．
- 時代遅れの命令や環境(`\bf`，`eqnarray`，`$$...$$`など)は使わない．
  使うと，nagパッケージがビルドのログ(`src/main.log`)に`Package nag Warning`として代わりの書き方を示し，
  `mise run lint:nag`と`mise run check`，GitHub Actionsのビルドが失敗する．

## TeX Liveのイメージについて

`texlive/texlive:latest`は，その年のTeX Liveのscheme-fullを含む公式イメージである．
latexindentが使うPerlモジュールも含むため，追加のインストールなしにlatexindentを使える．
TeX Liveの年度を固定したい場合は，`Dockerfile`の`FROM`を`texlive/texlive:TL2025-historic`などのタグにする．

## rtk(Rust Token Killer)について

シェルコマンドの出力を絞り込み，トークン消費を抑えるCLIプロキシ．
`.claude/settings.json`のフックが，Claude CodeのBashツール呼び出しを自動的に`rtk`経由に書き換える．
コマンドの詳しい対応表は[rtkのリポジトリ](https://github.com/rtk-ai/rtk)を参照．
`~/.claude/CLAUDE.md`からrtkの使い方を読み込ませておくと，全プロジェクトで効く．

## 共有スキルについて

`.claude/settings.json`は，[enunun/system-development-skills](https://github.com/enunun/system-development-skills)をプラグインのマーケットプレイスとして参照する設定を含む．成果物を仕上げる`finalize-artifacts`スキルなど，プロジェクトを問わず使うスキルはそちらに集約されている．

## claude-home / rtk-home について

`.devcontainer/claude-home/`と`.devcontainer/rtk-home/`は，コンテナ作成時に
`initializeCommand`が自動生成し，
コンテナ内の`/root/.claude`や`/root/.config/rtk`などにバインドマウントされる．
資格情報や履歴を含むため，`.gitignore`で除外している．
