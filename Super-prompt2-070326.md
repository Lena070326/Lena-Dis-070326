Hi please improve previous design by keeping all original features and adding additional features that 1. User can paste or upload submission materials (510(k) summary, estar pdf, md, json). Then agent will reorganized the submission materials into organized md and json. Then agent will create a wow webpage to present those submission materials with agent pre-review results. FDA officer then can modify review results. Then agent will create a comprehensive review report in wow webpage (html), user can edit the review report and export the report (md, html, pdf, json). Please also adding 3 additional wow ai features to this app. Please let user to select light/dark themes, default Traditional Chinese/English, 10 styles based on panton color palette. Please adding wow visualization effect for llm execution, wow interactive indicators, live log and wow interactive dashboard with 5 graph. Please don't create code and only create a comprehensive technical specification in markdown in 4000 to 5000 words. Please fix potential bugs and iterate until get excellent reuslts. Ending wiht 20 comprehensive follow up questions. 醫療器材 510(k) 自主性 AI 審查監管代理平台（eSTAR v7.0 兼容）

進階級架構技術規格說明書（Advanced Technical Specifications）

1. 系統性架構藍圖與無 RAG 設計原則

1.1 核心架構哲學

本系統的設計核心在於解決法規審查的「時效性硬傷」與「上下文斷裂問題」。傳統的 RAG 技術會將長篇的 FDA 指南切碎為向量塊，這會破壞法規文字之間的內在法規階層（Regulatory Hierarchy）。例如，一條豁免條款（Exception）的有效性往往取決於前文三個段落前的定義，RAG 檢索極易導致大模型「斷章取義」。

本平台完全摒棄靜態向量數據庫（No-RAG），轉而利用 SUPER LLM 的長文本脈絡窗口（Long-Context Window，支援單次 200k 至 1M Tokens 的高階推理），將整個審查 Session 的全景資料作為動態物件（Dynamic Context Objects）載入內存：

動態指引注入（Dynamic Guidance Ingestion）： 系統不預存任何指南。當審查員開啟一個 510(k) 案時，可直接粘貼最新的特定器材指引文字，或者直接上傳官方剛發布的 Draft Guidance PDF。這些文字將被完整、無切片地封裝在系統 System Prompt 的指令邊界層。

網頁搜索與 openFDA 雙軌 Proxy（Dynamic Web & API Search Proxy）： 平台封裝了具備自適應結構化語法轉換器的 Search Agent。當審查員指定了對比器材（Predicate Device）的 510(k) 號碼（K號）或產品代碼（Product Code）後，Agent 會自主發起異步請求，一條流向 api.fda.gov 提取結構化 JSON 決策元數據，另一條流向即時網絡搜索，抓取官方最新的公開 510(k) Summary 文本及 MAUDE 不良事件歷史記錄。

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

整合了 eSTAR v7.0 互動式結構渲染引擎。審查員在此分頁切換檢視「申報方上傳的技術文件」與「自行貼上的 FDA 審查指南」。  

具備 「AI 聯動高亮（AI-Driven Sync Highlighting）」 功能。當審查員點擊中欄 AI 產出的某條缺陷時，左欄文件會自動滾動並高亮對應的原始頁碼與段落。

中欄：自主代理初步審查簡報窗（Agentic Review Intelligence Pod）

流式實時串流（Streaming Live Execution）： 當 eSTAR 文件讀入時，各路 Agent 的推理過程會以結構化的方式即時流式輸出。

互動式缺陷矩陣（Interactive Deficiency Matrix）： AI 產出的初步審查結論不是死板的文本，而是一個個帶有狀態的卡片（卡片狀態包含：嚴重缺陷 (Major Deficiency)、輕微不合規 (Minor Discrepancy)、符合標準 (Compliant)）。

右欄：人類審查員即時筆記與決策覆核側邊欄（Reviewer Notes & Decision Override Sidecar）

人類覆核選框（HITL Checkboxes）： 每一條 AI 產出的缺陷卡片旁，皆有一個人類審查員的確認勾選框。

決策覆核（Override）機制： 如果審查員認為 AI 的判斷過於嚴苛，可以點擊 [Override/覆核] 按鈕，卡片狀態隨即轉為 [人类已修正: 接受申報方理由]。

動態筆記同步（Dynamic Notes Ingestion）： 審查員在右欄隨手寫下的草稿筆記（如：“這個台架測試的樣本數看起來有點少，幫我查查外網對照組一般是多少”），會立刻被中央協調代理捕獲，並動態觸發新一輪的外網搜索與重新推理。

3.2 駕駛艙多代理協同交互系統

以下透過交互式模擬展示 FDA 審查員在面對一款具有嵌入式 AI 生物訊號分析功能的穿戴式心電圖（ECG）監護儀（Class II）時，如何利用新版駕駛艙進行動態審查、修改 AI 初步結論、並驅使系統利用外網搜尋優化 skill.md：





4. 三項全新追加的突破性「Wow AI」核心技術特性

為進一步提升系統的智能化深度，本技術規格書在原有的基礎上，額外追加了三項獨創的 AI 監管技術特性：

Wow 特性 4：臨床試驗樣本數與人口統計學偏見自動審計代理（Demographic Bias & Clinical Sample Size Auditor）

技術背景： 依據 2025 年多項針對已核准 AI 醫療器材的獨立學術研究指出，高達 50% 以上的 510(k) 摘要中嚴重缺乏訓練數據與臨床試驗的樣本量、人口統計學（Demographics）分佈資訊。  

技術原理： 當審查員動態貼上申報書後，該專屬 Agent 會提取臨床評估章節的患者表格，自主啟動外部網絡搜索，拉取過去 3 年內所有同類已核准器材（同類產品代碼）的平均臨床樣本量、年齡、性別、種族分佈基線（Baseline Matrix）。

功能價值： 系統能自動計算並對審查員發出偏差警告：「警告：申報方的 AI 演算法測試數據中，黑人與亞裔族群樣本佔比僅為 1.2%，顯著低於 openFDA 該產品代碼 14.5% 的行業合規平均值，存在嚴重的人口統計學偏見與泛化失效風險。」

Wow 特性 5：eSTAR v7.0 跨章節矛盾關聯雷達（Cross-Section Narrative Contradiction Radar）

技術背景： 2026 年 6 月 1 日 FDA 正式發布 eSTAR Version 7.0，將「人類因素與可用性工程（Human Factors Guidance）」全面嵌入結構化表單中。申報廠商經常在不同章節由不同工程師分工撰寫，極易導致前後敘事矛盾。  

技術原理： 本特性利用 SUPER LLM 的超長上下文優勢，橫向對齊 eSTAR 的 Human Factors 章節與 Software Risk Management 及 Labeling (使用說明書) 章節。

功能價值： 一旦發現廠商在《使用說明書》中寫明 「本產品可由非醫學專業的家庭照顧者獨立操作」（Lay User），但在《人類因素驗證報告》中卻只招募了 「具備 5 年以上經驗的執業護理師」 作為可用性測試受試者，雷達會瞬間拉響警報，精準定位這一重大的合規性欺瞞。

Wow 特性 6：即時 QMSR / ISO 13485 品質體系差距預判引擎（Real-time QMSR Gap Synthesizer）

技術背景： 自 2026 年 2 月 2 日起，FDA 官方的質量體系法規（QSR）正式被全新的 QMSR（品質管理系統條例）取代，全面引進並接軌 ISO 13485:2016 國際標準。  

技術原理： 當審查員動態貼上或透過網路搜尋引進申報廠商的「設計控制（Design Controls）」與「風險管理程序書」摘要時，本 Agent 會逐條比對 ISO 13485:2016 的核心條款。  

功能價值： 系統能在一分鐘內為審查員產出《QMSR 合規風險預判報告》，指出該廠的追溯性矩陣（Traceability Matrix）是否仍停留在舊版的 QSR 框架下，幫助審查員在進行實質技術審查前，快速完成廠商體系合規性的全面摸底。

5. 詳細技術架構與數據流規範

為了支撐 skill.md 管道與駕駛艙的高互動性，底層 API 與數據模型必須嚴格遵循強製結構化（Strictly Structured）的數據協議。

5.1 擴充後的系統整體組件架構

SkillCompiler 模組： 負責將用戶上傳的 Markdown 轉換為大模型可識別的元 Prompts，並通過外網 API 進行合規性增強。

CockpitStreamer 模組： 基於 WebSocket 協議，負責將多代理辯論（Wow 特性 2）、紅隊審查結果即時推送到前端 React 駕駛艙介面。

ExternalSearchProxy 模組： 負責執行單向安全屏蔽（Masking）的 openFDA JSON API 請求與 Google/Bing 法規指南實時檢索。

5.2 全景端到端數據流 JSON 規範（Data Schema Specification）

當審查員在駕駛艙操作修改，同時觸發 skill.md 優化與外網搜尋時，系統核心組件之間流轉的完整 JSON 數據結構如下：

JSON



{

  "system_metadata": {

    "platform_version": "v2026.2.0",

    "active_estar_version": "7.0_Non-IVD",

    "qmsr_compliance_baseline": "ISO_13485:2016_Enforced",

    "timestamp": "2026-07-03T14:22:36Z"

  },

  "skill_md_evolution_pipeline": {

    "skill_id": "human-factors-expert-agent",

    "user_provided_guidance": "Please enhance this skill to ensure strict check against the brand new May 29, 2026 Human Factors Content Guidance embedded in eSTAR v7.0.",

    "original_skill_content": "# Role\nFDA Human Factors Reviewer.\n# Task\nCheck if the medical device usability test has enough participants.",

    "web_search_augmentation_results": [

      {

        "source": "site:fda.gov/regulatory-信息-2026",

        "scraped_criteria": "The final Human Factors guidance (May 29, 2026) mandates a risk-based framework for critical tasks, requiring a minimum of 15 participants per distinct user group for human factors validation testing."

      }

    ],

    "evolved_skill_content": "# Role\nFDA Senior Human Factors & Usability Regulatory Specialist (eSTAR v7.0 Compliant).\n# Task\nExecute strict risk-based framework auditing for medical device user interfaces.\n# Strict_Constraints\n1. Verify if the submission defines distinct user groups (e.g., Lay Users vs. Healthcare Professionals).\n2. Cross-check Section 14.1 with the May 29, 2026 Guidance requirement: Confirm that each critical task has a minimum validation sample size of N=15. If N<15, automatically flag as a Major Deficiency."

  },

  "interactive_cockpit_state": {

    "active_submission_k_number": "K260789",

    "current_memorandum_completion_rate": 65,

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

當系統為了優化 skill.md 或驗證駕駛艙中的技術參數而發起網絡搜尋時，系統底層會調用「語意抽象引擎」。該引擎會自動剔除所有與當前申報案相關的廠商名稱（如 Advanced Vision MedTech）、產品代號名稱（如 HeartShield）等專有名詞，僅提取中性的科學與法規關鍵字（例如："Class II" "ECG" "OTA update" "cybersecurity guidance 2026"）發送至外網，確保外界絕對無法透過搜尋引擎的流量分析逆向推導出 FDA 正在審查哪家公司的哪款創新型設備。

人類筆記動態脫敏層（Reviewer Notes Data Masking）：

審查員在右側邊欄寫下的動態筆記，可能包含尚未公開的臨床數據或實驗室內部八卦。系統在將這些筆記轉化為 AI Agent 的驅動指令前，會先經過本地端輕量級脫敏過濾器，自動識別並用遮罩字元（如 [CONFIDENTIAL_DATA_MASKED]）屏蔽潛在的敏感資訊，防止其作為 Context 誤送至外部 SUPER LLM 的公共 API 端點。

W3C 網頁無障礙與安全沙箱（UI Sandbox Protection）：

高互動駕駛艙網頁採用嚴格的內容安全策略（Content Security Policy, CSP），禁用所有外部未授權的 JavaScript 腳本注入。所有左欄渲染的 eSTAR 文件中的 PDF 內建巨集、惡意連結，在經由後端解析時均會被徹底剝離，防止黑客透過申報文件對 FDA 審查員的瀏覽器實施動態 Cookie 劫持或 XSS 攻擊。

7. 20 個深度架構、工程與法規延伸思考問題（Follow-up Questions）

為了推動這套融合了 skill.md 自主上演進技術與多功能監監管駕駛艙的 AI 平台順利通過 FDA 資訊安全與科學倫理審查，必須進一步探討以下 20 個深層核心問題：

skill.md 版本控制與法規溯源： 當多位審查員各自上傳並由 AI 優化出多個不同版本的 skill.md 技能檔時，系統底層應如何設計一個類似 Git 的版本控制與中央審核機制，防止「不合規或過於寬鬆的技能 Prompt」在 FDA 內部擴散？

人類修正（Override）的權重學習機制： 當審查員在駕駛艙中頻繁點擊 [Override/覆核] 駁回 AI 針對某條法規（如最新 2026 年 QMSR 條款）的缺陷判定時，系統是否應該、以及如何在不違反 ZDR（零數據留存）的前提下，利用這些人類回饋來反向微調 Agent 的推理閾值？

長文本 Context Window 內存膨脹控管： 當駕駛艙左欄載入了數份總計超過 800 頁的 eSTAR v7.0 附錄文件，且中欄的多代理辯論又產生了大量思考鏈 Token 時，系統如何利用動態 Context 剪裁技術，在不漏失關鍵法規證據的前提下，防止內存崩潰或超出 Token 預算？

即時網路搜尋的結果一致性（Determinism）： 搜索引擎的結果隨時在變。如果同一個 510(k) 案在週一和週五分別啟動審查，而週三外網剛好發布了一份新的安全召回通告，系統如何向審查員明示「因即時網絡數據變更導致的 AI 審查結論差異」？

Wow 特性 4（人口統計學審計）的數據不對稱處理： 若申報器材是一款罕見病（Orphan Device）診斷軟體，其臨床試驗本身就因客觀原因無法招募到充足的多族群樣本，該 Agent 如何智慧地放寬其偏見審計基線，而不至於誤報大量的「僞陽性缺陷」？  

eSTAR v7.0 互動表單腳本對齊： eSTAR v7.0 內部包含大量隱藏的動態邏輯（例如勾選某框後才彈出 Human Factors 欄位）。本系統的 Context Ingestor 在將文件餵給 LLM 時，如何確保大模型能百分之百還原這些動態表單的填寫因果鏈？

人類筆記（Reviewer Notes）的即時語意激發： 當審查員在右側欄寫下 “這款產品的演算法與 K221435 很像” 時，系統應如何設計背後的語意激發（Semantic Trigger）機制，讓 Search Agent 秒級自動去 openFDA 撈取 K221435 的全套公開 Summary 數據？

多代理辯論（Wow 特性 2）在 UI 上的視覺化收斂： 當正反兩方 Agent 在中欄激辯不可開交、各自引經據典時，前端介面應如何優化排版（Scannability），才能讓疲憊的 FDA 審查員在一秒鐘內看懂雙方爭論的核心科學矛盾點，而不是被迫閱讀數萬字的辯論日誌？

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

終極法律責任與 AI 決策追溯： 根據 2026 年最新醫療 AI 責任歸屬原則，任何經 AI 輔助生成的審查缺陷信函，最終法律責任均由簽名的 FDA 官員承擔。本平台的駕駛艙如何設計一套「最終決策確認清單（Final Signing Checkbox）」，強制審查員逐一滾動檢視所有被他手動覆核或忽略的 AI 警報，以在法律層面確保 Human-in-the-Loop 流程的形式與實質正義？   Sample webpage:

<title>Samsung Medison HERA Z20 | FDA 510(k) Compliance Portal</title> <script src="https://cdn.tailwindcss.com"></script> <script> tailwind.config = { theme: { extend: { colors: { slate: { 800: '#1E293B', 900: '#0F172A' }, coral: { 500: '#F87171' }, canvas: '#F8FAFC' }, fontFamily: { sans: ['Inter', 'sans-serif'], mono: ['JetBrains Mono', 'monospace'], serif: ['Merriweather', 'serif'] } } } } </script> <style> body { font-family: 'Inter', sans-serif; background-color: #F8FAFC; color: #1E293B; } .glass { background: rgba(255, 255, 255, 0.7); backdrop-filter: blur(10px); border: 1px solid rgba(255, 255, 255, 0.3); } .highlight-coral { background-color: rgba(248, 113, 113, 0.15); border-left: 3px solid #F87171; padding-left: 0.5rem; } ::-webkit-scrollbar { width: 6px; } ::-webkit-scrollbar-thumb { background: #CBD5E1; border-radius: 3px; } </style><!-- Header -->

<header class="h-16 glass flex items-center justify-between px-6 z-50 shadow-sm shrink-0">

    <div class="flex items-center gap-4">

        <h1 class="text-xl font-bold text-slate-900 tracking-tight">HERA<span class="text-coral-500 font-light">Z20</span> <span class="text-xs text-slate-400 font-mono">| FDA 510(k) Portal</span></h1>

    </div>

    <div class="flex items-center gap-4">

        <select class="bg-white border rounded-lg px-3 py-1 text-sm font-medium border-slate-200 outline-none">

            <option>Gemini 2.5 Flash</option>

        </select>

        <div class="flex gap-2">

            <button class="bg-slate-900 text-white px-4 py-1.5 rounded-lg text-sm font-semibold hover:bg-slate-700 transition">Export PDF</button>

            <button class="bg-white border border-slate-200 px-4 py-1.5 rounded-lg text-sm font-semibold hover:bg-slate-50 transition">Share</button>

        </div>

    </div>

</header>



<!-- Main Workspace -->

<main class="flex-1 flex overflow-hidden">

    

    <!-- Left Panel: Agent -->

    <aside class="w-80 border-r border-slate-200 flex flex-col bg-white overflow-hidden">

        <div class="p-4 border-b border-slate-100">

            <h2 class="text-xs font-bold text-slate-400 uppercase tracking-widest">Diagnostic Agent</h2>

            <div class="mt-4 h-2 bg-slate-100 rounded-full overflow-hidden">

                <div class="h-full bg-coral-500 w-[24%]"></div>

            </div>

            <p class="text-[10px] text-slate-500 mt-2">7 / 30 Questions Completed</p>

        </div>

        <div class="flex-1 overflow-y-auto p-4 space-y-4">

            <div class="p-3 bg-slate-50 rounded-lg text-sm border-l-2 border-slate-900">

                <p class="text-slate-600 font-medium">1. Model Specification</p>

                <p class="text-xs mt-1 text-slate-500">Subject model: HERA Z20. Verified against K241971 clearance.</p>

            </div>

            <div class="p-3 bg-coral-500/5 rounded-lg text-sm border-l-2 border-coral-500">

                <p class="text-coral-600 font-semibold text-xs mb-1 uppercase tracking-wider">Status: Awaiting Info</p>

                <p class="text-slate-700">Please attach the axial and lateral resolution specs for the CMV1-10 probe.</p>

            </div>

        </div>

    </aside>



    <!-- Center Workspace: Editor -->

    <section class="flex-1 flex flex-col">

        <div class="flex-1 grid grid-cols-2 divide-x divide-slate-200 overflow-hidden">

            <!-- Markdown Input -->

            <textarea class="w-full h-full p-8 font-mono text-sm bg-slate-50 outline-none resize-none focus:bg-white transition" placeholder="Draft your regulatory response here...">



Regulatory Gap Analysis: HERA Z20



1. Electrical Safety (IEC 60601-1)



Status: COMPLIANT Reference: Nemko Korea Report 2025.

2. AI/ML Functionality



Required: CADe/CADx technical documentation. [PENDING: User to input dataset provenance] </textarea>

            <!-- Preview Pane -->

            <div class="p-8 overflow-y-auto bg-white font-serif prose prose-slate max-w-none">

                <h1 class="text-3xl font-bold mb-4">510(k) Compliance: HERA Z20</h1>

                <div class="highlight-coral p-4 mb-6">

                    <h3 class="font-bold text-coral-600">Compliance Summary</h3>

                    <p class="text-sm">Based on internal validation, the system meets IEC 60601-1:2025. Attention required for probe specific resolution data (Ref: #3 in pending list).</p>

                </div>

                <p class="text-sm leading-relaxed">The Samsung Medison HERA Z20 is intended for diagnostic ultrasound imaging and body fluid analysis. The system supports multi-modal operation including 3D/4D, ELASTOSCAN+, and advanced AI features (ViewAssist, HeartAssist).</p>

            </div>

        </div>

    </section>



    <!-- Power Bar (Floating) -->

    <div class="w-16 border-l border-slate-200 bg-white flex flex-col items-center py-6 gap-4">

        <div class="group relative cursor-pointer" title="Gap Analysis">

            <div class="w-10 h-10 rounded-xl bg-slate-900 flex items-center justify-center text-white text-lg">⚖️</div>

        </div>

        <div class="group relative cursor-pointer" title="Evidence Mapping">

            <div class="w-10 h-10 rounded-xl bg-slate-100 flex items-center justify-center text-slate-600 text-lg">🔗</div>

        </div>

        <div class="group relative cursor-pointer" title="Standards Mapping">

            <div class="w-10 h-10 rounded-xl bg-slate-100 flex items-center justify-center text-slate-600 text-lg">🛡️</div>

        </div>

    </div>

</main>



<script>

    // Sync scrolling simulation

    const editor = document.querySelector('textarea');

    const preview = document.querySelector('.prose');

    editor.addEventListener('scroll', () => {

        preview.scrollTop = editor.scrollTop;

    });

</script>
