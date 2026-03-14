# API 保護與用量控管方案（簡易版）

## 目標

* 保護內部 / 對外 API，不被濫用或打爆
* 支援 **人員登入（帳密 / JWT）** 與 **系統呼叫（API Key）**
* 可限制即時流量（rate limit）
* 可統計週期用量（週 / 月）
* 超過用量可自動或手動暫停使用者
* 全部使用 **OSS，不花授權費**

---

## 整體架構

```
Human User ──帳密──▶ Auth Server ──JWT──▶ User
System     ──API Key──────────────────────┐
                                              ▼
User / System ──▶ Traefik (Gateway)
                    ├─ JWT 驗證
                    ├─ API Key 驗證
                    ├─ Rate Limit
                    ├─ Metrics / Access Log
                    ▼
                  API Services
                    ▲
                    │
        Prometheus ◀─┘
            │
        Grafana（查詢 / Dashboard）
            │
     Batch / Cron Job（週期統計）
            │
        Auth Server（停權 / 降級）
```

---

## 元件職責說明

### Traefik（API Gateway）

* API 唯一入口
* 驗證請求來源

  * JWT（人員）
  * API Key（系統）
* Rate limit（防止瞬間爆量）
* 產生 metrics 與 access log

---

### Auth Server（自行開發）

負責 **身分與狀態管理，不處理流量**

#### 功能

* 人員帳密登入 `/login`
* 發 JWT（含使用者資訊 / plan）
* API Key 驗證 `/apikey/verify`
* 管理 client / user 狀態

  * ACTIVE
  * SUSPENDED（超標）
  * BLOCKED

#### 不負責

* 即時計算 API 用量
* rate limit

---

### Prometheus

* 收集 Traefik metrics
* 儲存 API request 次數（時間序）
* 作為週期統計的資料來源

---

### Grafana

* 顯示 API 使用狀況
* 查詢週 / 月用量
* 用量排行榜
* 協助判斷是否超標

---

## 認證方式設計

### 1️⃣ JWT（人員登入）

* 使用者先向 Auth Server 登入
* Auth Server 驗帳密後發 JWT

```http
Authorization: Bearer <JWT>
```

JWT 建議包含：

```json
{
  "sub": "user123",
  "dept": "IT",
  "role": "admin",
  "plan": "pro",
  "exp": 1739000000
}
```

Traefik：

* 使用 public key / JWKS 驗證 JWT
* 不需每次呼叫 Auth Server

---

### 2️⃣ API Key（系統 / 自動化）

```http
X-API-Key: abc123xyz
```

流程：

```
Client → Traefik → Auth Server (/apikey/verify)
                    ↓
              200 / 401 / 403
```

Auth Server 驗證成功後回傳：

```http
X-Client-Id: system-a
X-Plan: free
```

Traefik 會把這些資訊轉給 API 與 metrics

---

## 流量控制策略

### 即時流量（Rate Limit）— Traefik

| Plan  | 限制          |
| ----- | ----------- |
| free  | 10 req/sec  |
| pro   | 100 req/sec |
| admin | unlimited   |

用途：

* 防止 API 被瞬間打爆

---

### 週期用量（Quota）— Prometheus + Batch

* 不在 request 當下判斷
* 透過 metrics 週期統計

PromQL 範例（30 天）：

```promql
sum(
  increase(traefik_service_requests_total[30d])
) by (client_id)
```

---

## 超標處理流程

```
Prometheus →（查詢）→ Batch Job
                        │
                        ├─ 未超標 → 不動作
                        └─ 超標 → 呼叫 Auth Server
                                      ↓
                               status = SUSPENDED
```

Auth Server 在驗證時：

```text
status != ACTIVE → 回 403
```

Traefik 直接拒絕請求

---

## JWT 超標處理策略

* JWT 本身無法即時失效
* 建議：

  * JWT 有效期短（例如 1 小時）
  * 超標後標記 SUSPENDED
  * 下次換 token 即失效

---

## 設計原則總結

* Auth Server：

  * 只管「你是誰、你還能不能用」
* Traefik：

  * 管「你現在能不能進、進多快」
* Prometheus / Grafana：

  * 管「你用了多少」

---

## 一句話版本

> **Traefik 擋流量
> Prometheus 記用量
> Grafana 看報表
> Auth Server 負責停權**

---

（此文件為簡易版，可再依實際部署補上設定與實作細節）
