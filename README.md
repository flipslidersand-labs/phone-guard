# Phone Guard — Landline Scam Call Protection Device

![Language](https://img.shields.io/badge/language-C%2B%2B%20%7C%20Go-blue)
![Platform](https://img.shields.io/badge/platform-ESP32%20%7C%20GCP-orange)
![License](https://img.shields.io/badge/license-MIT-green)

---

## English

An IoT device that physically taps into the landline and automatically sends a LINE notification to family members when a call arrives from an unknown number.

- **No recording** — voice audio is never sent to any server
- **No AI judgment** — the device does not determine whether a call is a scam; that decision is left to the family
- **Number sharing only** — users may optionally contribute numbers to the shared database

### Tech Stack

| Layer | Technology |
|---|---|
| Embedded | ESP32 + FSK decode (C++, PlatformIO) |
| Backend | Go (Cloud Run) |
| Database | PostgreSQL (Cloud SQL) |
| Notification | LINE Notify API |
| Deployment | GCP |

### Hardware Overview

```
Landline
    ↓
[Modular splitter jack]
    ↓              ↓
Telephone set    ESP32
                  ├── FSK decode
                  ├── Wi-Fi
                  └── LINE notification
```

Parts: ESP32 (¥1,000) + modular splitter (¥500) + analog circuit (¥500) + case (¥500) = approx. ¥3,000

### Phase Design

#### Phase 1 — Firmware MVP
- FSK decode on ESP32 standalone
- Local phone book lookup
- LINE Notify alert

#### Phase 2 — Cloud Integration
- Lookup against number DB (display national report count)

#### Phase 3 — Smart Features
- Automatic call rejection for high-risk numbers
- Statistical analysis

#### Phase 4 — Production
- PCB mass production, sold via e-commerce

### Project Structure

```
phone-guard/
├── esp32/            # Firmware (C++ / PlatformIO)
│   ├── src/
│   │   └── main.cpp
│   ├── lib/
│   └── platformio.ini
├── backend/          # Go API
│   ├── cmd/api/
│   ├── internal/
│   └── go.mod
├── db/
│   └── schema.sql
└── docs/
    ├── fsk-spec.md
    └── wiring.md
```

### Getting Started

#### Firmware (Phase 1)

```bash
cd esp32
pio run -t upload   # Upload to ESP32
pio device monitor  # Serial monitor
```

#### Backend (Phase 2)

```bash
cd backend
go mod download
go run cmd/api/main.go
```

#### Database (PostgreSQL)

```bash
psql -U postgres -f db/schema.sql
```

### API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/numbers/:number` | Look up number info |
| POST | `/api/calls` | Log an incoming call |
| GET | `/api/whitelist` | Retrieve whitelist |
| POST | `/api/whitelist` | Add to whitelist |

### CI

GitHub Actions calls reusable workflows from `flipslidersand-labs/qa-workflows`:

| Workflow | Purpose |
|---|---|
| `go-test.yml` | Go tests for `backend/` (`working-directory: backend`) |
| `gitleaks.yml` | Secrets scan (gitleaks) |

**Required setting:** the Actions variable `GATE_RUNNER` must be set. Both workflows read it as the runner (`runs-on`), with no default for public repos. It is set at the organization level; if you fork this repository, set `GATE_RUNNER` yourself (Settings → Secrets and variables → Actions → Variables), e.g. `ubuntu-latest`. Without it the jobs stay queued.

### License

MIT

---

## 日本語

固定電話の回線に物理割り込みして、知らない番号から着信があったら家族の LINE に自動通知する IoT デバイス。

- **録音しない** — 音声はサーバに送らない
- **判定しない** — AI で詐欺か判断しない。家族が判断する
- **番号だけ共有** — ユーザーが任意で DB に提供

### 技術スタック

| レイヤー | 技術 |
|---|---|
| 組み込み | ESP32 + FSK デコード（C++, PlatformIO） |
| バックエンド | Go（Cloud Run） |
| DB | PostgreSQL（Cloud SQL） |
| 通知 | LINE Notify API |
| デプロイ | GCP |

### ハードウェア構成

```
固定電話回線
    ↓
[分岐モジュラージャック]
    ↓          ↓
固定電話機   ESP32
               ├── FSKデコード
               ├── Wi-Fi
               └── LINE通知
```

部品: ESP32（1k）+ モジュラー分岐（500）+ アナログ回路（500）+ ケース（500）= 約3,000円

### フェーズ設計

#### Phase 1（Firmware MVP）
- ESP32 単体で FSK デコード
- ローカル電話帳照合
- LINE Notify 通知

#### Phase 2（Cloud Integration）
- 番号 DB と照合（全国記録件数表示）

#### Phase 3（Smart Features）
- 高スコア番号の自動切断
- 統計分析

#### Phase 4（Production）
- 基板量産、EC で販売

### プロジェクト構成

```
phone-guard/
├── esp32/            # ファームウェア (C++ / PlatformIO)
│   ├── src/
│   │   └── main.cpp
│   ├── lib/
│   └── platformio.ini
├── backend/          # Go API
│   ├── cmd/api/
│   ├── internal/
│   └── go.mod
├── db/
│   └── schema.sql
└── docs/
    ├── fsk-spec.md
    └── wiring.md
```

### Getting Started

#### ファームウェア（Phase 1）

```bash
cd esp32
pio run -t upload   # ESP32 へ書き込み
pio device monitor  # シリアルモニタ
```

#### バックエンド（Phase 2）

```bash
cd backend
go mod download
go run cmd/api/main.go
```

#### データベース（PostgreSQL）

```bash
psql -U postgres -f db/schema.sql
```

### API エンドポイント

| メソッド | エンドポイント | 説明 |
|---|---|---|
| GET | `/api/numbers/:number` | 番号情報照合 |
| POST | `/api/calls` | 着信ログ記録 |
| GET | `/api/whitelist` | ホワイトリスト取得 |
| POST | `/api/whitelist` | ホワイトリスト追加 |

### CI

GitHub Actions は `flipslidersand-labs/qa-workflows` の reusable workflow を呼び出します。

| Workflow | 内容 |
|---|---|
| `go-test.yml` | `backend/` の Go テスト（`working-directory: backend`） |
| `gitleaks.yml` | シークレット走査（gitleaks） |

**必須設定:** Actions 変数 `GATE_RUNNER` が必要です。どちらの workflow も runner（`runs-on`）としてこの変数を読み、公開リポでは既定値がありません。org 設定で定義済みです。fork する場合は各自で `GATE_RUNNER`（Settings → Secrets and variables → Actions → Variables）を設定してください（例: `ubuntu-latest`）。未設定だとジョブは queued のまま止まります。

### ライセンス

MIT
