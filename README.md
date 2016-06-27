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

今回導入されたのはdjango1.9

```
$ pip list | grep -i django
Django (1.9.7)
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

```
django=> \dt
                リレーションの一覧
 スキーマ |       名前        |    型    | 所有者
----------+-------------------+----------+--------
 public   | django_migrations | テーブル | django
(1 行)

django=> \d django_migrations
                                 テーブル "public.django_migrations"
   列    |            型            |                             修飾語
---------+--------------------------+----------------------------------------------------------------
 id      | integer                  | not null default nextval('django_migrations_id_seq'::regclass)
 app     | character varying(255)   | not null
 name    | character varying(255)   | not null
 applied | timestamp with time zone | not null
インデックス:
    "django_migrations_pkey" PRIMARY KEY, btree (id)

django=> select * from django_migrations; 
 id | app | name | applied
----+-----+------+---------
(0 行)
```

 * django applicationについて確認する

以下のdjangoアプリケーションがデフォルトで使用されるように設定されており、これらのアプリケーションは最低でも1つのテーブルを利用する。
使わないアプリケーションがあるならこの時点で外しておくのが望ましいが、今はよくわからないのでデフォルトでいく。

```
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```

 * databaseのmigrate

以前はこの操作はsyncdbという名前になっていたようだが、現在はmigrateという名前になっている模様。

```
$ python manage.py migrate
Operations to perform:
  Apply all migrations: sessions, admin, contenttypes, auth
Running migrations:
  Rendering model states... DONE
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying sessions.0001_initial... OK
```

migrate操作によって作成されたテーブルの一覧を見てみる。

```
django-> \dt
                    リレーションの一覧
 スキーマ |            名前            |    型    | 所有者
----------+----------------------------+----------+--------
 public   | auth_group                 | テーブル | django
 public   | auth_group_permissions     | テーブル | django
 public   | auth_permission            | テーブル | django
 public   | auth_user                  | テーブル | django
 public   | auth_user_groups           | テーブル | django
 public   | auth_user_user_permissions | テーブル | django
 public   | django_admin_log           | テーブル | django
 public   | django_content_type        | テーブル | django
 public   | django_migrations          | テーブル | django
 public   | django_session             | テーブル | django
```

テーブル名から大体、何に使用されるテーブルなのかは想像が付く。

 * timezoneの変更

[修正コミット](https://github.com/KosukeShimofuji/django-api-server/commit/c3fe7ebc81c59ea8926aac823830915c99b31bf3)

## Viewの作成

views.pyを追加して、indexメソッドを追記して、urls.pyにどのようなリクエストがきた時にindexメソッドを呼ぶのかを定義する。urls.pyは俗に言うcontrollerだと思われる。
[修正コミット](https://github.com/KosukeShimofuji/django-api-server/commit/706e06a4374e2df53f03cacb28b67f42da298dda)


# 参考文献

 * http://docs.djangoproject.jp/en/latest/index.html
 * https://docs.djangoproject.com/en/1.9/intro/tutorial01/
 * http://qiita.com/kimihiro_n/items/86e0a9e619720e57ecd8


