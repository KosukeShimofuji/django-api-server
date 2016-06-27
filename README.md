# djangoでAPIサーバを作成する

djangoでAPIサーバを作成し、rest形式のリクエストを受け取り、openstackを操作するapiサーバを作成する。

# 環境作成

ansibleディレクリに移動して、djangoの学習をするためのマシンを用意する

```
$ vagrant up
$ ansible-playbook -i development site.yml
```

## pyenv

```
$ pyenv install 3.5.1
$ pyenv global 3.5.1
```

## virtualenv

```
$ pyenv virtualenv 3.5.1 django 
$ pyenv global django
```

## djangoのインストール

 * djangoをpipでインストールする。

```
$ pip install --upgrade pip
$ pip install django
$ pip install psycopg2 # djangoからpostgresqlを操作するために必要
```

## postgresqlのインストール

djangoのチュートリアルではpython2.5以降から同梱されているsqlite3を使用することが推奨されていますが、私の場合は実運用する時はpostgresqlかmysqlになりそうなのでpostgresqlと連携してdjangoを使用します。
本環境では以下のようにpostgresqlのデータベースを設定しています。

```
dbname: "django"
dbuser: "django"
dbpass: "hagPijhoajdegocmuOtvethapOcvigry"
```

接続をテストしてみる。

```
$ psql -hlocalhost -Udjango
ユーザ django のパスワード:
psql (9.4.8)
SSL接続(プロトコル: TLSv1.2, 暗号化方式: ECDHE-RSA-AES256-GCM-SHA384, ビット長: 256, 圧縮: オフ)
"help" でヘルプを表示します.
```

# djangoのチュートリアル

[チュートリアル](http://docs.djangoproject.jp/en/latest/intro/tutorial01.html)を参考にdjangoの使い方な学んでいく。

## git branchの作成

```
$ git clone git@github.com:KosukeShimofuji/django-api-server.git
$ cd django-api-server/
$ git branch first-app
$ git checkout first-app
$ git add fitst-app
$ git commit -m "first commit"
$ git push origin first-app
```

## djangoでプロジェクトを作成

```
$ django-admin.py startproject first_app
$ tree first_app/
first_app/
├── first_app
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
└── manage.py

1 directory, 5 files
```

## 開発用サーバの起動

first_appディレクトリに移動してから以下のコマンドで開発用サーバを起動する。

```
$ python manage.py runserver 0.0.0.0:8000
```

## firtst_appの設定を行う

アプリケーションの設定はsettings.pyを編集することで行うことができる。

 * postgresqlを利用するに設定

[修正コミット](https://github.com/KosukeShimofuji/django-api-server/commit/fceb8011a82ee1ee1be496ec4bfdcd34bce6252f)


# 参考文献

 * http://docs.djangoproject.jp/en/latest/index.html
 * http://qiita.com/kimihiro_n/items/86e0a9e619720e57ecd8


