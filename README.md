<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>🌸 ふんわり体重＆バイタルダイアリー 🌸</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Chart.js -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <!-- Lucide Icons -->
    <script src="https://unpkg.com/lucide@latest"></script>
    <!-- Google Fonts -->
    <link href="https://fonts.googleapis.com/css2?family=Kiwi+Maru:wght@400;500&family=M+PLUS+Rounded+1c:wght@400;500;700;800&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'M PLUS Rounded 1c', 'Kiwi Maru', sans-serif;
            background-color: #fdf2f8; /* Soft pink bg */
            background-image: radial-gradient(#fbcfe8 1px, transparent 1px);
            background-size: 24px 24px;
        }
        /* Cute custom scrollbar */
        ::-webkit-scrollbar {
            width: 8px;
            height: 8px;
        }
        ::-webkit-scrollbar-track {
            background: #fbcfe8;
        }
        ::-webkit-scrollbar-thumb {
            background: #f472b6;
            border-radius: 9999px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: #e11d48;
        }

        /* 📄 A4 Print Stylesheet Optimization */
        @media print {
            @page {
                size: A4 portrait;
                margin: 8mm 10mm 8mm 10mm;
            }
            body {
                background: white !important;
                background-image: none !important;
                color: #000 !important;
                font-size: 11px !important;
                -webkit-print-color-adjust: exact !important;
                print-color-adjust: exact !important;
            }
            /* Hide non-printable components */
            header,
            .no-print,
            #apiKeyModal,
            #toast,
            #btnViewTimeline,
            #btnViewDaily,
            #btnViewDiff,
            button {
                display: none !important;
            }
            main {
                max-width: 100% !important;
                padding: 0 !important;
                margin: 0 !important;
                space-y: 12px !important;
            }
            .print-header {
                display: block !important;
                border-bottom: 2px solid #f472b6;
                padding-bottom: 6px;
                margin-bottom: 12px;
            }
            .bg-white {
                border: 1px solid #fbcfe8 !important;
                box-shadow: none !important;
                border-radius: 12px !important;
                break-inside: avoid;
                padding: 12px !important;
            }
            /* Chart height optimization for A4 */
            .chart-container-box {
                height: 260px !important;
            }
            /* Table formatting */
            table {
                font-size: 10px !important;
            }
            th, td {
                padding: 4px 6px !important;
            }
        }
        .print-header {
            display: none;
        }
    </style>
</head>
<body class="text-pink-950 min-h-screen flex flex-col">

    <!-- Header -->
    <header class="bg-white/90 backdrop-blur-md border-b-2 border-pink-200 sticky top-0 z-30 shadow-sm no-print">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-3 flex flex-col sm:flex-row items-center justify-between gap-3">
            <div class="flex items-center space-x-3">
                <div class="p-3 bg-gradient-to-tr from-rose-400 to-pink-300 text-white rounded-2xl shadow-md shadow-pink-200 text-2xl flex items-center justify-center">
                    🌸
                </div>
                <div>
                    <h1 class="text-xl font-extrabold text-pink-900 tracking-wide flex items-center gap-1.5">
                        14日間 体重ダイアリー
                        <span class="text-xs font-normal bg-pink-100 text-pink-700 px-2.5 py-0.5 rounded-full border border-pink-200">Gemini 搭載 ✨</span>
                    </h1>
                    <p class="text-xs text-pink-400 font-medium">起床時・朝食後・夕食後・お休み前のぽちぽち体重ログ 🎀</p>
                </div>
            </div>
            
            <div class="flex items-center gap-2 flex-wrap justify-center">
                <button id="btnPrintA4" class="px-3.5 py-2 text-xs font-bold text-indigo-800 bg-indigo-100/80 hover:bg-indigo-200 rounded-2xl transition-all flex items-center gap-1.5 border border-indigo-200 shadow-xs">
                    <i data-lucide="printer" class="w-4 h-4 text-indigo-600"></i>
                    📄 A4印刷 / PDF
                </button>
                <button id="btnExportCsv" class="px-3.5 py-2 text-xs font-bold text-emerald-800 bg-emerald-100/80 hover:bg-emerald-200 rounded-2xl transition-all flex items-center gap-1.5 border border-emerald-200 shadow-xs">
                    <i data-lucide="download" class="w-4 h-4 text-emerald-600"></i>
                    CSV保存 📄
                </button>
                <button id="btnApiKeyConfig" class="px-3.5 py-2 text-xs font-bold text-purple-700 bg-purple-100/80 hover:bg-purple-200 rounded-2xl transition-all flex items-center gap-1.5 border border-purple-200 shadow-xs">
                    <i data-lucide="key" class="w-4 h-4 text-purple-500"></i>
                    🔑 APIキー設定
                </button>
                <button id="btnSampleData" class="px-3.5 py-2 text-xs font-bold text-amber-800 bg-amber-100/80 hover:bg-amber-200 rounded-2xl transition-all flex items-center gap-1.5 border border-amber-200 shadow-xs">
                    <i data-lucide="sparkles" class="w-4 h-4 text-amber-500"></i>
                    ✨ サンプル入力
                </button>
                <button id="btnClearData" class="px-3.5 py-2 text-xs font-bold text-rose-700 bg-rose-100/80 hover:bg-rose-200 rounded-2xl transition-all flex items-center gap-1.5 border border-rose-200 shadow-xs">
                    <i data-lucide="trash-2" class="w-4 h-4 text-rose-500"></i>
                    リセット
                </button>
            </div>
        </div>
    </header>

    <!-- Main Content -->
    <main class="flex-1 max-w-7xl w-full mx-auto px-4 sm:px-6 lg:px-8 py-6 space-y-6">

        <!-- A4 Printable Header Title (Visible only when printed) -->
        <div class="print-header text-center">
            <h1 class="text-2xl font-black text-pink-950">🌸 14日間 ふんわり体重＆バイタルダイアリー 🌸</h1>
            <p id="printPeriodTitle" class="text-xs font-bold text-pink-800 mt-1">----年--月--日 〜 ----年--月--日</p>
        </div>

        <!-- Date Range Navigation Bar -->
        <div class="bg-white/90 backdrop-blur-sm p-4 rounded-3xl border-2 border-pink-100 shadow-sm flex flex-col md:flex-row items-center justify-between gap-4">
            <div class="flex items-center space-x-2">
                <button id="btnPrevPeriod" class="p-2.5 bg-pink-50 hover:bg-pink-100 rounded-2xl text-pink-600 transition-colors shadow-xs no-print">
                    <i data-lucide="chevron-left" class="w-5 h-5"></i>
                </button>
                <div class="text-center px-4">
                    <span id="periodLabel" class="text-sm sm:text-base font-extrabold text-pink-900">----年--月--日 〜 ----年--月--日</span>
                    <span class="text-xs text-pink-400 block font-medium">（月曜日スタート 🌷 14日間）</span>
                </div>
                <button id="btnNextPeriod" class="p-2.5 bg-pink-50 hover:bg-pink-100 rounded-2xl text-pink-600 transition-colors shadow-xs no-print">
                    <i data-lucide="chevron-right" class="w-5 h-5"></i>
                </button>
            </div>

            <button id="btnTodayPeriod" class="px-4 py-2 text-xs font-bold text-pink-700 bg-pink-100 hover:bg-pink-200 rounded-2xl transition-colors shadow-xs flex items-center gap-1.5 no-print">
                <span>🗓️ 今週を表示する</span>
            </button>
        </div>

        <!-- Quick Entry Form Card -->
        <div id="weightForm" class="bg-white rounded-3xl border-2 border-pink-200 shadow-sm p-5 sm:p-6 space-y-4 no-print">
            <div class="border-b border-pink-100 pb-3 flex items-center justify-between">
                <h3 class="font-extrabold text-pink-900 flex items-center gap-2 text-base">
                    <span class="text-xl">✍️</span>
                    きょうの記録を入力する
                </h3>
                <span class="text-xs text-pink-400 font-medium">数字を入れたら「保存」をおしてね♪</span>
            </div>

            <form id="innerWeightForm" class="space-y-3">
                <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-5 lg:grid-cols-6 gap-3 items-end">
                    <div>
                        <label class="block text-xs font-bold text-pink-700 mb-1">📅 日付</label>
                        <input type="date" id="inputDate" required class="w-full px-3 py-2 border-2 border-pink-200 rounded-2xl text-sm focus:outline-none focus:ring-2 focus:ring-pink-400 focus:border-pink-400 bg-pink-50/30">
                    </div>

                    <div>
                        <label class="block text-xs font-bold text-sky-600 mb-1 flex items-center gap-1">
                            <span>☀️</span> 起床時 (kg)
                        </label>
                        <input type="number" step="0.01" min="20" max="250" id="inputWake" placeholder="例: 52.0" class="w-full px-3 py-2 border-2 border-sky-200 rounded-2xl text-sm focus:outline-none focus:ring-2 focus:ring-sky-400 bg-sky-50/30">
                    </div>

                    <div>
                        <label class="block text-xs font-bold text-amber-600 mb-1 flex items-center gap-1">
                            <span>🥐</span> 朝食後 (kg)
                        </label>
                        <input type="number" step="0.01" min="20" max="250" id="inputAfterBreakfast" placeholder="例: 52.4" class="w-full px-3 py-2 border-2 border-amber-200 rounded-2xl text-sm focus:outline-none focus:ring-2 focus:ring-amber-400 bg-amber-50/30">
                    </div>

                    <div>
                        <label class="block text-xs font-bold text-emerald-600 mb-1 flex items-center gap-1">
                            <span>🍽️</span> 夕食後 (kg)
                        </label>
                        <input type="number" step="0.01" min="20" max="250" id="inputAfterDinner" placeholder="例: 52.8" class="w-full px-3 py-2 border-2 border-emerald-200 rounded-2xl text-sm focus:outline-none focus:ring-2 focus:ring-emerald-400 bg-emerald-50/30">
                    </div>

                    <div>
                        <label class="block text-xs font-bold text-purple-600 mb-1 flex items-center gap-1">
                            <span>🌙</span> お休み前 (kg)
                        </label>
                        <input type="number" step="0.01" min="20" max="250" id="inputBed" placeholder="例: 52.6" class="w-full px-3 py-2 border-2 border-purple-200 rounded-2xl text-sm focus:outline-none focus:ring-2 focus:ring-purple-400 bg-purple-50/30">
                    </div>

                    <div class="sm:col-span-2 md:col-span-1 lg:col-span-1">
                        <button type="submit" class="w-full py-2.5 bg-gradient-to-r from-rose-400 to-pink-500 hover:from-rose-500 hover:to-pink-600 text-white font-extrabold text-sm rounded-2xl transition-all shadow-md shadow-pink-200 flex items-center justify-center gap-1.5 active:scale-95">
                            <i data-lucide="heart" class="w-4 h-4 fill-white"></i>
                            保存する
                        </button>
                    </div>
                </div>

                <!-- Dynamic Alert inside Form -->
                <div id="formAlert" class="hidden p-3 rounded-2xl text-xs font-bold flex items-center gap-2"></div>
            </form>
        </div>

        <!-- Metric Summary Cards -->
        <div class="grid grid-cols-2 lg:grid-cols-4 gap-3 sm:gap-4">
            <div class="bg-white p-3.5 sm:p-4 rounded-3xl border-2 border-pink-100 shadow-sm flex items-center justify-between">
                <div>
                    <p class="text-[11px] font-bold text-pink-400">期間の平均体重 ⚖️</p>
                    <h3 id="statAvgWeight" class="text-xl sm:text-2xl font-black text-pink-900 mt-0.5">-- <span class="text-xs font-medium text-pink-400">kg</span></h3>
                </div>
                <div class="p-2.5 bg-sky-100 text-sky-600 rounded-2xl text-lg sm:text-xl">
                    🧸
                </div>
            </div>

            <div class="bg-white p-3.5 sm:p-4 rounded-3xl border-2 border-pink-100 shadow-sm flex items-center justify-between">
                <div>
                    <p class="text-[11px] font-bold text-pink-400">期間の体重変化 📈</p>
                    <h3 id="statWeightChange" class="text-xl sm:text-2xl font-black text-pink-900 mt-0.5">-- <span class="text-xs font-medium text-pink-400">kg</span></h3>
                </div>
                <div id="statWeightChangeIcon" class="p-2.5 bg-pink-50 text-pink-600 rounded-2xl text-lg sm:text-xl">
                    ✨
                </div>
            </div>

            <div class="bg-white p-3.5 sm:p-4 rounded-3xl border-2 border-pink-100 shadow-sm flex items-center justify-between">
                <div>
                    <p class="text-[11px] font-bold text-pink-400">就寝直前増えた日数 🍓</p>
                    <h3 id="statNightIncreaseCount" class="text-xl sm:text-2xl font-black text-rose-500 mt-0.5">0 <span class="text-xs font-medium text-pink-400">/ 14日</span></h3>
                </div>
                <div class="p-2.5 bg-rose-100 text-rose-500 rounded-2xl text-lg sm:text-xl">
                    🍓
                </div>
            </div>

            <div class="bg-white p-3.5 sm:p-4 rounded-3xl border-2 border-pink-100 shadow-sm flex items-center justify-between">
                <div>
                    <p class="text-[11px] font-bold text-pink-400">夕食→就寝 平均差分 🌙</p>
                    <h3 id="statAvgNightDiff" class="text-xl sm:text-2xl font-black text-pink-900 mt-0.5">-- <span class="text-xs font-medium text-pink-400">kg</span></h3>
                </div>
                <div class="p-2.5 bg-purple-100 text-purple-600 rounded-2xl text-lg sm:text-xl">
                    🌙
                </div>
            </div>
        </div>

        <!-- Chart Section -->
        <div class="bg-white rounded-3xl border-2 border-pink-200 shadow-sm p-4 sm:p-5 space-y-4">
            <div class="flex flex-col sm:flex-row sm:items-center justify-between gap-3 border-b border-pink-100 pb-3">
                <div>
                    <h2 class="text-base sm:text-lg font-black text-pink-900 flex items-center gap-2">
                        <span>📊</span>
                        14日間の体重グラフ
                    </h2>
                    <p class="text-xs text-pink-400 font-medium mt-0.5">「夕食後よりお休み前が増えたポイント」はピンク苺色🍓でハイライトされます</p>
                </div>

                <!-- View Switcher Tabs -->
                <div class="inline-flex p-1 bg-pink-100/80 rounded-2xl text-xs font-bold self-start sm:self-auto gap-1 no-print">
                    <button id="btnViewTimeline" class="px-3 py-1.5 rounded-xl bg-white text-pink-900 shadow-xs transition-all">
                        時系列 (56点) 連続
                    </button>
                    <button id="btnViewDaily" class="px-3 py-1.5 rounded-xl text-pink-600 hover:text-pink-900 transition-all">
                        時間帯別 4本線
                    </button>
                    <button id="btnViewDiff" class="px-3 py-1.5 rounded-xl text-pink-600 hover:text-pink-900 transition-all">
                        夕食後 vs お休み前 差分
                    </button>
                </div>
            </div>

            <!-- Legend and Indicator Help -->
            <div class="flex flex-wrap items-center gap-3 text-xs bg-pink-50/60 p-2.5 sm:p-3 rounded-2xl font-medium">
                <div class="flex items-center gap-1.5">
                    <span class="w-3 h-3 rounded-full bg-sky-400 inline-block"></span>
                    <span class="text-pink-800">起床時</span>
                </div>
                <div class="flex items-center gap-1.5">
                    <span class="w-3 h-3 rounded-full bg-amber-400 inline-block"></span>
                    <span class="text-pink-800">朝食後</span>
                </div>
                <div class="flex items-center gap-1.5">
                    <span class="w-3 h-3 rounded-full bg-emerald-400 inline-block"></span>
                    <span class="text-pink-800">夕食後</span>
                </div>
                <div class="flex items-center gap-1.5">
                    <span class="w-3 h-3 rounded-full bg-purple-400 inline-block"></span>
                    <span class="text-pink-800">お休み前(通常)</span>
                </div>
                <div class="flex items-center gap-1.5 font-bold text-rose-600 bg-rose-100/80 px-2 py-0.5 rounded-xl border border-rose-200">
                    <span class="w-2.5 h-2.5 rounded-full bg-rose-500 inline-block animate-ping"></span>
                    <span>お休み前 (夕食後より増加🍓)</span>
                </div>
            </div>

            <!-- Chart Canvas Container -->
            <div class="relative w-full h-[320px] sm:h-[380px] chart-container-box">
                <canvas id="weightChart"></canvas>
            </div>
        </div>

        <!-- 14-Day Table View -->
        <div class="bg-white rounded-3xl border-2 border-pink-200 shadow-sm p-4 sm:p-5 space-y-3">
            <div class="flex items-center justify-between border-b border-pink-100 pb-3">
                <h3 class="font-black text-pink-900 flex items-center gap-2 text-base">
                    <span>📖</span>
                    14日間のきろく一覧
                </h3>
                <span class="text-xs text-pink-400 font-medium no-print">※ 行をクリックすると上の入力欄にセットされるよ</span>
            </div>

            <div class="overflow-x-auto">
                <table class="w-full text-left border-collapse">
                    <thead>
                        <tr class="bg-pink-50/80 border-b border-pink-100 text-xs font-bold text-pink-800">
                            <th class="py-2.5 px-3">日付</th>
                            <th class="py-2.5 px-3">☀️ 起床時</th>
                            <th class="py-2.5 px-3">🥐 朝食後</th>
                            <th class="py-2.5 px-3">🍽️ 夕食後</th>
                            <th class="py-2.5 px-3">🌙 お休み前</th>
                            <th class="py-2.5 px-3">夕食後→お休み前</th>
                            <th class="py-2.5 px-3 text-center">判定</th>
                        </tr>
                    </thead>
                    <tbody id="dataTableBody" class="text-xs sm:text-sm divide-y divide-pink-100 font-medium">
                        <!-- Populated dynamically via JS -->
                    </tbody>
                </table>
            </div>
        </div>

        <!-- Gemini Health Coach Section (Pastel Sweet Theme) -->
        <div class="bg-gradient-to-br from-pink-500 via-purple-500 to-indigo-600 rounded-3xl p-5 sm:p-6 text-white shadow-xl shadow-purple-200/50 space-y-4 no-print">
            <div class="flex flex-col sm:flex-row sm:items-center justify-between gap-4 border-b border-white/20 pb-4">
                <div class="flex items-center space-x-3">
                    <div class="p-3 bg-amber-300 text-pink-950 rounded-2xl shadow-lg text-2xl flex items-center justify-center">
                        🐱
                    </div>
                    <div>
                        <div class="flex items-center gap-2">
                            <h2 class="text-lg font-black tracking-wide">Gemini ふんわりヘルスコーチ</h2>
                            <span class="px-2.5 py-0.5 text-[10px] font-bold bg-white/20 text-pink-100 rounded-full border border-white/30">Gemini 搭載</span>
                        </div>
                        <p class="text-xs text-pink-100 font-medium">14日間の「起床時・朝食後・夕食後・お休み前」のパターンをやさしく分析してアドバイスしてくれるよ✨</p>
                    </div>
                </div>

                <div class="flex flex-wrap items-center gap-2">
                    <button id="btnAiAnalyze" class="px-4 py-2.5 bg-amber-300 hover:bg-amber-200 text-pink-950 font-black text-xs rounded-2xl shadow-md transition-all flex items-center gap-1.5 active:scale-95">
                        <i data-lucide="sparkles" class="w-4 h-4 text-pink-600"></i>
                        ✨ 14日間を分析する
                    </button>
                    <button id="btnAiGenCard" class="px-3.5 py-2.5 bg-white/15 hover:bg-white/25 text-white font-bold text-xs rounded-2xl transition-all flex items-center gap-1.5 border border-white/25">
                        <i data-lucide="image" class="w-4 h-4 text-amber-300"></i>
                        🎨 ごほうびカード画像を作る
                    </button>
                </div>
            </div>

            <!-- Status / Spinner -->
            <div id="aiLoadingState" class="hidden py-8 text-center space-y-3">
                <div class="inline-block animate-bounce text-4xl">🌸</div>
                <p id="aiLoadingText" class="text-xs text-pink-100 font-bold">Gemini がみんなの記録を分析中だよ...♪</p>
            </div>

            <!-- Output Card (Hidden initially) -->
            <div id="aiResultCard" class="hidden bg-white/15 backdrop-blur-md border border-white/20 rounded-2xl p-4 sm:p-5 space-y-4 shadow-inner">
                <div class="flex flex-col sm:flex-row items-start sm:items-center justify-between gap-3 border-b border-white/15 pb-3">
                    <div class="flex items-center gap-3">
                        <div id="aiGradeBadge" class="w-12 h-12 rounded-2xl bg-amber-300 text-pink-950 font-black text-2xl flex items-center justify-center shadow-lg">
                            🌸
                        </div>
                        <div>
                            <span class="text-[10px] text-pink-200 uppercase tracking-widest block font-bold">14日間の習慣評価</span>
                            <h4 id="aiSummaryTitle" class="text-sm font-extrabold text-white">すばらしいペースです！生活リズムを意識してみよう♪</h4>
                        </div>
                    </div>

                    <!-- TTS Voice Playback Button -->
                    <button id="btnPlayTts" class="px-3.5 py-2 bg-purple-900/60 hover:bg-purple-800/80 text-white rounded-xl text-xs font-bold flex items-center gap-1.5 border border-purple-300/40 transition-colors">
                        <i data-lucide="volume-2" class="w-4 h-4 text-amber-300"></i>
                        <span id="ttsBtnText">🔊 声のアドバイスを聴く</span>
                    </button>
                </div>

                <div class="grid grid-cols-1 md:grid-cols-3 gap-4 text-xs font-medium">
                    <!-- Good Points -->
                    <div class="bg-black/20 p-3.5 rounded-2xl border border-white/10 space-y-1.5">
                        <h5 class="font-bold text-emerald-300 flex items-center gap-1.5">
                            <span>✨</span> 褒めポイント＆成果
                        </h5>
                        <ul id="aiGoodPoints" class="space-y-1 text-pink-100 list-disc list-inside">
                            <!-- Populated by JS -->
                        </ul>
                    </div>

                    <!-- Meal Gain Analysis -->
                    <div class="bg-black/20 p-3.5 rounded-2xl border border-white/10 space-y-1.5">
                        <h5 class="font-bold text-rose-300 flex items-center gap-1.5">
                            <span>🌙</span> 夕食・夜の間食アドバイス
                        </h5>
                        <p id="aiNightAnalysis" class="text-pink-100 leading-relaxed">
                            <!-- Populated by JS -->
                        </p>
                    </div>

                    <!-- Actionable Tips -->
                    <div class="bg-black/20 p-3.5 rounded-2xl border border-white/10 space-y-1.5">
                        <h5 class="font-bold text-amber-200 flex items-center gap-1.5">
                            <span>🎀</span> あしたからのラッキーアクション
                        </h5>
                        <ul id="aiActionTips" class="space-y-1 text-pink-100 list-disc list-inside">
                            <!-- Populated by JS -->
                        </ul>
                    </div>
                </div>

                <!-- Hidden audio container for TTS -->
                <audio id="ttsAudioPlayer" class="hidden"></audio>
            </div>

            <!-- Generated Illustration Badge Display -->
            <div id="aiImageContainer" class="hidden bg-black/20 border border-white/15 rounded-2xl p-4 flex flex-col sm:flex-row items-center gap-4">
                <img id="aiGeneratedBadgeImg" class="w-32 h-32 rounded-2xl object-cover shadow-lg border-2 border-white/40" alt="Motivation Badge" />
                <div class="space-y-1 text-center sm:text-left">
                    <span class="text-[10px] text-amber-300 font-bold uppercase tracking-wider">Gemini 3.1 Flash Image 生成</span>
                    <h5 class="text-sm font-bold text-white">🌸 14日間がんばったね！ごほうびバッジ 🌸</h5>
                    <p class="text-xs text-pink-100">継続的な測定おつかれさまでした！この画像を保存してモチベーションにしてくださいね♪</p>
                </div>
            </div>
        </div>

    </main>

    <!-- API Key Settings Modal -->
    <div id="apiKeyModal" class="fixed inset-0 bg-pink-950/50 backdrop-blur-sm z-50 flex items-center justify-center hidden p-4 no-print">
        <div class="bg-white rounded-3xl max-w-md w-full p-6 space-y-4 shadow-2xl border-2 border-pink-200">
            <div class="flex items-center justify-between border-b border-pink-100 pb-3">
                <h3 class="font-extrabold text-pink-900 text-base flex items-center gap-2">
                    🔑 Gemini APIキーの設定
                </h3>
                <button id="btnCloseApiModal" class="text-pink-400 hover:text-pink-600">
                    <i data-lucide="x" class="w-5 h-5"></i>
                </button>
            </div>
            
            <p class="text-xs text-pink-600 leading-relaxed font-medium">
                Gemini の分析や声のアドバイス、イラスト画像作成機能をつかうための無料APIキーを入力してね。<br>
                キーはお使いのブラウザ（localStorage）にのみ安全に保存されます。
            </p>

            <div>
                <label class="block text-xs font-bold text-pink-700 mb-1">Gemini API Key</label>
                <input type="password" id="inputApiKey" placeholder="AIzaSy..." class="w-full px-3 py-2 border-2 border-pink-200 rounded-2xl text-sm focus:outline-none focus:ring-2 focus:ring-purple-400">
            </div>

            <div class="flex items-center justify-between text-[11px] font-bold text-pink-500">
                <a href="https://aistudio.google.com/app/apikey" target="_blank" rel="noopener noreferrer" class="text-purple-600 hover:underline flex items-center gap-1">
                    APIキーを無料で取得する <i data-lucide="external-link" class="w-3 h-3"></i>
                </a>
            </div>

            <div class="flex gap-2 pt-2">
                <button id="btnSaveApiKey" class="w-full py-2.5 bg-gradient-to-r from-purple-500 to-pink-500 hover:from-purple-600 hover:to-pink-600 text-white font-extrabold text-xs rounded-2xl transition-all shadow-md shadow-pink-200">
                    設定を保存する 🎀
                </button>
            </div>
        </div>
    </div>

    <!-- Notification Toast -->
    <div id="toast" class="fixed bottom-5 right-5 transform translate-y-20 opacity-0 transition-all duration-300 bg-pink-900 text-white px-5 py-3 rounded-2xl shadow-xl flex items-center gap-2 text-sm z-50 font-bold border border-pink-700 no-print">
        <span class="text-lg">🌸</span>
        <span id="toastMessage">保存しました</span>
    </div>

    <script>
        // Global Application State
        const STORAGE_KEY = 'weight_tracker_data_v3';
        let weightData = {}; // Format: { "YYYY-MM-DD": { wake: 52.0, afterBreakfast: 52.4, afterDinner: 52.8, bed: 52.6 } }
        let currentStartDate = '';
        let currentView = 'timeline'; // 'timeline' | 'daily' | 'diff'
        let chartInstance = null;

        const DAY_NAMES = ['日', '月', '火', '水', '木', '金', '土'];

        function initIcons() {
            lucide.createIcons();
        }

        function getMondayOfWeek(d) {
            const date = new Date(d);
            const day = date.getDay();
            const diff = date.getDate() - day + (day === 0 ? -6 : 1);
            const monday = new Date(date.setDate(diff));
            return formatDate(monday);
        }

        function getStartOfCurrent14Days() {
            return getMondayOfWeek(new Date());
        }

        function parseDateStr(str) {
            if (!str) return new Date();
            const parts = str.split('-');
            return new Date(parseInt(parts[0], 10), parseInt(parts[1], 10) - 1, parseInt(parts[2], 10));
        }

        function formatDate(d) {
            const date = new Date(d);
            const year = date.getFullYear();
            const month = String(date.getMonth() + 1).padStart(2, '0');
            const day = String(date.getDate()).padStart(2, '0');
            return `${year}-${month}-${day}`;
        }

        function addDays(dateStr, days) {
            const d = parseDateStr(dateStr);
            d.setDate(d.getDate() + days);
            return formatDate(d);
        }

        function setDefaultFormDate() {
            document.getElementById('inputDate').value = formatDate(new Date());
        }

        function formatShortDate(dateStr) {
            const d = parseDateStr(dateStr);
            const m = String(d.getMonth() + 1).padStart(2, '0');
            const day = String(d.getDate()).padStart(2, '0');
            const dayOfWeek = DAY_NAMES[d.getDay()];
            return `${m}/${day}(${dayOfWeek})`;
        }

        function getCurrent14DaysList() {
            const list = [];
            for (let i = 0; i < 14; i++) {
                list.push(addDays(currentStartDate, i));
            }
            return list;
        }

        function updatePeriodLabel(dates) {
            if (!dates || dates.length === 0) return;
            const start = dates[0];
            const end = dates[dates.length - 1];
            
            const formatFull = (dStr) => {
                const d = parseDateStr(dStr);
                const y = d.getFullYear();
                const m = String(d.getMonth() + 1).padStart(2, '0');
                const day = String(d.getDate()).padStart(2, '0');
                const dow = DAY_NAMES[d.getDay()];
                return `${y}年${m}月${day}日(${dow})`;
            };
            
            const labelText = `${formatFull(start)} 〜 ${formatFull(end)}`;
            const labelEl = document.getElementById('periodLabel');
            if (labelEl) {
                labelEl.textContent = labelText;
            }

            const printLabelEl = document.getElementById('printPeriodTitle');
            if (printLabelEl) {
                printLabelEl.textContent = `${labelText} (14日間記録シート)`;
            }
        }

        function updateStats(dates) {
            let allWeights = [];
            let nightIncreaseCount = 0;
            let nightDiffs = [];

            dates.forEach(d => {
                const log = weightData[d];
                if (!log) return;

                ['wake', 'afterBreakfast', 'afterDinner', 'bed'].forEach(key => {
                    if (log[key] !== null && log[key] !== undefined && !isNaN(log[key])) {
                        allWeights.push(log[key]);
                    }
                });

                if (log.afterDinner !== null && log.afterDinner !== undefined &&
                    log.bed !== null && log.bed !== undefined) {
                    const diff = log.bed - log.afterDinner;
                    nightDiffs.push(diff);
                    if (diff > 0) {
                        nightIncreaseCount++;
                    }
                }
            });

            // 1. Avg Weight
            const avgWeightEl = document.getElementById('statAvgWeight');
            if (allWeights.length > 0) {
                const avg = allWeights.reduce((a, b) => a + b, 0) / allWeights.length;
                avgWeightEl.innerHTML = `${avg.toFixed(2)} <span class="text-xs font-normal text-pink-400">kg</span>`;
            } else {
                avgWeightEl.innerHTML = `-- <span class="text-xs font-normal text-pink-400">kg</span>`;
            }

            // 2. Weight Change
            const weightChangeEl = document.getElementById('statWeightChange');
            const changeIconEl = document.getElementById('statWeightChangeIcon');
            if (allWeights.length >= 2) {
                const firstVal = allWeights[0];
                const lastVal = allWeights[allWeights.length - 1];
                const change = lastVal - firstVal;
                const sign = change > 0 ? '+' : '';
                const colorClass = change > 0 ? 'text-rose-500' : (change < 0 ? 'text-emerald-600' : 'text-pink-900');
                
                weightChangeEl.className = `text-xl sm:text-2xl font-black mt-0.5 ${colorClass}`;
                weightChangeEl.innerHTML = `${sign}${change.toFixed(2)} <span class="text-xs font-normal text-pink-400">kg</span>`;
                
                if (changeIconEl) {
                    if (change > 0) {
                        changeIconEl.className = "p-2.5 bg-rose-100 text-rose-500 rounded-2xl text-lg sm:text-xl";
                        changeIconEl.innerHTML = '📈';
                    } else if (change < 0) {
                        changeIconEl.className = "p-2.5 bg-emerald-100 text-emerald-600 rounded-2xl text-lg sm:text-xl";
                        changeIconEl.innerHTML = '📉';
                    } else {
                        changeIconEl.className = "p-2.5 bg-pink-100 text-pink-600 rounded-2xl text-lg sm:text-xl";
                        changeIconEl.innerHTML = '✨';
                    }
                }
            } else {
                weightChangeEl.className = "text-xl sm:text-2xl font-black text-pink-900 mt-0.5";
                weightChangeEl.innerHTML = `-- <span class="text-xs font-normal text-pink-400">kg</span>`;
            }

            // 3. Night Increase Count
            const countEl = document.getElementById('statNightIncreaseCount');
            countEl.innerHTML = `${nightIncreaseCount} <span class="text-xs font-normal text-pink-400">/ 14日</span>`;

            // 4. Avg Night Diff
            const avgNightDiffEl = document.getElementById('statAvgNightDiff');
            if (nightDiffs.length > 0) {
                const avgDiff = nightDiffs.reduce((a, b) => a + b, 0) / nightDiffs.length;
                const sign = avgDiff > 0 ? '+' : '';
                const colorClass = avgDiff > 0 ? 'text-rose-500' : (avgDiff < 0 ? 'text-emerald-600' : 'text-pink-900');
                avgNightDiffEl.className = `text-xl sm:text-2xl font-black mt-0.5 ${colorClass}`;
                avgNightDiffEl.innerHTML = `${sign}${avgDiff.toFixed(2)} <span class="text-xs font-normal text-pink-400">kg</span>`;
            } else {
                avgNightDiffEl.className = "text-xl sm:text-2xl font-black text-pink-900 mt-0.5";
                avgNightDiffEl.innerHTML = `-- <span class="text-xs font-normal text-pink-400">kg</span>`;
            }

            lucide.createIcons();
        }

        function switchView(viewName) {
            currentView = viewName;
            const btnTimeline = document.getElementById('btnViewTimeline');
            const btnDaily = document.getElementById('btnViewDaily');
            const btnDiff = document.getElementById('btnViewDiff');

            const activeClass = "px-3 py-1.5 rounded-xl bg-white text-pink-900 shadow-xs transition-all";
            const inactiveClass = "px-3 py-1.5 rounded-xl text-pink-600 hover:text-pink-900 transition-all";

            btnTimeline.className = viewName === 'timeline' ? activeClass : inactiveClass;
            btnDaily.className = viewName === 'daily' ? activeClass : inactiveClass;
            btnDiff.className = viewName === 'diff' ? activeClass : inactiveClass;

            const dates = getCurrent14DaysList();
            renderChart(dates);
        }

        function renderChart(dates) {
            const canvas = document.getElementById('weightChart');
            if (!canvas) return;
            const ctx = canvas.getContext('2d');

            if (chartInstance) {
                chartInstance.destroy();
            }

            if (currentView === 'timeline') {
                renderTimelineChart(ctx, dates);
            } else if (currentView === 'daily') {
                renderDailyChart(ctx, dates);
            } else if (currentView === 'diff') {
                renderDiffChart(ctx, dates);
            }
        }

        function renderAll() {
            const dates = getCurrent14DaysList();
            updatePeriodLabel(dates);
            updateStats(dates);
            renderChart(dates);
            renderTable(dates);
        }

        function loadFormDataForDate(dateStr) {
            if (weightData[dateStr]) {
                const d = weightData[dateStr];
                document.getElementById('inputWake').value = d.wake ?? '';
                document.getElementById('inputAfterBreakfast').value = d.afterBreakfast ?? d.breakfast ?? '';
                document.getElementById('inputAfterDinner').value = d.afterDinner ?? d.dinner ?? '';
                document.getElementById('inputBed').value = d.bed ?? '';
            } else {
                document.getElementById('inputWake').value = '';
                document.getElementById('inputAfterBreakfast').value = '';
                document.getElementById('inputAfterDinner').value = '';
                document.getElementById('inputBed').value = '';
            }
        }

        function generateSampleData() {
            weightData = {};
            const startMon = parseDateStr(getStartOfCurrent14Days());
            let baseWeight = 52.0;

            for (let i = 0; i < 28; i++) {
                const d = new Date(startMon);
                d.setDate(startMon.getDate() - 14 + i);
                const dateStr = formatDate(d);

                baseWeight += (Math.random() - 0.52) * 0.18;
                
                const isWakeMissing = (i === 5 || i === 12);
                const isBedMissing = (i === 8);

                const wake = isWakeMissing ? null : parseFloat(baseWeight.toFixed(2));
                const afterBreakfast = parseFloat(((wake || baseWeight) + 0.2 + Math.random() * 0.25).toFixed(2));
                const afterDinner = parseFloat((afterBreakfast + 0.3 + Math.random() * 0.35).toFixed(2));
                
                const bedGain = (i % 3 === 0) ? (0.2 + Math.random() * 0.2) : (-0.15 - Math.random() * 0.2);
                const bed = isBedMissing ? null : parseFloat((afterDinner + bedGain).toFixed(2));

                weightData[dateStr] = { wake, afterBreakfast, afterDinner, bed };
            }
            saveDataToStorage();
        }

        // Storage Functions
        function loadDataFromStorage() {
            try {
                const saved = localStorage.getItem(STORAGE_KEY) || localStorage.getItem('weight_tracker_data_v2') || localStorage.getItem('weight_tracker_data_v1');
                if (saved) {
                    const parsed = JSON.parse(saved);
                    weightData = {};
                    Object.keys(parsed).forEach(k => {
                        const item = parsed[k];
                        weightData[k] = {
                            wake: item.wake ?? null,
                            afterBreakfast: item.afterBreakfast ?? item.breakfast ?? item.beforeBreakfast ?? null,
                            afterDinner: item.afterDinner ?? item.dinner ?? null,
                            bed: item.bed ?? null
                        };
                    });
                }
            } catch (e) {
                console.error("Failed to load from localStorage", e);
                weightData = {};
            }
        }

        function saveDataToStorage() {
            try {
                localStorage.setItem(STORAGE_KEY, JSON.stringify(stringify(weightData)));
            } catch (e) {
                console.error("Failed to save to localStorage", e);
            }
        }

        function stringify(obj) {
            return JSON.stringify(obj);
        }

        // Setup DOM Event Handlers
        function setupEventListeners() {
            document.getElementById('btnPrevPeriod').addEventListener('click', () => {
                currentStartDate = addDays(currentStartDate, -14);
                renderAll();
            });

            document.getElementById('btnNextPeriod').addEventListener('click', () => {
                currentStartDate = addDays(currentStartDate, 14);
                renderAll();
            });

            document.getElementById('btnTodayPeriod').addEventListener('click', () => {
                currentStartDate = getStartOfCurrent14Days();
                renderAll();
            });

            // Enhanced Print Action Handling
            document.getElementById('btnPrintA4').addEventListener('click', () => {
                try {
                    if (typeof window.print === 'function') {
                        window.print();
                    } else {
                        throw new Error('window.print is not supported');
                    }
                } catch (err) {
                    console.error("Print invocation failed:", err);
                    alert("お使いのブラウザ・環境では「印刷」ボタンがブロックされているか対応していません。\n\n【対処法】\nブラウザのメニュー（右上「⋮」や「共有」ボタン）から「印刷」または「PDFで保存」をお試しいただくか、Chrome / Safari などの標準ブラウザで開いてみてください。");
                }
            });

            document.getElementById('btnSampleData').addEventListener('click', () => {
                generateSampleData();
                showToast("かわいいサンプルデータをセットしたよ🌸");
                renderAll();
            });

            document.getElementById('btnClearData').addEventListener('click', () => {
                if (confirm("すべてのきろくを削除してもいいですか？")) {
                    weightData = {};
                    saveDataToStorage();
                    showToast("データをリセットしたよ");
                    renderAll();
                }
            });

            document.getElementById('btnExportCsv').addEventListener('click', exportCsv);

            document.getElementById('btnViewTimeline').addEventListener('click', () => switchView('timeline'));
            document.getElementById('btnViewDaily').addEventListener('click', () => switchView('daily'));
            document.getElementById('btnViewDiff').addEventListener('click', () => switchView('diff'));

            document.getElementById('innerWeightForm').addEventListener('submit', (e) => {
                e.preventDefault();
                const date = document.getElementById('inputDate').value;
                
                const parseInput = (id) => {
                    const val = document.getElementById(id).value.trim();
                    return val === '' || isNaN(parseFloat(val)) ? null : parseFloat(val);
                };

                const wake = parseInput('inputWake');
                const afterBreakfast = parseInput('inputAfterBreakfast');
                const afterDinner = parseInput('inputAfterDinner');
                const bed = parseInput('inputBed');

                if (!date) return;

                weightData[date] = { wake, afterBreakfast, afterDinner, bed };
                saveDataToStorage();

                const currentDates = getCurrent14DaysList();
                if (!currentDates.includes(date)) {
                    currentStartDate = getMondayOfWeek(date);
                }

                showToast(`${date} のきろくを保存しました🎀`);
                renderAll();
            });

            const afterDinnerInput = document.getElementById('inputAfterDinner');
            const bedInput = document.getElementById('inputBed');

            function checkFormDiff() {
                const dinnerVal = parseFloat(afterDinnerInput.value);
                const bedVal = parseFloat(bedInput.value);
                const alertEl = document.getElementById('formAlert');

                if (!isNaN(dinnerVal) && !isNaN(bedVal)) {
                    if (bedVal > dinnerVal) {
                        const diff = (bedVal - dinnerVal).toFixed(2);
                        alertEl.className = 'p-3 rounded-2xl text-xs font-bold flex items-center gap-2 bg-rose-100 text-rose-700 border border-rose-200';
                        alertEl.innerHTML = `🍓 お休み前が夕食後より <strong>+${diff}kg</strong> 増えています！（夜の間食注意・苺色ハイライト）`;
                        alertEl.classList.remove('hidden');
                    } else {
                        const diff = (dinnerVal - bedVal).toFixed(2);
                        alertEl.className = 'p-3 rounded-2xl text-xs font-bold flex items-center gap-2 bg-emerald-100 text-emerald-800 border border-emerald-200';
                        alertEl.innerHTML = `✨ お休み前が夕食後より <strong>-${diff}kg</strong> 減っているか同等です！素晴らしいナイトケア♪`;
                        alertEl.classList.remove('hidden');
                    }
                    lucide.createIcons();
                } else {
                    alertEl.classList.add('hidden');
                }
            }

            afterDinnerInput.addEventListener('input', checkFormDiff);
            bedInput.addEventListener('input', checkFormDiff);

            document.getElementById('inputDate').addEventListener('change', (e) => {
                const selectedDate = e.target.value;
                loadFormDataForDate(selectedDate);
                checkFormDiff();
            });
        }

        // VIEW 1: 56-Point Continuous Timeline Chart
        function renderTimelineChart(ctx, dates) {
            const labels = [];
            const dataPoints = [];
            const pointColors = [];
            const pointRadius = [];
            const isNightGainSegment = [];

            const timeKeys = [
                { key: 'wake', name: '起床時' },
                { key: 'afterBreakfast', name: '朝食後' },
                { key: 'afterDinner', name: '夕食後' },
                { key: 'bed', name: 'お休み前' }
            ];

            dates.forEach(d => {
                const dayLog = weightData[d] || {};
                const shortDate = formatShortDate(d);

                const dinnerVal = dayLog.afterDinner;
                const bedVal = dayLog.bed;
                const isRedDay = (dinnerVal !== null && dinnerVal !== undefined && bedVal !== null && bedVal !== undefined && bedVal > dinnerVal);

                timeKeys.forEach(tk => {
                    labels.push(`${shortDate} ${tk.name}`);
                    const val = (dayLog[tk.key] !== undefined && dayLog[tk.key] !== null) ? dayLog[tk.key] : null;
                    dataPoints.push(val);

                    if (tk.key === 'bed' && isRedDay) {
                        pointColors.push('#f43f5e'); // Soft Rose/Strawberry
                        pointRadius.push(7);
                    } else if (tk.key === 'wake') {
                        pointColors.push('#38bdf8'); // Sky Blue
                        pointRadius.push(4.5);
                    } else if (tk.key === 'afterBreakfast') {
                        pointColors.push('#fbbf24'); // Amber
                        pointRadius.push(4.5);
                    } else if (tk.key === 'afterDinner') {
                        pointColors.push('#34d399'); // Mint Green
                        pointRadius.push(4.5);
                    } else {
                        pointColors.push('#c084fc'); // Pastel Purple
                        pointRadius.push(4.5);
                    }

                    isNightGainSegment.push({
                        date: d,
                        type: tk.key,
                        isRed: isRedDay
                    });
                });
            });

            chartInstance = new Chart(ctx, {
                type: 'line',
                data: {
                    labels: labels,
                    datasets: [{
                        label: '体重 (kg)',
                        data: dataPoints,
                        borderColor: '#f472b6',
                        borderWidth: 2.5,
                        tension: 0.2,
                        pointBackgroundColor: pointColors,
                        pointBorderColor: pointColors,
                        pointRadius: pointRadius,
                        pointHoverRadius: 8,
                        spanGaps: false,
                        segment: {
                            borderColor: ctx => {
                                const p0 = ctx.p0DataIndex;
                                const p1 = ctx.p1DataIndex;
                                if (p0 !== undefined && p1 !== undefined) {
                                    if (p0 % 4 === 2 && p1 % 4 === 3) {
                                        const y0 = ctx.p0.parsed.y;
                                        const y1 = ctx.p1.parsed.y;
                                        if (y1 > y0) {
                                            return '#f43f5e';
                                        }
                                    }
                                }
                                return '#f472b6';
                            },
                            borderWidth: ctx => {
                                const p0 = ctx.p0DataIndex;
                                const p1 = ctx.p1DataIndex;
                                if (p0 % 4 === 2 && p1 % 4 === 3 && ctx.p1.parsed.y > ctx.p0.parsed.y) {
                                    return 3.5;
                                }
                                return 2.5;
                            }
                        }
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: {
                        legend: { display: false },
                        tooltip: {
                            callbacks: {
                                label: function(context) {
                                    const val = context.parsed.y;
                                    if (val === null || val === undefined) return '未測定 (空白)';
                                    const idx = context.dataIndex;
                                    const dayInfo = isNightGainSegment[idx];
                                    if (dayInfo && dayInfo.type === 'bed' && dayInfo.isRed) {
                                        return `お休み前: ${val} kg (🍓 夕食後より増えています)`;
                                    }
                                    return `体重: ${val} kg`;
                                }
                            }
                        }
                    },
                    scales: {
                        x: {
                            grid: { display: false },
                            ticks: {
                                maxRotation: 45,
                                autoSkip: true,
                                maxTicksLimit: 14,
                                callback: function(val, index) {
                                    if (index % 4 === 0) {
                                        return labels[index].split(' ')[0];
                                    }
                                    return '';
                                },
                                font: { size: 11, family: 'M PLUS Rounded 1c' }
                            }
                        },
                        y: {
                            suggestedMin: getMinWeight(dataPoints) - 0.5,
                            suggestedMax: getMaxWeight(dataPoints) + 0.5,
                            ticks: {
                                stepSize: 0.5,
                                callback: value => value.toFixed(1) + ' kg',
                                font: { family: 'M PLUS Rounded 1c' }
                            }
                        }
                    }
                }
            });
        }

        // VIEW 2: 4-Line Daily Comparison Chart
        function renderDailyChart(ctx, dates) {
            const shortLabels = dates.map(d => formatShortDate(d));

            const wakeData = [];
            const afterBfData = [];
            const afterDinnerData = [];
            const bedData = [];

            const bedPointColors = [];
            const bedPointRadius = [];

            dates.forEach(d => {
                const log = weightData[d] || {};
                wakeData.push(log.wake ?? null);
                afterBfData.push(log.afterBreakfast ?? null);
                afterDinnerData.push(log.afterDinner ?? null);
                bedData.push(log.bed ?? null);

                if (log.afterDinner !== null && log.afterDinner !== undefined &&
                    log.bed !== null && log.bed !== undefined && log.bed > log.afterDinner) {
                    bedPointColors.push('#f43f5e');
                    bedPointRadius.push(7);
                } else {
                    bedPointColors.push('#c084fc');
                    bedPointRadius.push(4.5);
                }
            });

            chartInstance = new Chart(ctx, {
                type: 'line',
                data: {
                    labels: shortLabels,
                    datasets: [
                        {
                            label: '☀️ 起床時',
                            data: wakeData,
                            borderColor: '#38bdf8',
                            backgroundColor: '#38bdf8',
                            spanGaps: false,
                            tension: 0.25
                        },
                        {
                            label: '🥐 朝食後',
                            data: afterBfData,
                            borderColor: '#fbbf24',
                            backgroundColor: '#fbbf24',
                            spanGaps: false,
                            tension: 0.25
                        },
                        {
                            label: '🍽️ 夕食後',
                            data: afterDinnerData,
                            borderColor: '#34d399',
                            backgroundColor: '#34d399',
                            spanGaps: false,
                            tension: 0.25
                        },
                        {
                            label: '🌙 お休み前 (夕食後＜お休み前で🍓色)',
                            data: bedData,
                            borderColor: '#c084fc',
                            backgroundColor: bedPointColors,
                            pointBorderColor: bedPointColors,
                            pointRadius: bedPointRadius,
                            pointHoverRadius: 8,
                            spanGaps: false,
                            tension: 0.25
                        }
                    ]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: {
                        legend: { position: 'top', labels: { font: { family: 'M PLUS Rounded 1c' } } },
                        tooltip: {
                            callbacks: {
                                afterLabel: function(context) {
                                    if (context.datasetIndex === 3) {
                                        const d = dates[context.dataIndex];
                                        const log = weightData[d];
                                        if (log && log.afterDinner !== undefined && log.afterDinner !== null && log.bed !== undefined && log.bed !== null && log.bed > log.afterDinner) {
                                            const diff = (log.bed - log.afterDinner).toFixed(2);
                                            return `🍓 夕食後より +${diff}kg 増えたよ！`;
                                        }
                                    }
                                }
                            }
                        }
                    },
                    scales: {
                        y: {
                            ticks: {
                                stepSize: 0.5,
                                callback: v => v.toFixed(1) + ' kg',
                                font: { family: 'M PLUS Rounded 1c' }
                            }
                        }
                    }
                }
            });
        }

        // VIEW 3: After Dinner vs Bed Diff Bar Chart
        function renderDiffChart(ctx, dates) {
            const shortLabels = dates.map(d => formatShortDate(d));
            const diffData = [];
            const barColors = [];

            dates.forEach(d => {
                const log = weightData[d] || {};
                if (log.afterDinner !== null && log.afterDinner !== undefined && log.bed !== null && log.bed !== undefined) {
                    const diff = parseFloat((log.bed - log.afterDinner).toFixed(2));
                    diffData.push(diff);
                    if (diff > 0) {
                        barColors.push('#f43f5e');
                    } else {
                        barColors.push('#34d399');
                    }
                } else {
                    diffData.push(null);
                    barColors.push('#cbd5e1');
                }
            });

            chartInstance = new Chart(ctx, {
                type: 'bar',
                data: {
                    labels: shortLabels,
                    datasets: [{
                        label: '夕食後 → お休み前の体重変化 (kg)',
                        data: diffData,
                        backgroundColor: barColors,
                        borderRadius: 8
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: {
                        legend: { display: false },
                        tooltip: {
                            callbacks: {
                                label: ctx => {
                                    const val = ctx.parsed.y;
                                    if (val > 0) return `増加: +${val} kg (🍓 夜の間食傾向)`;
                                    return `減少/維持: ${val} kg (✨ スッキリ就寝)`;
                                }
                            }
                        }
                    },
                    scales: {
                        y: {
                            title: { display: true, text: '体重差 (kg)', font: { family: 'M PLUS Rounded 1c' } },
                            ticks: {
                                stepSize: 0.5,
                                callback: v => (v > 0 ? '+' : '') + v.toFixed(2) + ' kg',
                                font: { family: 'M PLUS Rounded 1c' }
                            }
                        }
                    }
                }
            });
        }

        function getMinWeight(arr) {
            const valid = arr.filter(v => v !== null && !isNaN(v));
            return valid.length > 0 ? Math.min(...valid) : 45;
        }

        function getMaxWeight(arr) {
            const valid = arr.filter(v => v !== null && !isNaN(v));
            return valid.length > 0 ? Math.max(...valid) : 60;
        }

        function renderTable(dates) {
            const tbody = document.getElementById('dataTableBody');
            tbody.innerHTML = '';

            dates.forEach(d => {
                const log = weightData[d] || { wake: null, afterBreakfast: null, afterDinner: null, bed: null };
                const shortDate = formatShortDate(d);

                const wakeStr = log.wake !== null && log.wake !== undefined ? log.wake.toFixed(2) : '-';
                const afterBfStr = log.afterBreakfast !== null && log.afterBreakfast !== undefined ? log.afterBreakfast.toFixed(2) : '-';
                const afterDinnerStr = log.afterDinner !== null && log.afterDinner !== undefined ? log.afterDinner.toFixed(2) : '-';
                const bedStr = log.bed !== null && log.bed !== undefined ? log.bed.toFixed(2) : '-';

                let diffStr = '-';
                let isRed = false;
                let badge = '<span class="text-pink-300 text-xs">-</span>';

                if (log.afterDinner !== null && log.afterDinner !== undefined && log.bed !== null && log.bed !== undefined) {
                    const diff = parseFloat((log.bed - log.afterDinner).toFixed(2));
                    if (diff > 0) {
                        diffStr = `<span class="text-rose-500 font-bold">+${diff.toFixed(2)} kg</span>`;
                        isRed = true;
                        badge = `<span class="inline-flex items-center gap-1 px-2.5 py-0.5 rounded-full text-xs font-extrabold bg-rose-100 text-rose-600">
                                    🍓 増加
                                 </span>`;
                    } else if (diff < 0) {
                        diffStr = `<span class="text-emerald-600 font-bold">${diff.toFixed(2)} kg</span>`;
                        badge = `<span class="inline-flex items-center gap-1 px-2.5 py-0.5 rounded-full text-xs font-extrabold bg-emerald-100 text-emerald-700">
                                    ✨ 減少
                                 </span>`;
                    } else {
                        diffStr = `<span class="text-pink-800">0.00 kg</span>`;
                        badge = `<span class="inline-flex items-center gap-1 px-2.5 py-0.5 rounded-full text-xs font-bold bg-pink-100 text-pink-700">
                                    キープ
                                 </span>`;
                    }
                }

                const tr = document.createElement('tr');
                tr.className = `hover:bg-pink-100/60 cursor-pointer transition-colors ${isRed ? 'bg-rose-50/60' : ''}`;
                
                tr.innerHTML = `
                    <td class="py-2.5 px-3 font-bold text-pink-900 flex items-center gap-1.5">
                        ${d} <span class="text-xs text-pink-400 font-medium">(${shortDate.split('(')[1]}</span>
                        ${isRed ? '🍓' : ''}
                    </td>
                    <td class="py-2.5 px-3 text-pink-800">${wakeStr}</td>
                    <td class="py-2.5 px-3 text-pink-800">${afterBfStr}</td>
                    <td class="py-2.5 px-3 text-pink-800 font-bold">${afterDinnerStr}</td>
                    <td class="py-2.5 px-3 ${isRed ? 'text-rose-600 font-black' : 'text-pink-800'}">${bedStr}</td>
                    <td class="py-2.5 px-3">${diffStr}</td>
                    <td class="py-2.5 px-3 text-center">${badge}</td>
                `;

                tr.addEventListener('click', () => {
                    document.getElementById('inputDate').value = d;
                    loadFormDataForDate(d);
                    
                    tr.classList.add('bg-pink-200');
                    setTimeout(() => tr.classList.remove('bg-pink-200'), 500);

                    if (window.innerWidth < 1024) {
                        document.getElementById('weightForm').scrollIntoView({ behavior: 'smooth' });
                    }
                });

                tbody.appendChild(tr);
            });

            lucide.createIcons();
        }

        function exportCsv() {
            const dates = Object.keys(weightData).sort();
            if (dates.length === 0) {
                alert("エクスポートするデータがないよ");
                return;
            }

            let csvContent = "data:text/csv;charset=utf-8,日付,起床時(kg),朝食後(kg),夕食後(kg),お休み前(kg),夕食後→お休み前差分(kg)\n";

            dates.forEach(d => {
                const log = weightData[d];
                const wake = log.wake ?? '';
                const afterBf = log.afterBreakfast ?? '';
                const afterDinner = log.afterDinner ?? '';
                const bed = log.bed ?? '';
                let diff = '';
                if (afterDinner !== '' && bed !== '') {
                    diff = (bed - afterDinner).toFixed(2);
                }
                csvContent += `${d},${wake},${afterBf},${afterDinner},${bed},${diff}\n`;
            });

            const encodedUri = encodeURI(csvContent);
            const link = document.createElement("a");
            link.setAttribute("href", encodedUri);
            link.setAttribute("download", `weight_data_${formatDate(new Date())}.csv`);
            document.body.appendChild(link);
            link.click();
            document.body.removeChild(link);
            showToast("CSVファイルをダウンロードしたよ📄");
        }

        function showToast(message) {
            const toast = document.getElementById('toast');
            const toastMsg = document.getElementById('toastMessage');
            toastMsg.textContent = message;
            toast.classList.remove('translate-y-20', 'opacity-0');
            toast.classList.add('translate-y-0', 'opacity-100');

            setTimeout(() => {
                toast.classList.remove('translate-y-0', 'opacity-100');
                toast.classList.add('translate-y-20', 'opacity-0');
            }, 3000);
        }

        document.addEventListener('DOMContentLoaded', () => {
            initIcons();
            loadDataFromStorage();
            setDefaultFormDate();
            
            currentStartDate = getStartOfCurrent14Days();

            const initialDate = document.getElementById('inputDate').value;
            loadFormDataForDate(initialDate);

            setupEventListeners();
            setupGeminiFeatures();
            setupApiKeyModal();
            renderAll();
        });

        const API_KEY_STORAGE_KEY = 'gemini_api_key_v1';

        function getStoredApiKey() {
            return localStorage.getItem(API_KEY_STORAGE_KEY) || '';
        }

        function setupApiKeyModal() {
            const modal = document.getElementById('apiKeyModal');
            const btnOpen = document.getElementById('btnApiKeyConfig');
            const btnClose = document.getElementById('btnCloseApiModal');
            const btnSave = document.getElementById('btnSaveApiKey');
            const inputKey = document.getElementById('inputApiKey');

            btnOpen.addEventListener('click', () => {
                inputKey.value = getStoredApiKey();
                modal.classList.remove('hidden');
            });

            btnClose.addEventListener('click', () => {
                modal.classList.add('hidden');
            });

            btnSave.addEventListener('click', () => {
                const val = inputKey.value.trim();
                localStorage.setItem(API_KEY_STORAGE_KEY, val);
                modal.classList.add('hidden');
                showToast(val ? "APIキーを保存したよ🔑" : "APIキーをクリアしたよ");
            });
        }

        function ensureApiKey() {
            const key = getStoredApiKey();
            if (!key) {
                document.getElementById('apiKeyModal').classList.remove('hidden');
                alert("Gemini 機能をつかうには APIキー の設定が必要です🎀");
                return null;
            }
            return key;
        }

        // Helper Functions
        async function fetchWithRetry(url, options, maxRetries = 3) {
            let delay = 1000;
            for (let i = 0; i < maxRetries; i++) {
                try {
                    const res = await fetch(url, options);
                    if (res.ok) return res;
                } catch (e) {
                    if (i === maxRetries - 1) throw e;
                }
                await new Promise(r => setTimeout(r, delay));
                delay *= 2;
            }
            throw new Error('API request failed after retries');
        }

        function pcmToWav(pcm16Array, sampleRate = 24000) {
            const numChannels = 1;
            const bytesPerSample = 2;
            const blockAlign = numChannels * bytesPerSample;
            const byteRate = sampleRate * blockAlign;
            const dataSize = pcm16Array.length * bytesPerSample;
            const buffer = new ArrayBuffer(44 + dataSize);
            const view = new DataView(buffer);

            function writeString(offset, string) {
                for (let i = 0; i < string.length; i++) {
                    view.setUint8(offset + i, string.charCodeAt(i));
                }
            }

            writeString(0, 'RIFF');
            view.setUint32(4, 36 + dataSize, true);
            writeString(8, 'WAVE');
            writeString(12, 'fmt ');
            view.setUint32(16, 16, true);
            view.setUint16(20, 1, true);
            view.setUint16(22, numChannels, true);
            view.setUint32(24, sampleRate, true);
            view.setUint32(28, byteRate, true);
            view.setUint16(32, blockAlign, true);
            view.setUint16(34, 16, true);
            writeString(36, 'data');
            view.setUint32(40, dataSize, true);

            let offset = 44;
            for (let i = 0; i < pcm16Array.length; i++, offset += 2) {
                view.setInt16(offset, pcm16Array[i], true);
            }
            return new Blob([buffer], { type: 'audio/wav' });
        }

        let latestSpeechText = "";

        function setupGeminiFeatures() {
            const btnAnalyze = document.getElementById('btnAiAnalyze');
            const btnGenCard = document.getElementById('btnAiGenCard');
            const btnPlayTts = document.getElementById('btnPlayTts');

            btnAnalyze.addEventListener('click', runGeminiAnalysis);
            btnGenCard.addEventListener('click', runGeminiImageCardGen);
            btnPlayTts.addEventListener('click', playSpeechAdvice);
        }

        async function runGeminiAnalysis() {
            const apiKey = ensureApiKey();
            if (!apiKey) return;

            const loadingState = document.getElementById('aiLoadingState');
            const loadingText = document.getElementById('aiLoadingText');
            const resultCard = document.getElementById('aiResultCard');

            const dates = getCurrent14DaysList();
            
            let logsSummary = [];
            let nightGainsCount = 0;

            dates.forEach(d => {
                const log = weightData[d];
                if (!log) return;
                const wake = log.wake ?? '未測定';
                const afterBf = log.afterBreakfast ?? '未測定';
                const afterDinner = log.afterDinner ?? '未測定';
                const bed = log.bed ?? '未測定';
                let nightDiffStr = '計算不可';

                if (log.afterDinner !== null && log.afterDinner !== undefined &&
                    log.bed !== null && log.bed !== undefined) {
                    const diff = log.bed - log.afterDinner;
                    nightDiffStr = `${diff > 0 ? '+' : ''}${diff.toFixed(2)}kg`;
                    if (diff > 0) nightGainsCount++;
                }

                logsSummary.push(`日付:${d} | 起床時:${wake} | 朝食後:${afterBf} | 夕食後:${afterDinner} | お休み前:${bed} | 夕食後→お休み前差:${nightDiffStr}`);
            });

            if (logsSummary.length === 0) {
                alert("分析するためのデータがまだ入っていません。データを入力するか「サンプル入力」を押してね✨");
                return;
            }

            loadingText.textContent = "Gemini が14日間のバイタルログを可愛く分析中...🌸";
            loadingState.classList.remove('hidden');
            resultCard.classList.add('hidden');

            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-3-flash-preview:generateContent?key=${apiKey}`;

            const systemPrompt = "あなたはユーザーの健康と生活習慣を優しく見守る、可愛くて親身なフレンド・ヘルスコーチです。絵文字（🌸, 🥐, 🍽️, ✨, 🎀, 🌙など）を交えながら、ポジティブでやる気が出る言葉遣いでフィードバックしてください。特に「起床時」「朝食後」「夕食後」「お休み前」の傾向と、夕食後から就寝前までの体重変化・夜の間食アドバイスを行ってください。";
            
            const userPrompt = `ユーザーの14日間の記録です：

${logsSummary.join('\n')}

夕食後よりお休み前（就寝直前）に増えた日数は ${nightGainsCount} 日間です。
以下のJSON形式で回答してね：

{
  "grade": "🌸", "⭐", "🌙", "☘️" などの絵文字やランク記号,
  "summary": "やさしく励ます14日間の総評（50〜80文字）",
  "goodPoints": ["褒めポイント1", "褒めポイント2"],
  "nightAnalysis": "夕食後からお休み前までの変化に対する夜の間食・ナイトケアのやさしいアドバイス（100文字程度）",
  "actionTips": ["明日から試せるプチ改善アドバイス1", "アドバイス2", "アドバイス3"],
  "speechText": "音声用の短く明るい応援メッセージ（絵文字なしで読みやすい文章、70文字程度）"
}`;

            const payload = {
                contents: [{ parts: [{ text: userPrompt }] }],
                systemInstruction: { parts: [{ text: systemPrompt }] },
                generationConfig: {
                    responseMimeType: "application/json",
                    responseSchema: {
                        type: "OBJECT",
                        properties: {
                            grade: { type: "STRING" },
                            summary: { type: "STRING" },
                            goodPoints: { type: "ARRAY", items: { type: "STRING" } },
                            nightAnalysis: { type: "STRING" },
                            actionTips: { type: "ARRAY", items: { type: "STRING" } },
                            speechText: { type: "STRING" }
                        },
                        required: ["grade", "summary", "goodPoints", "nightAnalysis", "actionTips", "speechText"]
                    }
                }
            };

            try {
                const response = await fetchWithRetry(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });

                const result = await response.json();
                const jsonText = result.candidates?.[0]?.content?.parts?.[0]?.text;

                if (jsonText) {
                    const data = JSON.parse(jsonText);
                    
                    document.getElementById('aiGradeBadge').textContent = data.grade || '🌸';
                    document.getElementById('aiSummaryTitle').textContent = data.summary;
                    
                    const goodUl = document.getElementById('aiGoodPoints');
                    goodUl.innerHTML = (data.goodPoints || []).map(p => `<li>${p}</li>`).join('');

                    document.getElementById('aiNightAnalysis').textContent = data.nightAnalysis;

                    const actionUl = document.getElementById('aiActionTips');
                    actionUl.innerHTML = (data.actionTips || []).map(a => `<li>${a}</li>`).join('');

                    latestSpeechText = data.speechText || data.summary;

                    resultCard.classList.remove('hidden');
                    showToast("Gemini のアドバイスが届いたよ🌸");
                }
            } catch (err) {
                console.error("Gemini Analysis Error:", err);
                alert("Gemini の分析呼び出しに失敗しました。APIキーを確認してね。");
            } finally {
                loadingState.classList.add('hidden');
            }
        }

        async function playSpeechAdvice() {
            if (!latestSpeechText) {
                alert("最初に「14日間を分析する」を実行してね🌸");
                return;
            }

            const apiKey = ensureApiKey();
            if (!apiKey) return;

            const ttsBtnText = document.getElementById('ttsBtnText');
            const originalText = ttsBtnText.textContent;
            ttsBtnText.textContent = "⏳ 声を作っているよ...";

            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-tts:generateContent?key=${apiKey}`;

            const payload = {
                contents: [{
                    parts: [{ text: `可愛らしく明るい少女のような温かい声でメッセージを話してください: ${latestSpeechText}` }]
                }],
                generationConfig: {
                    responseModalities: ["AUDIO"],
                    speechConfig: {
                        voiceConfig: {
                            prebuiltVoiceConfig: { voiceName: "Aoede" }
                        }
                    }
                }
            };

            try {
                const response = await fetchWithRetry(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });

                const result = await response.json();
                const part = result?.candidates?.[0]?.content?.parts?.[0];
                const base64Audio = part?.inlineData?.data;
                const mimeType = part?.inlineData?.mimeType;

                if (base64Audio && mimeType && mimeType.startsWith("audio/")) {
                    const sampleRateMatch = mimeType.match(/rate=(\d+)/);
                    const sampleRate = sampleRateMatch ? parseInt(sampleRateMatch[1], 10) : 24000;

                    const pcmBuffer = base64ToArrayBuffer(base64Audio);
                    const pcm16 = new Int16Array(pcmBuffer);
                    const wavBlob = pcmToWav(pcm16, sampleRate);
                    const audioUrl = URL.createObjectURL(wavBlob);

                    const player = document.getElementById('ttsAudioPlayer');
                    player.src = audioUrl;
                    player.play();

                    ttsBtnText.textContent = "▶ お話し中...♪";
                    player.onended = () => {
                        ttsBtnText.textContent = originalText;
                    };
                } else {
                    alert("音声データの取得に失敗しました。");
                    ttsBtnText.textContent = originalText;
                }
            } catch (err) {
                console.error("Gemini TTS Error:", err);
                alert("音声アドバイスの生成に失敗しました。");
                ttsBtnText.textContent = originalText;
            }
        }

        async function runGeminiImageCardGen() {
            const apiKey = ensureApiKey();
            if (!apiKey) return;

            const loadingState = document.getElementById('aiLoadingState');
            const loadingText = document.getElementById('aiLoadingText');
            const imgContainer = document.getElementById('aiImageContainer');
            const imgEl = document.getElementById('aiGeneratedBadgeImg');

            loadingText.textContent = "Gemini 3.1 Flash Image で最高にかわいいイラスト画像を作成中...🎨";
            loadingState.classList.remove('hidden');

            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-3.1-flash-image:generateContent?key=${apiKey}`;

            const promptText = "A super cute Japanese kawaii style pastel illustration badge icon, fluffy cute cat character holding a tiny star and cherry blossoms, soft pink and strawberry colors, dreamy aesthetic, high detail illustration, Japanese text overlay reading '14日間がんばったね!'";

            const payload = {
                contents: [{
                    parts: [{ text: promptText }]
                }],
                generationConfig: {
                    responseModalities: ['IMAGE'],
                    imageConfig: { aspectRatio: "1:1" }
                }
            };

            try {
                const response = await fetchWithRetry(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });

                const result = await response.json();
                const part = result?.candidates?.[0]?.content?.parts?.find(p => p.inlineData);

                if (part && part.inlineData) {
                    const base64Data = part.inlineData.data;
                    const mime = part.inlineData.mimeType || 'image/png';
                    imgEl.src = `data:${mime};base64,${base64Data}`;
                    imgContainer.classList.remove('hidden');
                    showToast("かわいいごほうび画像を生成したよ🌸");
                }
            } catch (err) {
                console.error("Gemini Image Gen Error:", err);
                alert("画像の生成に失敗しました。");
            } finally {
                loadingState.classList.add('hidden');
            }
        }

        function base64ToArrayBuffer(base64) {
            const binaryString = atob(base64);
            const len = binaryString.length;
            const bytes = new Uint8Array(len);
            for (let i = 0; i < len; i++) {
                bytes[i] = binaryString.charCodeAt(i);
            }
            return bytes.buffer;
        }
    </script>
</body>
</html>
