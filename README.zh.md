<h1 align="center">Pencil Design</h1>

<p align="center">
  <a href="./README.md">English</a> |
  <a href="./README.zh.md"><b>繁體中文</b></a>
</p>

<p align="center">
  <a href="https://github.com/joneshong-skills/pencil-design/stargazers"><img src="https://img.shields.io/github/stars/joneshong-skills/pencil-design?style=flat-square" alt="GitHub Stars"></a>
  <a href="https://github.com/joneshong-skills/pencil-design/blob/main/LICENSE"><img src="https://img.shields.io/github/license/joneshong-skills/pencil-design?style=flat-square" alt="License"></a>
  <img src="https://img.shields.io/badge/version-0.2.0-blue?style=flat-square" alt="Version">
</p>

<p align="center">
  Pencil.dev 的專業設計助理 -- 透過 MCP 在 IDE 中進行 AI 原生向量設計，從構思到設計轉程式碼匯出。
</p>

---

## 功能特色

- **IDE 原生設計** -- 直接在 Claude Code 中透過 Pencil MCP 工具建立與編輯 .pen 向量檔案
- **元件重用** -- 在建構前探索既有的可重用元件與設計 token，確保一致性
- **逐區段建構** -- 迭代式建構-驗證-修正工作流，每個區段完成後皆以截圖視覺驗證
- **多畫面編排** -- 透過子代理並行處理多畫面設計，各自在獨立的畫布區域作業
- **設計轉程式碼匯出** -- 將 .pen 設計轉換為 Tailwind CSS + shadcn/ui 元件，搭配語意化 class 名稱
- **4 套內建設計系統** -- Shadcn UI、Halo、Lunaris、Nitro 風格系統，加速原型設計

## 使用方式

### 觸發語句

> 「Pencil 設計」、「建立 .pen 檔案」、「編輯 .pen」、「設計畫面」、「vibe coding」、「設計轉程式碼」

### 範例

```
# 載入 Pencil MCP 工具（每次對話一次）
mcp__mcpproxy__retrieve_tools(server_name: "pencil")

# 檢查當前編輯器狀態
get_editor_state

# 探索可重用元件
batch_get(reusable: true)

# 讀取設計 token
get_variables

# 建構一個區段
batch_design([...operations...])

# 視覺驗證
get_screenshot
snapshot_layout(problemsOnly: true)
```

## 工作流程

```
SELECT_MODE --> SCAFFOLD --> DESIGN --> REFINE --> EXPORT
```

### 標準 10 步驟流程

1. **mcpproxy** -- 載入 Pencil MCP 工具
2. **get_editor_state** -- 檢查當前檔案與選取狀態
3. **batch_get** -- 探索既有可重用元件
4. **get_variables** -- 讀取設計 token
5. **get_guidelines** -- 載入任務類型規則（web-app、mobile-app、landing-page 等）
6. **get_style_guide** -- 選擇視覺方向（選用）
7. **find_empty_space_on_canvas** -- 定位放置區域
8. **batch_design** -- 逐區段建構（每次呼叫最多 25 個操作）
9. **get_screenshot** -- 視覺驗證該區段
10. **snapshot_layout** -- 偵測重疊與裁切問題

重複步驟 8-10 處理每個區段。切勿在單次 batch_design 呼叫中建構整頁。

### batch_design 操作

| 操作 | 用途 |
|------|------|
| **Insert** `I()` | 建立新節點 |
| **Copy** `C()` | 複製既有節點 |
| **Update** `U()` | 修改屬性 |
| **Replace** `R()` | 替換元件實例 |
| **Move** `M()` | 重新定位於樹狀結構中 |
| **Delete** `D()` | 移除節點 |
| **Image** `G()` | 填入 AI 生成或圖庫圖片 |

## 整合

| 技能 / 工具 | 關係 |
|-------------|------|
| **frontend-design** | 載入以獲取獨特視覺識別（避免泛用 AI 美學） |
| **brand-guidelines** | 將品牌 token 套用至 .pen 設計 |
| **canvas-design** | 不同工具：PNG/PDF 藝術輸出，非 .pen 檔案 |

## 安裝

1. 安裝並執行 Pencil Desktop App 或 VS Code/Cursor 擴充套件
2. 確保 Pencil MCP 伺服器在啟動 Claude Code 前已可用
3. 將 `SKILL.md` 放置於 `~/.claude/skills/pencil-design/`

## 授權

[MIT](https://opensource.org/licenses/MIT)
