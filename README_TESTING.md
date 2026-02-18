# 測試指南 / Testing Guide

## OpenRouter API 測試

### 問題診斷

如果遇到 `Failed to authenticate request with Clerk` 錯誤（502），請檢查：

1. **API Key 是否有效**
   - 訪問 https://openrouter.ai/keys
   - 確認 API Key 是否正確複製（無多餘空格）
   - 確認 API Key 未過期

2. **API Key 格式**
   - OpenRouter API Key 應該以 `sk-proj-` 開頭
   - 長度約為 164 字符

3. **額度檢查**
   - 確認帳戶有足夠的額度
   - 檢查是否有使用限制

### 測試步驟

#### 方法 1: 使用 PowerShell 測試腳本

```powershell
# 運行測試腳本
.\test_openrouter.ps1
```

#### 方法 2: 使用 Atlas AI Agent

```powershell
# 設置環境變量
$env:OPENROUTER_API_KEY = "your-api-key-here"

# 測試單次消息
cargo run --release -- agent --message "你好"

# 測試互動模式
cargo run --release -- agent
```

#### 方法 3: 直接測試 API

```powershell
$env:OPENROUTER_API_KEY = "your-api-key-here"

$body = @{
    model = "anthropic/claude-sonnet-4-20250514"
    messages = @(@{ role = "user"; content = "ping" })
} | ConvertTo-Json -Depth 5

Invoke-RestMethod -Uri "https://openrouter.ai/api/v1/chat/completions" `
    -Headers @{
        Authorization = "Bearer $env:OPENROUTER_API_KEY"
        "Content-Type" = "application/json"
        "HTTP-Referer" = "https://github.com/Atlas-AIOS/Atlas-AI-Agent"
        "X-Title" = "Atlas AI Agent"
    } `
    -Method Post -Body $body
```

### 配置方式

#### 方式 1: 環境變量（推薦）

```powershell
# Windows PowerShell
$env:OPENROUTER_API_KEY = "your-api-key-here"

# 永久設置（用戶級）
[System.Environment]::SetEnvironmentVariable("OPENROUTER_API_KEY", "your-api-key-here", "User")
```

#### 方式 2: 配置文件

編輯 `~/.config/atlas-ai-agent/config.toml`：

```toml
[agent]
provider = "openrouter"
model = "anthropic/claude-sonnet-4-20250514"
api_key = "your-api-key-here"
```

### 常見問題

#### Q: 502 Bad Gateway 錯誤
**A:** API Key 認證失敗。請：
- 確認 API Key 正確
- 訪問 https://openrouter.ai/keys 重新生成
- 檢查帳戶狀態

#### Q: 401 Unauthorized 錯誤
**A:** API Key 無效或已過期。請重新生成 API Key。

#### Q: 429 Too Many Requests 錯誤
**A:** 請求過於頻繁。請稍後再試或檢查額度。

#### Q: 模型不存在錯誤
**A:** 檢查模型名稱是否正確。常用模型：
- `anthropic/claude-sonnet-4-20250514`
- `openai/gpt-3.5-turbo`
- `openai/gpt-4`
- `google/gemini-pro`

### 測試其他模型

如果 Claude 模型不可用，可以嘗試其他模型：

```powershell
# 使用 GPT-3.5
$body = @{
    model = "openai/gpt-3.5-turbo"
    messages = @(@{ role = "user"; content = "ping" })
} | ConvertTo-Json -Depth 5

# 使用 Gemini
$body = @{
    model = "google/gemini-pro"
    messages = @(@{ role = "user"; content = "ping" })
} | ConvertTo-Json -Depth 5
```

### 調試技巧

1. **啟用詳細日誌**
   ```powershell
   $env:RUST_LOG = "debug"
   cargo run --release -- agent --message "test"
   ```

2. **檢查網絡連接**
   ```powershell
   Test-NetConnection -ComputerName openrouter.ai -Port 443
   ```

3. **驗證 API Key 格式**
   ```powershell
   $env:OPENROUTER_API_KEY.Length  # 應該是 164
   $env:OPENROUTER_API_KEY.Substring(0,8)  # 應該是 "sk-proj-"
   ```
