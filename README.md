# 📚 Gyupgangi（겹강이）

## 🧩 プロジェクト概要

**Gyupgangiは、時間割ベースのマッチングアルゴリズムを通じて、大学生同士の自然な関係形成を支援するWebサービスです。**

ユーザーが時間割を登録すると、

* 同じ授業を受講している学生を自動的に検出し、

* 共通講義数に基づいて友達を推薦します。

また、

* 友達関係である場合のみチャットやプロフィール閲覧が可能となるよう設計し、

* **プライバシーと安全性を同時に確保したSNS型サービス**です。

---

## 🚀 問題定義

大学生は同じ授業を受けることで自然な接点はあるものの、

* 自分から話しかけることが難しい
* 情報共有やグループ課題のチーム構成が困難
* 特に内向的な学生は関係形成のハードルが高い

👉 そこで、
**「共通の授業」という自然な接点を活用したサービスを設計しました。**

---

## ⚙ 主な機能

### 1⃣ 時間割ベースの友達マッチング

* 共通講義数の計算
* 同一授業に基づくユーザー推薦

### 2⃣ 友達申請システム

* 申請 / 承認 / 保留の状態管理
* `FRIEND_STATUS` による関係制御

### 3⃣ 通知システム

* 友達申請・承認時の通知生成
* 通知履歴の保存

### 4⃣ 制限付きアクセス制御

* 友達関係の場合のみ：

  * チャット可能
  * プロフィール閲覧可能


---

## 🛠 技術スタック

* **Frontend:** HTML, CSS
* **Backend:** Python (Flask)
* **Database:** SQLite3
* **Realtime:** Flask-SocketIO
* **Architecture:** MVC Pattern

その他：

* Web Crawling → 講義データ収集
* PythonAnywhere → デプロイ
* React Native (Expo) → WebView化

---

## 🧠 コアロジック

### 📌 マッチングアルゴリズム

* ユーザー間の時間割比較
* 共通講義数に基づく推薦


### 📌 関係状態モデル（重要設計）

```text
FRIEND_STATUS
0 → 初期（マッチング状態）
1 → 申請中
2 → 友達確定
```

👉 単一の属性で関係フロー全体を制御


### 📌 通知システム

* 構造：`{sender_id, user_id, message}`
* イベントベース通知生成
* 履歴保存

👉 機能だけでなく**ユーザー体験の流れまで考慮した設計**

---

## 🗄 データベース設計

### 主要エンティティ

#### 1. MEMBERS（ユーザー）

* ID, PW, EMAIL, NAME, NICKNAME
* INTRO, YEAR, SEX
* MAJOR1, MAJOR2（複合属性）


#### 2. TOTALLECTURES（講義）

* 講義情報管理
* 曜日は多値属性として分離


#### 3. LECTURES（履修関係）

* ユーザー ↔ 講義（M:N関係）


#### 4. FRIEND（コアエンティティ🔥）

* (USER1, USER2) 複合キー
* FRIEND_STATUS

👉 **ユーザー関係の状態管理を担う中心テーブル**


#### 5. NOTIFICATION（通知）

* ユーザー間イベント保存


#### 6. MESSAGE（チャット）

* 友達関係時のみ利用可能

---

## 🏗 プロジェクト構造

```bash
KHU-Gyupgangi-app/
├── application.py
├── requirements.txt
├── README.md
├── .gitignore
├── docs/  
│   ├── Gyupgangi_Database_Design_Report.pdf
│   └── Gyupgangi_Final_Presentation.pdf
├── data/
│   └── lectures.csv
├── database/
│   ├── sqlite_temp.py
│   └── .gitkeep
├── static/
│   └── img/
│       ├── profile.png
│       ├── logo_name.png
│       └── splash.png
├── templates/
│   ├── auth/
│   │   ├── start.html
│   │   └── regmember.html
│   ├── timetable/
│   │   ├── search.html
│   │   ├── search_result.html
│   │   └── timetable.html
│   ├── friends/
│   │   ├── findfriend_home.html
│   │   ├── friend_list.html
│   │   ├── realfriend_list.html
│   │   └── friend_profile.html
│   ├── profile/
│   │   ├── myprof_home.html
│   │   └── myprof_rev.html
│   ├── chat/
│   │   ├── friend_chat.html
│   │   └── chat.html
│   ├── notifications/
│   │   └── notification_history.html
│   └── common/
│       ├── hello.html
│       └── home.html
└── utils/
    └── calculateTime.py
```

本プロジェクトは、現在 `application.py` にすべてのルーティングおよび主要なバックエンドロジックを実装している構成です。  ただし、可読性向上のために、リポジトリ上では機能ごとにテンプレート・データ・ユーティリティを整理しています。

---

### 1⃣ ルーティング構成（`application.py`）

`application.py` では、以下のように機能単位でルートを分類できます。

#### 🔹 auth
- `/`, `/login`, `/logout`, `/regmember`, `/addmem`

#### 🔹 timetable
- `/enternew`, `/search`, `/addsubject`, `/delsubject`, `/timetable`

#### 🔹 friends
- `/findfriend_home`, `/friend_list`, `/realfriend_list`, `/accept_friend`, `/accept_friend_request`

#### 🔹 profile
- `/myprofile`, `/enternewprof`, `/updateprof`, `/friend_profile`

#### 🔹 chat
- `/friend_chat`, `/chat/<friend_id>`, `/send_message`, `/chat_history/<room>`

#### 🔹 notifications
- `/notification_history`


---

### 2⃣ 各ファイル・フォルダの役割

#### 📁 data/

* **lectures.csv**
  講義情報を保存したCSVファイルです。
  初期講義データとして利用し、SQLiteデータベースへ投入するために使用します。


#### 📁 database/

* **sqlite_temp.py**
  SQLiteデータベースの初期化用スクリプトです。
  テーブル作成や講義データの登録処理を担当します。


#### 📁 templates/auth/

* **start.html**
  初期画面・ログイン画面です。

* **regmember.html**
  会員登録画面です。


#### 📁 templates/timetable/

* **search.html**
  講義検索画面です。

* **search_result.html**
  検索結果一覧および講義選択画面です。

* **timetable.html**
  ユーザー時間割の表示・追加・削除を行う画面です。


#### 📁 templates/friends/

* **findfriend_home.html**
  友達検索開始画面です。

* **friend_list.html**
  マッチングされたユーザー一覧および友達申請画面です。

* **realfriend_list.html**
  申請待ち・承認済みの友達一覧を表示する画面です。

* **friend_profile.html**
  他ユーザーのプロフィールと時間割を閲覧する画面です。


#### 📁 templates/profile/

* **myprof_home.html**
  自分のプロフィール表示画面です。

* **myprof_rev.html**
  自分のプロフィール編集画面です。


#### 📁 templates/chat/

* **friend_chat.html**
  チャット可能な友達一覧を表示する画面です。

* **chat.html**
  1対1リアルタイムチャット画面です。


#### 📁 templates/notifications/

* **notification_history.html**
  通知履歴一覧を表示する画面です。


#### 📁 templates/common/

* **hello.html**
  ログイン後のメインナビゲーション画面です。

* **home.html**
  ホーム画面です。通知アイコンや主要導線を提供します。


#### 📁 utils/

* **calculateTime.py**
  講義時間データを時間割テーブル形式に変換するユーティリティです。
  時間割生成、講義時刻変換、重複講義判定に利用されます。

---

## ▶ ローカル実行方法

```bash
# 1. リポジトリをクローン
git clone https://github.com/username/gyeopgang-i.git

# 2. 仮想環境作成・有効化
python -m venv venv
source venv/bin/activate   # Mac/Linux
venv\Scripts\activate      # Windows

# 3. 依存パッケージインストール
pip install -r requirements.txt

# 4. サーバー起動
flask run
```

---

## ⚠ 制約・課題および学び

### 🔹 1. サーバー環境と技術選定の制約

* Flask + SQLite構成で開発を進めたが、
  多くのホスティングサービスがMySQLを前提としており、選択肢が制限された
* PythonAnywhereを利用してデプロイを行ったが、
  **Flask-SocketIO（リアルタイム通信）が正常に動作しない問題が発生**

* **学び：** 技術選定は「開発のしやすさ」だけでなく「デプロイ環境との互換性」まで考慮する必要がある。
リアルタイム機能はインフラ依存度が高い


### 🔹 2. データベース設計とスケーラビリティ

* 初期設計では、ユーザー情報に関連データを統合しようとしたが
講義数、メッセージ数、友達数の増加によりデータ量が急増し、非効率が発生

* リレーション分離により改善したが、**DBサイズ増加とパフォーマンス問題を実感**

* **学び：** 正規化だけでなく**「データ増加時のスケーラビリティ」を考慮した設計が重要**


### 🔹 3. 状態管理（FRIEND_STATUS）の重要性

* 初期は複数のフラグで関係を管理しようとしたが、
  状態遷移が複雑化しバグが発生

* **単一の状態値（FRIEND_STATUS）に統一することで解決**

* **学び：****「状態を明確に定義すること」がシステム設計の安定性を大きく左右する。**
状態設計はDB設計だけでなく**アクセス制御やUXにも直結する**


### 🔹 4. 通知システムのUX課題

* 通知の既読（SEEN）処理により
  一度確認した通知が再表示されない問題が発生
* テストのために複数アカウントを作成する必要があり、検証コストが高かった

* **学び：**機能実装だけでなく **「ユーザー体験（UX）」を考慮した設計が必要。**
状態管理（既読/未読）は慎重に設計すべき


### 🔹 5. 機能実装と時間的制約

* iOS環境では一部機能が正常動作せず
* 実ユーザーによるベータテストを十分に実施できなかった

 **学び：** 機能完成度よりも **「ユーザー検証（フィードバック）」がサービス改善に重要。**
 クロスプラットフォーム対応の難しさを理解

---

## 🔧 今後の改善

* MySQL / PostgreSQLへの移行
* Flask Blueprintによるモジュール分割
* SocketIO対応可能なインフラへの移行
* インデックス設計およびクエリ最適化
* ユーザーテストの実施とUX改善
---

## 👩‍💻 チーム内役割

本プロジェクトは3名で開発し、それぞれの役割は以下の通りです。


### 🔹 권혜빈（バックエンド / コア設計担当）

- マッチングアルゴリズム設計および実装  
- `FRIEND_STATUS` による関係モデル設計  
- Flaskベースの「友達検索・管理」機能の実装  
- Friend / Notification リレーションのバックエンド構築  
- 通知履歴機能・プロフィール公開設定などの詳細機能実装  
- 発表資料（中間・最終）作成  


### 🔹 곽가영（データ収集 / フロントエンド担当）

- 初期プロトタイプ設計  
- Webクローリングによる講義データ収集  
- 統合時間割データ（TOTALLECTURES）構築  
- 「時間割テキスト → テーブル変換」アルゴリズム設計  
- マッチングロジック補助  
- 全体CSSおよびUIデザイン担当  


### 🔹 전지원（フルスタック / インフラ担当）

- ペーパープロトタイプ設計  
- Flaskアプリ基本構造（ログイン、会員登録、画面構成）実装  
- 時間割検索・管理機能およびマイページ開発  
- データベース設計およびSQL作成  
- Webホスティング（PythonAnywhere）担当  
- React Native（WebView）によるモバイルアプリ化  
- ロゴおよび画面デザイン制作  
