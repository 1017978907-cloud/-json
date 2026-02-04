<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SaaS 多站点文案智能生成器</title>
    <style>
        /* 页面整体风格 - 干净、现代 */
        body { font-family: -apple-system, "Segoe UI", Roboto, Helvetica, Arial, sans-serif; background-color: #f3f4f6; margin: 0; padding: 20px; color: #1f2937; }
        .container { max-width: 1400px; margin: 0 auto; }
        
        /* 顶部标题与站点选择 */
        .top-bar { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; background: white; padding: 15px 20px; border-radius: 12px; box-shadow: 0 1px 3px rgba(0,0,0,0.1); }
        h1 { margin: 0; font-size: 1.25rem; color: #111827; display: flex; align-items: center; gap: 10px; }
        .badge { background: #dbeafe; color: #1e40af; font-size: 0.75rem; padding: 4px 8px; border-radius: 4px; }

        /* 站点选择单选框 */
        .site-switch { display: flex; gap: 10px; background: #f3f4f6; padding: 5px; border-radius: 8px; }
        .site-option { cursor: pointer; padding: 8px 16px; border-radius: 6px; font-weight: 600; font-size: 0.9rem; transition: all 0.2s; border: 1px solid transparent; }
        .site-option:hover { background: #e5e7eb; }
        .site-option.active { background: #2563eb; color: white; shadow: 0 1px 2px rgba(0,0,0,0.1); }
        .site-option input { display: none; }

        /* API 配置区 (可折叠) */
        .api-config { background: #1f2937; color: #e5e7eb; padding: 15px 20px; border-radius: 12px; margin-bottom: 20px; }
        .api-header { display: flex; justify-content: space-between; cursor: pointer; font-weight: 600; font-size: 0.9rem; }
        .api-body { display: grid; grid-template-columns: 2fr 1fr 0.5fr; gap: 15px; margin-top: 15px; }
        .input-group label { display: block; font-size: 0.75rem; color: #9ca3af; margin-bottom: 5px; }
        .input-group input { width: 100%; background: #374151; border: 1px solid #4b5563; color: white; padding: 8px; border-radius: 6px; font-family: monospace; font-size: 0.85rem; box-sizing: border-box; }
        .input-group input:focus { border-color: #60a5fa; outline: none; }

        /* 主工作区 */
        .workspace { display: grid; grid-template-columns: 1fr 1fr; gap: 20px; height: calc(100vh - 220px); min-height: 600px; }
        
        /* 面板通用样式 */
        .panel { background: white; border-radius: 12px; border: 1px solid #e5e7eb; display: flex; flex-direction: column; overflow: hidden; box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.05); }
        .panel-header { padding: 15px; background: #f9fafb; border-bottom: 1px solid #e5e7eb; display: flex; justify-content: space-between; align-items: center; }
        .panel-title { font-weight: 600; color: #374151; font-size: 1rem; }
        
        textarea { flex: 1; border: none; padding: 20px; resize: none; font-family: "Menlo", "Monaco", "Courier New", monospace; font-size: 13px; line-height: 1.6; outline: none; }
        textarea::placeholder { color: #9ca3af; }

        /* 按钮样式 */
        button { border: none; border-radius: 6px; padding: 8px 16px; font-weight: 600; cursor: pointer; transition: 0.2s; font-size: 0.9rem; display: flex; align-items: center; gap: 6px; }
        .btn-run { background: #2563eb; color: white; }
        .btn-run:hover { background: #1d4ed8; }
        .btn-copy { background: #10b981; color: white; }
        .btn-copy:hover { background: #059669; }
        .btn-dl { background: #4b5563; color: white; }
        .btn-dl:hover { background: #374151; }

        /* 状态条 */
        .status-bar { padding: 10px 20px; font-size: 0.85rem; border-top: 1px solid #e5e7eb; background: #f9fafb; display: flex; align-items: center; color: #6b7280; }
        .status-dot { width: 8px; height: 8px; border-radius: 50%; background: #d1d5db; margin-right: 8px; }
        .status-dot.processing { background: #fbbf24; box-shadow: 0 0 0 2px rgba(251, 191, 36, 0.2); }
        .status-dot.success { background: #10b981; }
        .status-dot.error { background: #ef4444; }

    </style>
</head>
<body>

    <!-- 顶部导航 -->
    <div class="top-bar">
        <h1>
            SaaS 文案填充助手
            <span class="badge" id="currentSiteBadge">OpenMusic</span>
        </h1>
        
        <!-- 站点切换 -->
        <div class="site-switch">
            <label class="site-option active" onclick="switchSite('openMusic', this)">
                <input type="radio" name="site" checked> OpenMusic (Funk底板)
            </label>
            <label class="site-option" onclick="switchSite('musicArt', this)">
                <input type="radio" name="site"> MusicArt (Rap底板)
            </label>
            <label class="site-option" onclick="switchSite('musicAi', this)">
                <input type="radio" name="site"> MusicAI (MA组件)
            </label>
        </div>
    </div>

    <!-- API 设置 -->
    <div class="api-config">
        <div class="api-header" onclick="toggleApi()">
            <span>🛠️ API 设置 (点击折叠/展开)</span>
            <span style="opacity: 0.7; font-weight: normal;">默认使用 API Mart</span>
        </div>
        <div class="api-body" id="apiBody">
            <div class="input-group">
                <label>API Endpoint</label>
                <input type="text" id="apiUrl" value="https://api.apimart.ai/v1/chat/completions">
            </div>
            <div class="input-group">
                <label>API Key</label>
                <input type="password" id="apiKey" placeholder="输入 sk-...">
            </div>
            <div class="input-group">
                <label>Model</label>
                <input type="text" id="modelName" value="gpt-4o">
            </div>
        </div>
    </div>

    <!-- 工作区 -->
    <div class="workspace">
        <!-- 左侧 -->
        <div class="panel">
            <div class="panel-header">
                <span class="panel-title">1. 粘贴文案 (Raw Text)</span>
                <button class="btn-run" onclick="generateJson()">
                    <span>⚡ 开始生成</span>
                </button>
            </div>
            <textarea id="inputText" placeholder="请在此粘贴您的文案...
格式建议：
How to use (Block Name)
Your Tag Name Here
Your Title Here
Your Description Here..."></textarea>
            <div class="status-bar">
                <div class="status-dot" id="statusDot"></div>
                <span id="statusText">等待输入...</span>
            </div>
        </div>

        <!-- 右侧 -->
        <div class="panel">
            <div class="panel-header">
                <span class="panel-title">2. 生成结果 (JSON)</span>
                <div style="display: flex; gap: 10px;">
                    <button class="btn-copy" onclick="copyResult()">📋 复制</button>
                    <button class="btn-dl" onclick="downloadFile()">💾 下载</button>
                </div>
            </div>
            <textarea id="outputText" readonly placeholder="AI 生成的 JSON 将显示在这里。
我们会保留底板的所有样式、ID 和图片，仅替换文字。"></textarea>
        </div>
    </div>

<script>
    // ================== 1. 定义底板数据 (Hardcoded Templates) ==================
    
    // OpenMusic (基于 funk-music-generator.json - OP组件)
    const TEMPLATE_OPENMUSIC = {
      "modules": [
        {
          "id": "a05408e0-306a-4791-9f69-55d88bc7089d", "component": "OP1", "type": "OP1-type", "active": true, "style": "OP1Style",
          "config": { "theme": "light", "currentTheme": "openMusic", "bgColor": "bg-background", "TagShowInRightSetting": true, "TagName": "", "TagPositionIndex": 1, "TitleShowInRightSetting": true, "TitleLabelName": "标题", "TitleTranslateFlag": true, "TitleName": "", "TitlePositionIndex": 2, "DescriptionShowInRightSetting": true, "DescriptionLabelName": "描述", "DescriptionTranslateFlag": true, "DescriptionName": "", "DescriptionPositionIndex": 3, "CTALinkShowInRightSetting": true, "CTALinkShow": true, "CTALinkIsLink": true, "CTALinkLink": "/funk-music-generator", "CTALinkLinkRef": ["dofollow"], "CTALinkTranslateFlag": true, "CTALinkLabelName": "按钮", "CTALinkName": "Get Started", "CTALinkIconLeftSize": "(20px*20px)", "CTALinkIconLeftUseUpload": false, "CTALinkIconLeftImg": "", "CTALinkIconLeftIcon": "", "CTALinkPositionIndex": 4, "sectionList": [
              {"CardShowInRightSetting": true, "CardLabelName": "卡片1", "CardTitleShowInRightSetting": true, "CardTitleLabelName": "卡片标题", "CardTitleName": "", "CardTitleTranslateFlag": true, "CardTitlePositionIndex": 1, "CardDescriptionShowInRightSetting": true, "CardDescriptionLabelName": "卡片描述", "CardDescriptionName": "", "CardDescriptionTranslateFlag": true, "CardDescriptionPositionIndex": 2, "CardPositionIndex": 1},
              {"CardShowInRightSetting": true, "CardLabelName": "卡片2", "CardTitleShowInRightSetting": true, "CardTitleLabelName": "卡片标题", "CardTitleName": "", "CardTitleTranslateFlag": true, "CardTitlePositionIndex": 1, "CardDescriptionShowInRightSetting": true, "CardDescriptionLabelName": "卡片描述", "CardDescriptionName": "", "CardDescriptionTranslateFlag": true, "CardDescriptionPositionIndex": 2, "CardPositionIndex": 2},
              {"CardShowInRightSetting": true, "CardLabelName": "卡片3", "CardTitleShowInRightSetting": true, "CardTitleLabelName": "卡片标题", "CardTitleName": "", "CardTitleTranslateFlag": true, "CardTitlePositionIndex": 1, "CardDescriptionShowInRightSetting": true, "CardDescriptionLabelName": "卡片描述", "CardDescriptionName": "", "CardDescriptionTranslateFlag": true, "CardDescriptionPositionIndex": 2, "CardPositionIndex": 3}
          ], "source": [], "script": "" }
        },
        {
          "id": "c6fed41d-9c65-475c-af88-b26a44e23136", "component": "OP2", "type": "OP2-type", "active": false, "style": "OP2Style",
          "config": { "theme": "light", "currentTheme": "openMusic", "bgColor": "bg-background", "TagShowInRightSetting": true, "TagLabelName": "标签", "TagTranslateFlag": true, "TagName": "", "TagPositionIndex": 1, "TitleShowInRightSetting": true, "TitleLabelName": "标题", "TitleTranslateFlag": true, "TitleName": "", "TitlePositionIndex": 2, "DescriptionShowInRightSetting": true, "DescriptionLabelName": "描述", "DescriptionTranslateFlag": true, "DescriptionName": "", "DescriptionPositionIndex": 3, "CTALinkShowInRightSetting": true, "CTALinkShow": true, "CTALinkIsLink": true, "CTALinkLink": "", "CTALinkLinkRef": ["dofollow"], "CTALinkTranslateFlag": true, "CTALinkLabelName": "按钮", "CTALinkName": "", "CTALinkIconLeftSize": "(20px*20px)", "CTALinkIconLeftUseUpload": false, "CTALinkIconLeftImg": "", "CTALinkIconLeftIcon": "", "CTALinkPositionIndex": 4, "sectionList": [
              {"FeatureShowInRightSetting": true, "FeatureLabelName": "特性1", "FeatureTitleShowInRightSetting": true, "FeatureTitleLabelName": "特性标题", "FeatureTitleName": "", "FeatureTitleTranslateFlag": true, "FeatureTitlePositionIndex": 2, "FeatureDescriptionShowInRightSetting": true, "FeatureDescriptionLabelName": "特性描述", "FeatureDescriptionName": "", "FeatureDescriptionTranslateFlag": true, "FeatureDescriptionPositionIndex": 3, "FeatureImageShowInRightSetting": true, "FeatureImageLabelName": "特性图片", "FeatureImageUseUpload": true, "FeatureImageImg": "https://cdn.louhu.com/openmusic/images/176941894430618400.png", "FeatureImageSize": "(500px*400px)", "FeatureImagePositionIndex": 4, "FeatureIconShowInRightSetting": true, "FeatureIconLabelName": "图标", "FeatureIconUseUpload": false, "FeatureIconImg": "", "FeatureIconIcon": "", "FeatureIconSize": "(20px*20px)", "FeatureIconPositionIndex": 5, "FeatureCTAShowInRightSetting": true, "FeatureCTALabelName": "按钮", "FeatureCTAName": "", "FeatureCTALink": "/funk-music-generator", "FeatureCTALinkRef": ["dofollow"], "FeatureCTATranslateFlag": true, "FeatureCTAPositionIndex": 6, "FeatureIsReversed": false, "FeatureBadgeColor": "bg-green-100", "FeatureBadgeTextColor": "text-green-500", "FeaturePositionIndex": 1, "FeatureImageImgAlt": ""},
              {"FeatureShowInRightSetting": true, "FeatureLabelName": "特性2", "FeatureTitleShowInRightSetting": true, "FeatureTitleLabelName": "特性标题", "FeatureTitleName": "", "FeatureTitleTranslateFlag": true, "FeatureTitlePositionIndex": 2, "FeatureDescriptionShowInRightSetting": true, "FeatureDescriptionLabelName": "特性描述", "FeatureDescriptionName": "", "FeatureDescriptionTranslateFlag": true, "FeatureDescriptionPositionIndex": 3, "FeatureImageShowInRightSetting": true, "FeatureImageLabelName": "特性图片", "FeatureImageUseUpload": true, "FeatureImageImg": "https://cdn.louhu.com/openmusic/images/176941895353614391.png", "FeatureImageSize": "(500px*400px)", "FeatureImagePositionIndex": 4, "FeatureIconShowInRightSetting": true, "FeatureIconLabelName": "图标", "FeatureIconUseUpload": false, "FeatureIconImg": "", "FeatureIconSize": "(20px*20px)", "FeatureIconPositionIndex": 5, "FeatureCTAShowInRightSetting": true, "FeatureCTALabelName": "按钮", "FeatureCTAName": "", "FeatureCTALink": "/funk-music-generator", "FeatureCTALinkRef": ["dofollow"], "FeatureCTATranslateFlag": true, "FeatureCTAPositionIndex": 6, "FeatureIsReversed": true, "FeatureBadgeColor": "bg-blue-100", "FeatureBadgeTextColor": "text-blue-500", "FeaturePositionIndex": 2, "FeatureImageImgAlt": ""},
              {"FeatureShowInRightSetting": true, "FeatureLabelName": "特性3", "FeatureTitleShowInRightSetting": true, "FeatureTitleLabelName": "特性标题", "FeatureTitleName": "", "FeatureTitleTranslateFlag": true, "FeatureTitlePositionIndex": 2, "FeatureDescriptionShowInRightSetting": true, "FeatureDescriptionLabelName": "特性描述", "FeatureDescriptionName": "", "FeatureDescriptionTranslateFlag": true, "FeatureDescriptionPositionIndex": 3, "FeatureImageShowInRightSetting": true, "FeatureImageLabelName": "特性图片", "FeatureImageUseUpload": true, "FeatureImageImg": "https://cdn.louhu.com/openmusic/images/176941895786111765.png", "FeatureImageSize": "(500px*400px)", "FeatureImagePositionIndex": 4, "FeatureIconShowInRightSetting": true, "FeatureIconLabelName": "图标", "FeatureIconUseUpload": true, "FeatureIconImg": "", "FeatureIconSize": "(20px*20px)", "FeatureIconPositionIndex": 5, "FeatureCTAShowInRightSetting": true, "FeatureCTALabelName": "按钮", "FeatureCTAName": "", "FeatureCTALink": "/funk-music-generator", "FeatureCTALinkRef": ["dofollow"], "FeatureCTATranslateFlag": true, "FeatureCTAPositionIndex": 6, "FeatureIsReversed": false, "FeatureBadgeColor": "bg-purple-100", "FeatureBadgeTextColor": "text-purple-500", "FeaturePositionIndex": 3, "FeatureImageImgAlt": ""},
              {"FeatureShowInRightSetting": true, "FeatureLabelName": "特性1", "FeatureTitleShowInRightSetting": true, "FeatureTitleLabelName": "特性标题", "FeatureTitleName": "", "FeatureTitleTranslateFlag": true, "FeatureTitlePositionIndex": 2, "FeatureDescriptionShowInRightSetting": true, "FeatureDescriptionLabelName": "特性描述", "FeatureDescriptionName": "", "FeatureDescriptionTranslateFlag": true, "FeatureDescriptionPositionIndex": 3, "FeatureImageShowInRightSetting": true, "FeatureImageLabelName": "特性图片", "FeatureImageUseUpload": true, "FeatureImageImg": "https://cdn.louhu.com/openmusic/images/176941896497646021.png", "FeatureImageSize": "(500px*400px)", "FeatureImagePositionIndex": 4, "FeatureIconShowInRightSetting": true, "FeatureIconLabelName": "图标", "FeatureIconUseUpload": false, "FeatureIconImg": "", "FeatureIconIcon": "", "FeatureIconSize": "(20px*20px)", "FeatureIconPositionIndex": 5, "FeatureCTAShowInRightSetting": true, "FeatureCTALabelName": "按钮", "FeatureCTAName": "", "FeatureCTALink": "/funk-music-generator", "FeatureCTALinkRef": ["dofollow"], "FeatureCTATranslateFlag": true, "FeatureCTAPositionIndex": 6, "FeatureIsReversed": true, "FeatureBadgeColor": "bg-green-100", "FeatureBadgeTextColor": "text-green-500", "FeaturePositionIndex": 1, "FeatureImageImgAlt": ""}
          ], "source": [], "script": "" }
        },
        {
          "id": "1612ac90-0037-4483-b244-f4c342845c83", "component": "OP3", "type": "OP3-type", "active": false, "style": "OP3Style",
          "config": { "theme": "light", "currentTheme": "openMusic", "bgColor": "bg-background", "TitleShowInRightSetting": true, "TitleLabelName": "标题", "TitleTranslateFlag": true, "TitleName": "FAQ", "TitlePositionIndex": 1, "BackgroundImageShowInRightSetting": true, "BackgroundImageLabelName": "背景图片", "BackgroundImageUseUpload": true, "BackgroundImageImg": "", "BackgroundImageSize": "(1920px*1080px)", "BackgroundImagePositionIndex": 2, "sectionList": [
              {"FAQShowInRightSetting": true, "FAQLabelName": "FAQ项目1", "FAQQuestionShowInRightSetting": true, "FAQQuestionLabelName": "问题", "FAQQuestionName": "", "FAQQuestionTranslateFlag": true, "FAQQuestionPositionIndex": 2, "FAQAnswerShowInRightSetting": true, "FAQAnswerLabelName": "答案", "FAQAnswerName": "", "FAQAnswerTranslateFlag": true, "FAQAnswerPositionIndex": 3, "FAQPositionIndex": 1},
              {"FAQShowInRightSetting": true, "FAQLabelName": "FAQ项目2", "FAQQuestionShowInRightSetting": true, "FAQQuestionLabelName": "问题", "FAQQuestionName": "", "FAQQuestionTranslateFlag": true, "FAQQuestionPositionIndex": 2, "FAQAnswerShowInRightSetting": true, "FAQAnswerLabelName": "答案", "FAQAnswerName": "", "FAQAnswerTranslateFlag": true, "FAQAnswerPositionIndex": 3, "FAQPositionIndex": 2},
              {"FAQShowInRightSetting": true, "FAQLabelName": "FAQ项目1", "FAQQuestionShowInRightSetting": true, "FAQQuestionLabelName": "问题", "FAQQuestionName": "", "FAQQuestionTranslateFlag": true, "FAQQuestionPositionIndex": 2, "FAQAnswerShowInRightSetting": true, "FAQAnswerLabelName": "答案", "FAQAnswerName": "", "FAQAnswerTranslateFlag": true, "FAQAnswerPositionIndex": 3, "FAQPositionIndex": 1},
              {"FAQShowInRightSetting": true, "FAQLabelName": "FAQ项目1", "FAQQuestionShowInRightSetting": true, "FAQQuestionLabelName": "问题", "FAQQuestionName": "", "FAQQuestionTranslateFlag": true, "FAQQuestionPositionIndex": 2, "FAQAnswerShowInRightSetting": true, "FAQAnswerLabelName": "答案", "FAQAnswerName": "", "FAQAnswerTranslateFlag": true, "FAQAnswerPositionIndex": 3, "FAQPositionIndex": 1},
              {"FAQShowInRightSetting": true, "FAQLabelName": "FAQ项目1", "FAQQuestionShowInRightSetting": true, "FAQQuestionLabelName": "问题", "FAQQuestionName": "", "FAQQuestionTranslateFlag": true, "FAQQuestionPositionIndex": 2, "FAQAnswerShowInRightSetting": true, "FAQAnswerLabelName": "答案", "FAQAnswerName": "", "FAQAnswerTranslateFlag": true, "FAQAnswerPositionIndex": 3, "FAQPositionIndex": 1}
          ], "source": [], "script": "" }
        },
        {
          "id": "278d1d8c-d170-4e04-a1a6-d159ed53f6f5", "component": "OP4", "type": "OP4-type", "active": false, "style": "OP4Style",
          "config": { "theme": "light", "currentTheme": "openMusic", "bgColor": "bg-background", "TitleShowInRightSetting": true, "TitleLabelName": "标题", "TitleTranslateFlag": true, "TitleName": "", "TitlePositionIndex": 1, "DescriptionShowInRightSetting": true, "DescriptionLabelName": "描述", "DescriptionTranslateFlag": true, "DescriptionName": "", "DescriptionPositionIndex": 2, "TipShowInRightSetting": true, "TipLabelName": "提示", "TipTranslateFlag": true, "TipName": "", "TipPositionIndex": 3, "CTALinkShowInRightSetting": true, "CTALinkShow": true, "CTALinkIsLink": true, "CTALinkLink": "/funk-music-generator", "CTALinkLinkRef": ["dofollow"], "CTALinkTranslateFlag": true, "CTALinkLabelName": "按钮", "CTALinkName": "", "CTALinkIconLeftSize": "(20px*20px)", "CTALinkIconLeftUseUpload": false, "CTALinkIconLeftImg": "", "CTALinkIconLeftIcon": "", "CTALinkPositionIndex": 4, "CTALinkCTALinkSpecialCss": "bg-foreground hover:bg-foreground/90 active:bg-foreground/80", "BackgroundImageShowInRightSetting": true, "BackgroundImageLabelName": "背景图片", "BackgroundImageUseUpload": true, "BackgroundImageImg": "https://cdn.louhu.com/176164571810469445.png", "BackgroundImageSize": "(1920px*1080px)", "BackgroundImagePositionIndex": 5, "source": [], "script": "" }
        }
      ]
    };

    // MusicArt (基于 rap-beat-maker.json - MR组件)
    const TEMPLATE_MUSICART = {
      "modules": [
        {
          "id": "48103220-8686-4584-9a4b-5551fe9c5ebb", "component": "MR1", "type": "MR1-type", "active": true, "style": "MR1Style",
          "config": { "theme": "light", "currentTheme": "musicArt", "bgColor": "bg-background", "TagShowInRightSetting": true, "TagLabelName": "标签", "TagTranslateFlag": true, "TagName": "", "TagPositionIndex": 1, "TitleShowInRightSetting": true, "TitleLabelName": "标题", "TitleTranslateFlag": true, "TitleName": "", "TitlePositionIndex": 2, "DescriptionShowInRightSetting": true, "DescriptionLabelName": "描述", "DescriptionTranslateFlag": true, "DescriptionName": "", "DescriptionPositionIndex": 3, "CTALinkShowInRightSetting": true, "CTALinkShow": true, "CTALinkIsLink": true, "CTALinkLink": "/rap-beat-maker", "CTALinkTranslateFlag": true, "CTALinkLabelName": "按钮", "CTALinkName": "Make Beats", "CTALinkIconLeftSize": "(20px*20px)", "CTALinkIconLeftUseUpload": false, "CTALinkIconLeftImg": "", "CTALinkIconLeftIcon": "", "CTALinkPositionIndex": 4, "sectionList": [
              {"CardShowInRightSetting": true, "CardLabelName": "卡片1", "CardTitleShowInRightSetting": true, "CardTitleLabelName": "卡片标题", "CardTitleName": "", "CardTitleTranslateFlag": true, "CardTitlePositionIndex": 1, "CardDescriptionShowInRightSetting": true, "CardDescriptionLabelName": "卡片描述", "CardDescriptionName": "", "CardDescriptionTranslateFlag": true, "CardDescriptionPositionIndex": 2, "CardPositionIndex": 1},
              {"CardShowInRightSetting": true, "CardLabelName": "卡片2", "CardTitleShowInRightSetting": true, "CardTitleLabelName": "卡片标题", "CardTitleName": "", "CardTitleTranslateFlag": true, "CardTitlePositionIndex": 1, "CardDescriptionShowInRightSetting": true, "CardDescriptionLabelName": "卡片描述", "CardDescriptionName": "", "CardDescriptionTranslateFlag": true, "CardDescriptionPositionIndex": 2, "CardPositionIndex": 2},
              {"CardShowInRightSetting": true, "CardLabelName": "卡片3", "CardTitleShowInRightSetting": true, "CardTitleLabelName": "卡片标题", "CardTitleName": "", "CardTitleTranslateFlag": true, "CardTitlePositionIndex": 1, "CardDescriptionShowInRightSetting": true, "CardDescriptionLabelName": "卡片描述", "CardDescriptionName": "", "CardDescriptionTranslateFlag": true, "CardDescriptionPositionIndex": 2, "CardPositionIndex": 3}
          ], "source": [], "script": "" }
        },
        {
          "id": "a759598f-48f7-46ad-b194-32c67dac4b79", "component": "MR2", "type": "MR2-type", "active": false, "style": "MR2Style",
          "config": { "theme": "light", "currentTheme": "musicArt", "bgColor": "bg-background", "TagShowInRightSetting": true, "TagLabelName": "标签", "TagTranslateFlag": true, "TagName": "", "TagPositionIndex": 1, "TitleShowInRightSetting": true, "TitleLabelName": "标题", "TitleTranslateFlag": true, "TitleName": "", "TitlePositionIndex": 2, "DescriptionShowInRightSetting": true, "DescriptionLabelName": "描述", "DescriptionTranslateFlag": true, "DescriptionName": "", "DescriptionPositionIndex": 3, "CTALinkShowInRightSetting": true, "CTALinkShow": true, "CTALinkIsLink": true, "CTALinkLink": "", "CTALinkTranslateFlag": true, "CTALinkLabelName": "按钮", "CTALinkName": "", "CTALinkIconLeftSize": "(20px*20px)", "CTALinkIconLeftUseUpload": false, "CTALinkIconLeftImg": "", "CTALinkIconLeftIcon": "", "CTALinkPositionIndex": 4, "sectionList": [
              {"FeatureShowInRightSetting": true, "FeatureLabelName": "特性1", "FeatureTitleShowInRightSetting": true, "FeatureTitleLabelName": "特性标题", "FeatureTitleName": "", "FeatureTitleTranslateFlag": true, "FeatureTitlePositionIndex": 2, "FeatureDescriptionShowInRightSetting": true, "FeatureDescriptionLabelName": "特性描述", "FeatureDescriptionName": "", "FeatureDescriptionTranslateFlag": true, "FeatureDescriptionPositionIndex": 3, "FeatureImageShowInRightSetting": true, "FeatureImageLabelName": "特性图片", "FeatureImageUseUpload": true, "FeatureImageImg": "https://cdn.louhu.com/musicart/images/176759812394717909.png", "FeatureImageSize": "(500px*400px)", "FeatureImagePositionIndex": 4, "FeatureIconShowInRightSetting": true, "FeatureIconLabelName": "图标", "FeatureIconUseUpload": false, "FeatureIconImg": "", "FeatureIconIcon": "", "FeatureIconSize": "(20px*20px)", "FeatureIconPositionIndex": 5, "FeatureCTAShowInRightSetting": true, "FeatureCTALabelName": "按钮", "FeatureCTAName": "", "FeatureCTALink": "/rap-beat-maker", "FeatureCTATranslateFlag": true, "FeatureCTAPositionIndex": 6, "FeatureIsReversed": false, "FeatureBadgeColor": "bg-green-100", "FeatureBadgeTextColor": "text-green-500", "FeaturePositionIndex": 1, "FeatureImageImgAlt": ""},
              {"FeatureShowInRightSetting": true, "FeatureLabelName": "特性2", "FeatureTitleShowInRightSetting": true, "FeatureTitleLabelName": "特性标题", "FeatureTitleName": "", "FeatureTitleTranslateFlag": true, "FeatureTitlePositionIndex": 2, "FeatureDescriptionShowInRightSetting": true, "FeatureDescriptionLabelName": "特性描述", "FeatureDescriptionName": "", "FeatureDescriptionTranslateFlag": true, "FeatureDescriptionPositionIndex": 3, "FeatureImageShowInRightSetting": true, "FeatureImageLabelName": "特性图片", "FeatureImageUseUpload": true, "FeatureImageImg": "https://cdn.louhu.com/musicart/images/176759812988396035.png", "FeatureImageSize": "(500px*400px)", "FeatureImagePositionIndex": 4, "FeatureIconShowInRightSetting": true, "FeatureIconLabelName": "图标", "FeatureIconUseUpload": true, "FeatureIconImg": "", "FeatureIconSize": "(20px*20px)", "FeatureIconPositionIndex": 5, "FeatureCTAShowInRightSetting": true, "FeatureCTALabelName": "按钮", "FeatureCTAName": "", "FeatureCTALink": "/rap-beat-maker", "FeatureCTATranslateFlag": true, "FeatureCTAPositionIndex": 6, "FeatureIsReversed": true, "FeatureBadgeColor": "bg-blue-100", "FeatureBadgeTextColor": "text-blue-500", "FeaturePositionIndex": 2, "FeatureImageImgAlt": ""},
              {"FeatureShowInRightSetting": true, "FeatureLabelName": "特性3", "FeatureTitleShowInRightSetting": true, "FeatureTitleLabelName": "特性标题", "FeatureTitleName": "", "FeatureTitleTranslateFlag": true, "FeatureTitlePositionIndex": 2, "FeatureDescriptionShowInRightSetting": true, "FeatureDescriptionLabelName": "特性描述", "FeatureDescriptionName": "", "FeatureDescriptionTranslateFlag": true, "FeatureDescriptionPositionIndex": 3, "FeatureImageShowInRightSetting": true, "FeatureImageLabelName": "特性图片", "FeatureImageUseUpload": true, "FeatureImageImg": "https://cdn.louhu.com/musicart/images/176759813772737613.png", "FeatureImageSize": "(500px*400px)", "FeatureImagePositionIndex": 4, "FeatureIconShowInRightSetting": true, "FeatureIconLabelName": "图标", "FeatureIconUseUpload": true, "FeatureIconImg": "", "FeatureIconSize": "(20px*20px)", "FeatureIconPositionIndex": 5, "FeatureCTAShowInRightSetting": true, "FeatureCTALabelName": "按钮", "FeatureCTAName": "", "FeatureCTALink": "/rap-beat-maker", "FeatureCTATranslateFlag": true, "FeatureCTAPositionIndex": 6, "FeatureIsReversed": false, "FeatureBadgeColor": "bg-purple-100", "FeatureBadgeTextColor": "text-purple-500", "FeaturePositionIndex": 3, "FeatureImageImgAlt": ""},
              {"FeatureShowInRightSetting": true, "FeatureLabelName": "特性1", "FeatureTitleShowInRightSetting": true, "FeatureTitleLabelName": "特性标题", "FeatureTitleName": "", "FeatureTitleTranslateFlag": true, "FeatureTitlePositionIndex": 2, "FeatureDescriptionShowInRightSetting": true, "FeatureDescriptionLabelName": "特性描述", "FeatureDescriptionName": "", "FeatureDescriptionTranslateFlag": true, "FeatureDescriptionPositionIndex": 3, "FeatureImageShowInRightSetting": true, "FeatureImageLabelName": "特性图片", "FeatureImageUseUpload": true, "FeatureImageImg": "https://cdn.louhu.com/musicart/images/176759814569498134.png", "FeatureImageSize": "(500px*400px)", "FeatureImagePositionIndex": 4, "FeatureIconShowInRightSetting": true, "FeatureIconLabelName": "图标", "FeatureIconUseUpload": false, "FeatureIconImg": "", "FeatureIconIcon": "", "FeatureIconSize": "(20px*20px)", "FeatureIconPositionIndex": 5, "FeatureCTAShowInRightSetting": true, "FeatureCTALabelName": "按钮", "FeatureCTAName": "", "FeatureCTALink": "/rap-beat-maker", "FeatureCTATranslateFlag": true, "FeatureCTAPositionIndex": 6, "FeatureIsReversed": true, "FeatureBadgeColor": "bg-green-100", "FeatureBadgeTextColor": "text-green-500", "FeaturePositionIndex": 1, "FeatureImageImgAlt": ""}
          ], "source": [], "script": "" }
        },
        {
          "id": "7f3dbf60-372a-45ff-a257-c407940e4190", "component": "MR3", "type": "MR3-type", "active": false, "style": "MR3Style",
          "config": { "theme": "light", "currentTheme": "musicArt", "bgColor": "bg-background", "TitleShowInRightSetting": true, "TitleLabelName": "标题", "TitleTranslateFlag": true, "TitleName": "FAQ", "TitlePositionIndex": 1, "BackgroundImageShowInRightSetting": true, "BackgroundImageLabelName": "背景图片", "BackgroundImageUseUpload": true, "BackgroundImageImg": "https://cdn.louhu.com/176353448745196638.png", "BackgroundImageSize": "(1920px*1080px)", "BackgroundImagePositionIndex": 2, "sectionList": [
              {"FAQShowInRightSetting": true, "FAQLabelName": "FAQ项目1", "FAQQuestionShowInRightSetting": true, "FAQQuestionLabelName": "问题", "FAQQuestionName": "", "FAQQuestionTranslateFlag": true, "FAQQuestionPositionIndex": 2, "FAQAnswerShowInRightSetting": true, "FAQAnswerLabelName": "答案", "FAQAnswerName": "", "FAQAnswerTranslateFlag": true, "FAQAnswerPositionIndex": 3, "FAQPositionIndex": 1},
              {"FAQShowInRightSetting": true, "FAQLabelName": "FAQ项目2", "FAQQuestionShowInRightSetting": true, "FAQQuestionLabelName": "问题", "FAQQuestionName": "", "FAQQuestionTranslateFlag": true, "FAQQuestionPositionIndex": 2, "FAQAnswerShowInRightSetting": true, "FAQAnswerLabelName": "答案", "FAQAnswerName": "", "FAQAnswerTranslateFlag": true, "FAQAnswerPositionIndex": 3, "FAQPositionIndex": 2},
              {"FAQShowInRightSetting": true, "FAQLabelName": "FAQ项目1", "FAQQuestionShowInRightSetting": true, "FAQQuestionLabelName": "问题", "FAQQuestionName": "", "FAQQuestionTranslateFlag": true, "FAQQuestionPositionIndex": 2, "FAQAnswerShowInRightSetting": true, "FAQAnswerLabelName": "答案", "FAQAnswerName": "", "FAQAnswerTranslateFlag": true, "FAQAnswerPositionIndex": 3, "FAQPositionIndex": 1},
              {"FAQShowInRightSetting": true, "FAQLabelName": "FAQ项目1", "FAQQuestionShowInRightSetting": true, "FAQQuestionLabelName": "问题", "FAQQuestionName": "", "FAQQuestionTranslateFlag": true, "FAQQuestionPositionIndex": 2, "FAQAnswerShowInRightSetting": true, "FAQAnswerLabelName": "答案", "FAQAnswerName": "", "FAQAnswerTranslateFlag": true, "FAQAnswerPositionIndex": 3, "FAQPositionIndex": 1},
              {"FAQShowInRightSetting": true, "FAQLabelName": "FAQ项目1", "FAQQuestionShowInRightSetting": true, "FAQQuestionLabelName": "问题", "FAQQuestionName": "", "FAQQuestionTranslateFlag": true, "FAQQuestionPositionIndex": 2, "FAQAnswerShowInRightSetting": true, "FAQAnswerLabelName": "答案", "FAQAnswerName": "", "FAQAnswerTranslateFlag": true, "FAQAnswerPositionIndex": 3, "FAQPositionIndex": 1}
          ], "source": [], "script": "" }
        },
        {
          "id": "abe20b43-f71f-4eaa-8912-fe6345dd493e", "component": "MR4", "type": "MR4-type", "active": false, "style": "MR4Style",
          "config": { "theme": "light", "currentTheme": "musicArt", "bgColor": "bg-background", "TitleShowInRightSetting": true, "TitleLabelName": "标题", "TitleTranslateFlag": true, "TitleName": "", "TitlePositionIndex": 1, "DescriptionShowInRightSetting": true, "DescriptionLabelName": "描述", "DescriptionTranslateFlag": true, "DescriptionName": "", "DescriptionPositionIndex": 2, "TipShowInRightSetting": true, "TipLabelName": "提示", "TipTranslateFlag": true, "TipName": "", "TipPositionIndex": 3, "CTALinkShowInRightSetting": true, "CTALinkShow": true, "CTALinkIsLink": true, "CTALinkLink": "/rap-beat-maker", "CTALinkTranslateFlag": true, "CTALinkLabelName": "按钮", "CTALinkName": "", "CTALinkIconLeftSize": "(20px*20px)", "CTALinkIconLeftUseUpload": false, "CTALinkIconLeftImg": "", "CTALinkIconLeftIcon": "", "CTALinkPositionIndex": 4, "CTALinkCTALinkSpecialCss": "bg-foreground hover:bg-foreground/90 active:bg-foreground/80", "BackgroundImageShowInRightSetting": true, "BackgroundImageLabelName": "背景图片", "BackgroundImageUseUpload": true, "BackgroundImageImg": "https://cdn.louhu.com/176353432298086131.png", "BackgroundImageSize": "(1920px*1080px)", "BackgroundImagePositionIndex": 5, "source": [], "script": "" }
        }
      ]
    };

    // MusicAI (基于 musicArt 结构变体 - MA组件)
    const TEMPLATE_MUSICAI = {
      "modules": [
        {
          "id": "7b488bb5-424b-450e-843f-a0fd5cee67c3", "component": "MA1", "type": "MA1-type", "active": true, "style": "MA1Style",
          "config": { "theme": "light", "currentTheme": "musicAi", "bgColor": "bg-background", "TagShowInRightSetting": true, "TagLabelName": "标签", "TagTranslateFlag": true, "TagName": "", "TagPositionIndex": 1, "TitleShowInRightSetting": true, "TitleLabelName": "标题", "TitleTranslateFlag": true, "TitleName": "", "TitlePositionIndex": 2, "DescriptionShowInRightSetting": true, "DescriptionLabelName": "描述", "DescriptionTranslateFlag": true, "DescriptionName": "", "DescriptionPositionIndex": 3, "CTALinkShowInRightSetting": true, "CTALinkShow": true, "CTALinkIsLink": true, "CTALinkLink": "/rap-beat-maker", "CTALinkTranslateFlag": true, "CTALinkLabelName": "按钮", "CTALinkName": "Try Now", "CTALinkIconLeftSize": "(20px*20px)", "CTALinkIconLeftUseUpload": false, "CTALinkIconLeftImg": "", "CTALinkIconLeftIcon": "", "CTALinkPositionIndex": 4, "sectionList": [
              {"CardShowInRightSetting": true, "CardLabelName": "卡片1", "CardTitleShowInRightSetting": true, "CardTitleLabelName": "卡片标题", "CardTitleName": "", "CardTitleTranslateFlag": true, "CardTitlePositionIndex": 1, "CardDescriptionShowInRightSetting": true, "CardDescriptionLabelName": "卡片描述", "CardDescriptionName": "", "CardDescriptionTranslateFlag": true, "CardDescriptionPositionIndex": 2, "CardPositionIndex": 1},
              {"CardShowInRightSetting": true, "CardLabelName": "卡片2", "CardTitleShowInRightSetting": true, "CardTitleLabelName": "卡片标题", "CardTitleName": "", "CardTitleTranslateFlag": true, "CardTitlePositionIndex": 1, "CardDescriptionShowInRightSetting": true, "CardDescriptionLabelName": "卡片描述", "CardDescriptionName": "", "CardDescriptionTranslateFlag": true, "CardDescriptionPositionIndex": 2, "CardPositionIndex": 2},
              {"CardShowInRightSetting": true, "CardLabelName": "卡片3", "CardTitleShowInRightSetting": true, "CardTitleLabelName": "卡片标题", "CardTitleName": "", "CardTitleTranslateFlag": true, "CardTitlePositionIndex": 1, "CardDescriptionShowInRightSetting": true, "CardDescriptionLabelName": "卡片描述", "CardDescriptionName": "", "CardDescriptionTranslateFlag": true, "CardDescriptionPositionIndex": 2, "CardPositionIndex": 3}
          ], "source": [], "script": "" }
        },
        {
          "id": "8128d2a9-2ab9-4754-bcc2-17e16c681bab", "component": "MA2", "type": "MA2-type", "active": false, "style": "MA2Style",
          "config": { "theme": "light", "currentTheme": "musicAi", "bgColor": "bg-background", "TagShowInRightSetting": true, "TagLabelName": "标签", "TagTranslateFlag": true, "TagName": "", "TagPositionIndex": 1, "TitleShowInRightSetting": true, "TitleLabelName": "标题", "TitleTranslateFlag": true, "TitleName": "", "TitlePositionIndex": 2, "DescriptionShowInRightSetting": true, "DescriptionLabelName": "描述", "DescriptionTranslateFlag": true, "DescriptionName": "", "DescriptionPositionIndex": 3, "CTALinkShowInRightSetting": true, "CTALinkShow": true, "CTALinkIsLink": true, "CTALinkLink": "", "CTALinkTranslateFlag": true, "CTALinkLabelName": "按钮", "CTALinkName": "", "CTALinkIconLeftSize": "(20px*20px)", "CTALinkIconLeftUseUpload": false, "CTALinkIconLeftImg": "", "CTALinkIconLeftIcon": "", "CTALinkPositionIndex": 4, "sectionList": [
              {"FeatureShowInRightSetting": true, "FeatureLabelName": "特性1", "FeatureTitleShowInRightSetting": true, "FeatureTitleLabelName": "特性标题", "FeatureTitleName": "", "FeatureTitleTranslateFlag": true, "FeatureTitlePositionIndex": 2, "FeatureDescriptionShowInRightSetting": true, "FeatureDescriptionLabelName": "特性描述", "FeatureDescriptionName": "", "FeatureDescriptionTranslateFlag": true, "FeatureDescriptionPositionIndex": 3, "FeatureImageShowInRightSetting": true, "FeatureImageLabelName": "特性图片", "FeatureImageUseUpload": true, "FeatureImageImg": "https://cdn.louhu.com/musicai/images/176760525578921674.png", "FeatureImageSize": "(500px*400px)", "FeatureImagePositionIndex": 4, "FeatureIconShowInRightSetting": true, "FeatureIconLabelName": "图标", "FeatureIconUseUpload": false, "FeatureIconImg": "", "FeatureIconIcon": "", "FeatureIconSize": "(20px*20px)", "FeatureIconPositionIndex": 5, "FeatureCTAShowInRightSetting": true, "FeatureCTALabelName": "按钮", "FeatureCTAName": "", "FeatureCTALink": "/rap-beat-maker", "FeatureCTATranslateFlag": true, "FeatureCTAPositionIndex": 6, "FeatureIsReversed": false, "FeatureBadgeColor": "bg-green-100", "FeatureBadgeTextColor": "text-green-500", "FeaturePositionIndex": 1, "FeatureImageImgAlt": ""},
              {"FeatureShowInRightSetting": true, "FeatureLabelName": "特性3", "FeatureTitleShowInRightSetting": true, "FeatureTitleLabelName": "特性标题", "FeatureTitleName": "", "FeatureTitleTranslateFlag": true, "FeatureTitlePositionIndex": 2, "FeatureDescriptionShowInRightSetting": true, "FeatureDescriptionLabelName": "特性描述", "FeatureDescriptionName": "", "FeatureDescriptionTranslateFlag": true, "FeatureDescriptionPositionIndex": 3, "FeatureImageShowInRightSetting": true, "FeatureImageLabelName": "特性图片", "FeatureImageUseUpload": true, "FeatureImageImg": "https://cdn.louhu.com/musicai/images/176760528071637431.png", "FeatureImageSize": "(500px*400px)", "FeatureImagePositionIndex": 4, "FeatureIconShowInRightSetting": true, "FeatureIconLabelName": "图标", "FeatureIconUseUpload": true, "FeatureIconImg": "", "FeatureIconSize": "(20px*20px)", "FeatureIconPositionIndex": 5, "FeatureCTAShowInRightSetting": true, "FeatureCTALabelName": "按钮", "FeatureCTAName": "", "FeatureCTALink": "/rap-beat-maker", "FeatureCTATranslateFlag": true, "FeatureCTAPositionIndex": 6, "FeatureIsReversed": true, "FeatureBadgeColor": "bg-purple-100", "FeatureBadgeTextColor": "text-purple-500", "FeaturePositionIndex": 3, "FeatureImageImgAlt": ""},
              {"FeatureShowInRightSetting": true, "FeatureLabelName": "特性2", "FeatureTitleShowInRightSetting": true, "FeatureTitleLabelName": "特性标题", "FeatureTitleName": "", "FeatureTitleTranslateFlag": true, "FeatureTitlePositionIndex": 2, "FeatureDescriptionShowInRightSetting": true, "FeatureDescriptionLabelName": "特性描述", "FeatureDescriptionName": "", "FeatureDescriptionTranslateFlag": true, "FeatureDescriptionPositionIndex": 3, "FeatureImageShowInRightSetting": true, "FeatureImageLabelName": "特性图片", "FeatureImageUseUpload": true, "FeatureImageImg": "https://cdn.louhu.com/musicai/images/176760526343472529.png", "FeatureImageSize": "(500px*400px)", "FeatureImagePositionIndex": 4, "FeatureIconShowInRightSetting": true, "FeatureIconLabelName": "图标", "FeatureIconUseUpload": true, "FeatureIconImg": "", "FeatureIconSize": "(20px*20px)", "FeatureIconPositionIndex": 5, "FeatureCTAShowInRightSetting": true, "FeatureCTALabelName": "按钮", "FeatureCTAName": "", "FeatureCTALink": "/rap-beat-maker", "FeatureCTATranslateFlag": true, "FeatureCTAPositionIndex": 6, "FeatureIsReversed": false, "FeatureBadgeColor": "bg-blue-100", "FeatureBadgeTextColor": "text-blue-500", "FeaturePositionIndex": 2, "FeatureImageImgAlt": ""},
              {"FeatureShowInRightSetting": true, "FeatureLabelName": "特性1", "FeatureTitleShowInRightSetting": true, "FeatureTitleLabelName": "特性标题", "FeatureTitleName": "", "FeatureTitleTranslateFlag": true, "FeatureTitlePositionIndex": 2, "FeatureDescriptionShowInRightSetting": true, "FeatureDescriptionLabelName": "特性描述", "FeatureDescriptionName": "", "FeatureDescriptionTranslateFlag": true, "FeatureDescriptionPositionIndex": 3, "FeatureImageShowInRightSetting": true, "FeatureImageLabelName": "特性图片", "FeatureImageUseUpload": true, "FeatureImageImg": "https://cdn.louhu.com/musicai/images/176760528879055816.png", "FeatureImageSize": "(500px*400px)", "FeatureImagePositionIndex": 4, "FeatureIconShowInRightSetting": true, "FeatureIconLabelName": "图标", "FeatureIconUseUpload": false, "FeatureIconImg": "", "FeatureIconIcon": "", "FeatureIconSize": "(20px*20px)", "FeatureIconPositionIndex": 5, "FeatureCTAShowInRightSetting": true, "FeatureCTALabelName": "按钮", "FeatureCTAName": "", "FeatureCTALink": "/rap-beat-maker", "FeatureCTATranslateFlag": true, "FeatureCTAPositionIndex": 6, "FeatureIsReversed": true, "FeatureBadgeColor": "bg-green-100", "FeatureBadgeTextColor": "text-green-500", "FeaturePositionIndex": 1, "FeatureImageImgAlt": ""}
          ], "source": [], "script": "" }
        },
        {
          "id": "e1f96fc2-48ab-4b09-bd19-d8b476334ab0", "component": "MA3", "type": "MA3-type", "active": false, "style": "MA3Style",
          "config": { "theme": "light", "currentTheme": "musicAi", "bgColor": "bg-background", "TitleShowInRightSetting": true, "TitleLabelName": "标题", "TitleTranslateFlag": true, "TitleName": "FAQ", "TitlePositionIndex": 1, "BackgroundImageShowInRightSetting": true, "BackgroundImageLabelName": "背景图片", "BackgroundImageUseUpload": true, "BackgroundImageImg": "", "BackgroundImageSize": "(1920px*1080px)", "BackgroundImagePositionIndex": 2, "sectionList": [
              {"FAQShowInRightSetting": true, "FAQLabelName": "FAQ项目1", "FAQQuestionShowInRightSetting": true, "FAQQuestionLabelName": "问题", "FAQQuestionName": "", "FAQQuestionTranslateFlag": true, "FAQQuestionPositionIndex": 2, "FAQAnswerShowInRightSetting": true, "FAQAnswerLabelName": "答案", "FAQAnswerName": "", "FAQAnswerTranslateFlag": true, "FAQAnswerPositionIndex": 3, "FAQPositionIndex": 1},
              {"FAQShowInRightSetting": true, "FAQLabelName": "FAQ项目2", "FAQQuestionShowInRightSetting": true, "FAQQuestionLabelName": "问题", "FAQQuestionName": "", "FAQQuestionTranslateFlag": true, "FAQQuestionPositionIndex": 2, "FAQAnswerShowInRightSetting": true, "FAQAnswerLabelName": "答案", "FAQAnswerName": "", "FAQAnswerTranslateFlag": true, "FAQAnswerPositionIndex": 3, "FAQPositionIndex": 2},
              {"FAQShowInRightSetting": true, "FAQLabelName": "FAQ项目1", "FAQQuestionShowInRightSetting": true, "FAQQuestionLabelName": "问题", "FAQQuestionName": "", "FAQQuestionTranslateFlag": true, "FAQQuestionPositionIndex": 2, "FAQAnswerShowInRightSetting": true, "FAQAnswerLabelName": "答案", "FAQAnswerName": "", "FAQAnswerTranslateFlag": true, "FAQAnswerPositionIndex": 3, "FAQPositionIndex": 1},
              {"FAQShowInRightSetting": true, "FAQLabelName": "FAQ项目1", "FAQQuestionShowInRightSetting": true, "FAQQuestionLabelName": "问题", "FAQQuestionName": "", "FAQQuestionTranslateFlag": true, "FAQQuestionPositionIndex": 2, "FAQAnswerShowInRightSetting": true, "FAQAnswerLabelName": "答案", "FAQAnswerName": "", "FAQAnswerTranslateFlag": true, "FAQAnswerPositionIndex": 3, "FAQPositionIndex": 1},
              {"FAQShowInRightSetting": true, "FAQLabelName": "FAQ项目1", "FAQQuestionShowInRightSetting": true, "FAQQuestionLabelName": "问题", "FAQQuestionName": "", "FAQQuestionTranslateFlag": true, "FAQQuestionPositionIndex": 2, "FAQAnswerShowInRightSetting": true, "FAQAnswerLabelName": "答案", "FAQAnswerName": "", "FAQAnswerTranslateFlag": true, "FAQAnswerPositionIndex": 3, "FAQPositionIndex": 1}
          ], "source": [], "script": "" }
        },
        {
          "id": "d4132d9a-a43b-46b3-8674-3eac68f5163b", "component": "MA4", "type": "MA4-type", "active": false, "style": "MA4Style",
          "config": { "theme": "light", "currentTheme": "musicAi", "bgColor": "bg-background", "TitleShowInRightSetting": true, "TitleLabelName": "标题", "TitleTranslateFlag": true, "TitleName": "", "TitlePositionIndex": 1, "DescriptionShowInRightSetting": true, "DescriptionLabelName": "描述", "DescriptionTranslateFlag": true, "DescriptionName": "", "DescriptionPositionIndex": 2, "TipShowInRightSetting": true, "TipLabelName": "提示", "TipTranslateFlag": true, "TipName": "", "TipPositionIndex": 3, "CTALinkShowInRightSetting": true, "CTALinkShow": true, "CTALinkIsLink": true, "CTALinkLink": "/rap-beat-maker", "CTALinkTranslateFlag": true, "CTALinkLabelName": "按钮", "CTALinkName": "", "CTALinkIconLeftSize": "(20px*20px)", "CTALinkIconLeftUseUpload": false, "CTALinkIconLeftImg": "", "CTALinkIconLeftIcon": "", "CTALinkPositionIndex": 4, "CTALinkCTALinkSpecialCss": "bg-foreground hover:bg-foreground/90 active:bg-foreground/80", "BackgroundImageShowInRightSetting": true, "BackgroundImageLabelName": "背景图片", "BackgroundImageUseUpload": true, "BackgroundImageImg": "https://cdn.louhu.com/176352141513919785.png", "BackgroundImageSize": "(1920px*1080px)", "BackgroundImagePositionIndex": 5, "source": [], "script": "" }
        }
      ]
    };

    // 当前选中的模板
    let currentTemplate = TEMPLATE_OPENMUSIC;

    // ================== 2. 逻辑控制 ==================

    function switchSite(site, el) {
        document.querySelectorAll('.site-option').forEach(item => item.classList.remove('active'));
        el.classList.add('active');
        
        const badge = document.getElementById('currentSiteBadge');
        if (site === 'openMusic') {
            currentTemplate = TEMPLATE_OPENMUSIC;
            badge.innerText = 'OpenMusic';
        } else if (site === 'musicArt') {
            currentTemplate = TEMPLATE_MUSICART;
            badge.innerText = 'MusicArt';
        } else if (site === 'musicAi') {
            currentTemplate = TEMPLATE_MUSICAI;
            badge.innerText = 'MusicAI';
        }
        
        setStatus(`已切换底板为：${badge.innerText}。准备就绪。`);
        document.getElementById('outputText').value = ""; // 清空旧结果
    }

    function toggleApi() {
        const body = document.getElementById('apiBody');
        body.style.display = body.style.display === 'none' ? 'grid' : 'none';
    }

    function setStatus(msg, type = 'normal') {
        const text = document.getElementById('statusText');
        const dot = document.getElementById('statusDot');
        text.innerText = msg;
        dot.className = 'status-dot';
        
        if (type === 'processing') dot.classList.add('processing');
        else if (type === 'success') dot.classList.add('success');
        else if (type === 'error') dot.classList.add('error');
    }

    async function generateJson() {
        const apiUrl = document.getElementById('apiUrl').value.trim();
        const apiKey = document.getElementById('apiKey').value.trim();
        const modelName = document.getElementById('modelName').value.trim();
        const userText = document.getElementById('inputText').value.trim();

        if (!apiKey) {
            alert('⚠️ 请输入 API Key (SaaS 后台需要)');
            return;
        }
        if (!userText) {
            alert('⚠️ 请先粘贴文案');
            return;
        }

        setStatus('正在请求 API Mart 生成中，请稍候...', 'processing');

        try {
            const response = await fetch(apiUrl, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                    'Authorization': `Bearer ${apiKey}`
                },
                body: JSON.stringify({
                    model: modelName,
                    stream: false, // 关键：关闭流式传输以获得完整 JSON
                    messages: [
                        {
                            "role": "system",
                            "content": `You are a strict JSON editor for a SaaS website builder.
                            
                            TASK:
                            Fill the provided JSON template with the user's text content.
                            
                            CRITICAL RULES:
                            1. **MAPPING LOGIC**:
                               - "How to use" section:
                                 - 2nd line -> 'TagName'
                                 - 3rd line -> 'TitleName'
                                 - Paragraph -> 'DescriptionName'
                               - "Features" section:
                                 - 2nd line -> 'TagName'
                                 - 3rd line -> 'TitleName'
                                 - Paragraph -> 'DescriptionName'
                                 - List items -> FeatureTitleName / FeatureDescriptionName
                               - "FAQ" section -> FAQQuestionName / FAQAnswerName
                               - "CTA" section -> TitleName / DescriptionName / TipName / CTALinkName
                            
                            2. **IMMUTABLE FIELDS**:
                               - DO NOT change any 'id', 'component', 'style', 'type', 'active', 'theme', 'bgColor'.
                               - DO NOT change any 'FeatureImageImg' (keep original URLs).
                               - DO NOT change 'BackgroundImageImg'.
                               - DO NOT change 'FeatureBadgeColor' or 'FeatureBadgeTextColor'.
                            
                            3. **ALT TEXT**:
                               - If user provides Alt Text, put it in 'FeatureImageImgAlt' if the field exists.
                            
                            4. **OUTPUT**:
                               - Return ONLY valid JSON. No markdown.`
                        },
                        {
                            "role": "user",
                            "content": `TEMPLATE JSON:\n${JSON.stringify(currentTemplate)}\n\nUSER TEXT:\n${userText}`
                        }
                    ],
                    temperature: 0.1
                })
            });

            if (!response.ok) {
                throw new Error(`API Error: ${response.status}`);
            }

            const data = await response.json();
            let aiContent = data.choices[0].message.content;
            
            // 清理 Markdown 格式
            aiContent = aiContent.replace(/```json/g, '').replace(/```/g, '').trim();
            
            // 格式化 JSON
            const validJson = JSON.parse(aiContent);
            document.getElementById('outputText').value = JSON.stringify(validJson, null, 2);
            
            setStatus('✅ 生成成功！底板样式已锁定，文案已更新。', 'success');

        } catch (error) {
            console.error(error);
            setStatus(`❌ 失败: ${error.message}`, 'error');
            document.getElementById('outputText').value = "错误详情:\n" + error.message;
        }
    }

    function copyResult() {
        const text = document.getElementById('outputText');
        text.select();
        document.execCommand('copy');
        alert('已复制 JSON');
    }

    function downloadFile() {
        const text = document.getElementById('outputText').value;
        if (!text) return;
        const blob = new Blob([text], { type: 'application/json' });
        const url = URL.createObjectURL(blob);
        const a = document.createElement('a');
        a.href = url;
        
        // 根据当前站点命名
        const siteName = document.querySelector('.site-option.active').innerText.split(' ')[0];
        a.download = `${siteName}_generated.json`;
        
        a.click();
    }
</script>

</body>
</html>
