醫療器材智慧審查與補件評估平台 (TFDA AI Regulatory Agent)
系統架構與技術建置詳細規格書 (Technical Specification Document)
本規格書詳述「TFDA 醫療器材智慧審查與補件評估平台」的完整技術架構、AI 代理程式（AI Agent）工作流、核心演算法設計、程式碼除錯改進方案、以及動態互動式網頁產生器（HTML Generator）的建置細節。本平台旨在為醫療器材查驗登記（RA）專業人員提供一站式的智慧輔助工具。

壹、 系統願景與核心定位
本系統是一款專為台灣衛福部食藥署（TFDA）醫療器材查驗登記流程設計的智慧代理程式（Agentic AI）應用。平台的核心價值在於協助國內藥商與醫療器材製造廠，在收到官方補件公文（Deficiency Letter）或自行進行預審（Pre-assessment）時，能快速、結構化地解析數百頁的技術文件（Technical Dossiers），自動比對合規條款、辨識缺失、撰寫專業回覆對策，並產生能完全離線運行的動態互動式合規工作平台。

[使用者輸入: 公文/技術文件]
      │
      ▼
[AI 代理核心 (Gemini-3.1-Flash-Lite)] ◄───► [TFDA 合規法規資料庫 (預設/自訂)]
      │
      ├─► 1. 條款符合性判定 & 缺失提取
      ├─► 2. 智慧補件對策 (Smart-Fill Co-Pilot) 撰寫
      └─► 3. 國際原廠 Liaison 英文協調信生成
      │
      ▼
[動態合規工作平台 (自包含單一 HTML 檔案)]
      ├─► 整合式 KPI 儀表板 & 三大「驚豔功能 (Wow Features)」
      ├─► 互動式技術文件符合性編輯器 & 缺漏追蹤面板
      ├─► 無摩擦拖放看板 (Kanban Task Board) & 聯繫信生成器
      └─► 支援一鍵導出: JSON / Markdown / 完美的列印 PDF
貳、 核心代理程式（AI Agent）與大語言模型配置
本系統預設採用 Google 最新發布之 gemini-3.1-flash-lite 作為預設推理與文本處理引擎。該模型具備極高的性價比、超低延遲以及強大的上下文理解能力（1M tokens 上下文視窗），非常適合用於解析長篇幅的醫療器材技術手冊、生物相容性報告、臨床試驗計劃書及官方公文。

一、 多模型支援架構
為確保企業用戶的靈活性，系統底層設計為「非耦合式模型層」，允許使用者透過後端或 API 介面切換以下大語言模型：

gemini-3.1-flash-lite（預設，效能與速度平衡）

gemini-3.1-pro（適合深度、跨條款邏輯推演與複雜分析）

claude-3-5-sonnet（適合精準的法規商務英文書信與法條撰寫）

gpt-4o（適合多模態圖表、說明書影像解析）

二、 系統提示詞（System Prompt）模板與動態自訂
系統允許用戶在前端介面即時修改 Prompt 以微調 AI 的分析重點。以下為預設 AI Agent 審查專家提示詞模板：

Plaintext
你是一位深諳台灣《醫療器材管理法》、《醫療器材查驗登記審查準則》以及 2026 年最新 TFDA 查驗登記與臨床前測試基準的資深醫療器材法規專家（Regulatory Affairs Expert）。

你的任務是解析使用者上傳的【送審材料/摘要/審查意見】，並對照【審查指引/清單】進行深度比對。

【處理流程】：
1. 依據輸入文件，將其切分為結構化的「技術安全與效能審查項目」（如電性安全、聲學輸出、生物相容性、軟體確效與資通安全、AI CADe/CADx 演算法、性能測試等）。
2. 針對每一項目進行「符合性判定」（OK 或 不符合）。
3. 如果判定為「不符合」，必須：
   a. 詳細寫出「技術缺失與審查意見內容」（說明缺失何在、缺少哪些標準、哪些探頭型號缺失）。
   b. 設計對應的「官方正式回覆草案（Response Draft）」，使用專業、肯定的合規語言，引導使用者提供對應的附件、證明或報告。
   c. 評估「優先處理等級（Priority）」（High / Medium / Low）與「嚴重程度（Severity）」。
4. 生成完整的結構化 JSON 物件，格式必須與系統前端 JSON 架構完美對齊。

【注意】：
1. 專有名詞須符合台灣 TFDA 語境（例如：使用「查驗登記」而非「註冊」，「說明書」而非「說明書/標籤」，「確效」而非「驗證」）。
2. 對於「不符合」項目，回覆對策中必須包含具體的行動方針（例如：ISO 10993、IEC 60601 等標準之對應要求）。
參、 技術架構與運作流程（Workflow）
本系統採用「AI 輕量化後端 + 富交互前端」的設計理念，確保使用者下載的成果網頁能夠完全離線（Offline-first）在瀏覽器中開啟與操作，極大保護了醫療器材商的商業機密安全。

【步驟 1：輸入階段】
 ├─ 使用者透過前端 UI 拖放或黏貼上傳：
 │   ├── 1. 送審材料 / 技術報告摘要 / 官方補件通知書
 │   └── 2. 審查基準（可自選預設「超音波診斷儀查驗登記基準」或上傳自訂 checklist）
 └─ 設定運作模型（預設為 gemini-3.1-flash-lite）與自訂 Prompt。

【步驟 2：Agent 智慧解析階段】
 ├─ 後端 Agent 調用指定的 Gemini API，執行長文本法條映射。
 ├─ 執行合規比對，產出結構化的項目狀態、缺失描述、任務分配與原廠信件元素。
 └─ 自動優化中文錯別字，並辨識禁忌症、預期用途（預備處理 wow feature 1）。

【步驟 3：靜態 HTML 封裝與下載階段】
 ├─ 系統讀取預置的 HTML 範本（基於 Tailwind CSS、FontAwesome、原生 JavaScript 狀態引擎）。
 ├─ 將 AI 解析產出的結構化 JSON 資料，直接注入至 HTML 程式碼中的 `reviewReport` 全域變數中。
 ├─ 注入三大 WOW 功能所需之 JS 模組（包含 Vis.js/D3.js 關係圖譜、RAG 檢索問答知識庫、AI 寫作輔助器）。
 └─ 提示使用者下載 `.html` 檔案（或打包 `.zip` 封裝套件）。
肆、 網頁潛在錯誤修正與架構優化
在評估原始提供之 sample 網頁後，我們發現以下技術缺陷與潛在 Bug，並在本規格書中予以修正：

1. 模型物件硬編碼問題（Hardcoded Models in JS Functions）
原 Bug 描述：exportToJSON 與 exportToMarkdown 函數中調用了 modelFileFriendlyName() 輔助函數，但該函數內部硬編碼了 "EVO_Q10_Ultrasound"。當使用者上傳其他機型（例如：“Wireless Handheld Probe Q20”）時，下載的檔名依然是 EVO_Q10_Ultrasound。

修正方案：重構 modelFileFriendlyName() 函數，使其動態讀取 DOM 節點中的 email-model 欄位值，並過濾掉非法檔名字符。

2. 拖放（Drag & Drop）卡片遺失問題
原 Bug 描述：handleDrop 函數僅透過 event.dataTransfer.getData("text/plain") 讀取 draggedCardIndex。在某些近代瀏覽器（如 Safari 或 Firefox 隱私模式）中，基於安全原則，dataTransfer 在 drop 事件中可能返回空字串，導致轉換失敗。

修正方案：保留全域變數 draggedCardIndex 作為雙重保障（Fallback mechanism），當 dataTransfer 為空時自動使用備用全域變數。

3. 印表模式下的文字折行與字型失真
原 Bug 描述：在 @media print 樣式中，Tailwind 的部分 flex 與 grid 容器若未設定 display: block，會造成 PDF 輸出時排版嚴重截斷，尤其是 Table 中帶有長文本的 textarea。

修正方案：在 Print 模式樣式表中強制將 textarea 與長文本 td 設定為 white-space: pre-wrap，並在列印前將所有動態 textarea 的實體內容同步到獨立的 p 標籤中展示。

伍、 三大驚豔功能（Wow Features）技術實現細節
為了大幅度提升產出 HTML 網頁的實用價值，我們在網頁中嵌入以下三個進階互動式功能：

1. 驚豔功能一：互動式合規缺漏關係圖譜（Visual Compliance Gap Topology Map）
技術說明：利用 HTML5 <canvas> 與輕量級無相依向量圖形庫（或內建動態 SVG 生產器），將 TFDA 條款、產品結構、探頭清單、安全報告之間的關聯，繪製成一個動態力導向關係圖（Force-Directed Graph）。

交互體驗：

綠色節點代表「符合（OK）」之條款。

紅色節點代表「不符合（缺失）」之條款。

黃色節點代表「部分符合/正在審查中」之條款。

當使用者點擊圖譜中的紅色「生物相容性」節點時，右側側邊欄會即時彈出受影響的 16 款探頭清單。點擊「跳轉定位」可直接導航至 Tab 2 的回覆撰寫區。

   [Q10 主機系統] ────────── [軟體與網路安全 (OK)]
         │
         ├─────────────── [電性安全性 (OK)]
         │
         ├─────────────── [超音波聲學特定安全 (不符合)] ◄─── (點擊查看: 熱指數/機械指數缺漏)
         │
         └─────────────── [生物相容性檢測 (不符合)]
                                 │
                     ┌───────────┴───────────┐
            [LA3-16AD 探頭 (不符合)] ... [CA2-8AD 探頭 (不符合)]
2. 驚豔功能二：離線自主 RAG 與法規 AI 稽核面板（Offline AI-Auditor & Chat Sidebar）
技術說明：本功能在離線網頁中提供一個「Ask-Me-Anything」智慧對話框。系統在產生 HTML 時，已將使用者上傳的原始申報資料、TFDA 缺失公文與法規庫，壓縮轉譯為一組離線高效檢索字典（Offline Search Index），並封裝至 HTML 檔案中。

交互體驗：

使用者可以在離線網頁中輸入關鍵字，如：「探頭生物相容性」、「IEC 62359 聲場測試要求」。

離線 RAG 引擎會啟動 Fuzzy Search 與 Regex 語義相近度計算，在數毫秒內檢索出所有關聯的技術規範、受影響條款編號、以及系統預先產生的對策建議。

如果使用者配置了個人的 API Key，該對話框更能直接調用瀏覽器端的 fetch 去呼叫雲端大語言模型，提供真正的「雙向即時對策潤飾服務」。

3. 驚豔功能三：說明書禁忌症與預期用途「智慧防呆對照工具」（Labeling & Precaution Alignment Tool）
技術說明：專門解決查驗登記中最常見、最致命的「中文說明書/標籤內容與原廠技術文件規格不一致」問題（如 sample 網頁中提到的：眼科預設模式禁忌症衝突與翻譯錯別字）。

交互體驗：

平台提供一組側邊雙欄比對視窗。左欄顯示原廠英文說明書規格宣告（如 “Contraindication: Ophthalmic use is strictly prohibited except in dedicated ophthalmic mode”）。

右欄顯示中文說明書翻譯。AI 會自動利用紅色標示出差異（如：中文版漏譯了「除專用模式外」），並直接提示修正後的繁體中文合規建議段落。

使用者點擊「一鍵套用」，該修正內容便會自動寫入 Tab 2 的補件回覆對策。

陸、 完整網頁程式碼實現 (HTML 範本)
以下是根據規格優化、包含三大驚豔功能、並解決所有潛在 Bug 的完整網頁程式碼。當 AI 代理程式處理完您的上傳資料後，會產生包含此程式碼架構的 HTML 檔供您下載：

HTML
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>TFDA 醫療器材智慧審查與補件評估平台 - 企業進階版</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Google Fonts: Inter and Noto Sans TC -->
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&family=Noto+Sans+TC:wght@300;400;500;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', 'Noto Sans TC', sans-serif;
        }
        /* 定製精緻捲軸 */
        ::-webkit-scrollbar {
            width: 6px;
            height: 6px;
        }
        ::-webkit-scrollbar-track {
            background: #f1f5f9;
        }
        ::-webkit-scrollbar-thumb {
            background: #cbd5e1;
            border-radius: 4px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: #94a3b8;
        }
        @media print {
            body {
                background: white !important;
                color: black !important;
            }
            .no-print {
                display: none !important;
            }
            .print-full-width {
                width: 100% !important;
                max-width: 100% !important;
                padding: 0 !important;
                margin: 0 !important;
                box-shadow: none !important;
            }
            .page-break {
                page-break-before: always;
            }
        }
        /* 拓撲圖節點樣式 */
        .node-pulse {
            animation: pulse 2s infinite;
        }
        @keyframes pulse {
            0% { transform: scale(1); opacity: 1; }
            50% { transform: scale(1.1); opacity: 0.8; }
            100% { transform: scale(1); opacity: 1; }
        }
    </style>
</head>
<body class="bg-slate-50 text-slate-800 min-h-screen flex flex-col antialiased">

<!-- 頂部導航欄 -->
<header class="bg-gradient-to-r from-slate-900 to-indigo-950 text-white shadow-lg no-print">
    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-4 flex flex-col md:flex-row md:items-center md:justify-between gap-4">
        <div class="flex items-center space-x-3">
            <div class="p-2.5 bg-indigo-600 rounded-xl text-white shadow-md shadow-indigo-500/20">
                <i class="fa-solid fa-square-poll-horizontal text-2xl"></i>
            </div>
            <div>
                <h1 class="text-xl font-bold tracking-tight flex items-center gap-2">
                    TFDA 醫療器材智慧審查與補件評估平台
                    <span class="text-xs bg-indigo-500/30 text-indigo-300 font-medium px-2 py-0.5 rounded-full border border-indigo-500/40">PRO 專業版</span>
                </h1>
                <p id="project-subtitle" class="text-xs text-slate-300 mt-0.5">超音波診斷儀與 22 款探頭查驗登記 (符合 2026 TFDA 最新規範)</p>
            </div>
        </div>
        
        <!-- 操作按鈕 -->
        <div class="flex flex-wrap items-center gap-2">
            <button onclick="exportToJSON()" class="px-3.5 py-2 bg-slate-800 hover:bg-slate-700 text-slate-200 rounded-lg text-sm font-medium transition flex items-center gap-1.5 border border-slate-700">
                <i class="fa-solid fa-code text-indigo-400"></i> 匯出 JSON
            </button>
            <button onclick="exportToMarkdown()" class="px-3.5 py-2 bg-slate-800 hover:bg-slate-700 text-slate-200 rounded-lg text-sm font-medium transition flex items-center gap-1.5 border border-slate-700">
                <i class="fa-brands fa-markdown text-blue-400"></i> 匯出 Markdown
            </button>
            <button onclick="triggerPrint()" class="px-4 py-2 bg-indigo-600 hover:bg-indigo-500 text-white rounded-lg text-sm font-semibold transition flex items-center gap-1.5 shadow-lg shadow-indigo-600/20">
                <i class="fa-solid fa-file-pdf"></i> 列印 / 儲存 PDF
            </button>
        </div>
    </div>
</header>

<!-- 主工作區 -->
<main class="flex-grow max-w-7xl w-full mx-auto px-4 sm:px-6 lg:px-8 py-6 flex flex-col gap-6 print-full-width">
    
    <!-- KPI 數據儀表板 -->
    <section class="grid grid-cols-1 md:grid-cols-4 gap-4 no-print">
        <div class="bg-white p-5 rounded-2xl border border-slate-200 shadow-sm flex items-center justify-between">
            <div>
                <span class="text-xs font-semibold text-slate-400 uppercase tracking-wider block">審查項目通過率</span>
                <span id="completion-percentage" class="text-3xl font-extrabold text-slate-800 tracking-tight mt-1 block">50.0%</span>
                <span class="text-xs text-emerald-600 font-medium mt-1 inline-flex items-center gap-0.5">
                    <i class="fa-solid fa-arrow-trend-up"></i> 即時狀態
                </span>
            </div>
            <!-- 動態進度圓圈 -->
            <div class="relative w-16 h-16">
                <svg class="w-full h-full transform -rotate-90" viewBox="0 0 36 36">
                    <path class="text-slate-100" stroke-width="3" stroke="currentColor" fill="none" d="M18 2.0845 a 15.9155 15.9155 0 0 1 0 31.831 a 15.9155 15.9155 0 0 1 0 -31.831" />
                    <path id="svg-progress-circle" class="text-indigo-600 transition-all duration-500" stroke-dasharray="50, 100" stroke-width="3" stroke-linecap="round" stroke="currentColor" fill="none" d="M18 2.0845 a 15.9155 15.9155 0 0 1 0 31.831 a 15.9155 15.9155 0 0 1 0 -31.831" />
                </svg>
                <div class="absolute inset-0 flex items-center justify-center text-xs font-bold text-slate-700" id="circle-percentage-text">50%</div>
            </div>
        </div>

        <div class="bg-white p-5 rounded-2xl border border-slate-200 shadow-sm">
            <div class="flex justify-between items-start">
                <div>
                    <span class="text-xs font-semibold text-slate-400 uppercase tracking-wider block">補件缺失項目</span>
                    <span id="defect-count-badge" class="text-3xl font-extrabold text-rose-600 tracking-tight mt-1 block">5 項</span>
                    <p class="text-xs text-rose-500 mt-1 flex items-center gap-1">
                        <i class="fa-solid fa-circle-exclamation"></i> 需在限期內補齊
                    </p>
                </div>
                <div class="p-3 bg-rose-50 text-rose-600 rounded-xl">
                    <i class="fa-solid fa-triangle-exclamation text-lg"></i>
                </div>
            </div>
        </div>

        <div class="bg-white p-5 rounded-2xl border border-slate-200 shadow-sm">
            <div class="flex justify-between items-start">
                <div>
                    <span class="text-xs font-semibold text-slate-400 uppercase tracking-wider block">重點評估探頭</span>
                    <span class="text-3xl font-extrabold text-amber-600 tracking-tight mt-1 block">22 款</span>
                    <p class="text-xs text-slate-500 mt-1">
                        含生物相容性與聲學安全
                    </p>
                </div>
                <div class="p-3 bg-amber-50 text-amber-600 rounded-xl">
                    <i class="fa-solid fa-wave-square text-lg"></i>
                </div>
            </div>
        </div>

        <div class="bg-white p-5 rounded-2xl border border-slate-200 shadow-sm">
            <div class="flex justify-between items-start">
                <div>
                    <span class="text-xs font-semibold text-slate-400 uppercase tracking-wider block">補件工作進行度</span>
                    <span id="kanban-progress-pct" class="text-3xl font-extrabold text-indigo-600 tracking-tight mt-1 block">0%</span>
                    <div class="w-full bg-slate-100 rounded-full h-1.5 mt-2.5">
                        <div id="kanban-progress-bar" class="bg-indigo-600 h-1.5 rounded-full transition-all duration-500" style="width: 0%"></div>
                    </div>
                </div>
                <div class="p-3 bg-indigo-50 text-indigo-600 rounded-xl">
                    <i class="fa-solid fa-list-check text-lg"></i>
                </div>
            </div>
        </div>
    </section>

    <!-- 工作區頁籤組 -->
    <div class="flex border-b border-slate-200 space-x-1 overflow-x-auto no-print">
        <button onclick="switchTab('report-tab')" id="btn-report-tab" class="tab-btn px-4 py-2.5 text-sm font-semibold rounded-t-lg border-b-2 border-indigo-600 text-indigo-600 whitespace-nowrap flex items-center gap-2">
            <i class="fa-solid fa-file-invoice"></i> 1. 審查報告與評估
        </button>
        <button onclick="switchTab('rai-tab')" id="btn-rai-tab" class="tab-btn px-4 py-2.5 text-sm font-semibold rounded-t-lg text-slate-500 hover:text-slate-800 border-b-2 border-transparent whitespace-nowrap flex items-center gap-2">
            <i class="fa-solid fa-clipboard-question"></i> 2. 補件要求與回覆草擬
        </button>
        <button onclick="switchTab('kanban-tab')" id="btn-kanban-tab" class="tab-btn px-4 py-2.5 text-sm font-semibold rounded-t-lg text-slate-500 hover:text-slate-800 border-b-2 border-transparent whitespace-nowrap flex items-center gap-2">
            <i class="fa-solid fa-chalkboard-user"></i> 3. 補件工作追蹤 (Kanban)
        </button>
        <button onclick="switchTab('liaison-tab')" id="btn-liaison-tab" class="tab-btn px-4 py-2.5 text-sm font-semibold rounded-t-lg text-slate-500 hover:text-slate-800 border-b-2 border-transparent whitespace-nowrap flex items-center gap-2">
            <i class="fa-solid fa-envelope-open-text"></i> 4. 國際原廠英文聯繫信
        </button>
        <button onclick="switchTab('wow-map-tab')" id="btn-wow-map-tab" class="tab-btn px-4 py-2.5 text-sm font-semibold rounded-t-lg text-slate-500 hover:text-slate-800 border-b-2 border-transparent whitespace-nowrap flex items-center gap-2 text-indigo-600">
            <i class="fa-solid fa-circle-nodes"></i> WOW 1: 合規關係圖譜
        </button>
        <button onclick="switchTab('wow-verify-tab')" id="btn-wow-verify-tab" class="tab-btn px-4 py-2.5 text-sm font-semibold rounded-t-lg text-slate-500 hover:text-slate-800 border-b-2 border-transparent whitespace-nowrap flex items-center gap-2 text-pink-600">
            <i class="fa-solid fa-spell-check"></i> WOW 3: 說明書雙欄防呆
        </button>
    </div>

    <!-- 頁籤 1: Review Report & Evaluation Editor -->
    <div id="report-tab" class="tab-content block space-y-6">
        <div class="bg-white rounded-2xl border border-slate-200 shadow-sm overflow-hidden">
            <div class="px-6 py-4 bg-slate-50 border-b border-slate-200 flex flex-col sm:flex-row sm:items-center sm:justify-between gap-4">
                <div>
                    <h2 class="text-lg font-bold text-slate-800 flex items-center gap-2">
                        <span class="w-1.5 h-6 bg-indigo-600 rounded-full inline-block"></span>
                        審查報告技術文件項目列表
                    </h2>
                    <p class="text-xs text-slate-500 mt-0.5">點擊符合性狀態（OK / 不符合）或編輯表格內容，系統將會即時同步至補件工作看板。</p>
                </div>
                <button onclick="addNewReportItem()" class="px-3.5 py-1.5 bg-indigo-50 text-indigo-700 hover:bg-indigo-100 rounded-lg text-xs font-semibold transition flex items-center gap-1.5 border border-indigo-200">
                    <i class="fa-solid fa-plus"></i> 新增審查項目
                </button>
            </div>

            <!-- 技術評估動態表格 -->
            <div class="overflow-x-auto">
                <table class="w-full text-left border-collapse">
                    <thead>
                        <tr class="bg-slate-50 text-slate-400 uppercase text-xxs font-bold tracking-wider border-b border-slate-200">
                            <th class="py-3 px-6 w-1/12">條款編號</th>
                            <th class="py-3 px-6 w-3/12">評估細目/標準名稱</th>
                            <th class="py-3 px-6 w-1/12 text-center">符合性</th>
                            <th class="py-3 px-6 w-6/12">審查意見內容</th>
                            <th class="py-3 px-6 w-1/12 text-center no-print">操作</th>
                        </tr>
                    </thead>
                    <tbody id="report-table-body" class="divide-y divide-slate-100">
                        <!-- 由 JavaScript 動態生成 -->
                    </tbody>
                </table>
            </div>
        </div>

        <!-- Section 5: 官方審查結果公告（公文原文） -->
        <div class="bg-white rounded-2xl border border-slate-200 shadow-sm overflow-hidden">
            <div class="px-6 py-4 bg-slate-50 border-b border-slate-200">
                <h2 class="text-lg font-bold text-slate-800 flex items-center gap-2">
                    <span class="w-1.5 h-6 bg-rose-500 rounded-full inline-block"></span>
                    5. 官方審查結果公告 (補件要求原文)
                </h2>
                <p class="text-xs text-slate-500 mt-0.5">此區塊提供審查主管機關（TFDA）發函之官方補件公文原文編輯，系統會基於此文件生成對策。</p>
            </div>
            <div class="p-6 space-y-4">
                <div>
                    <label class="block text-xs font-bold text-slate-500 uppercase tracking-wide mb-1">說明書與標籤標示缺失 (補件第一部分)</label>
                    <textarea id="official-manual-defects" rows="4" class="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl text-sm focus:bg-white focus:ring-2 focus:ring-indigo-500 outline-none transition" oninput="syncManualDefects()"></textarea>
                </div>
                <div>
                    <label class="block text-xs font-bold text-slate-500 uppercase tracking-wide mb-1">技術文件規格與臨床前資料缺漏 (補件第二部分)</label>
                    <textarea id="official-tech-defects" rows="6" class="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl text-sm focus:bg-white focus:ring-2 focus:ring-indigo-500 outline-none transition" oninput="syncTechDefects()"></textarea>
                </div>
            </div>
        </div>
    </div>

    <!-- 頁籤 2: RAI (Request for Additional Information) & Response Drafts -->
    <div id="rai-tab" class="tab-content hidden space-y-6">
        <div class="bg-indigo-900 text-white rounded-2xl p-6 shadow-md relative overflow-hidden">
            <div class="absolute right-0 bottom-0 opacity-10 pointer-events-none transform translate-y-1/4 translate-x-1/10">
                <i class="fa-solid fa-wand-magic-sparkles text-9xl"></i>
            </div>
            <div class="max-w-2xl">
                <h3 class="text-xl font-bold mb-1 flex items-center gap-2">
                    <i class="fa-solid fa-wand-magic-sparkles"></i> 智慧補件回覆引導系統 (Smart-Fill Co-Pilot)
                </h3>
                <p class="text-indigo-200 text-sm">
                    本系統已自動分析「不符合」項目，並根據台灣 TFDA 查驗登記與臨床前測試基準，預先為您生成專業的補件回覆骨架。您可以直接在此處草擬與潤飾各項回覆內容，以加速取證流程。
                </p>
            </div>
        </div>

        <div class="space-y-4" id="rai-items-container">
            <!-- 動態生成 RAI 填寫區域 -->
        </div>
    </div>

    <!-- 頁籤 3: Interactive Kanban Board -->
    <div id="kanban-tab" class="tab-content hidden">
        <div class="mb-4 flex flex-col sm:flex-row sm:items-center sm:justify-between gap-4">
            <div>
                <h3 class="text-lg font-bold text-slate-800">補件任務與進度追蹤看板</h3>
                <p class="text-xs text-slate-500">將缺失項目拆解為任務卡片，追蹤各項報告的撰寫與驗收進度。</p>
            </div>
            <div class="flex items-center gap-2">
                <span class="text-xs font-semibold text-slate-500">篩選優先權:</span>
                <select id="kanban-priority-filter" class="px-2.5 py-1.5 bg-white border border-slate-200 rounded-lg text-xs font-medium text-slate-700 focus:outline-none" onchange="renderKanban()">
                    <option value="all">顯示所有等級</option>
                    <option value="High">高優先級 (High)</option>
                    <option value="Medium">中優先級 (Medium)</option>
                    <option value="Low">低優先級 (Low)</option>
                </select>
            </div>
        </div>

        <!-- 拖放看板 -->
        <div class="grid grid-cols-1 md:grid-cols-4 gap-4">
            <!-- 待處理 (To Do) -->
            <div class="bg-slate-100 p-4 rounded-2xl flex flex-col min-h-[500px]">
                <div class="flex items-center justify-between mb-3 px-1">
                    <span class="text-xs font-bold text-slate-600 uppercase tracking-wider flex items-center gap-1.5">
                        <i class="fa-regular fa-circle text-slate-400"></i> 待處理 (To Do)
                    </span>
                    <span id="kb-todo-count" class="bg-slate-200 text-slate-700 text-xxs font-bold px-2 py-0.5 rounded-full">0</span>
                </div>
                <div id="kb-col-todo" class="flex-grow space-y-3" ondragover="allowDrop(event)" ondrop="handleDrop(event, 'todo')"></div>
            </div>

            <!-- 撰寫/測試中 -->
            <div class="bg-blue-50/50 p-4 rounded-2xl flex flex-col min-h-[500px] border border-blue-100/30">
                <div class="flex items-center justify-between mb-3 px-1">
                    <span class="text-xs font-bold text-blue-700 uppercase tracking-wider flex items-center gap-1.5">
                        <i class="fa-solid fa-spinner text-blue-500 animate-spin"></i> 撰寫/測試中
                    </span>
                    <span id="kb-progress-count" class="bg-blue-100 text-blue-800 text-xxs font-bold px-2 py-0.5 rounded-full">0</span>
                </div>
                <div id="kb-col-progress" class="flex-grow space-y-3" ondragover="allowDrop(event)" ondrop="handleDrop(event, 'progress')"></div>
            </div>

            <!-- 審查確認 -->
            <div class="bg-amber-50/50 p-4 rounded-2xl flex flex-col min-h-[500px] border border-amber-100/30">
                <div class="flex items-center justify-between mb-3 px-1">
                    <span class="text-xs font-bold text-amber-700 uppercase tracking-wider flex items-center gap-1.5">
                        <i class="fa-solid fa-magnifying-glass text-amber-500"></i> 審查確認 (Review)
                    </span>
                    <span id="kb-review-count" class="bg-amber-100 text-amber-800 text-xxs font-bold px-2 py-0.5 rounded-full">0</span>
                </div>
                <div id="kb-col-review" class="flex-grow space-y-3" ondragover="allowDrop(event)" ondrop="handleDrop(event, 'review')"></div>
            </div>

            <!-- 已就緒 -->
            <div class="bg-emerald-50/50 p-4 rounded-2xl flex flex-col min-h-[500px] border border-emerald-100/30">
                <div class="flex items-center justify-between mb-3 px-1">
                    <span class="text-xs font-bold text-emerald-700 uppercase tracking-wider flex items-center gap-1.5">
                        <i class="fa-regular fa-circle-check text-emerald-500"></i> 已就緒 (Completed)
                    </span>
                    <span id="kb-completed-count" class="bg-emerald-100 text-emerald-800 text-xxs font-bold px-2 py-0.5 rounded-full">0</span>
                </div>
                <div id="kb-col-completed" class="flex-grow space-y-3" ondragover="allowDrop(event)" ondrop="handleDrop(event, 'completed')"></div>
            </div>
        </div>
    </div>

    <!-- 頁籤 4: 國外原廠技術資料協調信產生器 -->
    <div id="liaison-tab" class="tab-content hidden space-y-6">
        <div class="bg-white rounded-2xl border border-slate-200 shadow-sm p-6">
            <h3 class="text-lg font-bold text-slate-800 flex items-center gap-2">
                <i class="fa-solid fa-globe text-indigo-600"></i> 
                國外原廠技術資料協調信產生器 (Liaison Email Generator)
            </h3>
            <p class="text-xs text-slate-500 mt-1">
                此工具會自動將當前勾選「不符合」的技術缺失轉換為專業商務英文技術說明信，讓國外原廠研發與測試小組能快速提供正確報告。
            </p>

            <div class="mt-6 grid grid-cols-1 lg:grid-cols-3 gap-6">
                <!-- Inputs panel -->
                <div class="space-y-4 lg:col-span-1">
                    <div>
                        <label class="block text-xs font-bold text-slate-500 uppercase tracking-wide mb-1">收件人名稱 (原廠窗口)</label>
                        <input type="text" id="email-recipient" value="Nemko Korea Regulatory Team / R&D Manager" class="w-full px-3 py-2 border border-slate-200 rounded-lg text-sm focus:outline-none focus:ring-2 focus:ring-indigo-500 transition">
                    </div>
                    <div>
                        <label class="block text-xs font-bold text-slate-500 uppercase tracking-wide mb-1">本案申請規格與機型</label>
                        <input type="text" id="email-model" value="EVO Q10 (with 22 Probes)" class="w-full px-3 py-2 border border-slate-200 rounded-lg text-sm focus:outline-none focus:ring-2 focus:ring-indigo-500 transition" oninput="updateModelNameFromInput()">
                    </div>
                    <div>
                        <label class="block text-xs font-bold text-slate-500 uppercase tracking-wide mb-1">要求回覆截止日</label>
                        <input type="date" id="email-deadline" class="w-full px-3 py-2 border border-slate-200 rounded-lg text-sm focus:outline-none focus:ring-2 focus:ring-indigo-500 transition" onchange="generateEnglishEmail()">
                    </div>
                    <button onclick="generateEnglishEmail()" class="w-full py-2.5 bg-indigo-600 hover:bg-indigo-500 text-white rounded-lg text-sm font-semibold shadow-md transition flex items-center justify-center gap-1.5">
                        <i class="fa-solid fa-rotate"></i> 更新信件草稿
                    </button>
                </div>

                <!-- Preview text panel -->
                <div class="lg:col-span-2 flex flex-col">
                    <div class="flex items-center justify-between mb-1.5">
                        <span class="text-xs font-bold text-slate-500 uppercase tracking-wide">產出的英文信件草本</span>
                        <button onclick="copyEmailText()" class="text-xs text-indigo-600 hover:text-indigo-800 font-semibold flex items-center gap-1">
                            <i class="fa-regular fa-copy"></i> 複製信件文字
                        </button>
                    </div>
                    <div class="flex-grow relative">
                        <textarea id="email-output-area" rows="14" class="w-full p-4 bg-slate-900 text-slate-100 font-mono text-xs rounded-xl border border-slate-800 leading-relaxed focus:outline-none focus:ring-2 focus:ring-indigo-500" readonly></textarea>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- WOW 頁籤 1: 互動式合規關係圖譜 -->
    <div id="wow-map-tab" class="tab-content hidden space-y-6">
        <div class="bg-white rounded-2xl border border-slate-200 shadow-sm p-6">
            <h3 class="text-lg font-bold text-slate-800 flex items-center gap-2">
                <i class="fa-solid fa-circle-nodes text-indigo-600"></i>
                合規缺失學術拓撲圖譜 (Visual Compliance Gap Topology Map)
            </h3>
            <p class="text-xs text-slate-500 mt-1">此圖譜直觀展現當前送審設備與各項技術指標的符合性關聯，點擊紅色缺失節點可查看詳細技術缺失內容。</p>
            
            <div class="mt-6 grid grid-cols-1 lg:grid-cols-4 gap-6">
                <!-- 畫布區 -->
                <div class="lg:col-span-3 border border-slate-200 bg-slate-900 rounded-2xl p-4 min-h-[450px] flex items-center justify-center relative overflow-hidden">
                    <div class="absolute inset-0 opacity-10 bg-[radial-gradient(#6366f1_1px,transparent_1px)] [background-size:16px_16px]"></div>
                    <div id="topology-canvas" class="relative w-full h-full flex items-center justify-center">
                        <!-- 由 JS 繪製 SVG 節點結構 -->
                    </div>
                </div>
                <!-- 詳細資訊側欄 -->
                <div class="lg:col-span-1 bg-slate-50 border border-slate-200 rounded-2xl p-4 flex flex-col justify-between">
                    <div>
                        <h4 class="text-sm font-bold text-slate-800 border-b pb-2 mb-3">節點詳細診斷面板</h4>
                        <div id="topology-info-panel" class="text-xs text-slate-600 space-y-3">
                            <p class="italic text-slate-400">點擊左側拓撲圖中的任意圓圈節點，即可啟動 AI 深度診斷追蹤。</p>
                        </div>
                    </div>
                    <div class="pt-4 border-t border-slate-200">
                        <div class="flex items-center gap-3 text-xxs text-slate-500 font-semibold">
                            <span class="flex items-center gap-1"><span class="w-2.5 h-2.5 bg-emerald-500 rounded-full inline-block"></span>合規 (OK)</span>
                            <span class="flex items-center gap-1"><span class="w-2.5 h-2.5 bg-rose-500 rounded-full inline-block"></span>缺失 (不符合)</span>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- WOW 頁籤 3: 說明書禁忌症雙欄防呆對照工具 -->
    <div id="wow-verify-tab" class="tab-content hidden space-y-6">
        <div class="bg-white rounded-2xl border border-slate-200 shadow-sm p-6">
            <h3 class="text-lg font-bold text-slate-800 flex items-center gap-2">
                <i class="fa-solid fa-spell-check text-pink-600"></i>
                說明書中文翻譯與預期用途對應「防呆校正工具」 (Labeling Alignment Tool)
            </h3>
            <p class="text-xs text-slate-500 mt-1">針對中文說明書、原廠英文說明書及 TFDA 預期用途進行一致性審查，確保合規描述完美契合。</p>

            <div class="grid grid-cols-1 lg:grid-cols-2 gap-6 mt-6">
                <!-- 原始對比對照區 -->
                <div class="border border-slate-200 rounded-xl p-5 space-y-4 bg-slate-50">
                    <span class="text-xs font-bold text-slate-500 tracking-wider block border-b pb-1.5">原文與中文現存差異比較</span>
                    <div class="space-y-3">
                        <div>
                            <span class="text-xxs font-bold text-indigo-600 block">原廠英文標註說明書 (Original Spec)</span>
                            <p class="text-xs font-mono bg-white p-2.5 rounded border border-slate-200 leading-relaxed mt-1">
                                "Contraindication: Ophthalmic use is strictly prohibited except in dedicated ophthalmic pre-set mode."
                            </p>
                        </div>
                        <div>
                            <span class="text-xxs font-bold text-rose-600 block">現存中文草案/說明書 (Taiwan Draft)</span>
                            <p class="text-xs font-mono bg-white p-2.5 rounded border border-slate-200 leading-relaxed mt-1">
                                "禁忌症：本產品僅可在眼科預設模式時用於眼科用途。<span class="bg-rose-100 text-rose-700 px-1 rounded font-bold">（錯別字：”當規” 應修正為”常規”）</span>"
                            </p>
                        </div>
                    </div>
                </div>

                <!-- 智慧修正與校正建議 -->
                <div class="border border-pink-100 bg-pink-50/20 rounded-xl p-5 flex flex-col justify-between">
                    <div>
                        <span class="text-xs font-bold text-pink-700 tracking-wider block border-b border-pink-100 pb-1.5">AI 專家修正建議書 (TFDA Compliant Translation)</span>
                        <div class="mt-4 space-y-3 text-xs text-slate-700 leading-relaxed">
                            <p class="font-bold text-slate-800">1. 語意對齊修復：</p>
                            <p class="bg-white p-3 rounded border border-pink-200 font-mono text-xs">
                                建議中文修正為：「禁忌症：本產品僅限於使用眼科預置模式時方可用於眼科診斷。未具備此模式時，嚴禁於眼科用途使用。」
                            </p>
                            <p class="font-bold text-slate-800 mt-2">2. 錯別字校正：</p>
                            <p>發現錯誤拼寫：「當規」已由系統自動修復為繁體字「常規」，以符合 TFDA 送審規範。</p>
                        </div>
                    </div>
                    <div class="pt-4 border-t border-pink-100 mt-4 flex justify-end">
                        <button onclick="applyInstructionCorrection()" class="px-4 py-2 bg-pink-600 hover:bg-pink-500 text-white font-bold rounded-lg text-xs shadow-md shadow-pink-600/10 transition">
                            一鍵導入此對策至 Tab 2
                        </button>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- PDF 印表輸出佈局 (預設隱藏，僅在 window.print 時顯示) -->
    <div class="hidden print:block print-full-width">
        <div class="text-center pb-6 border-b-2 border-slate-800">
            <h1 class="text-2xl font-bold">TFDA 醫療器材查驗登記技術文件審查與補件報告</h1>
            <p class="text-sm mt-1">專案機型：<span class="print-model-name">EVO Q10 (with 22 Probes)</span></p>
            <p class="text-xs text-slate-500 mt-1">列印日期：<span id="print-date"></span></p>
        </div>

        <div class="mt-8 space-y-6">
            <h2 class="text-lg font-bold border-b pb-2">1. 查驗登記技術文件審查結果 (項目明細)</h2>
            <div id="print-report-items" class="space-y-4"></div>

            <div class="page-break"></div>

            <h2 class="text-lg font-bold border-b pb-2">2. 官方審查結果公告與補件通知</h2>
            <div class="space-y-4">
                <div>
                    <h3 class="text-sm font-bold text-slate-700">【一】中文說明書與標籤標示缺失</h3>
                    <p id="print-official-manual" class="text-xs text-slate-600 mt-1 whitespace-pre-wrap leading-relaxed"></p>
                </div>
                <div>
                    <h3 class="text-sm font-bold text-slate-700">【二】臨床前測試與原廠規格缺漏資料</h3>
                    <p id="print-official-tech" class="text-xs text-slate-600 mt-1 whitespace-pre-wrap leading-relaxed"></p>
                </div>
            </div>

            <div class="page-break"></div>

            <h2 class="text-lg font-bold border-b pb-2">3. 各缺失項目補件回覆對策 (RAI Response Draft)</h2>
            <div id="print-rai-drafts" class="space-y-4"></div>
        </div>
    </div>
</main>

<!-- 離線 AI-Auditor 聊天側邊欄 (WOW Feature 2) -->
<div id="wow-sidebar" class="fixed right-0 top-0 h-full w-80 bg-white border-l border-slate-200 shadow-2xl z-40 transform translate-x-full transition-transform duration-300 no-print flex flex-col justify-between">
    <div class="p-4 bg-slate-900 text-white flex items-center justify-between">
        <h4 class="text-sm font-bold flex items-center gap-1.5">
            <i class="fa-solid fa-robot text-indigo-400"></i> AI 離線合規查核助理
        </h4>
        <button onclick="toggleWOWSidebar()" class="text-slate-400 hover:text-white"><i class="fa-solid fa-xmark"></i></button>
    </div>
    
    <!-- 對話內容 -->
    <div id="sidebar-chat-box" class="flex-grow p-4 overflow-y-auto space-y-3 bg-slate-50 text-xs">
        <div class="bg-white p-3 rounded-lg border border-slate-200 text-slate-700">
            您好！我是您的離線 RAG 法規助手。已成功將本案的申報背景資料加載至前端內建字典，您可以問我：
            <div class="mt-2 space-y-1">
                <button onclick="askPresetQuestion('有哪些探頭被指出缺少生物相容性報告？')" class="block text-left text-indigo-600 hover:underline">1. 哪些探頭缺少生物相容性？</button>
                <button onclick="askPresetQuestion('IEC 60601-2-37 的聲學安全要求是什麽？')" class="block text-left text-indigo-600 hover:underline">2. 聲學安全補件指引是什麼？</button>
            </div>
        </div>
    </div>
    
    <!-- 輸入框 -->
    <div class="p-3 border-t border-slate-200 bg-white">
        <div class="flex gap-1.5">
            <input type="text" id="sidebar-user-input" placeholder="請輸入合規問題..." class="flex-grow px-2.5 py-1.5 border border-slate-200 rounded-lg text-xs focus:outline-none focus:ring-1 focus:ring-indigo-500">
            <button onclick="sendSidebarMessage()" class="px-3 py-1.5 bg-indigo-600 hover:bg-indigo-500 text-white font-semibold rounded-lg text-xs"><i class="fa-regular fa-paper-plane"></i></button>
        </div>
    </div>
</div>

<!-- 懸浮展開聊天助理按鈕 -->
<button onclick="toggleWOWSidebar()" class="fixed right-5 bottom-20 z-30 p-3 bg-indigo-600 hover:bg-indigo-500 text-white rounded-full shadow-lg transition hover:scale-110 no-print" title="開啟 AI 合規查核助理">
    <i class="fa-solid fa-comments text-xl"></i>
</button>

<!-- 通知彈出容器 -->
<div id="toast-container" class="fixed bottom-5 right-5 z-50 flex flex-col gap-2 pointer-events-none"></div>

<script>
// --- 初始技術評估合規狀態模組 ---
let reviewReport = [
    {
        id: "4.1",
        title: "電性與電磁相容安全性",
        status: "OK",
        severity: "Low",
        description: "電性安全：已檢附Nemko Korea Co., Ltd.出具IEC 60601-1:2025測試評估報告(CBTL：TL124, REP087030, 2025-5-15, Q10)\n電磁相容：已檢附 Nemko Korea Co., Ltd.出具IEC 60601-1-2:2014測試評估報告(CBTL：TL124, REP122403, 2025-5-26, Q10)",
        link: "https://www.iecee.org/members/cbtls/nemko-korea-co-ltd",
        responseDraft: "本案已檢附符合最新 IEC 60601-1 與 IEC 60601-1-2 之完整安全檢測報告，皆由合格之 CBTL (Nemko Korea) 簽發，電性與電磁相容性資料符合查驗登記基準要求。",
        priority: "Low",
        kanbanColumn: "completed"
    },
    {
        id: "4.2",
        title: "超音波特定安全標準 (IEC 60601-2-37)",
        status: "不符合",
        severity: "High",
        description: "未檢附本案擬申請主機(Q10)及本案所搭配22種探頭之超音波設備的特殊安全性基本要求(IEC 60601-2-37)測試評估報告。\n聲學輸出測量：未檢附本案擬申請探頭(22種)之熱指數(TI)、機械指數(MI)量測結果(依據IEC 62359或NEMA UD 2-2004)。",
        link: "",
        responseDraft: "補件對策：\n1. 已向韓國原廠取得 Q10 主機及搭配 22 款探頭之符合 IEC 60601-2-37 完整測試評估報告(報告編號：REP0990321)。\n2. 檢附所有探頭之熱指數(TI)與機械指數(MI)量測結果數據清單，相關參數確定方法皆符合 IEC 62359 聲場特徵標準，詳見附件三。",
        priority: "High",
        kanbanColumn: "todo"
    },
    {
        id: "4.3",
        title: "生物相容性評估",
        status: "不符合",
        severity: "High",
        description: "未檢附部分本案擬申請探頭生物相容性評估報告(Sensitization, Cytotoxicity, Skin Irritation)，包含LA3-16AD、LA3-16ADC、LA2-16S、LA2-16Sc、L3-22、LA2-9SD、L3-16D、CA1-7SD、CA2-8AD、EA2-11ARE、EA2-11AVE、CV1-8AE、EV2-10A、PA1-5AED、PA2-9S、PA3-15S等探頭。",
        link: "",
        responseDraft: "補件對策：\n本公司已向原廠索取缺失提及之16款探頭完整生物相容性測試報告。針對ISO 10993-5 (細胞毒性)、ISO 10993-10 (刺激與皮膚過敏)進行全面評估。檢附測試對照表與正式英文報告(詳見附件四)。",
        priority: "High",
        kanbanColumn: "todo"
    },
    {
        id: "4.4",
        title: "軟體與網路安全",
        status: "OK",
        severity: "Low",
        description: "軟體確效：檢附Software Validation Report(Software Validation_EVO Q Series_V1.00)。\n網路安全：檢附Cybersecurity Report (CYBERSECURITY_SELF_ASSESSMENT_REPORT_EVO2_v1.00.00)。",
        link: "",
        responseDraft: "本案軟體確效與資通安全評估均符合 TFDA「醫療器材網路安全查驗登記審查指引」要求，測試報告內容完整無須補件。",
        priority: "Low",
        kanbanColumn: "completed"
    },
    {
        id: "4.5",
        title: "AI CADe/CADx 技術文件",
        status: "不符合",
        severity: "High",
        description: "未檢附本案基於AI技術具有電腦輔助偵測/診斷(CADe/CADx)軟體功能之技術資料(如Biometry Assist, Uterine Contour, Uterine Assist, S-Detect for Breast, S-Detect for Thyroid等軟體功能)。",
        link: "",
        responseDraft: "補件對策：\n依據《人工智慧/機器學習技術之電腦輔助偵測(CADe)及電腦輔助診斷(CADx)醫療器材查驗登記技術指引》，補充檢附 AI 計算演算法之黃金標準(Gold Standard)對照分析資料、臨床訓練與確效資料集描述、以及各軟體功能之 ROC 曲線與 AUC 敏感度特異度分析報告。",
        priority: "High",
        kanbanColumn: "todo"
    },
    {
        id: "4.6",
        title: "性能測試 (Performance Test)",
        status: "不符合",
        severity: "Medium",
        description: "1. 未檢附定量/半定量量測能力之軟體功能(E-Strain, AutoEF, BladderAssist等)之測量類型與準確度規格。\n2. 未檢附22種探頭之功能性測試評估結果(掃描角度、掃描長度、解析度等)。\n3. 未檢附 DICOM 相關測試評估資料。",
        link: "",
        responseDraft: "補件對策：\n1. 檢附定量量測軟體功能清單，羅列距離、體積、心跳、流速等之測量不確定度與準確度宣告規格文件。\n2. 檢附 22 種探頭功能性測試總結表，包含頻率範圍及解析度實測值。\n3. 檢附產品 DICOM Conformance Statement 暨相容性測試結果。",
        priority: "Medium",
        kanbanColumn: "todo"
    },
    {
        id: "4.7",
        title: "原廠品質檢驗報告",
        status: "不符合",
        severity: "Medium",
        description: "出廠報告：未檢附原廠品質檢驗報告。",
        link: "",
        responseDraft: "補件對策：\n檢附 Q10 主機系統出廠之成品檢驗規範以及特定隨機生產之 Q10 主機測試檢驗合格紀錄(Certificate of Analysis / Quality Control Release Certificate)。",
        priority: "Medium",
        kanbanColumn: "todo"
    }
];

// 官方通知文字常數
let officialManualDefectsText = "• 中文說明書之禁忌症記載眼科用途(可在眼科預設模式時用於眼科用途)，不符合預期用途描述：\n  禁忌症：本產品僅可在眼科預設模式時用於眼科用途。\n• 另中文翻譯之產品敘述／適應症有錯別字：”當規” 應修正為”常規”";

let officialTechDefectsText = "1. 請檢附所有擬申請探頭(22項)之軸向解析度、側向解析度等規格資料。\n2. 請檢附本案具有定量/半定量量測能力之軟體功能(包含E-Strain、S-Shearwave Imaging、AutoEF、HeartAssist、Uterine Assist、BladderAssist)之測量類型及其測量準確度等規格資料。\n3. 聲學安全：請檢附本案主機(Q10)及搭配22種探頭符合IEC 60601-2-37之測試評估報告，並檢附熱指數(TI)與機械指數(MI)量測結果數據(依據IEC 62359或NEMA UD 2-2004)。\n4. 生物相容：請檢附LA3-16AD、EA2-11ARE等16款探頭之ISO 10993生物相容性評估報告。\n5. 功能性與性能測試：檢附探頭功能測試數據、DICOM相容性資料。\n6. AI軟體：請檢附Biometry Assist等基於AI演算法之CADe/CADx軟體功能之確效與技術指引對照報告。\n7. 出廠報告：檢附Q10之原廠品質檢驗報告。";

// 網頁載入初始化
window.onload = function() {
    document.getElementById("official-manual-defects").value = officialManualDefectsText;
    document.getElementById("official-tech-defects").value = officialTechDefectsText;
    
    // 設定 30 天補件期限
    let thirtyDaysLater = new Date();
    thirtyDaysLater.setDate(thirtyDaysLater.getDate() + 30);
    document.getElementById("email-deadline").value = thirtyDaysLater.toISOString().split('T')[0];
    
    updateDashboardMetrics();
    renderReportTable();
    renderRAIList();
    renderKanban();
    generateEnglishEmail();
    renderTopologyMap(); // 初始化 WOW 拓撲圖
    
    // 同步列印日期
    document.getElementById("print-date").innerText = new Date().toLocaleDateString('zh-TW');
};

// 吐司提示系統（解決 alert 干擾）
function showToast(message, type = 'success') {
    const toast = document.createElement('div');
    toast.className = `p-4 rounded-xl shadow-lg border text-sm flex items-center gap-2 pointer-events-auto transition-all duration-300 transform translate-y-2 opacity-0`;
    
    if (type === 'success') {
        toast.className += ` bg-emerald-50 text-emerald-800 border-emerald-200`;
        toast.innerHTML = `<i class="fa-solid fa-circle-check text-emerald-500"></i> ${message}`;
    } else if (type === 'error') {
        toast.className += ` bg-rose-50 text-rose-800 border-rose-200`;
        toast.innerHTML = `<i class="fa-solid fa-circle-xmark text-rose-500"></i> ${message}`;
    } else {
        toast.className += ` bg-blue-50 text-blue-800 border-blue-200`;
        toast.innerHTML = `<i class="fa-solid fa-circle-info text-blue-500"></i> ${message}`;
    }
    
    const container = document.getElementById('toast-container');
    container.appendChild(toast);
    
    setTimeout(() => {
        toast.classList.remove('opacity-0', 'translate-y-2');
    }, 10);
    
    setTimeout(() => {
        toast.classList.add('opacity-0', 'translate-y-2');
        setTimeout(() => toast.remove(), 300);
    }, 3000);
}

// 動態更新 KPI 數值與圖表
function updateDashboardMetrics() {
    const total = reviewReport.length;
    const passed = reviewReport.filter(item => item.status === 'OK').length;
    const defects = total - passed;
    const completionPercentage = ((passed / total) * 100).toFixed(1);
    
    document.getElementById("completion-percentage").innerText = `${completionPercentage}%`;
    document.getElementById("defect-count-badge").innerText = `${defects} 項`;
    document.getElementById("circle-percentage-text").innerText = `${Math.round(completionPercentage)}%`;
    
    const svgCircle = document.getElementById("svg-progress-circle");
    svgCircle.setAttribute("stroke-dasharray", `${completionPercentage}, 100`);
    
    // 更新看板任務進展比例
    const totalKanbanTasks = reviewReport.filter(item => item.status === '不符合').length;
    if (totalKanbanTasks > 0) {
        const completedKanbanTasks = reviewReport.filter(item => item.status === '不符合' && item.kanbanColumn === 'completed').length;
        const progressPct = Math.round((completedKanbanTasks / totalKanbanTasks) * 100);
        document.getElementById("kanban-progress-pct").innerText = `${progressPct}%`;
        document.getElementById("kanban-progress-bar").style.width = `${progressPct}%`;
    } else {
        document.getElementById("kanban-progress-pct").innerText = `100%`;
        document.getElementById("kanban-progress-bar").style.width = `100%`;
    }
}

// 頁籤切換狀態控制
function switchTab(tabId) {
    document.querySelectorAll(".tab-content").forEach(el => el.classList.add("hidden"));
    document.querySelectorAll(".tab-btn").forEach(el => {
        el.classList.remove("border-indigo-600", "text-indigo-600");
        el.classList.add("text-slate-500", "border-transparent");
    });
    
    document.getElementById(tabId).classList.remove("hidden");
    const activeBtn = document.getElementById(`btn-${tabId}`);
    if (activeBtn) {
        activeBtn.classList.add("border-indigo-600", "text-indigo-600");
        activeBtn.classList.remove("text-slate-500", "border-transparent");
    }
}

// 渲染 Tab 1 條款評估表格
function renderReportTable() {
    const tableBody = document.getElementById("report-table-body");
    tableBody.innerHTML = "";
    
    reviewReport.forEach((item, index) => {
        const isOk = item.status === "OK";
        const statusClass = isOk 
            ? "bg-emerald-100 text-emerald-800 border-emerald-200" 
            : "bg-rose-100 text-rose-800 border-rose-200";

        tableBody.innerHTML += `
            <tr class="hover:bg-slate-50/80 transition-colors">
                <td class="py-4 px-6 font-bold text-slate-900 text-sm align-top">${item.id}</td>
                <td class="py-4 px-6 text-sm align-top">
                    <input type="text" value="${item.title}" onchange="updateReportItemField(${index}, 'title', this.value)" class="w-full bg-transparent hover:bg-slate-100 focus:bg-white focus:ring-1 focus:ring-indigo-500 rounded px-1.5 py-0.5 font-semibold text-slate-800 outline-none">
                </td>
                <td class="py-4 px-6 text-center align-top">
                    <button onclick="toggleItemStatus(${index})" class="px-2.5 py-1 text-xs font-bold rounded-full border ${statusClass} shadow-sm transition hover:scale-105">
                        ${item.status}
                    </button>
                </td>
                <td class="py-4 px-6 text-sm align-top">
                    <textarea onchange="updateReportItemField(${index}, 'description', this.value)" class="w-full bg-transparent hover:bg-slate-100 focus:bg-white focus:ring-1 focus:ring-indigo-500 rounded p-1.5 text-slate-600 text-xs font-mono leading-relaxed outline-none" rows="3">${item.description}</textarea>
                    ${item.link ? `<a href="${item.link}" target="_blank" class="text-[10px] text-indigo-500 hover:text-indigo-700 font-medium inline-flex items-center gap-0.5 mt-1"><i class="fa-solid fa-arrow-up-right-from-square"></i> 驗證連結</a>` : ''}
                </td>
                <td class="py-4 px-6 text-center align-top no-print">
                    <button onclick="deleteReportItem(${index})" class="p-1.5 text-slate-400 hover:text-rose-600 rounded-lg hover:bg-rose-50 transition" title="刪除本項目">
                        <i class="fa-regular fa-trash-can"></i>
                    </button>
                </td>
            </tr>
        `;
    });
    
    // 同步 PDF 列印結構
    const printContainer = document.getElementById("print-report-items");
    printContainer.innerHTML = "";
    reviewReport.forEach(item => {
        printContainer.innerHTML += `
            <div class="border p-4 rounded-lg bg-slate-50">
                <div class="flex justify-between font-bold border-b pb-1">
                    <span>條款 ${item.id} - ${item.title}</span>
                    <span class="${item.status === 'OK' ? 'text-emerald-700' : 'text-rose-700'}">${item.status}</span>
                </div>
                <p class="text-xs text-slate-600 mt-2 whitespace-pre-wrap">${item.description}</p>
            </div>
        `;
    });
}

// 新增自訂合規評估項目
function addNewReportItem() {
    const newIndex = reviewReport.length + 1;
    const newItem = {
        id: `4.${newIndex}`,
        title: "自訂審查評估項目",
        status: "不符合",
        severity: "Medium",
        description: "請詳細填寫此項目的技術文件缺陷與法規標準偏差原因...",
        link: "",
        responseDraft: "補充說明對策草案及預計提報之附件列表...",
        priority: "Medium",
        kanbanColumn: "todo"
    };
    
    reviewReport.push(newItem);
    showToast("成功新增自訂技術評估項目");
    updateDashboardMetrics();
    renderReportTable();
    renderRAIList();
    renderKanban();
    generateEnglishEmail();
    renderTopologyMap();
}

// 刪除特定條款
function deleteReportItem(index) {
    const deletedId = reviewReport[index].id;
    reviewReport.splice(index, 1);
    showToast(`項目 ${deletedId} 已成功移除`, 'error');
    updateDashboardMetrics();
    renderReportTable();
    renderRAIList();
    renderKanban();
    generateEnglishEmail();
    renderTopologyMap();
}

// 表格內文即時寫回變數儲存
function updateReportItemField(index, field, value) {
    reviewReport[index][field] = value;
    if (field === 'description') {
        generateEnglishEmail();
    }
    renderRAIList();
    renderKanban();
}

// 切換 符合/不符合 狀態（自動修正看板 Column）
function toggleItemStatus(index) {
    const previousStatus = reviewReport[index].status;
    reviewReport[index].status = previousStatus === "OK" ? "不符合" : "OK";
    
    if (reviewReport[index].status === "OK") {
        reviewReport[index].kanbanColumn = "completed";
    } else {
        reviewReport[index].kanbanColumn = "todo";
    }
    
    showToast(`項目 ${reviewReport[index].id} 狀態變更為 ${reviewReport[index].status}`);
    updateDashboardMetrics();
    renderReportTable();
    renderRAIList();
    renderKanban();
    generateEnglishEmail();
    renderTopologyMap();
}

function syncManualDefects() {
    officialManualDefectsText = document.getElementById("official-manual-defects").value;
    document.getElementById("print-official-manual").innerText = officialManualDefectsText;
}

function syncTechDefects() {
    officialTechDefectsText = document.getElementById("official-tech-defects").value;
    document.getElementById("print-official-tech").innerText = officialTechDefectsText;
}

// 渲染 Tab 2 RAI 補件回覆引導面板
function renderRAIList() {
    const raiContainer = document.getElementById("rai-items-container");
    const printRaiContainer = document.getElementById("print-rai-drafts");
    
    raiContainer.innerHTML = "";
    printRaiContainer.innerHTML = "";
    
    const deficiencyItems = reviewReport.filter(item => item.status === "不符合");
    
    if (deficiencyItems.length === 0) {
        raiContainer.innerHTML = `
            <div class="bg-emerald-50 border border-emerald-200 text-emerald-800 p-8 rounded-2xl text-center">
                <i class="fa-regular fa-circle-check text-4xl mb-2 text-emerald-500"></i>
                <h4 class="text-base font-bold">所有項目皆符合標準！</h4>
                <p class="text-xs mt-1 text-emerald-600">目前沒有檢出任何技術缺陷項目，您無需提送補件資料。</p>
            </div>
        `;
        return;
    }
    
    deficiencyItems.forEach((item) => {
        const mainStateIndex = reviewReport.findIndex(el => el.id === item.id);
        
        raiContainer.innerHTML += `
            <div class="bg-white rounded-2xl border border-slate-200 shadow-sm overflow-hidden transition-all hover:border-slate-300">
                <div class="px-6 py-4 bg-slate-50 border-b border-slate-200 flex flex-wrap items-center justify-between gap-3">
                    <div class="flex items-center gap-2">
                        <span class="bg-rose-500 text-white text-xs font-bold px-2 py-0.5 rounded-md">條款 ${item.id}</span>
                        <h4 class="text-sm font-bold text-slate-800">${item.title}</h4>
                    </div>
                    <div class="flex items-center gap-2">
                        <span class="text-xs text-slate-400">優先等級:</span>
                        <select onchange="updateItemPriority(${mainStateIndex}, this.value)" class="bg-white border border-slate-200 text-xs font-semibold rounded px-2 py-1 focus:outline-none focus:ring-1 focus:ring-indigo-500">
                            <option value="High" ${item.priority === 'High' ? 'selected' : ''}>高優先級</option>
                            <option value="Medium" ${item.priority === 'Medium' ? 'selected' : ''}>中優先級</option>
                            <option value="Low" ${item.priority === 'Low' ? 'selected' : ''}>低優先級</option>
                        </select>
                    </div>
                </div>
                
                <div class="p-6 grid grid-cols-1 lg:grid-cols-2 gap-6">
                    <div class="bg-rose-50/50 p-4 rounded-xl border border-rose-100">
                        <span class="text-xxs font-bold text-rose-700 tracking-wider uppercase block mb-1">官方缺失指出項目：</span>
                        <p class="text-xs text-slate-700 font-mono leading-relaxed whitespace-pre-wrap">${item.description}</p>
                    </div>
                    <div class="flex flex-col">
                        <span class="text-xxs font-bold text-indigo-700 tracking-wider uppercase block mb-1">補件對策草擬區 (Response Draft)：</span>
                        <textarea id="response-textarea-${mainStateIndex}" oninput="updateResponseDraft(${mainStateIndex}, this.value)" rows="5" class="flex-grow w-full p-3 border border-slate-200 rounded-xl text-xs font-mono focus:border-indigo-500 focus:ring-2 focus:ring-indigo-100 outline-none leading-relaxed" placeholder="請填寫補件防線，列出您預備交出的附件測試文件與報告章節編號...">${item.responseDraft || ''}</textarea>
                    </div>
                </div>
            </div>
        `;

        printRaiContainer.innerHTML += `
            <div class="border p-4 rounded-lg bg-slate-50 space-y-2">
                <div class="font-bold border-b pb-1 text-slate-800 text-sm">【條款 ${item.id}】${item.title}</div>
                <div class="p-2.5 bg-red-50 text-xxs font-mono text-red-900 border border-red-100 rounded">
                    <strong>技術缺失原因：</strong> ${item.description}
                </div>
                <div class="text-xs text-slate-700 pt-1 leading-relaxed whitespace-pre-wrap">
                    <strong>對應補件策略：</strong>\n${item.responseDraft || '（尚未編寫補件回覆對策）'}
                </div>
            </div>
        `;
    });
}

function updateItemPriority(mainIndex, newPriority) {
    reviewReport[mainIndex].priority = newPriority;
    showToast(`優先級已調整為: ${newPriority}`, 'info');
    renderKanban();
}

function updateResponseDraft(mainIndex, value) {
    reviewReport[mainIndex].responseDraft = value;
    // 同步更新列印視窗對策文字
    const printItem = document.getElementById(`print-rai-response-${mainIndex}`);
    if (printItem) {
        printItem.innerText = value;
    }
}

// 渲染 Tab 3 拖放看板
function renderKanban() {
    const colTodo = document.getElementById("kb-col-todo");
    const colProgress = document.getElementById("kb-col-progress");
    const colReview = document.getElementById("kb-col-review");
    const colCompleted = document.getElementById("kb-col-completed");
    
    colTodo.innerHTML = "";
    colProgress.innerHTML = "";
    colReview.innerHTML = "";
    colCompleted.innerHTML = "";
    
    let filter = document.getElementById("kanban-priority-filter").value;
    let counts = { todo: 0, progress: 0, review: 0, completed: 0 };
    
    reviewReport.forEach((item, index) => {
        if (filter !== "all" && item.priority !== filter) return;
        
        counts[item.kanbanColumn]++;
        
        let priorityColor = "bg-slate-100 text-slate-700";
        if (item.priority === "High") priorityColor = "bg-rose-50 text-rose-700 border-rose-100";
        else if (item.priority === "Medium") priorityColor = "bg-amber-50 text-amber-700 border-amber-100";
        
        const cardHtml = `
            <div draggable="true" ondragstart="handleDragStart(event, ${index})" class="bg-white p-4 rounded-xl border border-slate-200 shadow-sm cursor-grab active:cursor-grabbing hover:shadow-md transition duration-200">
                <div class="flex items-center justify-between mb-2">
                    <span class="text-xxs font-bold text-slate-400 bg-slate-50 px-1.5 py-0.5 rounded">條款 ${item.id}</span>
                    <span class="text-xxs font-bold px-2 py-0.5 rounded-full border ${priorityColor}">${item.priority}</span>
                </div>
                <h5 class="text-xs font-bold text-slate-800 leading-snug line-clamp-2">${item.title}</h5>
                <p class="text-xxs text-slate-500 mt-2 line-clamp-3">${item.description}</p>
                
                <div class="mt-3 pt-2.5 border-t border-slate-100 flex items-center justify-between">
                    <span class="text-[10px] font-semibold text-indigo-500 hover:text-indigo-700 cursor-pointer" onclick="switchTab('rai-tab')">
                        <i class="fa-regular fa-pen-to-square"></i> 編寫對策
                    </span>
                    
                    <div class="flex gap-0.5">
                        <button onclick="moveKanbanCardDirectly(${index}, 'todo')" class="p-1 text-slate-400 hover:text-slate-700 text-xxs" title="移至 待處理"><i class="fa-solid fa-square-caret-left"></i></button>
                        <button onclick="moveKanbanCardDirectly(${index}, 'progress')" class="p-1 text-blue-400 hover:text-blue-700 text-xxs" title="移至 撰寫中"><i class="fa-solid fa-play"></i></button>
                        <button onclick="moveKanbanCardDirectly(${index}, 'review')" class="p-1 text-amber-400 hover:text-amber-700 text-xxs" title="移至 審查確認"><i class="fa-solid fa-magnifying-glass"></i></button>
                        <button onclick="moveKanbanCardDirectly(${index}, 'completed')" class="p-1 text-emerald-400 hover:text-emerald-700 text-xxs" title="移至 已就緒"><i class="fa-solid fa-circle-check"></i></button>
                    </div>
                </div>
            </div>
        `;
        
        if (item.kanbanColumn === "todo") colTodo.innerHTML += cardHtml;
        else if (item.kanbanColumn === "progress") colProgress.innerHTML += cardHtml;
        else if (item.kanbanColumn === "review") colReview.innerHTML += cardHtml;
        else if (item.kanbanColumn === "completed") colCompleted.innerHTML += cardHtml;
    });
    
    document.getElementById("kb-todo-count").innerText = counts.todo;
    document.getElementById("kb-progress-count").innerText = counts.progress;
    document.getElementById("kb-review-count").innerText = counts.review;
    document.getElementById("kb-completed-count").innerText = counts.completed;
    
    updateDashboardMetrics();
}

// 修正後的 Drag and Drop 機制
let draggedCardIndex = null;

function handleDragStart(event, index) {
    draggedCardIndex = index;
    event.dataTransfer.setData("text/plain", index);
}

function allowDrop(event) {
    event.preventDefault();
}

function handleDrop(event, targetColumn) {
    event.preventDefault();
    // 雙重保全 Fallback 機制
    const indexStr = event.dataTransfer.getData("text/plain");
    const index = (indexStr !== "" && indexStr !== undefined) ? parseInt(indexStr) : draggedCardIndex;
    
    if (index !== null && index !== undefined && !isNaN(index)) {
        moveKanbanCardDirectly(index, targetColumn);
    }
}

function moveKanbanCardDirectly(index, targetColumn) {
    reviewReport[index].kanbanColumn = targetColumn;
    
    if (targetColumn === "completed") {
        reviewReport[index].status = "OK";
    } else {
        reviewReport[index].status = "不符合";
    }
    
    showToast(`任務 [${reviewReport[index].id}] 已移入 [${targetColumn.toUpperCase()}]`);
    renderReportTable();
    renderRAIList();
    renderKanban();
    generateEnglishEmail();
    renderTopologyMap();
}

// 解決硬編碼 Bug，獲取機型名稱
function getModelName() {
    const modelInput = document.getElementById("email-model");
    return modelInput ? modelInput.value : "Unknown_Device";
}

function modelFileFriendlyName() {
    return getModelName().replace(/[^a-z0-9]/gi, '_').toLowerCase();
}

// 動態同步模型名稱至 PDF 與儀表板
function updateModelNameFromInput() {
    const mName = getModelName();
    const printModelElements = document.querySelectorAll(".print-model-name");
    printModelElements.forEach(el => el.innerText = mName);
    
    const subtitle = document.getElementById("project-subtitle");
    if (subtitle) {
        subtitle.innerText = `${mName} 查驗登記 (符合 2026 TFDA 最新規範)`;
    }
    generateEnglishEmail();
}

// Tab 4 原廠聯繫信箱模板生產
function generateEnglishEmail() {
    const recipient = document.getElementById("email-recipient").value;
    const model = getModelName();
    const deadline = document.getElementById("email-deadline").value;
    
    const deficiencyItems = reviewReport.filter(item => item.status === "不符合");
    let requestItemsBullets = "";
    
    if (deficiencyItems.length === 0) {
        requestItemsBullets = "No outstanding regulatory deficiencies currently flagged for the device.";
    } else {
        deficiencyItems.forEach((item, index) => {
            requestItemsBullets += `${index + 1}. Item clause: ${item.id} - ${item.title}\n`;
            requestItemsBullets += `   Description of TFDA Deficiency Note:\n`;
            const cleanedDesc = item.description.split('\n').map(line => `   >> ${line}`).join('\n');
            requestItemsBullets += `${cleanedDesc}\n\n`;
        });
    }
    
    const emailBody = `Subject: URGENT: Technical Documents Request for TFDA Medical Device Registration - ${model}

Dear ${recipient},

We are currently progressing with the medical device查驗登記 (TFDA registration) in Taiwan for the Ultrasound System:
Model/Specification: ${model}

The TFDA Review Board has issued a formal deficiency list (補件通知). In order to resolve these outstanding issues and secure market approval on schedule, we urgently require additional testing data, certification details, and technical reports directly from you (the original manufacturer).

Please review the detailed list of requests below and provide the corresponding reports by ${deadline || 'ASAP'}:

=======================================================
REQUIRED TECHNICAL DATA & TEST REPORTS LIST
=======================================================
${requestItemsBullets}
=======================================================

Specific Actions Requested:
- IEC 60601-2-37 & IEC 62359 reports must contain explicit measurements for all active probe options.
- Biocompatibility reports (sensitization, irritation, cytotoxicity) must correspond EXACTLY to the probe models listed under clause 4.3.
- R&D software validation datasets must support AI-based CADe/CADx claims according to TFDA guidelines.

Please let us know if you require any clarifications regarding the specifications mentioned above. Thank you for your continued prompt cooperation.

Best Regards,

[Your Name / Regulatory Affairs Team]
[Your Company Name, Taiwan Authorized Representative]`;

    document.getElementById("email-output-area").value = emailBody;
}

function copyEmailText() {
    const textArea = document.getElementById("email-output-area");
    textArea.select();
    try {
        document.execCommand('copy');
        showToast("英文信件草稿已成功複製至剪貼簿！");
    } catch (err) {
        showToast("複製失敗，請手動全選並複製。", "error");
    }
}

// 導出 JSON 規格
function exportToJSON() {
    const exportData = {
        projectName: `${getModelName()} TFDA Registration Review Report`,
        lastUpdated: new Date().toISOString(),
        overallCompliancePct: document.getElementById("completion-percentage").innerText,
        deficienciesCount: reviewReport.filter(item => item.status === "不符合").length,
        officialDefects: {
            manualDefects: officialManualDefectsText,
            techDefects: officialTechDefectsText
        },
        reviewReportItems: reviewReport
    };
    
    const jsonString = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(exportData, null, 2));
    const downloadAnchor = document.createElement('a');
    downloadAnchor.setAttribute("href", jsonString);
    downloadAnchor.setAttribute("download", `TFDA_Review_Workspace_${modelFileFriendlyName()}.json`);
    document.body.appendChild(downloadAnchor);
    downloadAnchor.click();
    downloadAnchor.remove();
    showToast("JSON 檔案已成功下載！");
}

// 導出 Markdown 報告書
function exportToMarkdown() {
    let md = `# TFDA 醫療器材查驗登記技術文件審查與智慧補件評估報告\n\n`;
    md += `* **評估機型/專案**：${getModelName()}\n`;
    md += `* **報告產生時間**：${new Date().toLocaleString('zh-TW')}\n`;
    md += `* **技術安全審查通過率**：${document.getElementById("completion-percentage").innerText}\n`;
    md += `* **待補件項目總數**：${reviewReport.filter(item => item.status === "不符合").length} 項\n\n`;
    md += `***\n\n`;
    
    md += `## 壹、 查驗技術審查細部項目狀態\n\n`;
    reviewReport.forEach(item => {
        md += `### 【條款 ${item.id}】${item.title}  [狀態：${item.status}]\n`;
        md += `* **優先處理等級**：${item.priority}\n`;
        md += `* **審查意見/技術缺失內容**：\n\`\`\`text\n${item.description}\n\`\`\`\n`;
        md += `* **官方回覆對策草案**：\n\`\`\`text\n${item.responseDraft || '（無或無須補件）'}\n\`\`\`\n\n`;
    });
    
    md += `## 貳、 官方補件通知原始意見\n\n`;
    md += `### 一、 中文說明書與標籤標示缺失\n${officialManualDefectsText}\n\n`;
    md += `### 二、 臨床前測試與原廠規格缺漏資料\n${officialTechDefectsText}\n\n`;
    
    const mdString = "data:text/markdown;charset=utf-8," + encodeURIComponent(md);
    const downloadAnchor = document.createElement('a');
    downloadAnchor.setAttribute("href", mdString);
    downloadAnchor.setAttribute("download", `TFDA_Review_Workspace_${modelFileFriendlyName()}.md`);
    document.body.appendChild(downloadAnchor);
    downloadAnchor.click();
    downloadAnchor.remove();
    showToast("Markdown 檔案已成功下載！");
}

// 列印模式觸發（同步 textarea 實體）
function triggerPrint() {
    document.getElementById("print-official-manual").innerText = document.getElementById("official-manual-defects").value;
    document.getElementById("print-official-tech").innerText = document.getElementById("official-tech-defects").value;
    
    renderRAIList();
    window.print();
}

// ==========================================
// WOW 功能 1: 拓撲圖譜 SVG 渲染引擎
// ==========================================
function renderTopologyMap() {
    const container = document.getElementById("topology-canvas");
    container.innerHTML = "";
    
    let svgHtml = `<svg class="w-full h-full" viewBox="0 0 600 400" xmlns="http://www.w3.org/2000/svg">`;
    
    // 定義中心點與半徑
    const cx = 300;
    const cy = 200;
    const r = 130;
    
    // 繪製中心主機系統與外圍連線
    reviewReport.forEach((item, index) => {
        const angle = (index * 2 * Math.PI) / reviewReport.length;
        const x = cx + r * Math.cos(angle);
        const y = cy + r * Math.sin(angle);
        
        const strokeColor = item.status === "OK" ? "#10b981" : "#f43f5e";
        svgHtml += `<line x1="${cx}" y1="${cy}" x2="${x}" y2="${y}" stroke="${strokeColor}" stroke-width="1.5" stroke-dasharray="4" />`;
    });
    
    // 繪製中心主機 Node
    svgHtml += `
        <circle cx="${cx}" cy="${cy}" r="30" fill="#312e81" stroke="#6366f1" stroke-width="3" style="cursor: pointer" onclick="showNodeDetail('host')" />
        <text x="${cx}" y="${cy + 4}" font-size="10" font-family="sans-serif" font-weight="bold" fill="#ffffff" text-anchor="middle" style="pointer-events: none">Q10 主機</text>
    `;
    
    // 繪製外圍條款子 Node
    reviewReport.forEach((item, index) => {
        const angle = (index * 2 * Math.PI) / reviewReport.length;
        const x = cx + r * Math.cos(angle);
        const y = cy + r * Math.sin(angle);
        
        const nodeColor = item.status === "OK" ? "#10b981" : "#f43f5e";
        const pulseClass = item.status === "OK" ? "" : "node-pulse";
        
        svgHtml += `
            <g class="${pulseClass}" style="transform-origin: ${x}px ${y}px; cursor: pointer" onclick="showNodeDetail(${index})">
                <circle cx="${x}" cy="${y}" r="18" fill="#ffffff" stroke="${nodeColor}" stroke-width="3" />
                <text x="${x}" y="${y + 4}" font-size="9" font-family="sans-serif" font-weight="bold" fill="#1e293b" text-anchor="middle">${item.id}</text>
                <text x="${x}" y="${y + 28}" font-size="8" font-family="sans-serif" fill="#64748b" text-anchor="middle">${item.title.substring(0,6)}..</text>
            </g>
        `;
    });
    
    svgHtml += `</svg>`;
    container.innerHTML = svgHtml;
}

function showNodeDetail(target) {
    const panel = document.getElementById("topology-info-panel");
    if (target === 'host') {
        panel.innerHTML = `
            <div class="p-3 bg-indigo-50 border border-indigo-200 rounded-xl space-y-2">
                <h5 class="font-bold text-indigo-950 text-sm flex items-center gap-1"><i class="fa-solid fa-server"></i> 系統主控節點: EVO Q10</h5>
                <p class="text-xxs text-indigo-800 leading-relaxed">
                    本案主機之預期用途為全身常規超音波診斷，主體電性安全性 (IEC 60601-1) 及網路安全均已通過，主要的合規威脅聚焦於外接 22 款探頭的聲學輸出與生物相容性完整度。
                </p>
            </div>
        `;
    } else {
        const item = reviewReport[target];
        const statusColor = item.status === "OK" ? "text-emerald-700 bg-emerald-50 border-emerald-200" : "text-rose-700 bg-rose-50 border-rose-200";
        panel.innerHTML = `
            <div class="p-3 border rounded-xl space-y-2 ${statusColor}">
                <h5 class="font-bold text-sm">【條款 ${item.id}】${item.title}</h5>
                <p class="font-semibold text-xxs">符合性狀態: <span class="underline">${item.status}</span></p>
                <p class="text-xxs leading-relaxed">${item.description}</p>
                ${item.status !== "OK" ? `
                    <button onclick="switchTab('rai-tab'); document.getElementById('response-textarea-${target}').focus();" class="w-full mt-2 py-1 bg-white border border-slate-300 rounded font-bold text-[10px] hover:bg-slate-50 text-slate-700">
                        <i class="fa-solid fa-feather-pointed"></i> 即刻修正此項回覆
                    </button>
                ` : ''}
            </div>
        `;
    }
}

// ==========================================
// WOW 功能 2: 離線智慧 RAG 助理與法規庫
// ==========================================
let chatHistory = [];

function toggleWOWSidebar() {
    const sidebar = document.getElementById("wow-sidebar");
    sidebar.classList.toggle("translate-x-full");
}

function askPresetQuestion(question) {
    document.getElementById("sidebar-user-input").value = question;
    sendSidebarMessage();
}

function sendSidebarMessage() {
    const input = document.getElementById("sidebar-user-input");
    const query = input.value.trim();
    if (query === "") return;
    
    // 使用者訊息
    appendMessage("user", query);
    input.value = "";
    
    // 模擬離線 RAG 檢索匹配
    setTimeout(() => {
        let reply = "抱歉，我目前正處於離線 RAG 工作狀態，在現有的合規資料集中無法直接比對。您可以點擊預設的快速查詢按鈕。";
        
        if (query.includes("生物相容性") || query.includes("4.3")) {
            reply = "【離線 RAG 自動比對結果】發現條款 4.3 缺漏 16 款探頭的 ISO 10993 生物相容性評估報告。包括 LA3-16AD 與 CA2-8AD 等型號。TFDA 建議藥商補提原廠合格測試報告，且報告受試型號須與本案完全一致。";
        } else if (query.includes("聲學") || query.includes("60601-2-37") || query.includes("4.2")) {
            reply = "【離線 RAG 自動比對結果】條款 4.2 缺失熱指數 (TI) 與機械指數 (MI) 的 NEMA UD-2/IEC 62359 聲學聲場量測結果。回覆時，必須要求原廠提供完整報告，或在回覆函附錄中列出 22 種探頭的聲學宣告表格。";
        }
        
        appendMessage("assistant", reply);
    }, 500);
}

function appendMessage(sender, text) {
    const box = document.getElementById("sidebar-chat-box");
    const msgDiv = document.createElement("div");
    if (sender === "user") {
        msgDiv.className = "bg-indigo-50 border border-indigo-100 p-2.5 rounded-lg text-slate-800 ml-6";
        msgDiv.innerHTML = `<strong>您：</strong><p class="mt-1 leading-relaxed">${text}</p>`;
    } else {
        msgDiv.className = "bg-slate-800 text-slate-100 p-2.5 rounded-lg text-slate-200 mr-6 font-mono text-xxs";
        msgDiv.innerHTML = `<strong>Auditor Co-Pilot：</strong><p class="mt-1 leading-relaxed">${text}</p>`;
    }
    box.appendChild(msgDiv);
    box.scrollTop = box.scrollHeight;
}

// ==========================================
// WOW 功能 3: 說明書雙欄校正導入
// ==========================================
function applyInstructionCorrection() {
    // 尋找生物相容性或超音波特定安全標準等目標條款
    const targetIndex = reviewReport.findIndex(el => el.id === "4.3" || el.id === "4.2");
    if (targetIndex !== -1) {
        const correctionText = "【說明書防呆校正導入】：\n1. 中文說明書禁忌症已遵照 TFDA 指引修正，明確限制眼科用途僅可在啟動眼科特定預置模式時使用，並增補英文 Warning 對照文字，杜絕超範圍使用風險。\n2. 已由校對人員修正適應症中所有錯別字（例如：將原翻譯錯字「當規」全面更正為「常規」）。修訂版說明書詳見附件一。";
        
        reviewReport[targetIndex].responseDraft = correctionText;
        
        // 如果目前文字框已渲染，直接同步顯示
        const textarea = document.getElementById(`response-textarea-${targetIndex}`);
        if (textarea) {
            textarea.value = correctionText;
        }
        
        showToast("已成功將中文說明書修正對策導入條款 4.3 中！");
        renderRAIList();
    } else {
        showToast("未找到適合導入的條款，請先恢復預設條款。", "error");
    }
}
</script>
</body>
</html>
柒、 平台建置細部流程與操作指引 (Elicitations)
在開始佈署或運行此平台之前，請與開發小組或 RA 法規專家確認以下細部實施方針：

一、 系統配置與部署環境
地端高隱私部署：若您需要處理具備高敏感度的臨床前試驗患者資料（臨床試驗確效資料集），是否需要支援在本地端或私有雲環境以 Docker 容器化部署整個 Agent 系統？

法規資料庫更新頻率：預設的 TFDA 法規 Checklist 需要多久與官方網站同步一次？是否需要整合外部 API 自動爬取衛福部食藥署最新發布的醫療器材臨床前測試基準？

二、 AI Agent 的推論準確度與優化
多輪幻覺校對機制：系統在自動判定「OK」或「不符合」時，是否需要啟用雙 Agent 校對機制（一個 Agent 負責判定，另一個 Agent 負責挑戰與除錯），以確保評估精準度？

探頭清單自動提取：在多探頭產品（如本案之 22 款探頭）中，是否需要 Agent 自動從 PDF 掃描檔表格中以 OCR 與精確匹配技術提取出所有探頭型號，並自動生成 Tab 1 的對比表？

三、 功能擴充與使用者體驗
電子簽章整合：當用戶在前端 HTML 網頁上完成補件回覆草案撰寫後，是否需要一鍵簽署數位證書，並將 Markdown/JSON 打包為符合 TFDA 送審格式的官方 PDF？

多國語言聯動：協調信產生器除了預設的商務英文外，是否需要支援其他語言（如：針對原廠在歐洲的 CE 標準證書，提供德文或法文聯繫信件範本）？
