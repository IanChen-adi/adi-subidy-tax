# adi-subidy-tax：產創補助資料上傳與查詢平台

Cloudflare Pages（靜態 `index.html`）＋ Pages Functions（`functions/api/[[path]].js`）＋ D1 資料庫，前面由 Cloudflare Access 保護。

- 正式網址：`https://inv-subsidy-adi.pages.dev`（另有 `adi-subsidy.pages.dev`）
- D1 綁定名稱：`DB`（在 Pages → Settings → Bindings 設定）

## 登入機制

後端只信任**經過簽章驗證**的 Cloudflare Access JWT（`Cf-Access-Jwt-Assertion` header 或 `CF_Authorization` cookie）。
沒有設定下列環境變數時，所有人都視為未登入（fail closed）。

| 環境變數（Pages → Settings → Variables and Secrets） | 說明 |
|---|---|
| `CF_ACCESS_TEAM_DOMAIN` | Zero Trust 團隊網域，例如 `xxx.cloudflareaccess.com`（Zero Trust → Settings → Custom Pages / Team domain） |
| `CF_ACCESS_AUD` | Access 應用程式的 **Application Audience (AUD) Tag**（Zero Trust → Access → Applications → 該 app → Overview）。多個 app 用逗號分隔 |
| `ADMIN_EMAILS` | （選填）追加管理員，逗號分隔。程式內預設管理員：`enyichen0413@adi.gov.tw`、`yswu@adi.gov.tw` |
| `ENVIRONMENT` | （僅本機）在 `.dev.vars` 設 `development` 時，`localhost` 可用畫面上的「開發模式 email」模擬登入。正式環境不要設 |

### 第一次設定 / 上線前檢查

1. Zero Trust → Access → Applications → Add an application → **Self-hosted**
   - Domain：本專案的正式網址，以及 `*.<專案>.pages.dev`（預覽部署）
   - Policy：Allow，Include → Emails ending in `@adi.gov.tw`
2. 複製該 app 的 AUD Tag 與團隊網域，填入上表的 `CF_ACCESS_AUD`、`CF_ACCESS_TEAM_DOMAIN`（Production 與 Preview 都要）。
3. （建議）Pages → Settings → General → **Enable access policy**，保護 preview 部署網址。
4. 部署後用無痕視窗打開網址：應先跳到 Access 登入頁，登入後右上角顯示自己的 email。

## 權限

| 動作 | 誰可以 |
|---|---|
| 查詢、匯出、建立服務、上傳資料 | 任何已登入者 |
| 刪除服務、刪除上傳紀錄、轉移負責人 | 該服務負責人或管理員 |

## 本機開發

```bash
cp .dev.vars.example .dev.vars   # 內含 ENVIRONMENT=development
npx wrangler pages dev . --d1 DB=<資料庫名稱>
```

打開 `http://localhost:8788`，在右上角填入 email 模擬登入。
