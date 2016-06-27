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

## アプリケーションの作成

プロジェクトとアプリケーションの違いを明確にしなければなりません。アプリケーションとはブログや公開レコードのデータベースなどの実際のwebアプリケーションを指します。
プロジェクトはwebアプリケーションを構築するための設定やアプリケーションを集めたものです。1つのプロジェクには複数のアプリケーションを入れることができ、1つのアプリケーションは複数のプロジェクトで使用することができます。

```
(django) login_user@django ~/django-api-server/first_app $ python manage.py startapp polls
(django) logsn_user@django ~/django-api-server/first_app $ tree
.
├── first_app
│   ├── __init__.py
│   ├── __pycache__
│   │   ├── __init__.cpython-35.pyc
│   │   ├── settings.cpython-35.pyc
│   │   ├── urls.cpython-35.pyc
│   │   ├── views.cpython-35.pyc
│   │   └── wsgi.cpython-35.pyc
│   ├── settings.py
│   ├── urls.py
│   ├── views.py
│   └── wsgi.py
├── manage.py
└── polls
    ├── __init__.py
    ├── admin.py
    ├── apps.py
    ├── migrations
    │   └── __init__.py
    ├── models.py
    ├── tests.py
    └── views.py
```

## Viewの作成

views.pyを追加して、indexメソッドを追記して、urls.pyにどのようなリクエストがきた時にindexメソッドを呼ぶのかを定義する。urls.pyは俗に言うcontrollerだと思われる。

 * [修正コミット](https://github.com/KosukeShimofuji/django-api-server/commit/706e06a4374e2df53f03cacb28b67f42da298dda)

```
$ curl django.test:8000
Hello, world. You're at the first_app index.
```

次はpollsにアクセスするためのURIを追記してみましょう。

 * [修正コミット](https://github.com/KosukeShimofuji/django-api-server/commit/6917f4226406d32cf3ec085c1027f07c40bff28b)

```
$ curl django.test:8000/polls/
Hello, world. You're at the polls index.
```

## Modelの作成

以下のコミットでPollテーブルの作成とquestionとpub_dateカラムの作成、Choiceテーブルの作成とpollとchoiseカラムの作成、及び、リレーションの作成を指示しています。

 * [修正コミット](https://github.com/KosukeShimofuji/django-api-server/commit/343a3a236a669eee705f76670eb81f1c0571c62b)

## Modelの有効化

モデルの有効化によって、djangoはcreate tableの実行と、テーブルのアクセスするapiの作成を自動的に行うことができますが、djangoにアプリケーションが作成されたことを教えてあげる必要があります。

 * [修正コミット](https://github.com/KosukeShimofuji/django-api-server/commit/cbe2123f3b5e2ad374f14af6552d25afa0dc0788)

### makemigrations

makemigrationsコマンドを利用してmigration用のスクリプトを生成します。

```
$ python manage.py makemigrations polls
Migrations for 'polls':
  0001_initial.py:
    - Create model Choice
    - Create model Poll
    - Add field poll to choice
```

作成された0001_initial.pyを観察してみましょう。

```
class Migration(migrations.Migration):

    initial = True

    dependencies = [
    ]

    operations = [
        migrations.CreateModel(
            name='Choice',
            fields=[
                ('id', models.AutoField(auto_created=True, primary_key=True, serialize=False, verbose_name='ID')),
                ('choice', models.CharField(max_length=200)),
                ('votes', models.IntegerField()),
            ],
        ),
        migrations.CreateModel(
            name='Poll',
            fields=[
                ('id', models.AutoField(auto_created=True, primary_key=True, serialize=False, verbose_name='ID')),
                ('question', models.CharField(max_length=200)),
                ('pub_date', models.DateTimeField(verbose_name='date published')),
            ],
        ),
        migrations.AddField(
            model_name='choice',
            name='poll',
            field=models.ForeignKey(on_delete=django.db.models.deletion.CASCADE, to='polls.Poll'),
        ),
    ]
```

first_app/polls/models.pyで指示した通りにデータベースを操作しようとしていることがわかります。

### sqlmigrate

sqlmigrateコマンドを利用するとこれから発行しようとしているSQL文を確認することができます。

```
$ python manage.py sqlmigrate polls 0001
BEGIN;
--
-- Create model Choice
--
CREATE TABLE "polls_choice" ("id" serial NOT NULL PRIMARY KEY, "choice" varchar(200) NOT NULL, "votes" integer NOT NULL);
--
-- Create model Poll
--
CREATE TABLE "polls_poll" ("id" serial NOT NULL PRIMARY KEY, "question" varchar(200) NOT NULL, "pub_date" timestamp with time zone NOT NULL);
--
-- Add field poll to choice
--
ALTER TABLE "polls_choice" ADD COLUMN "poll_id" integer NOT NULL;
ALTER TABLE "polls_choice" ALTER COLUMN "poll_id" DROP DEFAULT;
CREATE INDEX "polls_choice_582e9e5a" ON "polls_choice" ("poll_id");
ALTER TABLE "polls_choice" ADD CONSTRAINT "polls_choice_poll_id_3a553f1a_fk_polls_poll_id" FOREIGN KEY ("poll_id") REFERENCES "polls_poll" ("id") DEFERRABLE INITIALLY DEFERRED;

COMMIT;
```

### migrate

migrateコマンドを実行して実際にテーブルを作成してみましょう。

```
$ python manage.py migrate
Operations to perform:
  Apply all migrations: polls, admin, contenttypes, sessions, auth
Running migrations:
  Rendering model states... DONE
  Applying polls.0001_initial... OK
```

想定通りのテーブルができているかをpostgresqlクライアントを利用して確認してみます。

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
 public   | polls_choice               | テーブル | django
 public   | polls_poll                 | テーブル | django

django=> \d polls_choice;
                                テーブル "public.polls_choice"
   列    |           型           |                          修飾語
---------+------------------------+-----------------------------------------------------------
 id      | integer                | not null default nextval('polls_choice_id_seq'::regclass)
 choice  | character varying(200) | not null
 votes   | integer                | not null
 poll_id | integer                | not null
インデックス:
    "polls_choice_pkey" PRIMARY KEY, btree (id)
    "polls_choice_582e9e5a" btree (poll_id)
外部キー制約:
    "polls_choice_poll_id_3a553f1a_fk_polls_poll_id" FOREIGN KEY (poll_id) REFERENCES polls_poll(id) DEFERRABLE INITIALLY DEFERRED

django=> \d polls_poll;
                                 テーブル "public.polls_poll"
    列    |            型            |                         修飾語
----------+--------------------------+---------------------------------------------------------
 id       | integer                  | not null default nextval('polls_poll_id_seq'::regclass)
 question | character varying(200)   | not null
 pub_date | timestamp with time zone | not null
インデックス:
    "polls_poll_pkey" PRIMARY KEY, btree (id)
参照元：
    TABLE "polls_choice" CONSTRAINT "polls_choice_poll_id_3a553f1a_fk_polls_poll_id" FOREIGN KEY (poll_id) REFERENCES polls_poll(id) DEFERRABLE INITIALLY DEFERRED
```

## Modelの修正

モデルの修正はmodels.pyを修正してmakemigrationsを実行し、migrateすることで修正を適用することができます。

## APIで遊んでみる

```
(django) login_user@django ~/django-api-server/first_app $ python manage.py shell
Python 3.5.1 (default, Jun 27 2016, 11:44:53)
[GCC 4.9.2] on linux
Type "help", "copyright", "credits" or "license" for more information.
(InteractiveConsole)
>>> from polls.models import Question, Choice
>>> Question.objects.all()
[]
# レコードの追加
>>> from django.utils import timezone
>>> q = Question(question_text="What's new?", pub_date=timezone.now())
>>> q.save()
>>> Question.objects.all()
[<Question: Question object>]
# レコードの参照
>>> q.id
1
>>> q.question_text
"What's new?"
>>> q.pub_date
datetime.datetime(2016, 6, 27, 7, 27, 32, 951561, tzinfo=<UTC>)
>>> Question.objects.all()
[<Question: Question object>]
```

## modelsに__str__メソッドを追加する

APIでデータベースにアクセスすることができましたが、Question: Question
objectなんていう表記は親切ではないので、__str__メソッドをmodels.pyに付け加えます。

 * [修正コミット](https://github.com/KosukeShimofuji/django-api-server/commit/267fcd4316bdf19000fdd5bef1a75800ceb1be37)

__str__メソッドを加えると、以下のように親切な表記に変更されます。日本語の最新のdjangoのチュートリアルでは__str__メソッドより__unicode__メソッドを使うことを推奨していましたが、私の環境では動作しませんでした。

```
$ python manage.py shell
Python 3.5.1 (default, Jun 27 2016, 11:44:53)
[GCC 4.9.2] on linux
Type "help", "copyright", "credits" or "license" for more information.
(InteractiveConsole)
>>> from polls.models import Question, Choice
>>> Question.objects.all()
[<Question: What's new?>]
```


# 参考文献

 * http://docs.djangoproject.jp/en/latest/index.html
 * https://docs.djangoproject.com/en/1.9/intro/tutorial01/
 * http://qiita.com/kimihiro_n/items/86e0a9e619720e57ecd8


