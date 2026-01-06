# データベーススキーマ設計

## 概要
TaskFlowシステムのデータベーススキーマをPostgreSQL実装に基づいて詳細に定義します。
正規化、制約、インデックス設計を含めた実装レベルの設計を記載します。

---

## ER図（Entity Relationship Diagram）

### 全体構成図

```
┌─────────────────────────────────────────────────────────────────┐
│                          TaskFlow Database                       │
└─────────────────────────────────────────────────────────────────┘

┌────────────────┐
│     users      │
├────────────────┤
│ id (PK, UUID)  │───┐
│ name           │   │
│ email (UNIQUE) │   │
│ password_hash  │   │
│ role           │   │
│ created_at     │   │
│ updated_at     │   │
│ last_login_at  │   │
└────────────────┘   │
        │            │
        │ 1          │
        │            │
        │ creates    │
        │            │
        ▼ N          │
┌────────────────┐   │
│   projects     │   │
├────────────────┤   │
│ id (PK, UUID)  │───┼───┐
│ name           │   │   │
│ description    │   │   │
│ status         │   │   │
│ start_date     │   │   │
│ end_date       │   │   │
│ owner_id (FK)  │───┘   │
│ created_at     │       │
│ updated_at     │       │
└────────────────┘       │
        │                │
        │ 1              │
        │                │
        │ has            │
        │                │
        ▼ N              │
┌────────────────┐       │
│     tasks      │       │
├────────────────┤       │
│ id (PK, UUID)  │       │
│ title          │       │
│ description    │       │
│ status         │       │
│ priority       │       │
│ project_id(FK) │───────┘
│ assignee_id(FK)│───────┐
│ creator_id(FK) │───┐   │
│ due_date       │   │   │
│ estimated_hours│   │   │
│ actual_hours   │   │   │
│ created_at     │   │   │
│ updated_at     │   │   │
│ completed_at   │   │   │
└────────────────┘   │   │
        │            │   │
        │ 1          │   │
        │            │   │
        │ has        │   │
        │            │   │
        ▼ N          │   │
┌────────────────┐   │   │
│   comments     │   │   │
├────────────────┤   │   │
│ id (PK, UUID)  │   │   │
│ task_id (FK)   │───┘   │
│ author_id (FK) │───────┤
│ content        │       │
│ created_at     │       │
│ updated_at     │       │
└────────────────┘       │
                         │
┌────────────────┐       │
│project_members │       │
├────────────────┤       │
│ id (PK, UUID)  │       │
│ project_id(FK) │───────┤
│ user_id (FK)   │───────┘
│ role           │
│ joined_at      │
│ UNIQUE(proj,usr)│
└────────────────┘


┌────────────────┐       ┌────────────────┐       ┌────────────────┐
│    labels      │       │  task_labels   │       │     tasks      │
├────────────────┤       ├────────────────┤       ├────────────────┤
│ id (PK, UUID)  │◄──┐   │ id (PK, UUID)  │   ┌──►│ id (PK, UUID)  │
│ name           │   └───│ label_id (FK)  │   │   └────────────────┘
│ color          │       │ task_id (FK)   │───┘
│ project_id(FK) │───┐   │ assigned_at    │
│ created_at     │   │   │ UNIQUE(task,lbl)│
└────────────────┘   │   └────────────────┘
                     │
                     │
                     │
            ┌────────▼───────┐
            │   projects     │
            └────────────────┘
```

---

## テーブル定義詳細

### 1. users テーブル

**説明**: システムを利用するユーザー情報

| カラム名 | データ型 | NULL | デフォルト | 制約 | 説明 |
|---------|---------|------|-----------|------|------|
| id | UUID | NOT NULL | gen_random_uuid() | PRIMARY KEY | ユーザーID |
| name | VARCHAR(100) | NOT NULL | - | - | ユーザー名 |
| email | VARCHAR(255) | NOT NULL | - | UNIQUE | メールアドレス（ログインID） |
| password_hash | VARCHAR(255) | NOT NULL | - | - | bcryptハッシュ化されたパスワード |
| role | VARCHAR(20) | NOT NULL | 'Member' | CHECK(role IN ('Admin', 'Member', 'Viewer')) | システムロール |
| created_at | TIMESTAMP | NOT NULL | CURRENT_TIMESTAMP | - | 作成日時 |
| updated_at | TIMESTAMP | NOT NULL | CURRENT_TIMESTAMP | - | 更新日時 |
| last_login_at | TIMESTAMP | NULL | - | - | 最終ログイン日時 |

**インデックス**:
- PRIMARY KEY: `id`
- UNIQUE INDEX: `email`
- INDEX: `created_at` (ユーザー一覧のソート用)

**サンプルデータ**:
```sql
INSERT INTO users (id, name, email, password_hash, role) VALUES
('550e8400-e29b-41d4-a716-446655440000', 'John Doe', 'john@example.com', '$2a$10$...', 'Admin');
```

---

### 2. projects テーブル

**説明**: タスクをグループ化するプロジェクト

| カラム名 | データ型 | NULL | デフォルト | 制約 | 説明 |
|---------|---------|------|-----------|------|------|
| id | UUID | NOT NULL | gen_random_uuid() | PRIMARY KEY | プロジェクトID |
| name | VARCHAR(200) | NOT NULL | - | - | プロジェクト名 |
| description | TEXT | NULL | - | - | プロジェクト説明 |
| status | VARCHAR(20) | NOT NULL | 'Active' | CHECK(status IN ('Active', 'Archived', 'Completed')) | プロジェクトステータス |
| start_date | DATE | NULL | - | - | 開始日 |
| end_date | DATE | NULL | - | CHECK(end_date IS NULL OR end_date >= start_date) | 終了予定日 |
| owner_id | UUID | NOT NULL | - | FOREIGN KEY → users(id) ON DELETE SET NULL | プロジェクトオーナー |
| created_at | TIMESTAMP | NOT NULL | CURRENT_TIMESTAMP | - | 作成日時 |
| updated_at | TIMESTAMP | NOT NULL | CURRENT_TIMESTAMP | - | 更新日時 |

**インデックス**:
- PRIMARY KEY: `id`
- INDEX: `owner_id` (外部キー、検索用)
- INDEX: `status, created_at` (複合インデックス、一覧取得用)

**外部キー制約**:
- `owner_id` → `users(id)` ON DELETE SET NULL（ユーザー削除時はNULLに）

---

### 3. tasks テーブル

**説明**: 実行すべき作業単位

| カラム名 | データ型 | NULL | デフォルト | 制約 | 説明 |
|---------|---------|------|-----------|------|------|
| id | UUID | NOT NULL | gen_random_uuid() | PRIMARY KEY | タスクID |
| title | VARCHAR(200) | NOT NULL | - | - | タスクタイトル |
| description | TEXT | NULL | - | - | タスク詳細説明 |
| status | VARCHAR(20) | NOT NULL | 'Todo' | CHECK(status IN ('Todo', 'InProgress', 'InReview', 'Done', 'Blocked')) | タスクステータス |
| priority | VARCHAR(20) | NOT NULL | 'Medium' | CHECK(priority IN ('Low', 'Medium', 'High', 'Urgent')) | 優先度 |
| project_id | UUID | NOT NULL | - | FOREIGN KEY → projects(id) ON DELETE CASCADE | 所属プロジェクト |
| assignee_id | UUID | NULL | - | FOREIGN KEY → users(id) ON DELETE SET NULL | 担当者 |
| creator_id | UUID | NOT NULL | - | FOREIGN KEY → users(id) ON DELETE SET NULL | 作成者 |
| due_date | DATE | NULL | - | - | 期限日 |
| estimated_hours | DECIMAL(5,2) | NULL | - | CHECK(estimated_hours >= 0) | 見積もり時間 |
| actual_hours | DECIMAL(5,2) | NULL | - | CHECK(actual_hours >= 0) | 実績時間 |
| created_at | TIMESTAMP | NOT NULL | CURRENT_TIMESTAMP | - | 作成日時 |
| updated_at | TIMESTAMP | NOT NULL | CURRENT_TIMESTAMP | - | 更新日時 |
| completed_at | TIMESTAMP | NULL | - | - | 完了日時 |

**インデックス**:
- PRIMARY KEY: `id`
- INDEX: `project_id, status` (複合インデックス、プロジェクト内タスク一覧用)
- INDEX: `assignee_id, status` (複合インデックス、担当者のタスク一覧用)
- INDEX: `creator_id` (作成者検索用)
- INDEX: `due_date` (期限日ソート用)
- INDEX: `priority, created_at` (優先度ソート用)

**外部キー制約**:
- `project_id` → `projects(id)` ON DELETE CASCADE（プロジェクト削除時にタスクも削除）
- `assignee_id` → `users(id)` ON DELETE SET NULL（ユーザー削除時はNULLに）
- `creator_id` → `users(id)` ON DELETE SET NULL（ユーザー削除時はNULLに）

**トリガー（Phase 2で実装予定）**:
- ステータスが`Done`になった時に`completed_at`を自動設定

---

### 4. comments テーブル

**説明**: タスクに対するコメント

| カラム名 | データ型 | NULL | デフォルト | 制約 | 説明 |
|---------|---------|------|-----------|------|------|
| id | UUID | NOT NULL | gen_random_uuid() | PRIMARY KEY | コメントID |
| task_id | UUID | NOT NULL | - | FOREIGN KEY → tasks(id) ON DELETE CASCADE | タスクID |
| author_id | UUID | NOT NULL | - | FOREIGN KEY → users(id) ON DELETE SET NULL | 投稿者ID |
| content | TEXT | NOT NULL | - | CHECK(length(content) > 0 AND length(content) <= 10000) | コメント内容 |
| created_at | TIMESTAMP | NOT NULL | CURRENT_TIMESTAMP | - | 作成日時 |
| updated_at | TIMESTAMP | NOT NULL | CURRENT_TIMESTAMP | - | 更新日時 |

**インデックス**:
- PRIMARY KEY: `id`
- INDEX: `task_id, created_at DESC` (複合インデックス、タスクのコメント一覧用)
- INDEX: `author_id` (投稿者検索用)

**外部キー制約**:
- `task_id` → `tasks(id)` ON DELETE CASCADE（タスク削除時にコメントも削除）
- `author_id` → `users(id)` ON DELETE SET NULL（ユーザー削除時はNULLに）

---

### 5. labels テーブル

**説明**: タスクを分類するためのラベル

| カラム名 | データ型 | NULL | デフォルト | 制約 | 説明 |
|---------|---------|------|-----------|------|------|
| id | UUID | NOT NULL | gen_random_uuid() | PRIMARY KEY | ラベルID |
| name | VARCHAR(50) | NOT NULL | - | - | ラベル名 |
| color | VARCHAR(7) | NOT NULL | '#808080' | CHECK(color ~ '^#[0-9A-Fa-f]{6}$') | カラーコード（HEX形式） |
| project_id | UUID | NOT NULL | - | FOREIGN KEY → projects(id) ON DELETE CASCADE | プロジェクトID |
| created_at | TIMESTAMP | NOT NULL | CURRENT_TIMESTAMP | - | 作成日時 |

**インデックス**:
- PRIMARY KEY: `id`
- UNIQUE INDEX: `project_id, name` (プロジェクト内でラベル名は一意)
- INDEX: `project_id` (プロジェクトのラベル一覧用)

**外部キー制約**:
- `project_id` → `projects(id)` ON DELETE CASCADE（プロジェクト削除時にラベルも削除）

---

### 6. project_members テーブル

**説明**: プロジェクトとユーザーの多対多関係

| カラム名 | データ型 | NULL | デフォルト | 制約 | 説明 |
|---------|---------|------|-----------|------|------|
| id | UUID | NOT NULL | gen_random_uuid() | PRIMARY KEY | レコードID |
| project_id | UUID | NOT NULL | - | FOREIGN KEY → projects(id) ON DELETE CASCADE | プロジェクトID |
| user_id | UUID | NOT NULL | - | FOREIGN KEY → users(id) ON DELETE CASCADE | ユーザーID |
| role | VARCHAR(20) | NOT NULL | 'Member' | CHECK(role IN ('Owner', 'Member', 'Viewer')) | プロジェクト内ロール |
| joined_at | TIMESTAMP | NOT NULL | CURRENT_TIMESTAMP | - | 参加日時 |

**インデックス**:
- PRIMARY KEY: `id`
- UNIQUE INDEX: `project_id, user_id` (同じプロジェクトに同じユーザーは1度だけ参加)
- INDEX: `user_id` (ユーザーの参加プロジェクト一覧用)
- INDEX: `project_id, role` (プロジェクトメンバー一覧用)

**外部キー制約**:
- `project_id` → `projects(id)` ON DELETE CASCADE（プロジェクト削除時にメンバーも削除）
- `user_id` → `users(id)` ON DELETE CASCADE（ユーザー削除時にメンバーシップも削除）

---

### 7. task_labels テーブル

**説明**: タスクとラベルの多対多関係

| カラム名 | データ型 | NULL | デフォルト | 制約 | 説明 |
|---------|---------|------|-----------|------|------|
| id | UUID | NOT NULL | gen_random_uuid() | PRIMARY KEY | レコードID |
| task_id | UUID | NOT NULL | - | FOREIGN KEY → tasks(id) ON DELETE CASCADE | タスクID |
| label_id | UUID | NOT NULL | - | FOREIGN KEY → labels(id) ON DELETE CASCADE | ラベルID |
| assigned_at | TIMESTAMP | NOT NULL | CURRENT_TIMESTAMP | - | 付与日時 |

**インデックス**:
- PRIMARY KEY: `id`
- UNIQUE INDEX: `task_id, label_id` (同じタスクに同じラベルは1度だけ付与)
- INDEX: `label_id` (ラベルでタスク検索用)
- INDEX: `task_id` (タスクのラベル一覧用)

**外部キー制約**:
- `task_id` → `tasks(id)` ON DELETE CASCADE（タスク削除時にラベル関連も削除）
- `label_id` → `labels(id)` ON DELETE CASCADE（ラベル削除時にタスク関連も削除）

---

## 正規化の検討

### 第1正規形（1NF）
✅ **達成**: すべてのカラムが単一値（配列や繰り返しグループなし）

### 第2正規形（2NF）
✅ **達成**: すべての非キーカラムが主キーに完全関数従属
- 例: `tasks`テーブルでは、`title`, `description`などはすべて`id`に従属

### 第3正規形（3NF）
✅ **達成**: すべての非キーカラムが主キーに推移的でない関数従属
- 例: `tasks.creator_id`は`tasks.id`に従属し、`users.name`は`tasks.id`に推移的に従属しない（`users`テーブルで管理）

### 正規化の例外（意図的な非正規化）
**現時点ではなし**。Phase 2以降でパフォーマンス要件に応じて検討：
- タスク数のカウントをプロジェクトに持たせる（`projects.task_count`）
- 最終更新日時をプロジェクトに持たせる（`projects.last_activity_at`）

---

## インデックス設計の戦略

### 1. 主要なクエリパターン
| クエリパターン | 対象テーブル | インデックス |
|---------------|-------------|-------------|
| プロジェクトのタスク一覧（ステータスフィルタ） | tasks | `(project_id, status)` |
| 担当者のタスク一覧（ステータスフィルタ） | tasks | `(assignee_id, status)` |
| タスクのコメント一覧（新しい順） | comments | `(task_id, created_at DESC)` |
| プロジェクトのラベル一覧 | labels | `project_id` |
| プロジェクトメンバー一覧（ロールフィルタ） | project_members | `(project_id, role)` |
| ユーザーの参加プロジェクト一覧 | project_members | `user_id` |

### 2. インデックスの種類
- **B-Tree Index**: デフォルト、範囲検索、ソートに使用
- **UNIQUE Index**: 一意性制約（email, project_id+nameなど）
- **Partial Index（Phase 2）**: 条件付きインデックス（例: `status = 'Active'`のみ）
- **GIN Index（Phase 2）**: 全文検索用（`tasks.title`, `tasks.description`）

### 3. インデックス作成の優先順位
**Phase 1（MVP）**:
- ✅ PRIMARY KEY（自動作成）
- ✅ FOREIGN KEY（検索と結合用）
- ✅ UNIQUE制約（email, project_id+nameなど）
- ✅ 複合インデックス（頻繁に使用されるクエリ用）

**Phase 2以降**:
- GIN Index（全文検索）
- Partial Index（パフォーマンスチューニング）

---

## マイグレーション戦略

### 1. マイグレーションツール
**採用ツール**: `golang-migrate/migrate`

**理由**:
- Go標準の`database/sql`と統合可能
- up/downマイグレーションのサポート
- バージョン管理が容易
- CLIとライブラリの両方で使用可能

### 2. マイグレーションファイルの命名規則
```
db/migrations/
  000001_create_users_table.up.sql
  000001_create_users_table.down.sql
  000002_create_projects_table.up.sql
  000002_create_projects_table.down.sql
  000003_create_tasks_table.up.sql
  000003_create_tasks_table.down.sql
  ...
```

**規則**:
- 連番6桁 + アンダースコア + 説明的な名前
- `.up.sql`: 適用時のSQL
- `.down.sql`: ロールバック時のSQL

### 3. マイグレーション実行順序（Phase 1）
1. `000001_create_users_table.sql` - ユーザーテーブル（依存なし）
2. `000002_create_projects_table.sql` - プロジェクトテーブル（users依存）
3. `000003_create_tasks_table.sql` - タスクテーブル（projects, users依存）
4. `000004_create_comments_table.sql` - コメントテーブル（tasks, users依存）
5. `000005_create_labels_table.sql` - ラベルテーブル（projects依存）
6. `000006_create_project_members_table.sql` - プロジェクトメンバーテーブル（projects, users依存）
7. `000007_create_task_labels_table.sql` - タスクラベルテーブル（tasks, labels依存）
8. `000008_create_indexes.sql` - インデックス作成

### 4. マイグレーション実行コマンド
```bash
# データベース作成
createdb taskflow_dev

# マイグレーション実行
migrate -path db/migrations -database "postgresql://user:password@localhost:5432/taskflow_dev?sslmode=disable" up

# ロールバック（1つ戻す）
migrate -path db/migrations -database "postgresql://..." down 1

# バージョン確認
migrate -path db/migrations -database "postgresql://..." version
```

### 5. 環境別データベース
| 環境 | データベース名 | 説明 |
|------|---------------|------|
| 開発 | `taskflow_dev` | ローカル開発環境 |
| テスト | `taskflow_test` | テスト実行用（自動作成・削除） |
| 本番 | `taskflow_prod` | 本番環境（Phase 3） |

### 6. マイグレーションのベストプラクティス
- ✅ すべてのマイグレーションにはdownを用意（ロールバック可能に）
- ✅ 1つのマイグレーションファイルは1つの責務（テーブル作成、カラム追加など）
- ✅ マイグレーション適用前にバックアップ
- ✅ トランザクション内で実行（BEGIN/COMMIT）
- ✅ 本番環境では必ず事前にステージング環境でテスト

---

## データ整合性とカスケード

### 削除時の振る舞い

| 親テーブル | 子テーブル | 外部キー | ON DELETE動作 | 理由 |
|-----------|-----------|---------|--------------|------|
| users | projects.owner_id | owner_id | SET NULL | プロジェクトは残す |
| users | tasks.assignee_id | assignee_id | SET NULL | タスクは未割り当てに |
| users | tasks.creator_id | creator_id | SET NULL | タスクは残す |
| users | comments.author_id | author_id | SET NULL | コメントは残す |
| users | project_members.user_id | user_id | CASCADE | メンバーシップは削除 |
| projects | tasks.project_id | project_id | CASCADE | タスクも削除 |
| projects | labels.project_id | project_id | CASCADE | ラベルも削除 |
| projects | project_members.project_id | project_id | CASCADE | メンバーも削除 |
| tasks | comments.task_id | task_id | CASCADE | コメントも削除 |
| tasks | task_labels.task_id | task_id | CASCADE | ラベル関連も削除 |
| labels | task_labels.label_id | label_id | CASCADE | タスク関連も削除 |

### 制約による整合性保証

**CHECK制約**:
- `users.role`: Admin/Member/Viewerのみ
- `projects.status`: Active/Archived/Completedのみ
- `projects.end_date`: start_date以降であること
- `tasks.status`: Todo/InProgress/InReview/Done/Blockedのみ
- `tasks.priority`: Low/Medium/High/Urgentのみ
- `tasks.estimated_hours`, `actual_hours`: 0以上
- `labels.color`: HEX形式（#RRGGBB）
- `project_members.role`: Owner/Member/Viewerのみ
- `comments.content`: 1文字以上10000文字以下

**UNIQUE制約**:
- `users.email`: メールアドレスは一意
- `labels (project_id, name)`: プロジェクト内でラベル名は一意
- `project_members (project_id, user_id)`: 重複参加不可
- `task_labels (task_id, label_id)`: 重複ラベル付与不可

---

## サンプルCREATE TABLE文

### users テーブル
```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    role VARCHAR(20) NOT NULL DEFAULT 'Member' CHECK (role IN ('Admin', 'Member', 'Viewer')),
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    last_login_at TIMESTAMP
);

CREATE INDEX idx_users_created_at ON users(created_at);
```

### projects テーブル
```sql
CREATE TABLE projects (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(200) NOT NULL,
    description TEXT,
    status VARCHAR(20) NOT NULL DEFAULT 'Active' CHECK (status IN ('Active', 'Archived', 'Completed')),
    start_date DATE,
    end_date DATE CHECK (end_date IS NULL OR end_date >= start_date),
    owner_id UUID REFERENCES users(id) ON DELETE SET NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_projects_owner_id ON projects(owner_id);
CREATE INDEX idx_projects_status_created_at ON projects(status, created_at);
```

### tasks テーブル
```sql
CREATE TABLE tasks (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    title VARCHAR(200) NOT NULL,
    description TEXT,
    status VARCHAR(20) NOT NULL DEFAULT 'Todo' CHECK (status IN ('Todo', 'InProgress', 'InReview', 'Done', 'Blocked')),
    priority VARCHAR(20) NOT NULL DEFAULT 'Medium' CHECK (priority IN ('Low', 'Medium', 'High', 'Urgent')),
    project_id UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
    assignee_id UUID REFERENCES users(id) ON DELETE SET NULL,
    creator_id UUID REFERENCES users(id) ON DELETE SET NULL,
    due_date DATE,
    estimated_hours DECIMAL(5,2) CHECK (estimated_hours >= 0),
    actual_hours DECIMAL(5,2) CHECK (actual_hours >= 0),
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    completed_at TIMESTAMP
);

CREATE INDEX idx_tasks_project_status ON tasks(project_id, status);
CREATE INDEX idx_tasks_assignee_status ON tasks(assignee_id, status);
CREATE INDEX idx_tasks_creator ON tasks(creator_id);
CREATE INDEX idx_tasks_due_date ON tasks(due_date);
CREATE INDEX idx_tasks_priority_created ON tasks(priority, created_at);
```

*（その他のテーブルのCREATE文は省略、マイグレーションファイルに記載）*

---

## パフォーマンス最適化の考慮点

### 1. 想定データ量とパフォーマンス
| テーブル | Phase 1 | Phase 2 | Phase 3 |
|---------|---------|---------|---------|
| users | ~50 | ~200 | ~1,000 |
| projects | ~100 | ~500 | ~2,000 |
| tasks | ~10,000 | ~50,000 | ~200,000 |
| comments | ~5,000 | ~20,000 | ~100,000 |
| labels | ~200 | ~1,000 | ~5,000 |
| project_members | ~200 | ~1,000 | ~5,000 |
| task_labels | ~15,000 | ~75,000 | ~300,000 |

### 2. クエリパフォーマンス目標
- **単一レコード取得**: 10ms以内
- **一覧取得（100件）**: 100ms以内
- **複雑なJOINクエリ**: 200ms以内

### 3. インデックスのトレードオフ
**メリット**:
- SELECT性能向上（特に検索とソート）
- UNIQUE制約による整合性保証

**デメリット**:
- INSERT/UPDATE/DELETE性能低下（インデックス更新コスト）
- ディスク容量増加

**方針**: Phase 1では必要最小限、Phase 2以降でチューニング

---

## 将来の拡張（Phase 2以降）

### 1. 追加テーブル候補
- `notifications`: タスク割り当て通知
- `time_logs`: 作業時間記録
- `attachments`: ファイル添付
- `milestones`: マイルストーン管理
- `audit_logs`: 操作履歴

### 2. 全文検索の実装
```sql
-- Phase 2で追加
ALTER TABLE tasks ADD COLUMN search_vector tsvector;

CREATE INDEX idx_tasks_search ON tasks USING GIN(search_vector);

-- トリガーで自動更新
CREATE TRIGGER tasks_search_update
BEFORE INSERT OR UPDATE ON tasks
FOR EACH ROW EXECUTE FUNCTION
tsvector_update_trigger(search_vector, 'pg_catalog.english', title, description);
```

### 3. パーティショニング（Phase 3）
大量データ対応としてtasksテーブルを日付でパーティション化

---

**作成日**: 2026-01-06
**作成者**: 学習プロジェクト
**バージョン**: 1.0
**対象DBMS**: PostgreSQL 14+
