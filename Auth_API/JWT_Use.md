

登入 → 拿 JWT → 呼叫 API → 自動刷新 → 過期 → 重新登入


### 前端保存 Token（重點）

Access Token

放 memory（JS 變數）

或 Authorization header 管理器

❌ 不建議放 localStorage（XSS）

Refresh Token

httpOnly cookie（最安全）

或後端 session


### JWT 標準使用方式



```http
GET /api/report HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..
```
Bearer 是標準前綴

JWT 放在 header 裡面

API / Traefik / middleware 都直接讀 header

幾乎所有服務都用這個方式

### JWT 自動續約方式

核心概念

使用者身份固定（同一個帳號 / client_id）
但 Access Token（JWT）短效，需要定期換新的 JWT

Refresh Token 是長效憑證

Access Token 是短效憑證

**換新 JWT 不需要重新登入帳密**，只要拿舊 JWT 或 refresh token 換

以上都在前端製作

後端就使用 /refresh
驗證後發放
相同身份,但是更新的exp實用期限,新的JWT
