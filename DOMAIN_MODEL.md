# ドメインモデル図

## 概要
TaskFlowシステムの主要エンティティと、それらの関係性を定義します。

---

## エンティティ一覧

### 1. User（ユーザー）
**説明**: システムを利用するユーザー（チームメンバー）

**主要属性**:
- ID（UUID）
- 名前
- メールアドレス
- パスワードハッシュ
- ロール（Admin / Member / Viewer）
- 作成日時
- 最終ログイン日時

**責務**:
- システムへのログイン
- タスクの作成・更新
- コメントの投稿
- 自分が参加しているプロジェクトの閲覧

---

### 2. Project（プロジェクト）
**説明**: タスクをグループ化する単位（例: "Web API v2", "モバイルアプリ開発"）

**主要属性**:
- ID（UUID）
- 名前
- 説明
- ステータス（Active / Archived / Completed）
- 開始日
- 終了予定日
- 作成者ID（User）
- 作成日時
- 更新日時

**責務**:
- タスクのグループ化
- プロジェクト単位での進捗管理
- メンバーのアサイン

---

### 3. Task（タスク）
**説明**: 実行すべき作業単位

**主要属性**:
- ID（UUID）
- タイトル
- 説明
- ステータス（Todo / InProgress / InReview / Done / Blocked）
- 優先度（Low / Medium / High / Urgent）
- プロジェクトID（Project）
- 担当者ID（User）- nullable
- 作成者ID（User）
- 期限日 - nullable
- 見積もり時間（時間） - nullable
- 実績時間（時間） - nullable
- 作成日時
- 更新日時
- 完了日時 - nullable

**責務**:
- タスクの進捗管理
- 担当者への割り当て
- ステータスの遷移管理

---

### 4. Comment（コメント）
**説明**: タスクに対するコメント・議論

**主要属性**:
- ID（UUID）
- タスクID（Task）
- 投稿者ID（User）
- 内容（テキスト）
- 作成日時
- 更新日時

**責務**:
- タスクに関する議論の記録
- メンション機能（@username）
- 履歴の保持

---

### 5. Label（ラベル）
**説明**: タスクを分類するためのタグ（例: "bug", "feature", "frontend"）

**主要属性**:
- ID（UUID）
- 名前
- 色（HEX形式）
- プロジェクトID（Project）
- 作成日時

**責務**:
- タスクの分類
- フィルタリング・検索の補助

---

### 6. ProjectMember（プロジェクトメンバー）
**説明**: プロジェクトとユーザーの多対多の関連

**主要属性**:
- ID（UUID）
- プロジェクトID（Project）
- ユーザーID（User）
- ロール（Owner / Member / Viewer）
- 参加日時

**責務**:
- プロジェクトへのアクセス権限管理
- メンバー一覧の管理

---

### 7. TaskLabel（タスクラベル）
**説明**: タスクとラベルの多対多の関連

**主要属性**:
- ID（UUID）
- タスクID（Task）
- ラベルID（Label）
- 付与日時

**責務**:
- タスクへのラベル付け
- ラベルによる検索・フィルタ

---

## エンティティ関係図（ER図）

```
┌─────────────────┐
│     User        │
├─────────────────┤
│ id (PK)         │
│ name            │
│ email           │
│ password_hash   │
│ role            │
│ created_at      │
│ last_login_at   │
└─────────────────┘
         │
         │ 1
         │
         │ creates
         │
         ▼ N
┌─────────────────┐         ┌─────────────────┐
│    Project      │ 1     N │ ProjectMember   │
├─────────────────┤◄────────┤─────────────────┤
│ id (PK)         │         │ id (PK)         │
│ name            │         │ project_id (FK) │
│ description     │         │ user_id (FK)    │
│ status          │         │ role            │
│ start_date      │         │ joined_at       │
│ end_date        │         └─────────────────┘
│ owner_id (FK)   │                 │
│ created_at      │                 │ N
│ updated_at      │                 │
└─────────────────┘                 │
         │                          │
         │ 1                        │
         │                          │
         │ contains                 │
         │                          │
         ▼ N                        │
┌─────────────────┐                 │
│      Task       │                 │
├─────────────────┤                 │
│ id (PK)         │                 │
│ title           │                 │
│ description     │                 │
│ status          │                 │
│ priority        │                 │
│ project_id (FK) │                 │
│ assignee_id(FK) │◄────────────────┘
│ creator_id (FK) │         assigned to
│ due_date        │
│ estimated_hours │
│ actual_hours    │
│ created_at      │
│ updated_at      │
│ completed_at    │
└─────────────────┘
         │
         │ 1
         │
         │ has
         │
         ▼ N
┌─────────────────┐
│    Comment      │
├─────────────────┤
│ id (PK)         │
│ task_id (FK)    │
│ author_id (FK)  │
│ content         │
│ created_at      │
│ updated_at      │
└─────────────────┘


┌─────────────────┐         ┌─────────────────┐
│     Label       │ 1     N │   TaskLabel     │
├─────────────────┤◄────────┤─────────────────┤
│ id (PK)         │         │ id (PK)         │
│ name            │         │ task_id (FK)    │
│ color           │         │ label_id (FK)   │
│ project_id (FK) │         │ assigned_at     │
│ created_at      │         └─────────────────┘
└─────────────────┘                 │
                                    │ N
                                    │
                                    │
                                    ▼ 1
                            ┌─────────────────┐
                            │      Task       │
                            └─────────────────┘
```

---

## 関係性の詳細

### User ⇔ Project
- **関係**: 多対多（ProjectMemberを介して）
- **説明**: ユーザーは複数のプロジェクトに参加でき、プロジェクトは複数のメンバーを持つ
- **制約**:
  - プロジェクトには最低1人のOwnerが必要
  - Ownerは削除できない（別のメンバーをOwnerに昇格させてから削除）

### Project ⇔ Task
- **関係**: 1対多
- **説明**: 1つのプロジェクトは複数のタスクを持つ、タスクは必ず1つのプロジェクトに属する
- **制約**:
  - タスクは必ずプロジェクトに紐づく（プロジェクトなしのタスクは不可）
  - プロジェクトが削除されると、配下のタスクも削除される（CASCADE）

### Task → User（担当者）
- **関係**: 多対1
- **説明**: 複数のタスクが1人のユーザーに割り当てられる。1つのタスクは0または1人のユーザーに担当される
- **制約**:
  - タスクは未割り当て（assignee_id = NULL）も可能
  - 担当者はプロジェクトメンバーである必要がある

### Task ⇔ Comment
- **関係**: 1対多
- **説明**: 1つのタスクは複数のコメントを持つ
- **制約**:
  - タスクが削除されると、コメントも削除される（CASCADE）
  - コメントの編集履歴は保持しない（シンプル化のため）

### Task ⇔ Label
- **関係**: 多対多（TaskLabelを介して）
- **説明**: タスクは複数のラベルを持ち、ラベルは複数のタスクに付けられる
- **制約**:
  - ラベルはプロジェクトに紐づく
  - タスクに付けられるラベルは同じプロジェクト内のもののみ

---

## ドメインルール（ビジネスロジック）

### タスクステータスの遷移ルール
```
Todo ─────────────► InProgress ─────────────► InReview ─────────────► Done
  │                     │                          │                      │
  │                     │                          │                      │
  └─────────────────────┴──────────────────────────┴─────► Blocked       │
                                                                │          │
                                                                └──────────┘
                                                           (再開時はTodoへ)
```

**制約**:
- Doneに遷移すると completed_at が設定される
- Blockedから再開する場合は、Todoに戻す
- InReviewからDoneに遷移できるのはOwner/Adminのみ（将来的に実装）

### タスク優先度の自動調整
- 期限が近い（3日以内）タスクは自動的に High 以上に昇格
- Urgentタスクが10個以上ある場合、警告を表示

### プロジェクトメンバーの権限
| ロール | プロジェクト編集 | タスク作成 | タスク編集 | タスク削除 | メンバー招待 |
|--------|------------------|------------|------------|------------|--------------|
| **Owner** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Member** | ❌ | ✅ | ✅（自分のタスクのみ） | ❌ | ❌ |
| **Viewer** | ❌ | ❌ | ❌ | ❌ | ❌ |

---

## データ整合性の考慮

### 削除時の振る舞い
- **Userを削除**:
  - 作成したプロジェクト/タスクは残す（creator_id を NULL に）
  - 担当しているタスクは未割り当てに変更
- **Projectを削除**:
  - 配下のタスク、ラベル、コメントをすべて削除（CASCADE）
- **Taskを削除**:
  - 配下のコメント、ラベル関連を削除（CASCADE）

### 一意性制約
- User.email: ユニーク
- Label.name + project_id: ユニーク（同一プロジェクト内で同名ラベル不可）
- ProjectMember.project_id + user_id: ユニーク（重複参加不可）

---

## 拡張の余地（Phase 2以降）

### 追加予定エンティティ
1. **Notification（通知）**: タスク割り当て、コメントメンションの通知
2. **TimeLog（作業時間記録）**: タスクごとの作業時間トラッキング
3. **Attachment（添付ファイル）**: タスクへのファイル添付
4. **Milestone（マイルストーン）**: プロジェクトの中間目標

### 追加予定の関係
- User ⇔ Notification: 1対多
- Task ⇔ TimeLog: 1対多
- Task ⇔ Attachment: 1対多
- Project ⇔ Milestone: 1対多
- Task ⇔ Milestone: 多対1

---

## 参考資料

### ドメインモデリングの参考
- [Domain-Driven Design](https://domainlanguage.com/ddd/)
- [Event Storming](https://www.eventstorming.com/)

### 類似システムのER図
- [Linear Schema](https://linear.app/docs/graphql-api)
- [GitHub Projects Schema](https://docs.github.com/en/graphql/reference/objects)

---

**作成日**: 2026-01-06
**作成者**: 学習プロジェクト
**バージョン**: 1.0
