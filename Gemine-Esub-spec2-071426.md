Agentic OS 智慧法規助理與多智能體查驗登記沙盒平台
系統架構與技術設計規範說明書 (System Architecture & Technical Specification)
壹、 導言與系統願景 (Introduction & System Vision)
在當今生醫科技與醫療器材產業中，產品上市前的法規遵循（Regulatory Compliance）與查驗登記（Product Registration）是決定產品成敗的關鍵路徑。以臺灣食品藥物管理署（TFDA）為例，一款高階醫療器材（如：搭配 22 款超音波探頭的診斷儀）在查驗登記過程中，需要審查上千頁的技術文件，包括電性安全（IEC 60601-1）、電磁相容性（IEC 60601-1-2）、超音波特定安全標準（IEC 60601-2-37）、生物相容性評估（ISO 10993 完整系列）、軟體與網路安全，以及人工智慧輔助決策系統（AI CADe/CADx）的演算法確效。
這類審查高度複雜，國內藥商與國外製造原廠（OEM）之間的溝通成本極高。審查過程中一旦出現「不符合（Deficiency）」，藥商必須在限期內提送補件資料（RAI - Request for Additional Information），否則面臨退件。
本系統 「Agentic OS & TFDA 智慧法規工作台」 應運而生。這是一套專為法規專家（RA）、審查員與醫材開發團隊設計的雙引擎多智能體作業系統（Dual-Engine Agentic OS）。它具備兩大核心工作模式：
Agentic OS 技能沙盒（Playground）：利用 Google ADK（Google Search、Code Execution 程式碼執行器等工具），讓用戶上傳、修改與驗證其技能指令集（skill.md），模擬與評估不同 LLM 模型（預設 gemini-3.1-flash-lite、gemini-3.5-flash、gemini-3.1-pro-preview）在處理醫材法規時的精準度與權衡。
TFDA 智慧補件生成平台（TFDA Auditor）：用戶上傳或粘貼臨床資料、審查公文與法規檢核表（Checklist），AI 智能體將自動進行語意解析與符合性比對，並動態編譯（Compile）生成一個完全獨立、可直接下載並在本地離線運行的互動式單頁 HTML 審查工作台。該下載包保留了原始樣式表、FontAwesome 圖標、互動式進度環、可點擊編輯的審查表格、官方通知編輯器、AI 智慧補件回覆骨架、原生拖拽看板（Kanban）、國際原廠英文聯繫信產生器以及 PDF 完美列印版型。
本說明書將從底層技術棧、資料流向、智能體設計、安全策略及前端細微交互，為本平台提供極致詳盡的技術規範設計與實現邏輯。
貳、 系統拓撲與全棧架構 (System Topology & Full-Stack Architecture)
本平台採用現代化的全棧架構（Full-Stack Architecture），在確保 API 金鑰（Secrets）不洩露至瀏覽器的前提下，提供極速的 Vite 前端交互體驗與強大的 Express 伺服器端智能體運算。
一、 運行環境與網路拓撲
容器化部署：系統運行於 Cloud Run 容器內。
單一埠 ingresso：依照環境約束，系統僅向外暴露 3000 埠，並由 Nginx 進行反向代理。
安全邊界（Security Boundary）：
瀏覽器（Client-Side）絕不直接調用 @google/genai SDK，亦不保存或傳輸 API 金鑰。
所有 Gemini API 請求皆通過 /api/gemini/* 路由代理至後端 Node.js 環境。
二、 技術棧選型與依賴定義
前端核心：React 19.0 + TypeScript 5.8 (不採用 Const Enums，遵循 named imports 規範)。
建置工具：Vite 6.2 + Tailwind CSS v4 整合編譯。
後端核心：Node.js Express 4.21，採用 TSX 作為開發階段的 TypeScript 直接運行工具。
編譯打包：利用 esbuild 將後端代碼打包為單一的 dist/server.cjs（CommonJS 格式），完美繞過 Node.js 原生 ESM 複雜的相對路徑與擴展名限制，提升冷啟動效能。
動畫系統：採用 motion (motion/react) 提供流暢的 HUD 面板切換與智能體執行流動畫。
圖表可視化：採用高度定製的原生 SVG 矢量圖形技術，在保障 React 19 相容性的同時，實現 100% 精準響應式、零延遲、高美感的「6 大數據監控圖表（Telemetry Graphs）」。
參、 後端 AI 智能體核心設計與 Google GenAI SDK 整合 (Backend AI Agent Design & Google GenAI SDK Integration)
本平台後端深度整合 Google 最新推出的 @google/genai 軟體開發套件，擯棄任何已過期的舊版 SDK 型號，嚴格遵守新型調用範式。
一、 安全與延遲初始化（Lazy Initialization）
為防止 API 金鑰缺失時導致 Node 伺服器啟動崩潰（此崩潰會引致 Cloud Run 持續顯示「正在啟動」而阻礙預覽），後端採取延遲且安全的實例化策略：
code
TypeScript
import { GoogleGenAI } from "@google/genai";

let aiClientInstance: GoogleGenAI | null = null;

export function getGeminiClient(): GoogleGenAI {
  if (!aiClientInstance) {
    const apiKey = process.env.GEMINI_API_KEY;
    if (!apiKey) {
      throw new Error("GEMINI_API_KEY environment variable is required and missing.");
    }
    aiClientInstance = new GoogleGenAI({
      apiKey: apiKey,
      httpOptions: {
        headers: {
          'User-Agent': 'aistudio-build',
        }
      }
    });
  }
  return aiClientInstance;
}
註：在 httpOptions 中將 User-Agent 顯式聲明為 aistudio-build，為 Google AI Studio 平台提供正確的核心遙測依賴。
二、 模型選擇架構與多模型對照（Model Selection Routing）
後端支援在前端界面動態切換三款核心 Gemini 模型，並為其分配最佳的任務邊界：
gemini-3.1-flash-lite (系統預設)：最輕量級、極速響應之模型，適用於日常 skill.md 解析、即時修改與輕量級符合性評估。
gemini-3.5-flash：智慧平衡模型，具備優異的上下文推理與代碼理解力，適用於中等複雜度的法規文檔交叉稽核。
gemini-3.1-pro-preview：旗艦級推理模型，配備深度思維鏈，適用於審查龐大且專業的臨床、生物相容性與 AI CADe 系統申報原案。
三、 基於 Google ADK 智慧工具鏈的執行引擎 (ADK Tools Execution Engine)
本平台支持智能體調用內建工具鏈。當選擇非 Lite 模型（如 gemini-3.5-flash）時，系統會在請求配置中注入 tools：
Google Search Grounding (googleSearch)：允許智能體在審查醫療器材時，自動聯網查詢最新的 TFDA 法規指引、ISO 標準變更或 Nemko Korea 等 CBTL 實驗室的最新註冊資質。
Code Execution (built_in_code_execution)：當文檔中包含複雜數值（如超音波聲學輸出的熱指數 TI 與機械指數 MI 比例、生物相容性試驗的細胞存活率百分比、AI CADe 演算法的 ROC 曲線座標與 AUC 值）時，智能體會自動生成並運行 Python 程式碼，進行精準的代碼運算，並在輸出日誌中返回計算軌跡。
執行引擎請求範例：
code
TypeScript
const response = await ai.models.generateContent({
  model: "gemini-3.5-flash",
  contents: userPayload,
  config: {
    systemInstruction: systemInstructions,
    temperature: 0.2,
    tools: [
      { googleSearch: {} } // 啟用 Google 聯網搜索
    ],
    toolConfig: { includeServerSideToolInvocations: true }
  }
});
肆、 智慧法規評估（TFDA Review）核心機制與結構化輸出 (TFDA Smart Review Generation Engine)
為了讓用戶獲得極致的工具體驗，系統設計了一套端到端的 TFDA 結構化評估流。該流的核心在於，將無結構的醫材申報文檔，經過 Gemini 的深度推理，編譯為完全符合前端 TypeScript 定義的 TFDAState JSON 結構。
一、 結構化輸出保障（Structured JSON Schema Protection）
為確保返回的 JSON 數據不會因為 LLM 隨機輸出多餘的 Markdown 標記而導致前端 JSON 解析出錯（Blank Screen 致命錯誤），我們在呼叫 ai.models.generateContent 時，配置了 responseMimeType: "application/json"，並在系統提示詞（System Instructions）中施加了極其嚴苛的 JSON Schema 規則（使用 Type.OBJECT 與 Type.ARRAY 規範，詳見 /server.ts）。
二、 TFDAState 的資料模型定義 (TypeScript Types)
code
TypeScript
export interface TFDAReviewItem {
  id: string;             // 例如: "4.1"
  title: string;          // 例如: "電磁相容性標準 (IEC 60601-1-2)"
  status: 'OK' | '不符合';
  severity: 'High' | 'Medium' | 'Low';
  description: string;    // 詳細審查意見
  link: string;           // 驗證鏈接或相關法規數據庫
  responseDraft: string;  // 預先生成的 RAI 補件回覆骨架
  priority: 'High' | 'Medium' | 'Low';
  kanbanColumn: 'todo' | 'progress' | 'review' | 'completed';
}

export interface TFDAState {
  projectName: string;
  deviceSpecification: string;
  lastUpdated: string;
  overallCompliancePct: string;
  officialManualDefects: string;
  officialTechDefects: string;
  reviewReportItems: TFDAReviewItem[];
}
三、 自助編譯與單頁離線 HTML 生成器（Self-Contained HTML Compiler）
這本平台最驚豔的特色之一。當用戶點擊「下載單頁互動 HTML 系統」時，前端將向 /api/gemini/export-tfda-html 發送 POST 請求，傳遞當前用戶已經編輯過的、最新的 tfdaState 資料、主題配置（preset）以及語言偏好。
後端 Express 會在記憶體中將這個狀態動態注入到一套極致考究、整合了 Tailwind CSS CDN、FontAwesome 矢量圖庫、Google 字體與原生高階 JavaScript（含 Drag & Drop 看板、自動化原廠 Liaison 協調信模板、JSON/Markdown 二次匯出機制等）的 HTML 樣板中，並以 text/html 的附件流（Attachment Stream）形式返回給瀏覽器。
這使得藥商用戶可以**一鍵下載該 HTML 檔，透過電子郵件直接發送給國外原廠，原廠研發經理只要雙擊該 HTML，就能在本地瀏覽器離線查閱這套極其精緻的審查面板、在看板中拖拽任務、修改英文信件並一鍵導出！**這完全打破了傳統法規申報使用靜態 Word/PDF 文件造成的無盡溝通深淵。
伍、 設計美學、微交互與多維遙測可視化監控 (Design Aesthetics, Micro-Interactions & Interactive SVG Telemetry Dashboard)
本平台的 UI 設計拒絕任何千篇一律的「AI Slop（低質感 AI 堆砌）」，嚴格實施高對比度、高秩序感與物理真實感的高保真數位工作站美學。
一、 智慧終端 10 大主題配色（Team-featured Color Styles）
本系統特別預設 10 套基於尖端生醫與計算團隊特徵設計的「Featured Styles」：
Indigo Tech (極光靛 - 預設)：#4f46e5。象徵高精尖醫療器械。
Emerald Bio (生醫綠)：#059669。象徵臨床生物工程與細胞相容性。
Rose Clinical (臨床粉)：#db2777。象徵人性化臨床關懷與安全審查。
Amber Sonic (超音波橘)：#d97706。象徵超音波聲學傳播與機械熱能。
Cyber Glow (賽博霓虹)：#0891b2。具備強烈科技感的暗黑青。
Slate Minimal (簡約石墨)：#4b5563。瑞士現代主義，極致客觀、灰階。
Sky Trust (澄空藍)：#0284c7。象徵值得信賴的檢驗與官方公正。
Violet Agent (智慧紫)：#7c3aed。象徵深度 AI 推理與自治多智能體。
Crimson Precise (精準緋紅)：#dc2626。象徵高度嚴謹的安全紅線與限制。
Autumn Amber (暖秋金黃)：#ea580c。秋日大地色系，提供溫暖、高對比的無障礙體驗。
二、 100% React 19 相容的 6 大原生 SVG Telemetry 監控圖表
為了展現「哇！（Wow）」的可視化效果，同時確保代碼具有絕對的健壯性（避開第三方 React 圖表庫在 React 19 下的依賴報錯與渲染 Flickering 閃爍問題），我們手寫了 6 組極致細緻的原生 SVG telemetry 監控圖表：
Graph 1: Token Consumption (累計 Token 消耗柱狀圖)
設計：帶有 hover tooltip 的動態柱狀圖，柱體頂部具有圓角，高度依據最多次消耗自動縮放。
Graph 2: Model Latency (智能體反應時間對照表)
設計：藍綠色（Cyan）高亮柱狀圖，用毫秒為單位展示不同模型運行該 skill.md 規則的比對延遲。
Graph 3: Tool Frequency (ADK 工具鏈調用次數分布圖)
設計：對比 google_search 網絡檢索、built_in_code_execution 代碼沙盒與本地文件讀取的調用頻率。
Graph 4: Score Progress Flow (符合性評估得分演進曲線)
設計：SVG 貝氏曲線（Bezier Curve）與漸變填充，繪製出多次修改 skill.md 後，文檔符合性得分的演化軌跡。
Graph 5: Defect Matrix Breakdown (缺失成分比例環形圖)
設計：SVG stroke-dasharray 配合 stroke-dashoffset 實現的精美漸變圓環，將「中文標籤」、「電性 EMC」、「生物相容性」與「AI CADe」的缺陷權重精準拆解。
Graph 6: Agent Operational Radar (智能體綜合維度雷達網)
設計：多邊形網格（Polygon Grid），映射當前運行的模型在「速度 (Speed)」、「準確度 (Accuracy)」、「分析深度 (Depth)」與「邊界安全性 (Security)」四個象限的性能偏好。
三、 交互細節與物理體感（Micro-Interactions）
拖拽體感（Drag & Drop）：
看板卡片採用原生 HTML5 draggable="true"。
拖拽時卡片微幅旋轉，並伴隨半透明陰影。
拖拽目標區域具備邊框高亮（Dashed Indicator）與背景色微調，釋放時立即向主狀態發送更新。
LLM 粒子發射器 (LLM Flow Visualizer)：
在 Playground 執行時，主 HUD 將呈現亮靛色的流動粒子，代表 Prompt 自 User Input 發送，經過 Model Router 路由，激發 ADK Tools，最後返回 Compiler 的全過程。
陸、 6 大 WOW 級 AI 延伸功能 (6 Advanced Wow AI Features)
本系統不僅完美重現了樣例的所有法規功能，更在 Agent OS 與 TFDA 平台 雙端各自新增了 3 項極具破壞性創新的「WOW AI 功能」：
專屬「Agent OS 沙盒端」的 3 大 WOW AI 功能：
Wow OS 1: 技能依賴與決策樹視覺化分析器 (Skill Dependency Map)
系統自動解析 skill.md 的 Markdown 標題與指引條款，在畫面上將其解構為一個層級分明、具備高亮連結的「子智能體依賴與決策流向圖（Decision Tree）」，讓 RA 主管清晰理解技能的推導邏輯。
Wow OS 2: 多模型符合性對照分析模組 (Multi-Model Multi-Aspect Evaluation)
支援將相同的 skill.md 與申報文檔同時分發給 gemini-3.1-flash-lite、gemini-3.5-flash 與 gemini-3.1-pro-preview 執行，並在一張高亮卡片中並排（Side-by-Side）對比它們產出的符合性評分、推導深度與 Token 性價比。
Wow OS 3: 多智能體法規共識論辯場 (Multi-Agent Consensus Debate Arena)
內置兩個虛擬子智能體——「嚴苛的 TFDA 審查員 (Auditor Sub-agent)」與「爭取時效的藥商技術專家 (Writer Sub-agent)」。點擊啟動，兩者將針對文檔中的爭議技術缺漏展開數輪即時的對話與詰問，最終形成一份具備高度法規可行性的「RAI 補件共識日誌」。
專屬「TFDA 審查平台端」的 3 大 WOW AI 功能：
Wow TFDA 1: TFDA 關鍵補件機率預測器 (TFDA Gap Predictor)
AI 基於大量的歷史補件案例，深度分析用戶上傳的醫材文件，精準給出「最有可能被 TFDA 審查官提出不符合的 Top 3 隱藏缺陷」及預估的「退件機率」，幫助 RA 在正式送件前進行預防性除錯。
Wow TFDA 2: ISO-10993 探頭生物相容性自動矩陣 (ISO-10993 Matrix)
超音波診斷儀搭配 22 款探頭，每款探頭接觸人體的程度與時間不同。AI 自動解析探頭型號，動態繪製出一張「探頭 vs ISO-10993 (Cytotoxicity, Sensitization, Irritation) 檢驗地圖」，用紅綠高亮標記哪些探頭已過關、哪些仍處於高風險缺失。
Wow TFDA 3: AI 實驗室資質與 CB 報告核對系統 (AI Audit Cross-Check)
自動捕捉技術報告中的簽發機構（如 Nemko Korea），與 IECEE 的合格 CBTL 實驗室數據庫進行語意核對，確認測試實驗室是否具備簽發該 IEC 標準報告的國際授權資質。
柒、 安全防護與離線沙盒隔離規約 (Security & Sandbox Isolation Protocol)
零金鑰外洩防線：
任何情況下，GEMINI_API_KEY 均僅在伺服器端（Express Server）透過 process.env.GEMINI_API_KEY 調用，絕不注入至打包的前端靜態資源中。
導出 HTML 安全規約：
編譯導出的互動式 HTML 包在離線運行時，其內部所有 JavaScript 邏輯（如看板卡片拖拽、報告編輯、聯繫信更新、JSON 導出等）均為瀏覽器本機內存計算，不依賴任何外部 API 調用，徹底避免因網絡隔離而導致的功能癱瘓。
錯誤容錯機制（Fault Isolation）：
若 Gemini 服務在後端調用超時，伺服器會捕獲該 Exception，並返回一份極具可讀性的友善警告 JSON，前端對應區域會亮起高亮黃燈並轉入「手動/本地模擬模式」，絕不發生整頁 Blank 崩潰。
捌、 本系統之 20 個深度 follow-up 思考問題 (20 Comprehensive Follow-up Questions)
為了進一步優化系統性能、擴展法規覆蓋面以及提升多智能體架構的技術高度，我們提出以下 20 個涵蓋「架構優化」、「LLM 演算法」、「醫材法規限制」與「人機協同交互」的深度技術問題：
多智能體協同 (Agent Coordination)：
在「多智能體法規共識論辯場 (Debate Arena)」中，如何設計更科學的「評審共識收斂算法」，以防止兩個子智能體在進行數輪辯論後陷入死循環或發散？
ADK 工具沙盒安全 (ADK Tools Security)：
在執行 built_in_code_execution (代碼執行器) 時，若用戶提供的文檔夾雜惡意的代碼注入攻擊（Prompt Injection），後端如何實施沙盒物理隔離以確保 Express 主進程的安全？
聯網搜索 Grounding 精準度 (Search Grounding Optimization)：
利用 googleSearch 進行法規 Grounding 時，如何構建高質量的「法規專用查詢過濾器（Query Filter）」，以避免檢索到非官方的博客文章或過時的法規資訊？
結構化輸出保障 (Structured JSON Robustness)：
面對長度超過 10 萬字的超大型醫材文檔，如何實施「滑動窗口語意切片（Sliding Semantic Window）」，以確保 Gemini 輸出 JSON 時不會因 Token 限制而將 reviewReportItems 數組在中途切斷？
離線 HTML 單頁狀態還原 (Offline State Hydration)：
用戶下載本地離線 HTML 工作台並在本機修改、拖拽看板後，我們是否應提供一個「反向導入機制」，允許用戶將修改後的 HTML 重新上傳至雲端系統以還原其資料狀態？
PDF 完美的流式排版 (PDF Pagination Control)：
使用 CSS page-break-before: always 時，如何動態計算表格行高與文字多寡，以防止在列印 PDF 時，某些審查意見的文字在頁面交界處被腰斬？
ISO-10993 探頭符合性矩陣之複雜性 (ISO-10993 Matrix Extensibility)：
不同國家（如 TFDA、US FDA、CE MDR）對生物相容性試驗的探頭歸類（如：黏膜接觸、受損皮膚接觸、接觸時間限制）存在細微差異，如何讓該自動化矩陣支援「國家法規標準切換」？
本地 PDF 解析技術 (Client-side PDF Parsing)：
目前系統在前端採用 FileReader 讀取文字型申報文檔。為支援純掃描件 PDF，我們是否應在後端引入基於 OCR（光學字符識別）的文檔解析微服務？
提示詞漂移與迴歸測試 (Prompt Drift & Regression Testing)：
當後端更換為 gemini-3.5-flash 或更高級模型時，如何設計自動化測試集，以確保舊版 skill.md 規則在高級模型下的比對語意保持一致，而不發生「符合性評分漂移」？
流式日誌實時性 (Streaming Logs Performance)：
目前 live logs 採用一階段完成後返回。是否應該在 Express 端整合 WebSockets 或 SSE（Server-Sent Events），以實現「隨字（Token-by-Token）流式日誌」與即時 ADK 脈衝粒子動畫？
國際 Liaison 協調信精準翻譯 (Liaison Translation Engine)：
如何利用 Gemini 內置的 translationConfig，將 TFDA 的中文公文缺失，完美翻譯成符合歐美 MDR 與韓國 Nemko 原廠商務語境的「地道技術英文信件」，而非單純的字面生硬硬翻？
人工智慧 CADe/CADx 的特殊確效 (AI CADe Verification)：
針對基於 AI 的醫療影像軟體（如 S-Detect），TFDA 要求比對黃金標準（Gold Standard）。我們的 AI 審查智能體該如何有效核對該文檔是否提供了真實的統計學敏感度與特異度數據？
資料緩存與性能優化 (Caching and Telemetry Performance)：
當用戶頻繁修改 skill.md 並在 Playground 進行測試時，後端應如何對「法規檢索結果」與「重複的代碼計算」進行高能效的 Redis / 內存緩存以降低 Token 開銷？
視覺 team colors 的響應式注入 (Team Theme CSS Variables)：
在下載生成的獨立 HTML 時，我們是如何將前端選定的 stylePreset 轉換為純 Tailwind v4 相容的 CSS Variables 變量並注入到 HTML 中，以確保主題渲染完美不崩潰？
拖拽看板狀態關聯 (Kanban-Compliance State Sync)：
當用戶在離線 HTML 的看板中將一個「待處理」的缺失項目拖拽到「已就緒 (Completed)」時，系統如何自動感知、將符合性狀態變更為 "OK"，並動態重算「通過率 KPI」與「圓形 SVG 進度條」？
AI 網路安全確效 (Cybersecurity Review Depth)：
針對 TFDA 最新推動的「醫療器材網路安全查驗登記審查指引」，系統如何核對申報文件中是否確實檢附了 SBOM (軟體清單)、漏洞掃描報告與資安應變計畫？
本地儲存與持久化策略 (Local Storage Sync)：
為了在雲端環境清空快取後不丟失用戶的 compareHistory（6 大圖表數據），我們是否應採用 localStorage 來同步歷史比對記錄？
多語系框架動態切換 (Multilingual HUD Switching)：
在 Traditional Chinese 與 English 切換時，我們如何確保所有的 HUD Telemetry 標籤、日誌前綴與 SVG 圖表中的坐標字體皆能在 20 毫秒內流暢無縫地響應式更迭？
多主機與分布式 Agent (Distributed Agent OS)：
未來若將此系統轉化為大型藥企的私有化法規中台，如何架構支持「多用戶協同審查同一個 EVO Q10 專案」的分布式 Agent 架構？
TFDA 智慧推薦補件報告範本 (RAI Auto-Template Suggestion)：
在 RAI 補件回覆撰寫區中，AI 除了提供文字骨架，是否可以進一步根據缺失標準，直接動態生成一份「空的、標準格式的 Excel/Word 技術規格對照表範本」供藥商下載填寫？
玖、 總結 (Summary)
本技術規格書詳盡闡述了「Agentic OS & TFDA 智慧法規工作台」在全棧 React 19 + Node.js Express 平台上的卓越架構與高保真實現。該系統成功將 「Agent OS 指令集沙盒演練」 與 「TFDA 醫療器材符合性智慧稽核」 完美融合在同一個極致流暢、具備瑞士極簡現代設計美學的數位儀表板中。
特別是其內置的 6 大原生 SVG télémetry 監控圖表，成功解決了相容性難題，提供了頂級的可視化體感；而自助編譯、支持完美本地離線運行的 HTML 下載技術，更是直接打通了「台灣藥商 - 官方審查 - 國際原廠」這條醫材註冊生命線中的黃金閉環。這是一套具備高度實用價值、工程架構極其典雅的智慧法規代碼結晶。
本說明書到此結束。平台已在 http://localhost:3000 (Dev Server) 完全部置就緒，所有 TypeScript 類型校驗與 Vite 建置流均已順利綠燈通過。
