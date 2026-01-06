# API構成図・フロー

## 概要
TaskFlowのAPIエンドポイントの構成と、典型的な利用フローを図示します。
※ フロントエンドは作成しないため、画面遷移図ではなくAPI呼び出しフローとして記述

---

## 1. APIエンドポイント構成図

```
/api/v1
│
├── /auth                      # 認証（認証不要）
│   ├── POST   /signup         # ユーザー登録
│   └── POST   /login          # ログイン
│
├── /users                     # ユーザー管理（認証必須）
│   └── /me
│       ├── GET    /           # プロフィール取得
│       ├── PUT    /           # プロフィール更新
│       └── PUT    /password   # パスワード変更
│
├── /projects                  # プロジェクト管理（認証必須）
│   ├── GET    /               # プロジェクト一覧
│   ├── POST   /               # プロジェクト作成
│   └── /:projectId
│       ├── GET    /           # プロジェクト詳細
│       ├── PUT    /           # プロジェクト更新
│       ├── DELETE /           # プロジェクト削除
│       ├── POST   /archive    # プロジェクトアーカイブ
│       │
│       ├── /tasks             # タスク管理
│       │   ├── GET  /         # タスク一覧
│       │   └── POST /         # タスク作成
│       │
│       ├── /labels            # ラベル管理
│       │   ├── GET  /         # ラベル一覧
│       │   └── POST /         # ラベル作成
│       │
│       └── /members           # メンバー管理
│           ├── GET    /       # メンバー一覧
│           ├── POST   /       # メンバー招待
│           └── /:userId
│               ├── PATCH /    # 権限変更
│               └── DELETE /   # メンバー削除
│
├── /tasks                     # タスク管理（認証必須）
│   ├── GET    /me             # 自分のタスク一覧
│   ├── GET    /search         # タスク検索
│   ├── PATCH  /bulk           # 一括更新
│   └── /:taskId
│       ├── GET    /           # タスク詳細
│       ├── PUT    /           # タスク更新
│       ├── DELETE /           # タスク削除
│       ├── PATCH  /status     # ステータス変更
│       │
│       ├── /comments          # コメント管理
│       │   ├── GET  /         # コメント一覧
│       │   └── POST /         # コメント追加
│       │
│       └── /labels            # タスクラベル管理
│           ├── POST   /       # ラベル付与
│           └── DELETE /:labelId  # ラベル削除
│
├── /comments                  # コメント管理（認証必須）
│   └── /:commentId
│       ├── PUT    /           # コメント更新
│       └── DELETE /           # コメント削除
│
├── /labels                    # ラベル管理（認証必須）
│   └── /:labelId
│       ├── PUT    /           # ラベル更新
│       └── DELETE /           # ラベル削除
│
└── /notifications             # 通知（認証必須、Phase 3）
    ├── GET    /               # 通知一覧
    ├── POST   /read-all       # 全既読
    └── /:notificationId
        └── PATCH /read        # 既読化
```

---

## 2. 典型的な利用フロー

### フロー1: 初回ユーザー登録からタスク作成まで

```
┌──────────┐
│  Client  │
└────┬─────┘
     │
     │ 1. POST /api/v1/auth/signup
     │    { email, password, name }
     ▼
┌──────────────┐
│  TaskFlow    │ ──► ユーザー作成
│     API      │ ──► JWTトークン発行
└──────┬───────┘
       │ Response: { token, user }
       │
       │ 2. POST /api/v1/projects
       │    Authorization: Bearer <token>
       │    { name, description }
       ▼
┌──────────────┐
│  TaskFlow    │ ──► プロジェクト作成
│     API      │ ──► 作成者をOwnerに設定
└──────┬───────┘
       │ Response: { project }
       │
       │ 3. POST /api/v1/projects/:projectId/tasks
       │    Authorization: Bearer <token>
       │    { title, description, priority }
       ▼
┌──────────────┐
│  TaskFlow    │ ──► タスク作成
│     API      │
└──────┬───────┘
       │ Response: { task }
       ▼
    完了
```

---

### フロー2: 既存ユーザーのログインからタスク確認まで

```
┌──────────┐
│  Client  │
└────┬─────┘
     │
     │ 1. POST /api/v1/auth/login
     │    { email, password }
     ▼
┌──────────────┐
│  TaskFlow    │ ──► 認証情報検証
│     API      │ ──► JWTトークン発行
└──────┬───────┘
       │ Response: { token }
       │
       │ 2. GET /api/v1/tasks/me
       │    Authorization: Bearer <token>
       ▼
┌──────────────┐
│  TaskFlow    │ ──► 自分のタスク一覧取得
│     API      │ ──► ステータス、優先度でフィルタ
└──────┬───────┘
       │ Response: { tasks: [...] }
       │
       │ 3. GET /api/v1/tasks/:taskId
       │    Authorization: Bearer <token>
       ▼
┌──────────────┐
│  TaskFlow    │ ──► タスク詳細取得
│     API      │ ──► コメント含む
└──────┬───────┘
       │ Response: { task, comments }
       ▼
    完了
```

---

### フロー3: タスクステータス更新フロー

```
┌──────────┐
│  Client  │
└────┬─────┘
     │
     │ 1. PATCH /api/v1/tasks/:taskId/status
     │    Authorization: Bearer <token>
     │    { status: "InProgress" }
     ▼
┌──────────────┐
│  TaskFlow    │ ──► 権限チェック（担当者または作成者）
│     API      │ ──► ステータス遷移ルール検証
└──────┬───────┘    （Todo→InProgress OK）
       │ Response: { task }
       │
       │ （作業実施）
       │
       │ 2. PATCH /api/v1/tasks/:taskId/status
       │    { status: "Done" }
       ▼
┌──────────────┐
│  TaskFlow    │ ──► ステータス更新
│     API      │ ──► completed_at 設定
└──────┬───────┘
       │ Response: { task }
       ▼
    完了
```

---

### フロー4: チームコラボレーションフロー

```
┌─────────────┐
│   Owner     │
└──────┬──────┘
       │
       │ 1. POST /api/v1/projects/:projectId/members
       │    { user_id, role: "Member" }
       ▼
┌──────────────┐
│  TaskFlow    │ ──► メンバー追加
│     API      │ ──► 権限設定
└──────┬───────┘
       │ Response: { member }
       │
       ▼
┌─────────────┐
│   Member    │ ◄── 通知（Phase 3で実装）
└──────┬──────┘
       │
       │ 2. GET /api/v1/projects/:projectId
       │    Authorization: Bearer <member_token>
       ▼
┌──────────────┐
│  TaskFlow    │ ──► プロジェクト詳細取得
│     API      │ ──► メンバー確認
└──────┬───────┘
       │ Response: { project, tasks }
       │
       │ 3. POST /api/v1/tasks/:taskId/comments
       │    { content: "質問があります" }
       ▼
┌──────────────┐
│  TaskFlow    │ ──► コメント追加
│     API      │ ──► メンション検出
└──────┬───────┘
       │ Response: { comment }
       │
       ▼
┌─────────────┐
│   Owner     │ ◄── 通知（Phase 3で実装）
└─────────────┘
```

---

### フロー5: ラベルでタスク整理フロー

```
┌──────────┐
│  Client  │
└────┬─────┘
     │
     │ 1. POST /api/v1/projects/:projectId/labels
     │    { name: "bug", color: "#FF0000" }
     ▼
┌──────────────┐
│  TaskFlow    │ ──► ラベル作成
│     API      │
└──────┬───────┘
       │ Response: { label }
       │
       │ 2. POST /api/v1/tasks/:taskId/labels
       │    { label_id: "..." }
       ▼
┌──────────────┐
│  TaskFlow    │ ──► タスクにラベル付与
│     API      │
└──────┬───────┘
       │ Response: { task_label }
       │
       │ 3. GET /api/v1/projects/:projectId/tasks?label=bug
       ▼
┌──────────────┐
│  TaskFlow    │ ──► ラベルでフィルタ
│     API      │
└──────┬───────┘
       │ Response: { tasks: [bugタスクのみ] }
       ▼
    完了
```

---

## 3. 認証・認可フロー

### JWT認証フロー

```
┌──────────┐
│  Client  │
└────┬─────┘
     │
     │ POST /api/v1/auth/login
     ▼
┌──────────────┐
│  TaskFlow    │
│     API      │
└──────┬───────┘
       │
       ├─► 1. ユーザー検索（email）
       ├─► 2. パスワード検証（bcrypt）
       └─► 3. JWTトークン生成
               ├─ claims: { user_id, role }
               ├─ exp: 24時間後
               └─ signature: HMAC-SHA256
       │
       │ Response: { token: "eyJhbGc..." }
       ▼
┌──────────┐
│  Client  │ ─► トークンを保存（ローカルストレージ等）
└────┬─────┘
     │
     │ 以降のリクエスト
     │ Authorization: Bearer eyJhbGc...
     ▼
┌──────────────┐
│  TaskFlow    │
│     API      │
│ (Middleware) │
└──────┬───────┘
       │
       ├─► 1. Authorizationヘッダーから抽出
       ├─► 2. JWT署名検証
       ├─► 3. 有効期限チェック
       └─► 4. claims取得（user_id, role）
       │
       │ contextにuser情報を追加
       ▼
   Handler実行
```

---

### 権限チェックフロー

```
リクエスト
    │
    ▼
認証ミドルウェア
    │
    ├─ トークン検証 ──► 失敗 ──► 401 Unauthorized
    │
    ▼ 成功
ハンドラ
    │
    ├─► リソース取得（プロジェクト/タスク）
    │
    ├─► 権限チェック
    │   ├─ プロジェクトメンバーか？
    │   ├─ ロールは適切か？（Owner/Member/Viewer）
    │   └─ 操作が許可されているか？
    │
    ├─ OK ──► 処理実行 ──► 200 OK
    │
    └─ NG ──► 403 Forbidden
```

---

## 4. エラーレスポンスの統一

すべてのエラーは以下の形式で返します：

```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable error message",
    "details": {
      "field": "validation error details (optional)"
    }
  }
}
```

### 主なエラーコード

| HTTPステータス | エラーコード | 説明 |
|---------------|-------------|------|
| 400 | `VALIDATION_ERROR` | 入力検証エラー |
| 401 | `UNAUTHORIZED` | 認証が必要 |
| 403 | `FORBIDDEN` | 権限不足 |
| 404 | `NOT_FOUND` | リソースが見つからない |
| 409 | `CONFLICT` | リソースの競合（重複など） |
| 500 | `INTERNAL_ERROR` | サーバー内部エラー |

---

## 5. ページネーション・フィルタ・ソート

### クエリパラメータ仕様

```
GET /api/v1/projects/:projectId/tasks
  ?status=Todo,InProgress    # フィルタ（カンマ区切りで複数指定）
  &assignee=user-id          # 担当者フィルタ
  &priority=High,Urgent      # 優先度フィルタ
  &sort=priority,-due_date   # ソート（-は降順）
  &limit=20                  # 1ページあたりの件数
  &offset=0                  # オフセット
```

### レスポンス形式

```json
{
  "data": [...],
  "pagination": {
    "total": 150,
    "limit": 20,
    "offset": 0,
    "has_next": true
  }
}
```

---

## 6. APIバージョニング戦略

### 現在: v1
- パス: `/api/v1/...`
- 互換性: 維持する

### 将来: v2（破壊的変更がある場合）
- パス: `/api/v2/...`
- v1は少なくとも6ヶ月間維持

### 非推奨化のプロセス
1. v2リリース
2. v1に`Deprecation`ヘッダー追加
3. 6ヶ月後にv1廃止予告
4. 12ヶ月後にv1削除

---

## 7. レート制限（Phase 3で実装）

### 制限値
- 認証済みユーザー: 1000 req/hour
- 未認証: 100 req/hour

### レスポンスヘッダー
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 950
X-RateLimit-Reset: 1609459200
```

### 超過時
- HTTPステータス: 429 Too Many Requests
- Retry-Afterヘッダーで復帰時刻を通知

---

**作成日**: 2026-01-06
**作成者**: 学習プロジェクト
**バージョン**: 1.0
