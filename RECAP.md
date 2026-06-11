# oásis 教課回報系統 — 復盤

## 建了什麼

### 前端（GitHub Pages）

| 檔案 | 說明 |
|------|------|
| `index.html` | LIFF 身份驗證表單，欄位：課程日期、主辦單位、費用、備註 |
| `webhook-test.html` | n8n webhook 測試工具 |

- **Repo：** https://github.com/AndyuLIU/oasis-edu-form
- **部署網址：** https://andyuliu.github.io/oasis-edu-form/
- **測試工具：** https://andyuliu.github.io/oasis-edu-form/webhook-test.html

---

### 後端流程（n8n）

```
Webhook POST /edu-report
  → Notion Query（員工資料庫，用 Line UID 查人）
  → IF 員工是否存在？
      TRUE  → Notion Create（教課回報 DB，含員工 relation）
      FALSE → Notion Create（教課回報 DB，無 relation）
  → 回應 200
```

---

### Notion 資料庫欄位

| 欄位 | 類型 | 說明 |
|------|------|------|
| 名稱 | Title | 自動組合：`設計師 · 課程日期` |
| 設計師 | Text | 從 employee-lookup 取得 |
| 課程日期 | Date | 課程發生日期 |
| 主辦單位 | Text | 邀約方／主辦單位 |
| 費用 | Number | 單位：新台幣 |
| 備註 | Text | 選填 |
| Line UID | Text | 送出者的 LINE ID |
| 送出時間 | Date | 表單送出的 ISO timestamp |
| 員工 | Relation | 雙向連結員工資料庫 |

---

## 系統架構

```
LINE LIFF（2010336788-nDwPd2If）
  → 取得 LINE UID
  → employee-lookup 查身份（role = 設計師 才放行）
  → 填表送出
  → n8n webhook /edu-report
  → Notion 教課回報 DB（relation 連結員工資料庫）
```

---

## 過程中踩到的坑

| 問題 | 原因 | 解法 |
|------|------|------|
| Notion integration 看不到員工 DB | 員工資料庫未加入 n8n integration | 員工 DB → Connections → 加入 n8n integration |
| Relation 欄位不出現 | 教課回報 DB 沒有建立 Relation 欄位 | 在 Notion 手動建 Relation 欄位指向員工 DB |
| Relation pageId 帶 `-` 報錯 | Notion API 不接受 UUID 格式的 `-` | expression 加 `.replaceAll('-', '')` |
| `texts is not iterable` | JSON 匯入後 Notion node 的 `rich_text` 格式異常 | 刪掉 Create 節點，改用 UI 手動重建 |
| `備註` 空字串炸節點 | Notion node 不接受空字串的 `rich_text` | expression 改用 `\|\| undefined`，不傳空字串 |
