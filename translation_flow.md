# 翻譯流程設計

## 1. 系統輸入

-   原始輸入：TXT 文稿（已確認固定為 txt 格式）

## 2. 預處理

- 	偵測原文語言（使用detect_language函式）
-   如果是中文就做繁轉簡（使用OpenCC）

## 3. 佔位符管理

-   在正式翻譯前，建立一個暫存物件（Dict Array）作為佔位符

-   格式範例：

    ``` json
    [
      {"term": "王男", "translation": null, "placeholder": "[PER_1]"},
      {"term": "臺北市", "translation": null, "placeholder": "[LOC_2]"}
    ]
    ```
    
## 4. 詞彙儲存邏輯

-   資料庫：MySQL（localhost:3306, database name: aidb, table: translations）
	
  	``` SQL
	id INT AUTO_INCREMENT PRIMARY KEY,
    source_lang VARCHAR(10) NOT NULL COMMENT '來源語言 (zh, en, jp...)',
    target_lang VARCHAR(10) NOT NULL COMMENT '目標語言 (en, zh, jp...)',
    source_text VARCHAR(255) NOT NULL COMMENT '來源文字',
    target_text VARCHAR(255) DEFAULT NULL COMMENT '翻譯後文字',
    status ENUM('pending','translated') DEFAULT 'pending' COMMENT '翻譯狀態',
    term_type varchar(50) COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT '詞彙來源（正則,NER,UNK等分類）', 
    confidence decimal(5,4) DEFAULT NULL COMMENT '信心分數 (0.0~1.0)，表示翻譯可信度', 
    category varchar(50) COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT '詞彙類型，例如: 人名, 地名, 組織名, 專有名詞等',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT '建立時間',
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '最後更新時間',
    UNIQUE KEY unique_translation (source_lang, target_lang, source_text)
    ```
   
-   寫入流程：
    -   若詞彙已存在資料庫，不再重複寫入
    -   若詞彙不存在，新增一筆記錄（詞彙本身先存入，翻譯欄位可空白）

## 5. 詞彙擷取第一步
-   使用自訂定義的 JSON File 並使用正則的方式把文章的詞彙找出來，並用佔位浮替代，寫入Dict Array 跟寫入資料庫
-   [REG_1], [REG_2], … 代表「正則擷取」詞彙

## 6. 詞彙擷取第二步
-   使用NER（ckiplab/bert-base-chinese-ner）找出專有名詞、關鍵詞（如人名、地名、組織名等）
-   並用佔位浮替代，寫入Dict Array 跟寫入資料庫

## 7. 詞彙擷取第三步
-   先切詞，再作詞表對照，再使用翻譯模型（m2m100）先找出unk的字
-   並用佔位浮替代，寫入Dict Array 跟寫入資料庫
-   [UNK_1], [UNK_2], … 代表「未知/模型無法識別」詞彙


## 8. 翻譯流程

-   **步驟 A**：先使用 M2M100 進行全文的翻譯（做完詞彙擷取的文稿）
-   **步驟 B**：之後再使用科大訊飛進行Dict Array的翻譯，並重新寫入Dict Array，與此同時也要更新資料庫（寫入、強制更新）


## 9. 完整翻譯輸出

-   替換譯文的佔位符詞彙（從Dict Array替換）
-   確保輸出文本完整無遺漏

------------------------------------------------------------------------
