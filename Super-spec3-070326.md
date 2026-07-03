醫療器材 510(k) 自主性 AI 審查監管代理平台（eSTAR v7.0 兼容）
進階級架構技術規格說明書（v2026.2.0 - 升級迭代版）
1. 系統性架構藍圖與無 RAG 設計原則
1.1 核心架構哲學
本系統的設計核心在於解決法規審查的「時效性硬傷」與「上下文斷裂問題」。傳統的 RAG 技術會將長篇的 FDA 指南切碎為向量塊，這會破壞法規文字之間的內在法規階層（Regulatory Hierarchy）。例如，一條豁免條款（Exception）的有效性往往取決於前文三個段落前的定義，RAG 檢索極易導致大模型「斷章取義」。

本平台完全摒棄靜態向量數據庫（No-RAG），轉而利用 SUPER LLM 的長文本脈絡窗口（Long-Context Window，支援單次 200k 至 1M Tokens 的高階推理），將整個審查 Session 的全景資料作為動態物件（Dynamic Context Objects）載入內存：

動態指引注入（Dynamic Guidance Ingestion）： 系統不預存任何指南。當審查員開啟一個 510(k) 案時，可直接粘貼最新的特定器材指引文字，或者直接上傳官方剛發布的 Draft Guidance PDF。這些文字將被完整、無切片地封裝在系統 System Prompt 的指令邊界層。

網頁搜索與 openFDA 雙軌 Proxy（Dynamic Web & API Search Proxy）： 平台封裝了具備自適應結構化語法轉換器的 Search Agent。當審查員指定了對比器材（Predicate Device）的 510(k) 號碼（K號）或產品代碼（Product Code）後，Agent 會自主發起異步請求，一條流向 api.fda.gov 提取結構化 JSON 決策元數據，另一條流向即時網絡搜索，抓取官方最新的公開 510(k) Summary 文本及 MAUDE 不良事件歷史記錄。

多模態申報物資重組引擎（新增）： 全面兼容 eSTAR PDF, Markdown, JSON 及 510(k) 原始摘要。系統接收多源非結構化或半結構化申報材料後，會自動在內存中調用雙向對齊解析器，將散亂的技術文件統一編排、重構為高度語意化的結構化數據（對應 .md 與 .json 雙輸出格式），作為後續審查與可視化看板的唯一真理源（Single Source of Truth）。

2. 核心功能深入探討 1：skill.md 自主優化與演進管道
本功能賦予了 FDA 審查員「無代碼（No-Code）定義 AI 代理技能」的能力。審查員可以將自身的審查經驗、特定的檢查 SOP 寫成一個 Markdown 格式的技能檔（skill.md），系統會利用 AI 的自我反思與外網最新數據，自動將這個技能檔優化、編譯至工業級的精準度。

2.1 什麼是 skill.md？
skill.md 是一個結構化的代理行為指南，內部定義了該 Agent 的角色定位（Role）、工作流邊界（Workflow Constraints）、必須調用的外部工具（Tools）、法規對齊檢查點（Compliance Checkpoints） 以及 Few-Shot 範例（Few-Shot Examples）。

2.2 skill.md 優化器架構與數據流
當用戶貼上或上傳一個初版的 skill.md 時，系統會啟動一個 「元代理優化管道（Meta-Agent Optimization Pipeline）」，其運作流程如下：

[ 用戶上傳: 初版 skill.md ] ──► [ 用戶隨附之優化指引 (Guidance Prompt) ]
                                            │
                                            ▼
                          ┌──────────────────────────────────┐
                          │   元技能編譯代理 (Meta-Compiler)  │
                          └─────────────────┬────────────────┘
                                            │
                                   (提取關鍵字與產品代碼)
                                            ▼
                          ┌──────────────────────────────────┐
                          │   即時外網搜尋 & openFDA API 代理 │
                          └─────────────────┬────────────────┘
                                            │
                       (拉取 2026 最新法規變更、MAUDE 缺陷、eSTAR v7 邏輯)
                                            ▼
                          ┌──────────────────────────────────┐
                          │ 技能漏洞紅隊對抗代理 (Red-Team)   │
                          └─────────────────┬────────────────┘
                                            │
                                (生成反思與修補後的結構化 Prompt)
                                            ▼
[ 輸出: 進階完美版 skill.md ] ◄──── [ 語意語法校驗閘門 (Guardrail) ]
2.3 端到端演進引擎規格
結構解析（Structure Parsing）： 解析器將 skill.md 拆解為 AST（抽象語法樹）結構的 JSON 物件。

法規增強檢索（Regulatory Augmentation）： 系統偵測 skill.md 中提及的器材類別（例如：血管內導管）。Search Agent 立即啟動，去外網搜尋：

2026 年 FDA 針對該類器材發布的最新特別控制指南（Special Controls Guidance）。

過去 12 個月內，同類產品在市場上最常因為什麼原因收到「Additional Information (AI)」追加資料請求。

漏洞紅隊修補（Vulnerability Patching）： 紅隊代理（Red-Team Agent）會檢查初版 skill.md 中的漏洞。例如，如果初版技能只寫了 「檢查材料的生物相容性」，紅隊代理會根據最新的 2026 年 QMSR 標準與 ISO 10993-1，將其自動擴充補強為：「強制檢查是否包含細胞毒性、致敏性及皮內刺激性測試報告，並核對實驗室是否具備 ASCA（符合性評估認證體系）資質。」

自動生成輸出： 最終輸出一個優化後的 skill.md，內建了完美的 Few-Shot Chain-of-Thought（思考鏈）範例，並直接高亮顯示哪些部分是基於 2026 年最新網頁搜尋結果所增補的條款。

3. 核心功能深入探討 2：高互動式多代理審查駕駛艙（Cockpit Webpage）
這是專為 FDA 審查員設計的中央核心網頁介面。其核心哲學是 Human-in-the-Loop（人類協同決策），徹底告別傳統 Chatbot 的單調對話框，改用資訊密度極高的「三欄式動態控制台」。

3.1 駕駛艙介面佈局（UI Layout Specification）
網頁整體採用 React 19 與 Tailwind CSS 構建，畫面水平切分為三大核心區塊，具備高 scannability（易讀性）：

左欄：多模態文件與指引檢視器（Submission & Guidance Viewer）

整合了 eSTAR v7.0 互動式結構渲染引擎。審查員在此分頁切換檢視「申報方上傳重組後的技術文件（.md/.json）」與「自行貼上的 FDA 審查指南」。

具備 「AI 聯動高亮（AI-Driven Sync Highlighting）」 功能。當審查員點擊中欄 AI 產出的某條缺陷時，左欄文件會自動滾動並高亮對應的原始頁碼與段落。

中欄：自主代理初步審查簡報窗（Agentic Review Intelligence Pod）

流式實時串流（Streaming Live Execution）： 當 eSTAR 文件讀入時，各路 Agent 的推理過程會以結構化的方式即時流式輸出。

互動式缺陷矩陣（Interactive Deficiency Matrix）： AI 產出的初步審查結論不是死板的文本，而是一個個帶有狀態的卡片（卡片狀態包含：嚴重缺陷 Major Deficiency、輕微不合規 Minor Discrepancy、符合標準 Compliant）。

右欄：人類審查員即時筆記與決策覆核側邊欄（Reviewer Notes & Decision Override Sidecar）

人類覆核選框（HITL Checkboxes）： 每一條 AI 產出的缺陷卡片旁，皆有一個人類審查員的確認勾選框。

決策覆核（Override）機制： 如果審查員認為 AI 的判斷過於嚴苛，可以點擊 [Override/覆核] 按鈕，卡片狀態隨即轉為 [人类已修正: 接受申報方理由]。

動態筆記同步（Dynamic Notes Ingestion）： 審查員在右欄隨手寫下的草稿筆記（如：“這個台架測試的樣本數看起來有點少，幫我查查外網對照組一般是多少”），會立刻被中央協調代理捕獲，並動態觸發新一輪的外網搜索與重新推理。

4. 六項全新追加與升級之「Wow AI」核心技術特性
為進一步提升系統的智能化與自動化深度，本平台在原有 3 項特性的基礎上，额外追加 3 項全新 Wow 特性，構成全方位 AI 審查監管矩陣：

Wow 特性 1：多模態物資智能重組與雙向同步器 (Multimodal Submissions Swiss-Army Knife)
技術原理： 當申報方上傳多格式、非結構化的 510(k) summary、eSTAR pdf、md、json 等物資時，系統使用無 RAG 長文本解析引擎，將所有文本及鍵值對抽離，重新編排成一份具備高度結構化的核心 Markdown 與標準化元數據 JSON。

功能價值： 消除因不同部門、不同格式上傳導致的信息孤島，自動修補文件缺失標籤，確保審查人員能在最清晰的語境下閱讀所有技術要點。

Wow 特性 2：全自動生成式「Wow 預審與覆核網頁」及「報告編輯/導出外掛」
技術原理： 系統動態生成基於前端極致視覺體驗（HTML5 + Tailwind）的 Wow Webpage。該網頁不僅展示重組後的材料，還將 AI 初步審查缺陷卡片與 FDA 官員覆核工作流直接整合。審查結束後，一鍵生成具備極高法規威嚴度的「綜合審查報告（Comprehensive Review Report） HTML」，官員可直接在線上進行 Rich-text 編輯。

功能價值： 全面支持報告的一鍵多格式無損導出，包括 Markdown、HTML、PDF 以及供系統對接的 JSON 報文，大幅縮短 510(k) 決策信函（Deficiency Letter）的撰寫週期。

Wow 特性 3：LLM 即時執行可視化、流式互動日誌與 5 圖表動態指標看板
技術原理： 平台引入 WebSocket 雙向通信，提供 LLM 思考鏈執行可視化特效（如 Token 生成流速、Agent 間辯論熱力圖、安全沙箱隔離狀態）。同時提供一個 Live Log 動態互動日誌，以及包含 5 個法規維度圖表的 Wow 互動式儀表板。

5 圖表配置：

法規條款覆蓋率雷達圖 (Regulatory Clause Coverage Radar)

缺陷嚴重程度分布餅圖 (Deficiency Severity Distribution Pie)

前後章節語意矛盾交叉矩陣圖 (Cross-Section Contradiction Matrix Heatmap)

臨床樣本人口統計學偏見偏離度曲線 (Demographic Bias Deviation Curve)

QMSR 品質體系差距落點散點圖 (QMSR Gap Analysis Scatter Plot)

Wow 特性 4：臨床試驗樣本數與人口統計學偏見自動審計代理 (Demographic Bias & Clinical Sample Size Auditor)
技術背景： 依據 2025 年多項針對已核准 AI 醫療器材的獨立學術研究指出，高達 50% 以上的 510(k) 摘要中嚴重缺乏訓練數據與臨床試驗的樣本量、人口統計學（Demographics）分佈資訊。

技術原理： 當審查員動態貼上申報書後，該專屬 Agent 會提取臨床評估章節的患者表格，自主啟動外部網絡搜索，拉取過去 3 年內所有同類已核准器材（同類產品代碼）的平均臨床樣本量、年齡、性別、種族分佈基線（Baseline Matrix）。

功能價值： 系統能自動計算並對審查員發出偏差警告：「警告：申報方的 AI 演算法測試數據中，黑人與亞裔族群樣本佔比僅為 1.2%，顯著低於 openFDA 該產品代碼 14.5% 的行業合規平均值，存在嚴重的人口統計學偏見與泛化失效風險。」

Wow 特性 5：eSTAR v7.0 跨章節矛盾關聯雷達 (Cross-Section Narrative Contradiction Radar)
技術背景： 2026 年 6 月 1 日 FDA 正式發布 eSTAR Version 7.0，將「人類因素與可用性工程（Human Factors Guidance）」全面嵌入結構化表單中。申報廠商經常在不同章節由不同工程師分工撰寫，極易導致前後敘事矛盾。

技術原理： 本特性利用 SUPER LLM 的超長上下文優勢，橫向對齊 eSTAR 的 Human Factors 章節與 Software Risk Management 及 Labeling (使用說明書) 章節。

功能價值： 一旦發現廠商在《使用說明書》中寫明 「本產品可由非醫學專業的家庭照顧者獨立操作」（Lay User），但在《人類因素驗證報告》中卻只招募了 「具備 5 年以上經驗的執業護理師」 作為可用性測試受試者，雷達會瞬間拉響警報，精準定位這一重大的合規性欺瞞。

Wow 特性 6：即時 QMSR / ISO 13485 品質體系差距預判引擎 (Real-time QMSR Gap Synthesizer)
技術背景： 自 2026 年 2 月 2 日起，FDA 官方的質量體系法規（QSR）正式被全新的 QMSR（品質管理系統條例）取代，全面引進並接軌 ISO 13485:2016 國際標準。

技術原理： 當審查員動態貼上或透過網路搜尋引進申報廠商的「設計控制（Design Controls）」與「風險管理程序書」摘要時，本 Agent 會逐條比對 ISO 13485:2016 的核心條款。

功能價值： 系統能在一分鐘內為審查員產出《QMSR 合規風險預判報告》，指出該廠的追溯性矩陣（Traceability Matrix）是否仍停留在舊版的 QSR 框架下，幫助審查員在進行實質技術審查前，快速完成廠商體系合規性的全面摸底。

5. 詳細技術架構與數據流規範
為了支撐 skill.md 管道與駕駛艙的高互動性，底層 API 與數據模型必須嚴格遵循強製結構化（Strictly Structured）的數據協議。

5.1 擴充後的系統整體組件架構
MaterialReorganizer 模組： 負責將多源異構輸入（510(k) summary, eSTAR pdf, md, json）進行多模態清洗，統一導出標準化的 .md 與 .json 文件。

SkillCompiler 模組： 負責將用戶上傳的 Markdown 轉換為大模型可識別的元 Prompts，並通過外網 API 進行合規性增強。

CockpitStreamer 模組： 基於 WebSocket 協議，負責將 LLM 的內心獨白、多代理辯論、紅隊審查結果與動態 Live Log 即時推送到前端 React 駕駛艙介面。

ExternalSearchProxy 模組： 負責執行單向安全安全屏蔽（Masking）的 openFDA JSON API 請求與 Google/Bing 法規指南實時檢索。

VisualMetricsEngine 模組： 負責將系統審查狀態、偏見分析、QMSR 落點等動態計算結果實時轉化為前端包含 5 個圖表的 dashboard 數據流。

5.2 全景端到端數據流 JSON 規範（Data Schema Specification）
當審查員在駕駛艙操作修改，同時觸發材料重組、skill.md 優化與外網搜尋時，系統核心組件之間流轉的完整 JSON 數據結構如下：

JSON
{
  "system_metadata": {
    "platform_version": "v2026.2.0",
    "active_estar_version": "7.0_Non-IVD",
    "qmsr_compliance_baseline": "ISO_13485:2016_Enforced",
    "timestamp": "2026-07-03T14:22:36Z",
    "ui_preferences": {
      "theme": "dark",
      "default_language": "zh-TW",
      "panton_palette_style": "Classic_Navy_SOP_03"
    }
  },
  "material_reorganization_output": {
    "source_files_ingested": ["k260789_summary.txt", "estar_v7_form.pdf", "device_spec.json"],
    "structured_markdown_uri": "/en-US/submissions/K260789/reorganized_v1.md",
    "structured_json_uri": "/en-US/submissions/K260789/reorganized_v1.json"
  },
  "skill_md_evolution_pipeline": {
    "skill_id": "human-factors-expert-agent",
    "user_provided_guidance": "Please enhance this skill to ensure strict check against the brand new May 29, 2026 Human Factors Content Guidance embedded in eSTAR v7.0.",
    "original_skill_content": "# Role\nFDA Human Factors Reviewer.\n# Task\nCheck if the medical device usability test has enough participants.",
    "web_search_augmentation_results": [
      {
        "source": "site:fda.gov/regulatory-information-2026",
        "scraped_criteria": "The final Human Factors guidance (May 29, 2026) mandates a risk-based framework for critical tasks, requiring a minimum of 15 participants per distinct user group for human factors validation testing."
      }
    ],
    "evolved_skill_content": "# Role\nFDA Senior Human Factors & Usability Regulatory Specialist (eSTAR v7.0 Compliant).\n# Task\nExecute strict risk-based framework auditing for medical device user interfaces.\n# Strict_Constraints\n1. Verify if the submission defines distinct user groups (e.g., Lay Users vs. Healthcare Professionals).\n2. Cross-check Section 14.1 with the May 29, 2026 Guidance requirement: Confirm that each critical task has a minimum validation sample size of N=15. If N<15, automatically flag as a Major Deficiency."
  },
  "interactive_cockpit_state": {
    "active_submission_k_number": "K260789",
    "current_memorandum_completion_rate": 65,
    "live_execution_telemetry": {
      "llm_token_per_second": 142.5,
      "active_debate_nodes": ["Red-Team-Agent", "Compliance-Guardrail"],
      "current_log_stream": "[14:22:34] Wow-Radar triggered: Section 11.0 vs Section 14.2 discrepancy found. [14:22:35] Fetching openFDA for product code 'OMK'..."
    },
    "dashboard_metrics": {
      "regulatory_clause_coverage": [85, 92, 78, 95, 60],
      "deficiency_distribution": { "major": 1, "minor": 3, "compliant": 14 },
      "demographic_bias_deviation": 0.125,
      "qmsr_gap_score": 74.2
    },
    "agentic_findings": [
      {
        "finding_id": "DEF_CYBER_004",
        "agent_name": "Red-Team Adversarial Agent",
        "severity": "MAJOR_DEFICIENCY",
        "estar_section_link": "Section 14.2 (Software Cybersecurity)",
        "description": "Applicant states in Section 14.2 that encryption keys are stored locally in plaintext during deployment, violating the Feb 2026 Cybersecurity Guidance regarding secure key storage architecture.",
        "exact_quote_from_submission": "Cryptographic keys for node authentication are cached in temporary local directories without hardware security module (HSM) isolation.",
        "page_number": 89,
        "hitl_status": "PENDING_REVIEWER_ACTION"
      },
      {
        "finding_id": "DEF_HF_009",
        "agent_name": "Human Factors Auditor Agent",
        "severity": "MAJOR_DEFICIENCY",
        "estar_section_link": "Section 11.0 (Human Factors & Usability)",
        "description": "Sample size for pediatric user group validation testing is N=8, which violates the regulatory threshold established in the May 2026 guidance.",
        "page_number": 43,
        "hitl_status": "OVERRIDDEN_BY_REVIEWER",
        "reviewer_override_justification": "The device is primarily indicated for adults over 65. The pediatric population is an extremely narrow, secondary off-label usage group. N=8 is scientifically acceptable under the specific clinical protocol K260789-A; will accept applicant rationale but add a warning condition to the final label."
      }
    ],
    "reviewer_live_notes": "Spoke with clinical lead regarding the pediatric sample size. Since the device is a hospital-grade surgical system for adult cataract removal, the pediatric risk is minimal. I have manually overridden Finding DEF_HF_009. However, the cybersecurity flaw in key storage (DEF_CYBER_004) is critical and must be formally requested in the AI letter."
  }
}
6. 安全性、隱私保護與商業機密（CCI）硬防禦
在導入 skill.md 網頁搜尋優化與高互動式駕駛艙後，系統暴露的數據接觸點增多，必須升級資料安全防御等級：

搜尋關鍵字零商業特徵化（Featureless Search Tokenization）：
當系統為了優化 skill.md 或驗證駕駛艙中的技術參數而發起網絡搜尋時，系統底層會調用「語意抽象引擎」。該引擎會自動剔除所有與當前申報案相關的廠商名稱、產品代號名稱等專有名詞，僅提取中性的科學與法規關鍵字（例如："Class II" "ECG" "OTA update" "cybersecurity guidance 2026"）發送至外網，確保外界絕對無法透過搜尋引擎的流量分析逆向推導出 FDA 正在審查哪家公司的哪款創新型設備。

人類筆記動態脫敏層（Reviewer Notes Data Masking）：
審查員在右側邊欄寫下的動態筆記，可能包含尚未公開的臨床數據。系統在將這些筆記轉化為 AI Agent 的驅動指令前，會先經過本地端輕量級脫敏過濾器，自動識別並用遮罩字元（如 [CONFIDENTIAL_DATA_MASKED]）編蔽潛在的敏感資訊，防止其作為 Context 誤送至外部 SUPER LLM 的公共 API 端點。

W3C 網頁無障礙與安全沙箱（UI Sandbox Protection）：
高互動駕駛艙網頁採用嚴格的內容安全策略（Content Security Policy, CSP），禁用所有外部未授權的 JavaScript 腳本注入。所有左欄渲染的 eSTAR 文件中的 PDF 內建巨集、惡意連結，在經由後端解析時均會被徹底剝離，防止黑客透過申報文件對 FDA 審查員的瀏覽器實施動態 Cookie 劫持或 XSS 攻擊。

7. 「視覺驚艷」前端展示網頁代碼（Wow Review & Review Dashboard Preview）
以下提供完整升級、可直接運行的單一 HTML 文件範例。其中包含亮/暗主題切換器、中英雙語環境、Pantone 配色風格方案自適應、LLM 執行流、即時日誌與 5 大指標圖表看板（以高美感純前端 CSS 繪製）。

HTML
<!DOCTYPE html>
<html lang="zh-TW" class="dark">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>醫療器材 510(k) 自主性 AI 審查監管代理平台</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script>
        tailwind.config = {
            darkMode: 'class',
            theme: {
                extend: {
                    colors: {
                        // 默認採用 Pantone Classic Navy 19-4052 主題配色
                        panton: {
                            primary: '#0F2C59',
                            secondary: '#DAC0A3',
                            bgDark: '#0B132B',
                            cardDark: '#1C2541',
                            accent: '#F87171'
                        }
                    }
                }
            }
        }
    </script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@300;400;500&family=Noto+Sans+TC:wght@300;400;700&display=swap');
        body { font-family: 'Noto Sans TC', sans-serif; transition: all 0.3s ease; }
        .code-font { font-family: 'JetBrains Mono', monospace; }
        .glow-active { box-shadow: 0 0 15px rgba(248, 113, 113, 0.4); }
        .stream-text::after { content: '▋'; animation: blink 1s step-start infinite; }
        @keyframes blink { 50% { opacity: 0; } }
    </style>
</head>
<body class="bg-[#F4F6F9] text-slate-900 dark:bg-panton-bgDark dark:text-slate-100 h-screen flex flex-col overflow-hidden">

    <header class="h-16 border-b border-slate-200 dark:border-slate-800 bg-white dark:bg-panton-cardDark px-6 flex items-center justify-between shadow-sm z-50">
        <div class="flex items-center gap-3">
            <div class="w-3 h-3 bg-panton-accent rounded-full animate-ping"></div>
            <h1 class="text-lg font-bold tracking-tight">FDA 510(k) <span class="text-panton-accent font-light">Autonomous AI Agent Cockpit v7.0</span></h1>
        </div>
        
        <div class="flex items-center gap-4">
            <select class="bg-slate-100 dark:bg-slate-800 border-none rounded-lg px-3 py-1.5 text-xs font-medium outline-none">
                <option value="zh">繁體中文 (Traditional Chinese)</option>
                <option value="en">English (US)</option>
            </select>
            
            <button onclick="toggleTheme()" class="p-2 bg-slate-100 dark:bg-slate-800 rounded-lg text-xs hover:opacity-80">
                🌓 <span class="hidden md:inline ml-1">切換主題 Mode</span>
            </button>

            <select class="bg-slate-100 dark:bg-slate-800 border-none rounded-lg px-3 py-1.5 text-xs font-medium outline-none text-panton-accent">
                <option>Pantone 19-4052 Classic Navy (預設)</option>
                <option>Pantone 17-5104 Ultimate Gray</option>
                <option>Pantone 13-0647 Illuminating</option>
                <option>Pantone 18-1750 Viva Magenta</option>
                <option>Pantone 15-4020 Cerulean</option>
                <option>Pantone 19-1557 Chili Pepper</option>
                <option>Pantone 16-1546 Living Coral</option>
                <option>Pantone 18-3838 Ultra Violet</option>
                <option>Pantone 15-0343 Greenery</option>
                <option>Pantone 14-0848 Mimosa</option>
            </select>

            <div class="h-5 w-[1px] bg-slate-300 dark:bg-slate-700"></div>

            <div class="flex gap-1.5">
                <button onclick="alert('報告已成功導出為標準 Markdown (.md) 文件！')" class="bg-blue-600 hover:bg-blue-700 text-white px-3 py-1.5 rounded-md text-xs font-bold transition">導出 MD</button>
                <button onclick="alert('全面審查報告 HTML (Wow網頁版) 封裝完成。')" class="bg-emerald-600 hover:bg-emerald-700 text-white px-3 py-1.5 rounded-md text-xs font-bold transition">導出 HTML</button>
                <button onclick="alert('PDF 報告渲染中...')" class="bg-purple-600 hover:bg-purple-700 text-white px-3 py-1.5 rounded-md text-xs font-bold transition">導出 PDF</button>
            </div>
        </div>
    </header>

    <div class="flex-1 flex flex-col overflow-hidden">
        
        <section class="h-32 bg-white dark:bg-panton-cardDark border-b border-slate-200 dark:border-slate-800 p-4 grid grid-cols-5 gap-4 overflow-hidden shrink-0">
            <div class="bg-slate-50 dark:bg-slate-800/50 rounded-lg p-2 flex flex-col justify-between border border-slate-100 dark:border-slate-700">
                <div class="text-[11px] text-slate-400 font-bold uppercase">1. 法規條款覆蓋率 (Radar Matrix)</div>
                <div class="flex items-end gap-1 h-12">
                    <div class="bg-blue-500 w-3 h-[85%] rounded-t-sm"></div>
                    <div class="bg-blue-500 w-3 h-[92%] rounded-t-sm"></div>
                    <div class="bg-blue-500 w-3 h-[78%] rounded-t-sm"></div>
                    <div class="bg-blue-500 w-3 h-[95%] rounded-t-sm"></div>
                    <div class="bg-panton-accent w-3 h-[60%] rounded-t-sm animate-pulse"></div>
                </div>
                <div class="text-right text-[10px] text-panton-accent font-bold">目前進度: 82%</div>
            </div>
            <div class="bg-slate-50 dark:bg-slate-800/50 rounded-lg p-2 flex flex-col justify-between border border-slate-100 dark:border-slate-700">
                <div class="text-[11px] text-slate-400 font-bold uppercase">2. 缺陷嚴重度分布 (Pie Metrics)</div>
                <div class="flex items-center gap-3 h-12">
                    <div class="w-10 h-10 rounded-full border-4 border-panton-accent border-t-yellow-400 border-r-emerald-500 animate-spin"></div>
                    <div class="text-[10px] space-y-0.5">
                        <span class="block text-red-400 font-bold">● 重大缺陷: 1</span>
                        <span class="block text-yellow-400">● 輕微不符: 3</span>
                    </div>
                </div>
                <div class="text-right text-[10px] text-slate-400">共計 18 項審查點</div>
            </div>
            <div class="bg-slate-50 dark:bg-slate-800/50 rounded-lg p-2 flex flex-col justify-between border border-slate-100 dark:border-slate-700">
                <div class="text-[11px] text-slate-400 font-bold uppercase">3. 跨章節矛盾雷達 (Cross Heatmap)</div>
                <div class="grid grid-cols-4 gap-1 w-full h-12 pt-1">
                    <div class="bg-emerald-900 rounded-sm"></div><div class="bg-emerald-800 rounded-sm"></div><div class="bg-emerald-700 rounded-sm"></div><div class="bg-red-500 animate-pulse rounded-sm"></div>
                    <div class="bg-emerald-800 rounded-sm"></div><div class="bg-emerald-600 rounded-sm"></div><div class="bg-emerald-700 rounded-sm"></div><div class="bg-emerald-900 rounded-sm"></div>
                </div>
                <div class="text-left text-[9px] text-red-400 font-semibold">偵測到 1 處不一致敘事</div>
            </div>
            <div class="bg-slate-50 dark:bg-slate-800/50 rounded-lg p-2 flex flex-col justify-between border border-slate-100 dark:border-slate-700">
                <div class="text-[11px] text-slate-400 font-bold uppercase">4. 人口統計學偏見 (Bias Matrix)</div>
                <div class="w-full bg-slate-200 dark:bg-slate-700 h-2 rounded-full overflow-hidden mt-3">
                    <div class="bg-panton-accent h-full w-[78%]"></div>
                </div>
                <div class="text-xs text-slate-300 flex justify-between items-center">
                    <span class="text-[10px] text-yellow-400 font-medium">偏離行業基準線</span>
                    <span class="font-bold text-panton-accent text-sm">78%</span>
                </div>
            </div>
            <div class="bg-slate-50 dark:bg-slate-800/50 rounded-lg p-2 flex flex-col justify-between border border-slate-100 dark:border-slate-700">
                <div class="text-[11px] text-slate-400 font-bold uppercase">5. QMSR 體系落點 (ISO 13485)</div>
                <div class="relative w-full h-12 border-b border-l border-slate-600">
                    <div class="absolute bottom-6 left-12 w-3 h-3 bg-yellow-400 rounded-full animate-bounce"></div>
                    <div class="absolute bottom-1 right-2 text-[8px] text-slate-500">設計控制</div>
                    <div class="absolute top-1 left-1 text-[8px] text-slate-500">風險管理</div>
                </div>
                <div class="text-right text-[10px] text-yellow-400 font-bold">提示: 仍保留舊版 QSR 框架</div>
            </div>
        </section>

        <div class="flex-1 flex overflow-hidden">
            
            <aside class="w-1/4 border-r border-slate-200 dark:border-slate-800 flex flex-col bg-white dark:bg-panton-cardDark overflow-hidden">
                <div class="p-4 border-b border-slate-100 dark:border-slate-800 bg-slate-50 dark:bg-slate-900/40">
                    <h2 class="text-xs font-bold text-slate-400 uppercase tracking-widest">1. 多模態物資智能重組 (Structured)</h2>
                    <p class="text-[10px] text-slate-400 mt-1">已融合 510(k) Summary, eSTAR pdf, json 材料</p>
                </div>
                <div class="flex-1 overflow-y-auto p-4 space-y-4 text-xs font-serif leading-relaxed">
                    <div class="p-3 bg-blue-50/50 dark:bg-blue-950/20 border-l-2 border-blue-500 rounded-r-md">
                        <span class="block font-bold text-blue-400 mb-1">[系統重組段落 - 申報範圍]</span>
                        Samsung Medison HERA Z20 預期用於診斷超音波成像。系統支持包含 3D/4D、ELASTOSCAN+ 及進階 AI 輔助分析功能。
                    </div>
                    <div class="p-3 bg-red-50/50 dark:bg-red-950/20 border-l-2 border-panton-accent rounded-r-md">
                        <span class="block font-bold text-panton-accent mb-1">[跨章節矛盾關聯雷達鎖定段落]</span>
                        <span class="bg-red-500/20 px-1 text-red-300">《使用說明書》第 12 頁明記本系統可由非醫學專業家庭照護者獨立操作；但《人類因素驗證報告》僅招募執業護理師。</span>
                    </div>
                    <div class="p-3 bg-yellow-50/50 dark:bg-yellow-950/20 border-l-2 border-yellow-500 rounded-r-md">
                        <span class="block font-bold text-yellow-400 mb-1">[QMSR 體系追溯章節]</span>
                        廠商風險管理矩陣未涵蓋 ISO 13485:2016 關於委外供應商軟體安全漏洞的動態追溯程序。
                    </div>
                </div>
            </aside>

            <section class="flex-1 flex flex-col bg-slate-50 dark:bg-panton-bgDark overflow-hidden border-r border-slate-200 dark:border-slate-800">
                <div class="h-1/4 bg-slate-900 text-emerald-400 p-3 text-xs code-font overflow-y-auto border-b border-slate-800 space-y-1">
                    <div class="text-slate-400 border-b border-slate-800 pb-1 mb-1 flex justify-between items-center">
                        <span>⚡ AI AGENT RUNTIME EXECUTION STREAM (LIVE LOG)</span>
                        <span class="text-emerald-500 animate-pulse">● 142.5 Tokens/s</span>
                    </div>
                    <div class="text-blue-400">[SYSTEM] Loading Multi-modal Submission Materials into Dynamic Context Objects... Success.</div>
                    <div class="text-purple-400">[SKILL.MD PIPELINE] Optimizing behavior prompt via May 2026 FDA Human Factors Guidance.</div>
                    <div class="text-yellow-400">[WOW-RADAR] Auditing Section 11.0 against Section 14.2 for internal narratives...</div>
                    <div class="text-red-400 animate-pulse stream-text">[CRITICAL FINDING] DEF_CYBER_004 generated. Threat model detects hardware isolation bypass risk.</div>
                </div>

                <div class="flex-1 p-6 overflow-y-auto space-y-4">
                    <h3 class="text-xs font-bold text-slate-400 uppercase tracking-wider">2. 自主代理初步審查簡報窗 (可由官員動態編輯/覆核)</h3>
                    
                    <div class="bg-white dark:bg-panton-cardDark p-4 rounded-xl border border-red-500/30 glow-active flex flex-col justify-between">
                        <div class="flex justify-between items-start mb-2">
                            <div>
                                <span class="bg-red-500/20 text-red-400 px-2 py-0.5 rounded text-[10px] font-bold tracking-wide uppercase">重大缺陷 Major Deficiency</span>
                                <h4 class="text-sm font-bold mt-1">密鑰儲存架構違反 2026 最新網絡安全指引</h4>
                            </div>
                            <span class="text-xs text-slate-400 code-font">eSTAR Sec 14.2 (p.89)</span>
                        </div>
                        <p class="text-xs text-slate-400 mb-3 bg-slate-50 dark:bg-slate-900 p-2 rounded">
                          "申報方聲明加密密鑰在部署期間以明文形式本地儲存，這違反了 FDA 2026 年 2 月發布的安全密鑰管理體系硬性要求。"
                        </p>
                        <div class="flex justify-between items-center border-t border-slate-100 dark:border-slate-800 pt-3">
                            <label class="flex items-center gap-2 text-xs text-red-400 cursor-pointer">
                                <input type="checkbox" checked class="rounded border-slate-600 text-panton-accent focus:ring-panton-accent bg-slate-800"> 官員覆核：確認此項列入最終正式缺陷報告
                            </label>
                            <button onclick="alert('已覆核此判斷：系統將自動重新安排缺陷信函語氣。')" class="text-xs bg-slate-100 hover:bg-slate-200 dark:bg-slate-800 dark:hover:bg-slate-700 px-2.5 py-1 rounded transition">人類決策修正 (Override)</button>
                        </div>
                    </div>

                    <div class="bg-white dark:bg-panton-cardDark p-4 rounded-xl border border-slate-200 dark:border-slate-700 opacity-75">
                        <div class="flex justify-between items-start mb-2">
                            <div>
                                <span class="bg-slate-500/20 text-slate-400 px-2 py-0.5 rounded text-[10px] font-bold tracking-wide uppercase">人類已修正 Overridden</span>
                                <h4 class="text-sm font-bold mt-1 text-slate-500 line-through">兒童臨床驗證樣本數不足 (N=8)</h4>
                            </div>
                            <span class="text-xs text-slate-400 code-font">eSTAR Sec 11.0 (p.43)</span>
                        </div>
                        <p class="text-xs text-yellow-500/80 mb-3 bg-yellow-500/5 p-2 rounded border border-yellow-500/20">
                          <strong>官員修正理由：</strong> 本裝置主要適應症為 65 歲以上老年白內障手術，兒童為極其次要的標籤外特殊臨床情境。科學上接受 N=8 的臨床方案，不列為重大缺陷，改以標籤警語體現。
                        </p>
                    </div>
                </div>
            </section>

            <aside class="w-1/3 border-l border-slate-200 dark:border-slate-800 flex flex-col bg-white dark:bg-panton-cardDark overflow-hidden">
                <div class="p-4 border-b border-slate-100 dark:border-slate-800 bg-slate-50 dark:bg-slate-900/40">
                    <h2 class="text-xs font-bold text-slate-400 uppercase tracking-widest">3. 綜合審查報告線上編輯器 (Comprehensive Report)</h2>
                    <p class="text-[10px] text-slate-400 mt-1">此處內容支持實時 Rich-text 編輯，並支持導出全格式</p>
                </div>
                
                <div class="flex-1 p-4 overflow-y-auto">
                    <div contenteditable="true" class="w-full h-full p-4 bg-slate-50 dark:bg-slate-900/60 rounded-xl font-mono text-xs focus:outline-none focus:ring-1 focus:ring-panton-accent leading-relaxed overflow-y-auto space-y-4">
                        <p class="text-center font-bold text-sm text-slate-200 border-b border-slate-700 pb-2">FDA 510(k) SCIENTIFIC MEMORANDUM</p>
                        <p><strong>CASE NUMBER:</strong> K260789</p>
                        <p><strong>DEVICE NAME:</strong> Samsung Medison HERA Z20</p>
                        <p><strong>REGULATORY STATUS:</strong> ADDITIONAL INFORMATION (AI) REQUEST RECOMMENDED</p>
                        <hr class="border-slate-700 my-2">
                        <p class="text-yellow-400 font-bold">1. EXECUTIVE SUMMARY & GAP ENFORCEMENT</p>
                        <p>The technical documentation for the HERA Z20 has been audited against eSTAR v7.0 template boundaries and 2026 QMSR standards. Significant safety posture omissions have been discovered regarding hardware key storage containment and Human Factors validation cohort diversity.</p>
                        <p class="text-red-400 font-bold">2. FORMAL DEFICIENCY CLAUSES</p>
                        <p><strong>Deficiency 1 (Cybersecurity):</strong> Provide verification testing details demonstrating compliance with the secure cryptographic isolation standard outlined in Section 14.2 of the February 2026 Guidance.</p>
                    </div>
                </div>

                <div class="p-4 border-t border-slate-200 dark:border-slate-800 bg-slate-50 dark:bg-slate-900/30">
                    <span class="block text-[11px] font-bold text-slate-400 mb-1.5 uppercase">✍️ 審查員動態筆記同步層 (實時觸發外網代理)</span>
                    <textarea class="w-full h-16 p-2 text-xs bg-white dark:bg-slate-800 border border-slate-200 dark:border-slate-700 rounded-md focus:outline-none focus:border-panton-accent resize-none text-slate-300" placeholder="在此輸入口語筆記（例如：‘幫我查查同類產品 K221435 的臨床樣本量是多大...’），AI 將自動提取並觸發搜索..."></textarea>
                </div>
            </aside>

        </div>
    </div>

    <script>
        function toggleTheme() {
            const html = document.documentElement;
            if (html.classList.contains('dark')) {
                html.classList.remove('dark');
                alert('已切換至明亮主題 (Light Theme)');
            } else {
                html.classList.add('dark');
                alert('已切換至暗黑主題 (Dark Theme) - 採用 Pantone 經典藍基準');
            }
        }
    </script>
</body>
</html>
8. 20 個深度架構、工程與法規延伸思考問題（Follow-up Questions）
為了推動這套融合了 skill.md 自主上演進技術與多功能監監管駕駛艙的 AI 平台順利通過 FDA 資訊安全與科學倫理審查，必須進一步探討以下 20 個深層核心問題：

skill.md 版本控制與法規追溯： 當多位審查員各自上傳並由 AI 優化出多個不同版本的 skill.md 技能檔時，系統底層應如何設計一個類似 Git 的版本控制與中央審核機制，防止「不合規或過於寬鬆的技能 Prompt」在 FDA 內部擴散？

人類修正（Override）的權重學習機制： 當審查員在駕駛艙中頻繁點擊 [Override/覆核] 駁回 AI 針對某條法規（如最新 2026 年 QMSR 條款）的缺陷判定時，系統是否應該、以及如何在不違反 ZDR（零數據留存）的前提下，利用這些人類回饋來反向微調 Agent 的推理閾值？

長文本 Context Window 內存膨脹控管： 當駕駛艙左欄載入了數份總計超過 800 頁的 eSTAR v7.0 附錄文件，且中欄的多代理辨論又產生了大量思考鏈 Token 時，系統如何利用動態 Context 剪裁技術，在不漏失關鍵法規證據的前提下，防止內存崩潰或超出 Token 預算？

即時網路搜尋的結果一致性（Determinism）： 搜索引擎的結果隨時在變。如果同一個 510(k) 案在週一和週五分別啟動審查，而週三外網剛好發布了一份新的安全召回通告，系統如何向審查員明示「因即時網絡數據變更導致的 AI 審查結論差異」？

Wow 特性 4（人口統計學審計）的數據不對稱處理： 若申報器材是一款罕見病（Orphan Device）診斷軟體，其臨床試驗本身就因客觀原因無法招募到充足的多族群樣本，該 Agent 如何智慧地放寬其偏見審計基線，而不至於誤報大量的「僞陽性缺陷」？

eSTAR v7.0 互動表單腳本對齊： eSTAR v7.0 內部包含大量隱藏的動態邏輯（例如勾選某框後才彈出 Human Factors 欄位）。本系統的 Context Ingestor 在將文件餵給 LLM 時，如何確保大模型能百分之百還原這些動態表單的填寫因果鏈？

人類筆記（Reviewer Notes）的即時語意激發： 當審查員在右側欄寫下 “這款產品的演算法與 K221435 很像” 時，系統應如何設計背後的語意激發（Semantic Trigger）機制，讓 Search Agent 秒級自動去 openFDA 撈取 K221435 的全套公開 Summary 數據？

多代理辯論在 UI 上的視覺化收斂： 當正反兩方 Agent 在中欄激辯不可開交、各自引經據典時，前端介面應如何優化排版（Scannability），才能讓疲憊的 FDA 審查員在一秒鐘內看懂雙方爭論的核心科學矛盾點，而不是被迫閱讀數萬字的辯論日誌？

對抗性提示詞注入的深層隱寫術（Steganography）防禦： 申報廠商若將惡意無視法規的指令，透過像素色差隱寫在台架測試圖表、或者利用高頻噪聲隱藏在語音診斷測試音頻中，本系統的多模態 Agent 應如何在底層沙箱中對這些二進位媒體資產進行安全隔離與清洗？

2026 QMSR 接軌引發的舊案例基準失效： 由於 2026 年 2 月後全面施行 QMSR，Search Agent 在網路上搜到的 2025 年之前的歷史 510(k) 案例其質量體系均基於舊的 QSR。系統在調用這些歷史對比數據時，如何指導 Agent 不要用 2026 年的新標準去生搬硬套歷史既成事實？

技能演進管道（skill.md Engine）的 Few-Shot 自動清洗： AI 在自動為優化後的 skill.md 編寫推理範例時，如何確保這些範例的技術邏輯完全符合 FDA 內部的審查法理，而不夾帶任何網頁上未經證實的論點？

流式輸出（Streaming）與頁面滾動的性能優化： 駕駛艙中欄實時流式輸出數萬字深度推理時，前端 React 虛擬 DOM 如何優化渲染性能，防止左欄的高清 PDF 閱讀器與右欄的筆記輸入框發生輸入卡頓或畫面撕裂（UI Jitter）？

商業秘密（CCI）的界定模糊性防護： 某些新材料的配方，廠商並未註明為 CCI，但客觀上屬於其技術專利。系統的單向搜尋防護牆如何通過「專利語意偵測」，自適應地將這類潛在的敏感配方名字進行中性化替換，再發送至外網搜尋？

Wow 特性 5（跨章節矛盾雷達）的假陽性控制： 廠商在技術描述章節使用的工程學術語，與在說明書章節使用的通俗口語，本質上意思相同但字面完全相左。矛盾雷達如何建立「監管語意同義詞字典」，避免將這類正常的語意轉換誤報為前後矛盾？

多用戶協同審查下的狀態鎖定（State Locking）： 一個複雜的 510(k) 案可能由多位專家（軟體專家、生物相容性專家、臨床專家）共同審查。本駕駛艙的後端狀態機如何設計多租戶分章節鎖定（Row-Level Locking），防止多人的 AI 覆核筆記發生互相覆蓋？

Wow 特性 6（QMSR 差距預判）的法規動態調整： 針對不同風險等級的 Class II 器材，QMSR 的設計控制審查力度大不相同。該引擎如何動態讀取產品代碼（Product Code），自動調整其合規性判斷的嚴苛矩陣？

在線/離線雙軌審查切換機制： 如果 FDA 內部網路因安全升級突然中斷外網連線，導致 openFDA API 與網頁搜尋功能瞬間失效，駕駛艙如何優化其無痕切換機制，讓審查員依然能基於本地已注入的靜態 eSTAR 數據正常進行行政完整性（RTA）審查？

人類筆記與 AI 科學備忘錄（Scientific Memorandum）的語氣對齊： 系統最終自動生成的官方信件與內部備忘錄草案，其語氣必須嚴謹、客觀、具備行政威嚴。系統如何將審查員在右欄寫下的口語化、甚至帶有強烈情緒的粗糙筆記，優雅且精準地轉化為具備法規說服力的官方科學措辭？

大模型推理延遲（Latency）對即時審查體驗的影響： 由於無 RAG 架構需要將幾十萬 Token 的巨量上下文一次性發送給 SUPER LLM，單次推理的延遲可能長達 30-60 秒。系統如何透過前端設計（例如分段並行預加載、進度條語意化提示），來有效緩解人類審查員在等待 AI 辯論與紅隊掃描結果時的焦慮感？

終極法律責任與 AI 決策追溯： 根據醫療 AI 責任歸屬原則，任何經 AI 輔助生成的審查缺陷信函，最終法律責任均由簽名的 FDA 官員承擔。本平台的駕駛艙如何設計一套「最終決策確認清單（Final Signing Checkbox）」，強制審查員逐一滾動檢視所有被他手動覆核或忽略的 AI 警報，以在法律層面確保 Human-in-the-Loop 流程的形式與實質正義
