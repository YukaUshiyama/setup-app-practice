# Laravel環境構築する手順

# １　プロジェクト用のディレクトリを作成、そのディレクトリに移動

# ２　ディレクトリ内にLaravelプロジェクトを作成

例：
・Laravel 10.xプロジェクトを作成
＿＿＿＿＿＿＿＿＿＿
docker run --rm \
 -u "$(id -u):$(id -g)" \
 -v "$(pwd):/var/www/html" \
 -w /var/www/html \
 -e COMPOSER_CACHE_DIR=/tmp/composer_cache \
 laravelsail/php82-composer:latest \
 composer create-project laravel/laravel:^10.0 〇〇
＿＿＿＿＿＿＿＿＿＿

//Laravel 10.xをインストールした〇〇(ディレクトリ名)を作成

# ３　 Laravel Sailをインストールする

例：
・プロジェクトディレクトリに移動
cd 〇〇(２で作成したディレクトリ)

・Laravel Sailのインストール
＿＿＿＿＿＿＿＿＿＿
docker run --rm \
 -u "$(id -u):$(id -g)" \
 -v "$(pwd):/var/www/html" \
 -w /var/www/html \
 -e COMPOSER_CACHE_DIR=/tmp/composer_cache \
 laravelsail/php82-composer:latest \
 composer require laravel/sail --dev
＿＿＿＿＿＿＿＿＿＿

# ４　Sailの設定ファイル作成

例：
・Sailの設定ファイルを生成
＿＿＿＿＿＿＿＿＿＿
docker run --rm \
 -u "$(id -u):$(id -g)" \
 -v "$(pwd):/var/www/html" \
 -w /var/www/html \
 -e COMPOSER_CACHE_DIR=/tmp/composer_cache \
 laravelsail/php82-composer:latest \
 php artisan sail:install --with=mysql
＿＿＿＿＿＿＿＿＿＿

# 5　.envファイルを確認する(必要あれば)

# 6 phpMyAdminを追加する(データベースをブラウザ管理可能にさせるため)

手順
compose.yamlを開いてmysqlサービスの後に
＿＿＿＿＿＿＿＿＿＿
phpmyadmin:
image: 'phpmyadmin:latest'
ports: - '${FORWARD_PHPMYADMIN_PORT:-8080}:80'
        environment:
            PMA_HOST: mysql
            PMA_USER: '${DB_USERNAME}'
PMA_PASSWORD: '${DB_PASSWORD}'
networks: - sail
depends_on: - mysql
＿＿＿＿＿＿＿＿＿＿
を記入。
！この際インデントがずれると正しく動作しないため注意が必要！

# 7 ファイルの構築は完了Sailを起動する

./vendor/bin/sail ps

# 8　エイリアス設定(マストではない)

なんのため？->毎回立ち上げの度./vendor/bin/sail psと打つのは面倒かつタイプミスがあると時間のロスに繋がるためコードを簡潔で簡単なものにするため
＿＿＿＿＿＿＿＿＿＿
echo "alias sail='[ -f sail ] && bash sail || bash vendor/bin/sail'" >> ~/.zshrc
＿＿＿＿＿＿＿＿＿＿
を打ち込んで設定->その後設定を反映するために
＿＿＿＿＿＿＿＿＿＿
exec $SHELL
＿＿＿＿＿＿＿＿＿＿
これをすると今後このプロジェクト起動の際は
＿＿＿＿＿＿＿＿＿＿
sail up -d
＿＿＿＿＿＿＿＿＿＿
だけで起動させることができる！

# 9 アプリケーションキー作成

セキュリティーのため必要
＿＿＿＿＿＿＿＿＿＿
エイリアス未設定の場合
->
./vendor/bin/sail artisan key:generate

エイリアス設定済の場合
->
sail artisan key:generate
＿＿＿＿＿＿＿＿＿＿

# 10 環境構築完了動作確認を行う

ブラウザでURLにアクセスして正しく表示されているか確認

# 最後にGitHubにpushする

＿＿＿＿＿＿＿＿＿＿
初めてgitにコミットさせる場合
->
１Gitにpushしたいディレクトリに移動
２[git init]で今いるディレクトリをgit環境に置く
３[git add .]変更したファイルを全部「次のコミットに含めますよ」という準備（ステージング）をする
→ . は今いるディレクトリ以下全部という指定
4[git commit -m "first commit"]ステージングした内容を履歴として保存する
→" "内はコミットメッセージ。何をコミットするのかわかりやすく簡潔に書く
5[git remote add origin <リポジトリURL>]このローカルリポジトリ(今いるディレクトリ)とGitHub上のリポジトリを関連付ける
6[git branch -M main]現在のブランチ名を「main」に変更する
→ブランチとは？　開発するための作業スペース
→ -M 強制的に名前を変更するという意味合いのコマンド
7[git push -u origin main]mainブランチのコミット履歴をorigin（GitHub）へ送る
→ -u 今後このmainはorigin/mainとセットですよと宣言するコマンド次からgit pushだけで済む

二回目以降gitにコミットさせる場合
１[git status]で変更を確認(pushが必要なファイルがあるかどうか)
２[git add 〇〇]変更したファイルを「次のコミットに含めますよ」という準備（ステージング）をする。〇〇に変更したファイル名
３[git commit -m "〇〇"]〇〇にわかりやすいコミットメッセージ
４[git push]GitHubにコミット履歴を送る
＿＿＿＿＿＿＿＿＿＿
