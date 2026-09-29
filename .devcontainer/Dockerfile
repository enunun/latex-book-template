# miseの公式イメージからは，実行ファイルだけを取り出す．
FROM jdxcode/mise:2026.9.12-debian AS mise

# TeX Liveの公式イメージ(Island of TeX)．Debianをベースに，scheme-fullのTeX Liveを含む．
# latexindent，llmk，biber，upmendex，chktexと，latexindentが使うPerlモジュールも入っている．
# latestはその年のTeX Liveを追いかける．年度を固定したい場合は，TL2025-historicなどのタグにする．
FROM texlive/texlive:latest

# miseの公式イメージと同じ場所にmiseを置き，同じ環境変数を設定する．
COPY --from=mise /usr/local/bin/mise /usr/local/bin/mise
ENV MISE_DATA_DIR=/mise \
	MISE_CONFIG_DIR=/mise \
	MISE_CACHE_DIR=/mise/cache \
	PATH=/mise/shims:$PATH

RUN apt-get update \
	&& apt-get install -y --no-install-recommends \
	ca-certificates \
	curl \
	git \
	openssh-client \
	procps \
	jq \
	libatomic1 \
	&& rm -rf /var/lib/apt/lists/*

# CIなどで，所有者の異なるリポジトリをマウントしてもgitを使えるようにする．
RUN git config --system --add safe.directory /workspaces

# ここにプロジェクト固有のシステムパッケージのインストールを追加する
# (例: 図の作成や変換に使うツールなど)．

# ツール(rtk，lefthookなど)はmise.tomlのとおりにイメージへ入れる．
# 版を固定したい場合は，先に`mise lock`でmise.lockを作り，
# COPYの対象に加えたうえで`mise install --locked`にする．
WORKDIR /tmp/mise-bootstrap
COPY mise.toml ./
RUN mise trust \
	&& mise ls --current \
	&& mise install --verbose

# rtkのhookを有効化する．
RUN mkdir -p /root/.claude \
	&& rtk init -g --hook-only

# 起動後は，リポジトリのmise.tomlをグローバル設定として読む．
# これにより，/workspacesの外のディレクトリでも同じツールを使える．
# グローバル設定のルートも/workspacesにして，タスクをリポジトリ直下で実行させる
# (未指定だとホームディレクトリで実行される)．
ENV MISE_GLOBAL_CONFIG_FILE=/workspaces/mise.toml \
	MISE_GLOBAL_CONFIG_ROOT=/workspaces \
	MISE_TRUSTED_CONFIG_PATHS=/workspaces

WORKDIR /workspaces
