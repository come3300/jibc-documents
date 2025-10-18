# 第4回：TODOの保存先をMySQLに変更

作成日: 2025年4月13日 3:41

# **第4回：TODOの保存先をMySQLに変更しよう**

## **🎯 今回の目標**

- DockerでMySQLを追加する
- データベースに接続してTODOを保存・取得する
- PDOを使った安全なデータ操作を実装する

---

## **📁 構成の変更**

```
todo-app/
├── docker-compose.yml           ← ★MySQL追加
├── php/
│   ├── public/
│   │   └── index.php
│   ├── todo/
│   │   ├── index.php
│   │   ├── add.php
│   │   └── delete.php
│   ├── db/
│   │   └── db.php               ← ★PDO接続用ファイル
│   └── views/
│       └── todo.php
```

---

## **①docker-compose.ymlに MySQL を追加**

```
services:
  app:
    build: ./php
    ports:
      - "8000:80"
    volumes:
      - ./php:/var/www/html
    depends_on:
      - db

  db:
    platform: linux/amd64
    image: mysql:8.0
    container_name: mysql
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: todo_db
      MYSQL_USER: user
      MYSQL_PASSWORD: password
    ports:
      - "3306:3306"
    volumes:
      - db-data:/var/lib/mysql

  phpmyadmin:
    platform: linux/amd64
    image: phpmyadmin/phpmyadmin
    container_name: phpmyadmin
    depends_on:
      - db
    ports:
      - "8080:80"
    environment:
      PMA_HOST: db
      PMA_USER: user
      PMA_PASSWORD: password

volumes:
  db-data:
```

---

## **② TODOテーブルを作成（初回起動後に実行）**

以下を phpmyadmin または CLI で実行：

```
CREATE TABLE todos (
  id INT AUTO_INCREMENT PRIMARY KEY,
  task TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## **③db/db.php（PDO接続）**

```
<?php

$host = 'db';
$dbname = 'todo_db';
$user = 'user';
$password = 'password';

try {
    $pdo = new PDO("mysql:host=$host;dbname=$dbname;charset=utf8mb4", $user, $password);
    $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
} catch (PDOException $e) {
    die('DB接続エラー: ' . $e->getMessage());
}
```

---

## **④todo/index.php（データ取得）**

```
<?php
require_once __DIR__ . '/../db/db.php';

// DBからTODOを全件取得
$stmt = $pdo->query("SELECT * FROM todos ORDER BY created_at DESC");
$todos = $stmt->fetchAll(PDO::FETCH_ASSOC);

// ビューに渡す
require_once __DIR__ . '/../views/todo.php';
```

---

## **⑤todo/add.php（DBに追加）**

```
<?php
require_once __DIR__ . '/../db/db.php';

$task = $_POST['task'] ?? '';

if (trim($task) !== '') {
    $stmt = $pdo->prepare("INSERT INTO todos (task) VALUES (:task)");
    $stmt->execute(['task' => $task]);
}

header('Location: /');
exit;
```

---

## **⑥todo/delete.php（DBから削除）**

```
<?php
require_once __DIR__ . '/../db/db.php';

$id = $_POST['id'] ?? null;

if ($id !== null) {
    $stmt = $pdo->prepare("DELETE FROM todos WHERE id = :id");
    $stmt->execute(['id' => $id]);
}

header('Location: /');
exit;
```

---

## **⑦views/todo.php（ID付きで表示・削除）**

```
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <title>TODOアプリ</title>
</head>
<body>
    <h1>TODOリスト</h1>

    <form action="/add" method="POST">
        <input type="text" name="task" placeholder="やることを入力" required>
        <button type="submit">追加</button>
    </form>

    <h2>やること一覧</h2>
    <ul>
        <?php foreach ($todos as $todo): ?>
            <li>
                <?= htmlspecialchars($todo['task'], ENT_QUOTES, 'UTF-8') ?>
                <form action="/delete" method="POST" style="display:inline;">
                    <input type="hidden" name="id" value="<?= $todo['id'] ?>">
                    <button type="submit">削除</button>
                </form>
            </li>
        <?php endforeach; ?>
    </ul>
</body>
</html>
```

---

## **✅ 動作確認手順**

1. docker compose up -d で環境を起動
2. ブラウザで http://localhost:8000 にアクセス
3. TODOを追加・削除して、データベースに反映されていればOK！

---

## **🔚 今回のまとめ**

- セッション管理からDB管理へ移行
- DockerでMySQLとPHPを連携
- PDOを用いた安全なSQL実行

---

## **📌 次回予告**

次回は「TODOに完了フラグを追加し、完了済みの表示切り替え」を実装していきます！

チェックボックスで完了管理できる仕組みを導入していきます ✅