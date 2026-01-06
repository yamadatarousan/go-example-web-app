# 技術選定の判断基準

## 概要
TaskFlowシステムで採用する技術スタックの選定理由と判断基準を記録します。
学習目的と実務での実用性のバランスを考慮した選定を行います。

---

## 技術選定の基本方針

### 1. 選定の軸
| 軸 | 重み | 説明 |
|----|------|------|
| **学習価値** | 30% | Go開発の実務スキル習得に役立つか |
| **実用性** | 30% | 実際のプロジェクトで使われているか |
| **シンプルさ** | 20% | 学習コストが適切か、過度に複雑でないか |
| **コミュニティ** | 10% | ドキュメント、サンプルコードが豊富か |
| **パフォーマンス** | 10% | 非機能要件を満たせるか |

### 2. 選定基準
- **実績**: 実際のプロダクション環境で使用されている
- **メンテナンス**: アクティブに開発・メンテナンスされている
- **ライセンス**: MITまたはApache 2.0など商用利用可能
- **依存関係**: 依存ライブラリが少なく、安定している

---

## 採用技術スタック

### 1. 言語・フレームワーク

#### Go 1.24
**選定理由**:
- ✅ 学習目的がGo言語の習得
- ✅ 並行処理、高速なコンパイル、シンプルな文法
- ✅ 標準ライブラリが充実（net/http, database/sql）
- ✅ 静的型付けによる安全性
- ✅ クロスコンパイルが容易

**代替候補と比較**:
| 項目 | Go 1.24 | Node.js | Python | Java |
|------|---------|---------|--------|------|
| 学習価値 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| パフォーマンス | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐ |
| シンプルさ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐ |
| Web開発実績 | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |

**判断**: 学習目的に最適、かつ実務でも十分使われている

---

#### Webフレームワーク: Gin
**リポジトリ**: https://github.com/gin-gonic/gin
**ライセンス**: MIT

**選定理由**:
- ✅ Goで最も人気のあるWebフレームワーク（GitHub Star 75k+）
- ✅ 高速（httprouterベース）
- ✅ ミドルウェアのエコシステムが充実
- ✅ シンプルで直感的なAPI設計
- ✅ バリデーション機能が組み込み（go-playground/validator）
- ✅ JSON処理が高速

**代替候補と比較**:
| 項目 | Gin | Echo | Chi | net/http（標準） |
|------|-----|------|-----|------------------|
| 学習価値 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| パフォーマンス | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| 機能の豊富さ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ |
| コミュニティ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| シンプルさ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |

**net/httpを選ばなかった理由**:
- ルーティング、バリデーション、ミドルウェアを自前実装する必要がある
- 学習時間が限られている中で、車輪の再発明は避けたい

**Ginを選んだ決め手**:
- 実務で最もよく使われている（転職市場での価値が高い）
- 豊富なミドルウェア（CORS, Logger, Recovery, JWT）
- ドキュメントとサンプルコードが充実

**使用する主要機能**:
```go
// ルーティング
r.GET("/api/v1/tasks/:id", handler.GetTask)
r.POST("/api/v1/projects", handler.CreateProject)

// ミドルウェア
r.Use(gin.Logger())
r.Use(gin.Recovery())
r.Use(middleware.Auth())

// バリデーション
type CreateTaskRequest struct {
    Title       string `json:"title" binding:"required,min=1,max=200"`
    Description string `json:"description" binding:"max=2000"`
    Priority    string `json:"priority" binding:"oneof=Low Medium High Urgent"`
}
```

---

### 2. データベース

#### PostgreSQL 14+
**選定理由**:
- ✅ エンタープライズグレードのRDBMS
- ✅ 豊富な機能（JSON型、全文検索、トランザクション）
- ✅ Go標準の`database/sql`で接続可能
- ✅ オープンソースで無料
- ✅ 実務での採用実績が豊富

**代替候補と比較**:
| 項目 | PostgreSQL | MySQL | SQLite | MongoDB |
|------|-----------|-------|--------|---------|
| 学習価値 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| 機能の豊富さ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐ |
| パフォーマンス | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| 実務での採用 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐ |
| トランザクション | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |

**PostgreSQLを選んだ決め手**:
- **JSON型サポート**: コメントのメンション情報などをJSON形式で保存可能
- **全文検索**: タスク検索機能に使用（`to_tsvector`, `to_tsquery`）
- **制約の豊富さ**: CHECK制約、UNIQUE制約、外部キー制約
- **実績**: メルカリ、GitHub、Instagramなど大規模サービスで採用

**使用する主要機能**:
- **UUID型**: 主キーに使用（`gen_random_uuid()`）
- **ENUM型**: ステータス、優先度などの固定値
- **JSON型**: 柔軟なデータ保存（Phase 2以降）
- **全文検索インデックス**: タスク検索（Phase 2）
- **トランザクション**: 複数テーブルの整合性保証

---

#### データベースドライバ: lib/pq
**リポジトリ**: https://github.com/lib/pq
**ライセンス**: MIT

**選定理由**:
- ✅ Go標準の`database/sql`インターフェースに準拠
- ✅ PostgreSQLの公式推奨ドライバ
- ✅ シンプルで安定している

**代替候補**: pgx（より高機能だが、学習コストが高い）

---

#### マイグレーションツール: golang-migrate
**リポジトリ**: https://github.com/golang-migrate/migrate
**ライセンス**: MIT

**選定理由**:
- ✅ データベーススキーマのバージョン管理
- ✅ up/down マイグレーションのサポート
- ✅ CLIとライブラリの両方で使用可能
- ✅ 複数データベース対応

**使用例**:
```bash
migrate create -ext sql -dir db/migrations -seq create_users_table
migrate -path db/migrations -database "postgres://..." up
```

---

### 3. 認証・認可

#### JWT（JSON Web Token）
**ライブラリ**: https://github.com/golang-jwt/jwt
**ライセンス**: MIT

**選定理由**:
- ✅ ステートレス認証（サーバー側でセッション管理不要）
- ✅ スケーラビリティが高い（水平スケーリングが容易）
- ✅ モバイルアプリ・SPAとの相性が良い
- ✅ 業界標準（RFC 7519）

**代替候補と比較**:
| 項目 | JWT | セッションCookie | OAuth 2.0 |
|------|-----|------------------|-----------|
| ステートレス | ⭐⭐⭐⭐⭐ | ⭐ | ⭐⭐⭐⭐⭐ |
| シンプルさ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐ |
| スケーラビリティ | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐⭐ |
| セキュリティ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |

**JWTを選んだ決め手**:
- API専用システムなので、ステートレス認証が最適
- 複数サーバーへのスケールが容易
- 実務での採用事例が豊富

**セキュリティ対策**:
- HMAC-SHA256で署名
- 有効期限24時間
- リフレッシュトークン（Phase 2で検討）

**使用例**:
```go
token := jwt.NewWithClaims(jwt.SigningMethodHS256, jwt.MapClaims{
    "user_id": user.ID,
    "role":    user.Role,
    "exp":     time.Now().Add(24 * time.Hour).Unix(),
})
tokenString, _ := token.SignedString([]byte(secretKey))
```

---

#### パスワードハッシュ: bcrypt
**ライブラリ**: `golang.org/x/crypto/bcrypt`
**ライセンス**: BSD-3-Clause

**選定理由**:
- ✅ 業界標準のパスワードハッシュアルゴリズム
- ✅ ソルト自動生成
- ✅ コスト調整可能（計算量を増やせる）
- ✅ レインボーテーブル攻撃に強い

**代替候補**: scrypt, argon2（より安全だが、学習コストとのバランスでbcryptを選択）

**使用例**:
```go
hashedPassword, _ := bcrypt.GenerateFromPassword([]byte(password), bcrypt.DefaultCost)
err := bcrypt.CompareHashAndPassword(hashedPassword, []byte(inputPassword))
```

---

### 4. ログ・監視

#### ログライブラリ: zap
**リポジトリ**: https://github.com/uber-go/zap
**ライセンス**: MIT

**選定理由**:
- ✅ 高速（Uberが開発）
- ✅ 構造化ログ（JSON形式）
- ✅ レベル別ログ出力（Debug, Info, Warn, Error）
- ✅ フィールドベースのログ記録

**代替候補と比較**:
| 項目 | zap | logrus | zerolog | log（標準） |
|------|-----|--------|---------|-------------|
| パフォーマンス | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| 機能の豊富さ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐ |
| 構造化ログ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐ |
| シンプルさ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |

**zapを選んだ決め手**:
- 実務での採用実績（Uber, Twitch）
- 構造化ログでログ解析が容易
- パフォーマンスが重要な場面でも安心

**使用例**:
```go
logger.Info("task created",
    zap.String("task_id", taskID),
    zap.String("user_id", userID),
    zap.String("project_id", projectID),
)
```

---

### 5. テスト

#### テストフレームワーク: 標準ライブラリ（testing）
**選定理由**:
- ✅ Go標準なので追加の依存なし
- ✅ シンプルで学習コストが低い
- ✅ `go test`コマンドで実行

**補助ライブラリ**: testify/assert
**リポジトリ**: https://github.com/stretchr/testify
**ライセンス**: MIT

**選定理由**:
- ✅ アサーションが読みやすい
- ✅ モック機能（testify/mock）
- ✅ Go標準のtestingと併用可能

**使用例**:
```go
func TestCreateTask(t *testing.T) {
    task, err := service.CreateTask(ctx, req)
    assert.NoError(t, err)
    assert.Equal(t, "Todo", task.Status)
    assert.NotEmpty(t, task.ID)
}
```

#### HTTPテスト: httptest（標準ライブラリ）
**選定理由**:
- ✅ Go標準なので追加の依存なし
- ✅ Ginとの統合が容易
- ✅ レスポンスの検証が簡単

**使用例**:
```go
w := httptest.NewRecorder()
req, _ := http.NewRequest("POST", "/api/v1/tasks", body)
router.ServeHTTP(w, req)
assert.Equal(t, 201, w.Code)
```

---

### 6. バリデーション

#### validator
**リポジトリ**: https://github.com/go-playground/validator
**ライセンス**: MIT

**選定理由**:
- ✅ Ginに組み込まれている
- ✅ タグベースのバリデーション
- ✅ カスタムバリデーション関数の定義が可能
- ✅ 多言語エラーメッセージ対応

**使用例**:
```go
type CreateTaskRequest struct {
    Title    string `json:"title" binding:"required,min=1,max=200"`
    Priority string `json:"priority" binding:"required,oneof=Low Medium High Urgent"`
    DueDate  string `json:"due_date" binding:"omitempty,datetime=2006-01-02"`
}
```

---

### 7. 環境変数管理

#### godotenv
**リポジトリ**: https://github.com/joho/godotenv
**ライセンス**: MIT

**選定理由**:
- ✅ `.env`ファイルから環境変数を読み込み
- ✅ ローカル開発環境で便利
- ✅ 本番環境では実際の環境変数を使用

**使用例**:
```go
// ローカル開発時のみ
if err := godotenv.Load(); err != nil {
    log.Println("No .env file found")
}
dbURL := os.Getenv("DATABASE_URL")
```

---

### 8. その他のライブラリ

#### UUID生成: google/uuid
**リポジトリ**: https://github.com/google/uuid
**ライセンス**: BSD-3-Clause

**選定理由**:
- ✅ UUIDv4の生成
- ✅ Googleが開発・メンテナンス
- ✅ シンプルで安定

**使用例**:
```go
id := uuid.New().String()
```

---

## 採用しない技術とその理由

### 1. ORM（GORM, ent）
**理由**:
- ❌ 学習目的ではSQL理解が重要
- ❌ ORMのマジックを理解する学習コストが高い
- ❌ 複雑なクエリでORMの制約にハマる可能性

**代わりに**: `database/sql`を直接使用してSQLを書く

---

### 2. GraphQL
**理由**:
- ❌ RESTful APIの学習が先決
- ❌ 学習コストが高い
- ❌ 小規模システムではオーバースペック

**代わりに**: RESTful API（シンプルで広く使われている）

---

### 3. Redis（Phase 1）
**理由**:
- ❌ MVP段階ではキャッシュ不要
- ❌ システムを複雑にする

**Phase 2以降で検討**: パフォーマンス要件次第で導入

---

### 4. Docker（Phase 1のアプリケーション実行）
**理由**:
- ❌ ローカル開発ではGoバイナリを直接実行（シンプル）
- ❌ Dockerfileの作成・メンテナンスコスト

**ただし**: PostgreSQLはDocker Composeで起動（環境統一のため）

---

## 技術スタック全体像

```
┌─────────────────────────────────────────────────────────┐
│                    クライアント                          │
│         （Postman / curl / フロントエンド）               │
└────────────────────┬────────────────────────────────────┘
                     │ HTTPS (TLS 1.2+)
                     │
┌────────────────────▼────────────────────────────────────┐
│                  Webサーバー                             │
│  ┌──────────────────────────────────────────────────┐  │
│  │  Gin Web Framework                                │  │
│  │  ├─ ルーティング                                   │  │
│  │  ├─ ミドルウェア（Logger, Recovery, CORS, Auth）   │  │
│  │  └─ バリデーション（go-playground/validator）      │  │
│  └──────────────────────────────────────────────────┘  │
│                                                          │
│  ┌──────────────────────────────────────────────────┐  │
│  │  アプリケーション層                               │  │
│  │  ├─ Handler（リクエスト処理）                     │  │
│  │  ├─ Service（ビジネスロジック）                   │  │
│  │  ├─ Repository（データアクセス）                  │  │
│  │  └─ Middleware（認証・認可）                      │  │
│  └──────────────────────────────────────────────────┘  │
│                                                          │
│  ┌──────────────────────────────────────────────────┐  │
│  │  認証・セキュリティ                               │  │
│  │  ├─ JWT（golang-jwt/jwt）                         │  │
│  │  └─ bcrypt（golang.org/x/crypto）                 │  │
│  └──────────────────────────────────────────────────┘  │
│                                                          │
│  ┌──────────────────────────────────────────────────┐  │
│  │  ログ・監視                                       │  │
│  │  ├─ zap（構造化ログ）                             │  │
│  │  └─ /health エンドポイント                        │  │
│  └──────────────────────────────────────────────────┘  │
└────────────────────┬────────────────────────────────────┘
                     │ database/sql + lib/pq
                     │
┌────────────────────▼────────────────────────────────────┐
│              PostgreSQL 14+                              │
│  ├─ Users, Projects, Tasks, Comments, Labels             │
│  ├─ トランザクション                                     │
│  ├─ 全文検索インデックス（Phase 2）                      │
│  └─ JSON型サポート                                       │
└──────────────────────────────────────────────────────────┘
```

---

## Phase別の技術導入計画

### Phase 1（MVP）- Day68-74
**採用技術**:
- ✅ Go 1.24 + Gin
- ✅ PostgreSQL 14+ + lib/pq
- ✅ JWT + bcrypt
- ✅ zap（ログ）
- ✅ testify（テスト）
- ✅ validator（バリデーション）
- ✅ godotenv（環境変数）
- ✅ golang-migrate（マイグレーション）

**目標**: 基本的なCRUD APIの実装

---

### Phase 2（コラボレーション）- Day75-85
**追加技術**:
- Redis（キャッシュ）
- Prometheus + Grafana（監視）
- Swagger/OpenAPI 3.0（API仕様書自動生成）
- golangci-lint（静的解析）
- GitHub Actions（CI/CD強化）

**目標**: パフォーマンス向上と運用改善

---

### Phase 3（高度な機能）- Day86-100
**追加技術**:
- WebSocket（リアルタイム通知）
- ElasticSearch（高度な検索）
- Docker + Kubernetes（コンテナオーケストレーション）
- Jaeger（分散トレーシング）

**目標**: スケーラビリティと高度な機能

---

## 学習リソース

### 公式ドキュメント
- [Go公式チュートリアル](https://go.dev/doc/tutorial/)
- [Gin公式ドキュメント](https://gin-gonic.com/docs/)
- [PostgreSQL公式ドキュメント](https://www.postgresql.org/docs/)

### 参考書籍
- "Go言語による並行処理" - Katherine Cox-Buday
- "実用Go言語" - 渋川よしき
- "Clean Architecture 達人に学ぶソフトウェアの構造と設計" - Robert C. Martin

### オンラインリソース
- [Go by Example](https://gobyexample.com/)
- [Awesome Go](https://github.com/avelino/awesome-go)

---

## まとめ

### 選定の決め手
1. **実務での採用実績が高い技術**を優先
2. **学習コストとのバランス**を考慮（複雑すぎるものは避ける）
3. **コミュニティが活発**でドキュメントが豊富
4. **シンプルさを重視**（過度な抽象化は避ける）

### 技術選定の振り返り（Phase 1完了後に実施予定）
- パフォーマンス目標を達成できたか
- 学習コストは適切だったか
- 代替技術を検討すべき箇所はあるか

---

**作成日**: 2026-01-06
**作成者**: 学習プロジェクト
**バージョン**: 1.0
**対象システム**: TaskFlow - プロジェクト・タスク管理システム
