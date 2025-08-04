# Dante DAPI UART 功能偵錯報告

## 目的

本文件旨在記錄透過 Dante DAPI (dapi-4.9.1) 對 Dante 進行 UART 功能測試的過程與發現。

## 結論

經過一系列測試，我們得出以下結論：

**設備韌體目前並未支援透過標準 DAPI/ConMon 訊息進行的序列埠 (UART) 控制。**

雖然設備的電路圖上存在 UART 硬體，但韌體並未宣告擁有此項能力，也未對相關的控制指令做出反應。因此，需要韌體查詢是否能加入對 `HAS_SERIAL_PORTS` 的支援。

---

## 測試環境

*   **DAPI 版本:** 4.9.1
*   **作業系統:** macOS
*   **測試工具:** DAPI C 範例程式 (`conmon_audinate_listener`, `conmon_audinate_controller`)
*   **目標設備:** UltimoX2-d1d2b7
*   **韌體版本:** 0.0.1

---

## 偵錯過程與關鍵證據

### 1. 確認通訊管道正常

我們使用 `conmon_audinate_controller` 嘗試向設備發送查詢指令，設備正確地回覆了 ACK，確認 ConMon 通訊管道是暢通的。

**指令:**
```bash
./conmon_audinate_controller UltimoX2-d1d2b7 serial
```

**輸出日誌:**
```
sent control message with request id 0x7f7d3271b290
Got response for request 0x7f7d3271b290: success
```
這證明了應用程式可以成功發送指令，且設備可以成功接收。

### 2. 發現設備未宣告 UART 能力

我們使用 `conmon_audinate_listener` 監聽設備上電時發出的狀態訊息。在收到的 `VERSIONS_STATUS (0x0060)` 訊息中，我們檢查了設備的能力宣告 (`capabilities`) 欄位。

**關鍵日誌 (`VERSIONS_STATUS`):**
```
#EVENT Mon Aug  4 10:08:14 2025: Received status message from 001dc1fffed1d2b7/0000 (UltimoX2-d1d2b7)
:  chan=status (rx) size=216 aud-version=0x0734 aud-type=0x0060
> Audinate message: VERSIONS_STATUS (0x0060)
>> ...
>> capabilities=0x0d5050db
>>   0x00000001=CAN_IDENTIFY
>>   0x00000002=CAN_SYS_RESET
... (其他能力) ...
>>   0x08000000=CAN_LOCK
```

**分析:**
根據 DAPI 的頭檔案 (`include/audinate/conmon/conmon_audinate_messages.h`)，支援序列埠控制的能力由第 10 個位元 (`0x400`) 的 `HAS_SERIAL_PORTS` 旗標表示。

對設備回傳的能力值 `0x0d5050db` 進行位元運算：
`0x0d5050db & 0x400 = 0`

運算結果為 0，這明確表示該韌體版本**並未宣告**支援序列埠控制。

### 3. 確認製造商資訊

從 `MANF_VERSIONS_STATUS (0x00c0)` 訊息中，我們確認了設備的製造商。

**關鍵日誌 (`MANF_VERSIONS_STATUS`):**
```
#EVENT Mon Aug  4 10:08:14 2025: Received status message from 001dc1fffed1d2b7/0000 (UltimoX2-d1d2b7)
:  chan=status (rx) size=344 aud-version=0x0734 aud-type=0x00c0
> Audinate message: MANF_VERSIONS_STATUS (0x00c0)
>> manufacturer=4d65696c6f6f6e00
>> manufacturer name="Meiloon Industries"
>> model name="Meiloon"
>> model version=0.0.1
```

---

## 請求

基於以上驗證，Dante support：

1.  確認目前的韌體版本 (`0.0.1`) 是否計畫支援 DAPI 的序列埠控制功能。
2.  如果支援，我們應該如何啟用它？
3.  如果不支援，是否可以在未來的韌體版本中加入此功能？具體來說，就是在 `VERSIONS_STATUS` 的 `capabilities` 中啟用 `HAS_SERIAL_PORTS` (0x400) 旗標，並實作對標準 `SERIAL_PORT_CONTROL` (0x0027) ConMon 訊息的處理。
