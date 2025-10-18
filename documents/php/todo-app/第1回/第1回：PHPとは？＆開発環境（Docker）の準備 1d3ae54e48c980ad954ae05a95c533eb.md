# 第1回：PHPとは？＆開発環境（Docker）の準備

作成日: 2025年4月13日 3:23

# **🧑‍🏫 第1回：PHPとは？＆開発環境（Docker）の準備**

## **🌟 はじめに**

PHPを使った、完全初心者向けWebアプリケーション開発講座を始めます！

この講座では **TODOリストアプリ** を作りながら、実際に動くアプリケーションを完成させることを目指します 💪

## **💡 PHPってなに？**

- サーバーサイドで動作するプログラミング言語
- WebサイトやWebアプリケーションの開発に広く利用されている
- WordPressなどのCMSにも採用されており、学習コストが比較的低い

---

## **💻 開発環境を準備しよう**

この講座では **Docker** を使って、PHPの実行環境を簡単に構築します。

Laravelの構成を意識して、以下のようにフォルダを分けて作っていきます。

---

### **📂 プロジェクト構成（初期）**

プロジェクト名：todo-app

```
todo-app/
├── docker-compose.yml        # Docker全体の設定
├── php/
│   ├── Dockerfile            # PHP+Apacheの環境
│   ├── public/               # ブラウザからアクセスされる入口
│   │   └── index.php         # 初期表示用ファイル
│   ├── todo/                 # TODOアプリ本体ロジックを入れる
│   │   └── index.php         # index.phpへルーティングされた処理
│   └── views/                # HTML表示を切り出すビュー
│       └── todo.php          # TODOリストの表示用テンプレート
```

---

### **📝 ファイルを作成しよう**

1️⃣ **docker-compose.yml**

```
version: '3.8'
services:
  app:
    build: ./php
    ports:
      - "8000:80"
    volumes:
      - ./php:/var/www/html
```

2️⃣ **php/Dockerfile**

```
FROM php:8.2-apache

# DocumentRoot を public に変更
RUN sed -i 's!/var/www/html!/var/www/html/public!g' /etc/apache2/sites-available/000-default.conf

# AllowOverride All を追加して .htaccess を有効化
RUN echo '<Directory /var/www/html/public>\n\
    AllowOverride All\n\
</Directory>' >> /etc/apache2/apache2.conf

# 必要な PHP 拡張をインストール（ここが重要！）
RUN docker-php-ext-install pdo pdo_mysql

# ソースコードの配置
COPY . /var/www/html/

# mod_rewrite を有効化
RUN a2enmod rewrite
```

3️⃣ **php/public/index.php**

```
<?php

// TODOアプリのメイン処理に処理を渡す
require_once __DIR__ . '/../todo/index.php';
```

4️⃣ **php/todo/index.php**

（この時点では仮でOK、次回以降で中身を実装）

```
<?php
echo "TODOアプリ開発スタート！";
```

---

### **🚀 環境を起動しよう**

1. ターミナルでプロジェクトフォルダに移動

```
cd todo-app
```

1. Dockerを起動

```
docker compose up -d
```

1. ブラウザで以下のURLを開く

```
http://localhost:8000
```

### **✨ 表示確認**

> 「TODOアプリ開発スタート！」というメッセージが表示されればOKです！
> 

---

## **✅ まとめ**

- PHPはサーバーサイドで動作するWeb開発向け言語
- Dockerを使えば手軽に環境構築できる
- 今後の開発のベースとなるフォルダ構成を用意できた！

---