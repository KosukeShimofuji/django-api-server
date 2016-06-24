# djangoでAPIサーバを作成する

djangoでAPIサーバを作成し、rest形式のリクエストを受け取り、openstackを操作するapiサーバを作成する。

# 環境作成

ansibleディレクリに移動して、djangoの学習をするためのマシンを用意する

```
$ vagrant up
$ ansible-playbook -i development site.yml
```

# pythonの基礎環境を用意する

```
$ pyenv install 2.7.11
$ pyenv global 2.7.11
```

# djangoのインストール

 * djangoをpipでインストールする。

```
$ pip install --upgrade pip
$ pip install django
```

# djangoのチュートリアル

[チュートリアル](http://docs.djangoproject.jp/en/latest/intro/tutorial01.html)を参考にdjangoの使い方な学んでいく。

 * プロジェクトの作成

```
$ django-admin.py startproject mysite
$ tree mysite/
mysite/
├── manage.py
└── mysite
    ├── __init__.py
    ├── settings.py
    ├── urls.py
    └── wsgi.py

1 directory, 5 files
```

# 参考文献

 * http://docs.djangoproject.jp/en/latest/index.html

