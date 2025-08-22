# 安裝與設定 Google Gemini CLI

## 1. 安裝 Node.js

請先下載並安裝 [Node.js](https://nodejs.org/zh-tw/download)

確認版本：

``` powershell
node -v
npm -v
```

------------------------------------------------------------------------

## 2. 使用管理者權限開啟 PowerShell

### 確認 PowerShell 執行原則

``` powershell
Get-ExecutionPolicy -List
```

### 變更原則

``` powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned
```

------------------------------------------------------------------------

## 3. 暫時關閉 npm SSL

``` powershell
npm config set strict-ssl false
```

------------------------------------------------------------------------

## 4. 開始安裝 Gemini CLI

``` powershell
npm install -g @google/gemini-cli
```

------------------------------------------------------------------------

## 5. 設定環境變數

### 永久設定環境變數

``` powershell
[Environment]::SetEnvironmentVariable(
  "Path",
  $env:Path + ";C:\Users\你的使用者名稱\AppData\Roaming\npm",
  "User"
)
```

### 暫時設定環境變數

``` powershell
$env:Path += ";C:\Users\meiloon\AppData\Roaming\npm"
```

------------------------------------------------------------------------

## 6. 登入時關閉憑證驗證

### 永久關閉

``` powershell
[Environment]::SetEnvironmentVariable("NODE_TLS_REJECT_UNAUTHORIZED", "0", "User")
```

顯示是否寫入成功：

``` powershell
[Environment]::GetEnvironmentVariable("NODE_TLS_REJECT_UNAUTHORIZED", "User")
```

### 暫時關閉

``` powershell
$env:NODE_TLS_REJECT_UNAUTHORIZED = "0"
```

查看變數：

``` powershell
$env:NODE_TLS_REJECT_UNAUTHORIZED
```

------------------------------------------------------------------------

## 7. 設定 Google API Key

``` powershell
[Environment]::SetEnvironmentVariable("GEMINI_API_KEY", "你的金鑰", "User")
```
