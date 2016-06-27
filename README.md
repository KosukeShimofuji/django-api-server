# djangoでAPIサーバを作成する

djangoでAPIサーバを作成し、rest形式のリクエストを受け取り、openstackを操作するapiサーバを作成する。

# 環境作成

ansibleディレクリに移動して、djangoの学習をするためのマシンを用意する

```
$ vagrant up
$ ansible-playbook -i development site.yml
```

# pythonとvirtualenvで開発環境を用意する

 * pyenv

```
$ pyenv install 3.5.1
$ pyenv global 3.5.1
```

 * virtualenv

```
$ pyenv virtualenv 3.5.1 django 
$ pyenv global django
```

# djangoのインストール

 * djangoをpipでインストールする。

```
$ pip install --upgrade pip
$ pip install django
```

# djangoのチュートリアル

[チュートリアル](http://docs.djangoproject.jp/en/latest/intro/tutorial01.html)を参考にdjangoの使い方な学んでいく。

 * git branchの作成

```
$ git clone git@github.com:KosukeShimofuji/django-api-server.git
$ cd django-api-server/
$ git branch first-app
$ git checkout first-app
```

 * djangoでプロジェクトを作成

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

 * 開発用サーバの起動

first_appディレクトリに移動してから以下のコマンドで開発用サーバを起動する。

```
$ python manage.py runserver 0.0.0.0:8000
```

# 参考文献

 * http://docs.djangoproject.jp/en/latest/index.html
 * http://qiita.com/kimihiro_n/items/86e0a9e619720e57ecd8


