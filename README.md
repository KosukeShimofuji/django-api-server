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
 * http://qiita.com/kimihiro_n/items/86e0a9e619720e57ecd8


