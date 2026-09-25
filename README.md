<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>2週間 体重管理・分析トラッカー</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Chart.js -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <!-- Lucide Icons -->
    <script src="https://unpkg.com/lucide@latest"></script>
    <!-- Google Fonts -->
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&family=Noto+Sans+JP:wght@300;400;500;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Noto Sans JP', 'Inter', sans-serif;
            background-color: #f8fafc;
        }
        /* Custom scrollbar */
        ::-webkit-scrollbar {
            width: 6px;
            height: 6px;
        }
        ::-webkit-scrollbar-track {
            background: #f1f5f9;
        }
        ::-webkit-scrollbar-thumb {
            background: #cbd5e1;
            border-radius: 3px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: #94a3b8;
        }
    </style>
</head>
<body class="text-slate-800 min-h-screen flex flex-col">

    <!-- Header -->
    <header class="bg-white border-b border-slate-200 sticky top-0 z-30 shadow-sm">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-4 flex flex-col sm:flex-row items-center justify-between gap-4">
            <div class="flex items-center space-x-3">
                <div class="p-2.5 bg-rose-500 text-white rounded-xl shadow-md shadow-rose-200">
                    <i data-lucide="activity" class="w-6 h-6"></i>
                </div>
                <div>
                    <h1 class="text-xl font-bold text-slate-900 tracking-tight">14日間 体重管理トラッカー</h1>
                    <p class="text-xs text-slate-500">起床・朝食後・夕食後・就寝前の4回測定バイタルログ</p>
                </div>
            </div>
            
            <div class="flex items-center gap-2">
                <button id="btnApiKeyConfig" class="px-3.5 py-2 text-xs font-medium text-purple-700 bg-purple-50 hover:bg-purple-100 rounded-lg transition-colors flex items-center gap-1.5 border border-purple-200">
                    <i data-lucide="key" class="w-4 h-4 text-purple-600"></i>
                    APIキー設定
                </button>
                <button id="btnSampleData" class="px-3.5 py-2 text-xs font-medium text-slate-600 bg-slate-100 hover:bg-slate-200 rounded-lg transition-colors flex items-center gap-1.5">
                    <i data-lucide="sparkles" class="w-4 h-4 text-amber-500"></i>
                    サンプル入力
                </button>
                <button id="btnClearData" class="px-3.5 py-2 text-xs font-medium text-rose-600 bg-rose-50 hover:bg-rose-100 rounded-lg transition-colors flex items-center gap-1.5">
                    <i data-lucide="trash-2" class="w-4 h-4"></i>
                    データ初期化
                </button>
                <button id="btnExportCsv" class="px-3.5 py-2 text-xs font-medium text-emerald-700 bg-emerald-50 hover:bg-emerald-100 rounded-lg transition-colors flex items-center gap-1.5">
                    <i data-lucide="download" class="w-4 h-4"></i>
                    CSV出力
                </button>
            </div>
        </div>
    </header>

    <!-- Main Content -->
    <main class="flex-1 max-w-7xl w-full mx-auto px-4 sm:px-6 lg:px-8 py-6 space-y-6">

        <!-- Date Range Navigation Bar (Top Position) -->
        <div class="bg-white p-4 rounded-2xl border border-slate-200 shadow-sm flex flex-col md:flex-row items-center justify-between gap-4">
            <div class="flex items-center space-x-2">
                <button id="btnPrevPeriod" class="p-2 hover:bg-slate-100 rounded-lg text-slate-600 transition-colors">
                    <i data-lucide="chevron-left" class="w-5 h-5"></i>
                </button>
                <div class="text-center px-4">
                    <span id="periodLabel" class="text-sm font-bold text-slate-800">----年--月--日 〜 ----年--月--日</span>
                    <span class="text-xs text-slate-500 block">（月曜始まり 14日間）</span>
                </div>
                <button id="btnNextPeriod" class="p-2 hover:bg-slate-100 rounded-lg text-slate-600 transition-colors">
                    <i data-lucide="chevron-right" class="w-5 h-5"></i>
                </button>
            </div>

            <button id="btnTodayPeriod" class="px-3.5 py-1.5 text-xs font-medium text-indigo-600 bg-indigo-50 hover:bg-indigo-100 rounded-lg transition-colors">
                今週(月曜始まり)を表示
            </button>
        </div>

        <!-- Quick Entry Form Card -->
        <div class="bg-white rounded-2xl border border-slate-200 shadow-sm p-5 space-y-4">
            <div class="border-b border-slate-100 pb-3 flex items-center justify-between">
                <h3 class="font-bold text-slate-900 flex items-center gap-2 text-base">
                    <i data-lucide="edit-3" class="w-5 h-5 text-indigo-600"></i>
                    体重データの入力
                </h3>
                <span class="text-xs text-slate-400">1日4回測定（入力後「保存」をクリック）</span>
            </div>

            <form id="weightForm" class="space-y-3">
                <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-5 lg:grid-cols-6 gap-3 items-end">
                    <div>
                        <label class="block text-xs font-semibold text-slate-600 mb-1">日付</label>
                        <input type="date" id="inputDate" required class="w-full px-3 py-2 border border-slate-200 rounded-xl text-sm focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500">
                    </div>

                    <div>
                        <label class="block text-xs font-medium text-slate-600 mb-1 flex items-center gap-1">
                            <span class="w-2 h-2 rounded-full bg-blue-500"></span> 起床直後 (kg)
                        </label>
                        <input type="number" step="0.01" min="20" max="250" id="inputWake" placeholder="例: 65.0" class="w-full px-3 py-2 border border-slate-200 rounded-xl text-sm focus:outline-none focus:ring-2 focus:ring-indigo-500">
                    </div>

                    <div>
                        <label class="block text-xs font-medium text-slate-600 mb-1 flex items-center gap-1">
                            <span class="w-2 h-2 rounded-full bg-emerald-500"></span> 朝食直後 (kg)
                        </label>
                        <input type="number" step="0.01" min="20" max="250" id="inputBreakfast" placeholder="例: 65.5" class="w-full px-3 py-2 border border-slate-200 rounded-xl text-sm focus:outline-none focus:ring-2 focus:ring-indigo-500">
                    </div>

                    <div>
                        <label class="block text-xs font-medium text-slate-600 mb-1 flex items-center gap-1">
                            <span class="w-2 h-2 rounded-full bg-amber-500"></span> 夕食直後 (kg)
                        </label>
                        <input type="number" step="0.01" min="20" max="250" id="inputDinner" placeholder="例: 66.2" class="w-full px-3 py-2 border border-slate-200 rounded-xl text-sm focus:outline-none focus:ring-2 focus:ring-indigo-500">
                    </div>

                    <div>
                        <label class="block text-xs font-medium text-slate-600 mb-1 flex items-center gap-1">
                            <span class="w-2 h-2 rounded-full bg-indigo-500"></span> 就寝直前 (kg)
                        </label>
                        <input type="number" step="0.01" min="20" max="250" id="inputBed" placeholder="例: 66.0" class="w-full px-3 py-2 border border-slate-200 rounded-xl text-sm focus:outline-none focus:ring-2 focus:ring-indigo-500">
                    </div>

                    <div class="sm:col-span-2 md:col-span-1 lg:col-span-1">
                        <button type="submit" class="w-full py-2 bg-indigo-600 hover:bg-indigo-700 text-white font-medium text-sm rounded-xl transition-colors shadow-sm shadow-indigo-200 flex items-center justify-center gap-2">
                            <i data-lucide="save" class="w-4 h-4"></i>
                            保存する
                        </button>
                    </div>
                </div>

                <!-- Dynamic Alert inside Form -->
                <div id="formAlert" class="hidden p-3 rounded-xl text-xs flex items-center gap-2"></div>
            </form>
        </div>

        <!-- Metric Summary Cards -->
        <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4">
            <div class="bg-white p-4 rounded-2xl border border-slate-200 shadow-sm flex items-center justify-between">
                <div>
                    <p class="text-xs font-medium text-slate-500">期間内 平均体重</p>
                    <h3 id="statAvgWeight" class="text-2xl font-bold text-slate-900 mt-1">-- <span class="text-sm font-normal text-slate-500">kg</span></h3>
                </div>
                <div class="p-3 bg-blue-50 text-blue-600 rounded-xl">
                    <i data-lucide="scale" class="w-5 h-5"></i>
                </div>
            </div>

            <div class="bg-white p-4 rounded-2xl border border-slate-200 shadow-sm flex items-center justify-between">
                <div>
                    <p class="text-xs font-medium text-slate-500">期間内 体重変化</p>
                    <h3 id="statWeightChange" class="text-2xl font-bold text-slate-900 mt-1">-- <span class="text-sm font-normal text-slate-500">kg</span></h3>
                </div>
                <div id="statWeightChangeIcon" class="p-3 bg-slate-50 text-slate-600 rounded-xl">
                    <i data-lucide="trending-down" class="w-5 h-5"></i>
                </div>
            </div>

            <div class="bg-white p-4 rounded-2xl border border-slate-200 shadow-sm flex items-center justify-between">
                <div>
                    <p class="text-xs font-medium text-slate-500">夕食＜就寝（体重増加）</p>
                    <h3 id="statNightIncreaseCount" class="text-2xl font-bold text-rose-600 mt-1">0 <span class="text-sm font-normal text-slate-500">/ 14日</span></h3>
                </div>
                <div class="p-3 bg-rose-50 text-rose-600 rounded-xl">
                    <i data-lucide="alert-triangle" class="w-5 h-5"></i>
                </div>
            </div>

            <div class="bg-white p-4 rounded-2xl border border-slate-200 shadow-sm flex items-center justify-between">
                <div>
                    <p class="text-xs font-medium text-slate-500">平均 夜間増加量</p>
                    <h3 id="statAvgNightDiff" class="text-2xl font-bold text-slate-900 mt-1">-- <span class="text-sm font-normal text-slate-500">kg</span></h3>
                </div>
                <div class="p-3 bg-amber-50 text-amber-600 rounded-xl">
                    <i data-lucide="moon" class="w-5 h-5"></i>
                </div>
            </div>
        </div>

        <!-- Gemini AI Health Coach Section -->
        <div class="bg-gradient-to-br from-indigo-900 via-purple-900 to-slate-900 rounded-2xl p-5 sm:p-6 text-white shadow-xl space-y-4">
            <div class="flex flex-col sm:flex-row sm:items-center justify-between gap-4 border-b border-white/10 pb-4">
                <div class="flex items-center space-x-3">
                    <div class="p-2.5 bg-gradient-to-tr from-amber-400 to-rose-400 rounded-xl text-slate-900 shadow-lg">
                        <i data-lucide="sparkles" class="w-6 h-6"></i>
                    </div>
                    <div>
                        <div class="flex items-center gap-2">
                            <h2 class="text-lg font-bold tracking-tight">Gemini AI ヘルスコーチ & アドバイザー</h2>
                            <span class="px-2 py-0.5 text-[10px] font-semibold bg-indigo-500/30 text-indigo-200 rounded-full border border-indigo-400/30">AI Powered</span>
                        </div>
                        <p class="text-xs text-indigo-200">14日間の4回測定データを解析し、生活習慣改善と夜間増加の原因をアドバイス</p>
                    </div>
                </div>

                <div class="flex flex-wrap items-center gap-2">
                    <button id="btnAiAnalyze" class="px-4 py-2 bg-gradient-to-r from-amber-400 to-orange-400 hover:from-amber-300 hover:to-orange-300 text-slate-950 font-bold text-xs rounded-xl shadow-md transition-all flex items-center gap-1.5 active:scale-95">
                        <i data-lucide="brain-circuit" class="w-4 h-4"></i>
                        ✨ AIで14日間を分析する
                    </button>
                    <button id="btnAiGenCard" class="px-3.5 py-2 bg-white/10 hover:bg-white/20 text-white font-medium text-xs rounded-xl transition-all flex items-center gap-1.5 border border-white/15">
                        <i data-lucide="image" class="w-4 h-4 text-purple-300"></i>
                        🎨 応援バッジ画像生成
                    </button>
                </div>
            </div>

            <!-- AI Status / Spinner -->
            <div id="aiLoadingState" class="hidden py-8 text-center space-y-3">
                <div class="inline-block animate-spin rounded-full h-8 w-8 border-4 border-amber-400 border-t-transparent"></div>
                <p id="aiLoadingText" class="text-xs text-indigo-200 font-medium">Gemini AI が14日間の体重パターンを分析中...</p>
            </div>

            <!-- AI Output Card (Hidden initially) -->
            <div id="aiResultCard" class="hidden bg-white/10 backdrop-blur-md border border-white/10 rounded-xl p-4 sm:p-5 space-y-4">
                <div class="flex flex-col sm:flex-row items-start sm:items-center justify-between gap-3 border-b border-white/10 pb-3">
                    <div class="flex items-center gap-3">
                        <div id="aiGradeBadge" class="w-12 h-12 rounded-xl bg-amber-400 text-slate-950 font-black text-2xl flex items-center justify-center shadow-lg">
                            A
                        </div>
                        <div>
                            <span class="text-[10px] text-indigo-300 uppercase tracking-widest block font-bold">14日間の習慣評価</span>
                            <h4 id="aiSummaryTitle" class="text-sm font-bold text-white">順調なペースです！夜間の管理を意識しましょう</h4>
                        </div>
                    </div>

                    <!-- TTS Voice Playback Button -->
                    <button id="btnPlayTts" class="px-3 py-1.5 bg-indigo-600/80 hover:bg-indigo-500 text-white rounded-lg text-xs font-medium flex items-center gap-1.5 border border-indigo-400/40 transition-colors">
                        <i data-lucide="volume-2" class="w-4 h-4 text-amber-300"></i>
                        <span id="ttsBtnText">🔊 AI音声アドバイスを聴く</span>
                    </button>
                </div>

                <div class="grid grid-cols-1 md:grid-cols-3 gap-4 text-xs">
                    <!-- Good Points -->
                    <div class="bg-slate-950/40 p-3.5 rounded-xl border border-white/5 space-y-1.5">
                        <h5 class="font-bold text-emerald-400 flex items-center gap-1.5">
                            <i data-lucide="check-circle" class="w-4 h-4"></i>
                            良い点・成果
                        </h5>
                        <ul id="aiGoodPoints" class="space-y-1 text-slate-300 list-disc list-inside">
                            <!-- Populated by JS -->
                        </ul>
                    </div>

                    <!-- Night Gain Analysis -->
                    <div class="bg-slate-950/40 p-3.5 rounded-xl border border-white/5 space-y-1.5">
                        <h5 class="font-bold text-rose-400 flex items-center gap-1.5">
                            <i data-lucide="moon" class="w-4 h-4"></i>
                            夜間増加の傾向と対策
                        </h5>
                        <p id="aiNightAnalysis" class="text-slate-300 leading-relaxed">
                            <!-- Populated by JS -->
                        </p>
                    </div>

                    <!-- Actionable Tips -->
                    <div class="bg-slate-950/40 p-3.5 rounded-xl border border-white/5 space-y-1.5">
                        <h5 class="font-bold text-amber-300 flex items-center gap-1.5">
                            <i data-lucide="lightbulb" class="w-4 h-4"></i>
                            明日からの改善アクション
                        </h5>
                        <ul id="aiActionTips" class="space-y-1 text-slate-300 list-disc list-inside">
                            <!-- Populated by JS -->
                        </ul>
                    </div>
                </div>

                <!-- Hidden audio container for TTS -->
                <audio id="ttsAudioPlayer" class="hidden"></audio>
            </div>

            <!-- Generated Illustration Badge Display -->
            <div id="aiImageContainer" class="hidden bg-white/5 border border-white/10 rounded-xl p-4 flex flex-col sm:flex-row items-center gap-4">
                <img id="aiGeneratedBadgeImg" class="w-32 h-32 rounded-xl object-cover shadow-md border border-white/20" alt="AI Motivation Badge" />
                <div class="space-y-1 text-center sm:text-left">
                    <span class="text-[10px] text-amber-300 font-bold uppercase tracking-wider">Gemini 3.1 Flash Image 生成</span>
                    <h5 class="text-sm font-bold text-white">14日間達成モチベーションカード</h5>
                    <p class="text-xs text-slate-300">継続的な測定お疲れ様です！このカードを励みに次の14日間も頑張りましょう。</p>
                </div>
            </div>
        </div>

        <!-- Chart Section -->
        <div class="bg-white rounded-2xl border border-slate-200 shadow-sm p-5 space-y-4">
            <div class="flex flex-col sm:flex-row sm:items-center justify-between gap-4 border-b border-slate-100 pb-4">
                <div>
                    <h2 class="text-lg font-bold text-slate-900 flex items-center gap-2">
                        <i data-lucide="line-chart" class="w-5 h-5 text-indigo-600"></i>
                        14日間の体重推移グラフ
                    </h2>
                    <p class="text-xs text-slate-500 mt-0.5">「夕食直後より就寝直前が数値が大きい日」は赤色で強調表示されています</p>
                </div>

                <!-- View Switcher Tabs -->
                <div class="inline-flex p-1 bg-slate-100 rounded-xl text-xs font-medium self-start sm:self-auto">
                    <button id="btnViewTimeline" class="px-3 py-1.5 rounded-lg bg-white text-slate-900 shadow-xs transition-all">
                        時系列連続表示 (56点)
                    </button>
                    <button id="btnViewDaily" class="px-3 py-1.5 rounded-lg text-slate-600 hover:text-slate-900 transition-all">
                        時間帯別14日比較 (4本線)
                    </button>
                    <button id="btnViewDiff" class="px-3 py-1.5 rounded-lg text-slate-600 hover:text-slate-900 transition-all">
                        夕食 vs 就寝 差分表示
                    </button>
                </div>
            </div>

            <!-- Legend and Indicator Help -->
            <div class="flex flex-wrap items-center gap-4 text-xs bg-slate-50 p-3 rounded-xl">
                <div class="flex items-center gap-1.5">
                    <span class="w-3 h-3 rounded-full bg-blue-500 inline-block"></span>
                    <span class="text-slate-600">起床直後</span>
                </div>
                <div class="flex items-center gap-1.5">
                    <span class="w-3 h-3 rounded-full bg-emerald-500 inline-block"></span>
                    <span class="text-slate-600">朝食直後</span>
                </div>
                <div class="flex items-center gap-1.5">
                    <span class="w-3 h-3 rounded-full bg-amber-500 inline-block"></span>
                    <span class="text-slate-600">夕食直後</span>
                </div>
                <div class="flex items-center gap-1.5">
                    <span class="w-3 h-3 rounded-full bg-indigo-500 inline-block"></span>
                    <span class="text-slate-600">就寝直前（通常）</span>
                </div>
                <div class="flex items-center gap-1.5 font-bold text-rose-600 bg-rose-50 px-2 py-0.5 rounded border border-rose-200">
                    <span class="w-3 h-3 rounded-full bg-rose-600 inline-block animate-pulse"></span>
                    <span>就寝直前（夕食より増加：赤色警告）</span>
                </div>
                <div class="text-slate-400 italic">
                    ※ 測り忘れた項目は空欄（グラフの線が切れて空白表示）になります
                </div>
            </div>

            <!-- Chart Canvas Container -->
            <div class="relative w-full h-[380px] sm:h-[420px]">
                <canvas id="weightChart"></canvas>
            </div>
        </div>

        <!-- 14-Day Table View (Full Width) -->
        <div class="bg-white rounded-2xl border border-slate-200 shadow-sm p-5 space-y-3">
            <div class="flex items-center justify-between border-b border-slate-100 pb-3">
                <h3 class="font-bold text-slate-900 flex items-center gap-2 text-base">
                    <i data-lucide="table" class="w-4 h-4 text-indigo-600"></i>
                    14日間の記録一覧
                </h3>
                <span class="text-xs text-slate-500">※ 表の行をクリックすると上の入力フォームにデータが読み込まれます</span>
            </div>

            <div class="overflow-x-auto">
                <table class="w-full text-left border-collapse">
                    <thead>
                        <tr class="bg-slate-50 border-b border-slate-200 text-xs font-semibold text-slate-600">
                            <th class="py-2.5 px-3">日付</th>
                            <th class="py-2.5 px-3">起床直後</th>
                            <th class="py-2.5 px-3">朝食直後</th>
                            <th class="py-2.5 px-3">夕食直後</th>
                            <th class="py-2.5 px-3">就寝直前</th>
                            <th class="py-2.5 px-3">夕食→就寝 差分</th>
                            <th class="py-2.5 px-3 text-center">判定</th>
                        </tr>
                    </thead>
                    <tbody id="dataTableBody" class="text-sm divide-y divide-slate-100">
                        <!-- Populated dynamically via JS -->
                    </tbody>
                </table>
            </div>
        </div>

    </main>

    <!-- API Key Settings Modal -->
    <div id="apiKeyModal" class="fixed inset-0 bg-slate-900/60 backdrop-blur-sm z-50 flex items-center justify-center hidden p-4">
        <div class="bg-white rounded-2xl max-w-md w-full p-6 space-y-4 shadow-2xl border border-slate-100">
            <div class="flex items-center justify-between border-b border-slate-100 pb-3">
                <h3 class="font-bold text-slate-900 text-base flex items-center gap-2">
                    <i data-lucide="key" class="w-5 h-5 text-purple-600"></i>
                    Gemini APIキーの設定
                </h3>
                <button id="btnCloseApiModal" class="text-slate-400 hover:text-slate-600">
                    <i data-lucide="x" class="w-5 h-5"></i>
                </button>
            </div>
            
            <p class="text-xs text-slate-600 leading-relaxed">
                GitHub Pages等での安全な運用のために、ご自身の Gemini APIキーを入力してください。<br>
                キーはブラウザ（localStorage）にのみ保存され、外部に送信されることはありません。
            </p>

            <div>
                <label class="block text-xs font-semibold text-slate-700 mb-1">Gemini API Key</label>
                <input type="password" id="inputApiKey" placeholder="AIzaSy..." class="w-full px-3 py-2 border border-slate-200 rounded-xl text-sm focus:outline-none focus:ring-2 focus:ring-purple-500">
            </div>

            <div class="flex items-center justify-between text-[11px] text-slate-500">
                <a href="https://aistudio.google.com/app/apikey" target="_blank" rel="noopener noreferrer" class="text-indigo-600 hover:underline flex items-center gap-1">
                    APIキーを無料で取得する <i data-lucide="external-link" class="w-3 h-3"></i>
                </a>
            </div>

            <div class="flex gap-2 pt-2">
                <button id="btnSaveApiKey" class="w-full py-2 bg-purple-600 hover:bg-purple-700 text-white font-bold text-xs rounded-xl transition-colors">
                    設定を保存
                </button>
            </div>
        </div>
    </div>

    <!-- Notification Toast -->
    <div id="toast" class="fixed bottom-5 right-5 transform translate-y-20 opacity-0 transition-all duration-300 bg-slate-900 text-white px-4 py-3 rounded-xl shadow-lg flex items-center gap-2 text-sm z-50">
        <i data-lucide="check-circle" class="w-4 h-4 text-emerald-400"></i>
        <span id="toastMessage">保存しました</span>
    </div>

    <script>
        // Global Application State
        const STORAGE_KEY = 'weight_tracker_data_v1';
        let weightData = {}; // Format: { "YYYY-MM-DD": { wake: 65.0, breakfast: 65.5, dinner: 66.2, bed: 66.0 } }
        let currentStartDate = '';
        let currentView = 'timeline'; // 'timeline' | 'daily' | 'diff'
        let chartInstance = null;

        const DAY_NAMES = ['日', '月', '火', '水', '木', '金', '土'];

        function initIcons() {
            lucide.createIcons();
        }

        // Calculates the Monday of the week for a given date
        function getMondayOfWeek(d) {
            const date = new Date(d);
            const day = date.getDay();
            // day: 0 (Sun) -> diff -6, 1 (Mon) -> diff 0, ..., 6 (Sat) -> diff -5
            const diff = date.getDate() - day + (day === 0 ? -6 : 1);
            const monday = new Date(date.setDate(diff));
            return formatDate(monday);
        }

        // Returns start date (Monday of the current week)
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
            
            const labelEl = document.getElementById('periodLabel');
            if (labelEl) {
                labelEl.textContent = `${formatFull(start)} 〜 ${formatFull(end)}`;
            }
        }

        function updateStats(dates) {
            let allWeights = [];
            let nightIncreaseCount = 0;
            let nightDiffs = [];

            dates.forEach(d => {
                const log = weightData[d];
                if (!log) return;

                ['wake', 'breakfast', 'dinner', 'bed'].forEach(key => {
                    if (log[key] !== null && log[key] !== undefined && !isNaN(log[key])) {
                        allWeights.push(log[key]);
                    }
                });

                if (log.dinner !== null && log.dinner !== undefined && log.bed !== null && log.bed !== undefined) {
                    const diff = log.bed - log.dinner;
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
                avgWeightEl.innerHTML = `${avg.toFixed(2)} <span class="text-sm font-normal text-slate-500">kg</span>`;
            } else {
                avgWeightEl.innerHTML = `-- <span class="text-sm font-normal text-slate-500">kg</span>`;
            }

            // 2. Weight Change
            const weightChangeEl = document.getElementById('statWeightChange');
            const changeIconEl = document.getElementById('statWeightChangeIcon');
            if (allWeights.length >= 2) {
                const firstVal = allWeights[0];
                const lastVal = allWeights[allWeights.length - 1];
                const change = lastVal - firstVal;
                const sign = change > 0 ? '+' : '';
                const colorClass = change > 0 ? 'text-rose-600' : (change < 0 ? 'text-emerald-600' : 'text-slate-900');
                
                weightChangeEl.className = `text-2xl font-bold mt-1 ${colorClass}`;
                weightChangeEl.innerHTML = `${sign}${change.toFixed(2)} <span class="text-sm font-normal text-slate-500">kg</span>`;
                
                if (changeIconEl) {
                    if (change > 0) {
                        changeIconEl.className = "p-3 bg-rose-50 text-rose-600 rounded-xl";
                        changeIconEl.innerHTML = '<i data-lucide="trending-up" class="w-5 h-5"></i>';
                    } else if (change < 0) {
                        changeIconEl.className = "p-3 bg-emerald-50 text-emerald-600 rounded-xl";
                        changeIconEl.innerHTML = '<i data-lucide="trending-down" class="w-5 h-5"></i>';
                    } else {
                        changeIconEl.className = "p-3 bg-slate-50 text-slate-600 rounded-xl";
                        changeIconEl.innerHTML = '<i data-lucide="minus" class="w-5 h-5"></i>';
                    }
                }
            } else {
                weightChangeEl.className = "text-2xl font-bold text-slate-900 mt-1";
                weightChangeEl.innerHTML = `-- <span class="text-sm font-normal text-slate-500">kg</span>`;
            }

            // 3. Night Increase Count
            const countEl = document.getElementById('statNightIncreaseCount');
            countEl.innerHTML = `${nightIncreaseCount} <span class="text-sm font-normal text-slate-500">/ 14日</span>`;

            // 4. Avg Night Diff
            const avgNightDiffEl = document.getElementById('statAvgNightDiff');
            if (nightDiffs.length > 0) {
                const avgDiff = nightDiffs.reduce((a, b) => a + b, 0) / nightDiffs.length;
                const sign = avgDiff > 0 ? '+' : '';
                const colorClass = avgDiff > 0 ? 'text-rose-600' : (avgDiff < 0 ? 'text-emerald-600' : 'text-slate-900');
                avgNightDiffEl.className = `text-2xl font-bold mt-1 ${colorClass}`;
                avgNightDiffEl.innerHTML = `${sign}${avgDiff.toFixed(2)} <span class="text-sm font-normal text-slate-500">kg</span>`;
            } else {
                avgNightDiffEl.className = "text-2xl font-bold text-slate-900 mt-1";
                avgNightDiffEl.innerHTML = `-- <span class="text-sm font-normal text-slate-500">kg</span>`;
            }

            lucide.createIcons();
        }

        function switchView(viewName) {
            currentView = viewName;
            const btnTimeline = document.getElementById('btnViewTimeline');
            const btnDaily = document.getElementById('btnViewDaily');
            const btnDiff = document.getElementById('btnViewDiff');

            const activeClass = "px-3 py-1.5 rounded-lg bg-white text-slate-900 shadow-xs transition-all";
            const inactiveClass = "px-3 py-1.5 rounded-lg text-slate-600 hover:text-slate-900 transition-all";

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
                document.getElementById('inputBreakfast').value = d.breakfast ?? '';
                document.getElementById('inputDinner').value = d.dinner ?? '';
                document.getElementById('inputBed').value = d.bed ?? '';
            } else {
                document.getElementById('inputWake').value = '';
                document.getElementById('inputBreakfast').value = '';
                document.getElementById('inputDinner').value = '';
                document.getElementById('inputBed').value = '';
            }
        }

        function generateSampleData() {
            weightData = {};
            const startMon = parseDateStr(getStartOfCurrent14Days());
            let baseWeight = 65.0;

            // Generate 28 days of data (2 full 2-week periods starting on Monday)
            for (let i = 0; i < 28; i++) {
                const d = new Date(startMon);
                d.setDate(startMon.getDate() - 14 + i);
                const dateStr = formatDate(d);

                baseWeight += (Math.random() - 0.52) * 0.2;
                
                // Simulate occasional missing data (測り忘れ)
                const isWakeMissing = (i === 5 || i === 12);
                const isBedMissing = (i === 8);

                const wake = isWakeMissing ? null : parseFloat(baseWeight.toFixed(2));
                const breakfast = parseFloat(((wake || baseWeight) + 0.3 + Math.random() * 0.4).toFixed(2));
                const dinner = parseFloat(((wake || baseWeight) + 0.8 + Math.random() * 0.6).toFixed(2));
                
                const bedGain = (i % 3 === 0) ? (0.2 + Math.random() * 0.4) : (-0.1 - Math.random() * 0.3);
                const bed = isBedMissing ? null : parseFloat((dinner + bedGain).toFixed(2));

                weightData[dateStr] = { wake, breakfast, dinner, bed };
            }
            saveDataToStorage();
        }

        // Storage Functions
        function loadDataFromStorage() {
            try {
                const saved = localStorage.getItem(STORAGE_KEY);
                if (saved) {
                    weightData = JSON.parse(saved);
                }
            } catch (e) {
                console.error("Failed to load from localStorage", e);
                weightData = {};
            }
        }

        function saveDataToStorage() {
            try {
                localStorage.setItem(STORAGE_KEY, JSON.stringify(weightData));
            } catch (e) {
                console.error("Failed to save to localStorage", e);
            }
        }

        // Setup DOM Event Handlers
        function setupEventListeners() {
            // Navigation
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

            // Action Buttons
            document.getElementById('btnSampleData').addEventListener('click', () => {
                generateSampleData();
                showToast("サンプルデータを生成しました（測り忘れ空白含む）");
                renderAll();
            });

            document.getElementById('btnClearData').addEventListener('click', () => {
                if (confirm("全てのデータを消去してもよろしいですか？")) {
                    weightData = {};
                    saveDataToStorage();
                    showToast("データを初期化しました");
                    renderAll();
                }
            });

            document.getElementById('btnExportCsv').addEventListener('click', exportCsv);

            // View Tabs
            document.getElementById('btnViewTimeline').addEventListener('click', () => switchView('timeline'));
            document.getElementById('btnViewDaily').addEventListener('click', () => switchView('daily'));
            document.getElementById('btnViewDiff').addEventListener('click', () => switchView('diff'));

            // Form Submit
            document.getElementById('weightForm').addEventListener('submit', (e) => {
                e.preventDefault();
                const date = document.getElementById('inputDate').value;
                
                const parseInput = (id) => {
                    const val = document.getElementById(id).value.trim();
                    return val === '' || isNaN(parseFloat(val)) ? null : parseFloat(val);
                };

                const wake = parseInput('inputWake');
                const breakfast = parseInput('inputBreakfast');
                const dinner = parseInput('inputDinner');
                const bed = parseInput('inputBed');

                if (!date) return;

                weightData[date] = { wake, breakfast, dinner, bed };
                saveDataToStorage();

                // Auto-adjust view period if input date is outside current 14 days
                const currentDates = getCurrent14DaysList();
                if (!currentDates.includes(date)) {
                    currentStartDate = getMondayOfWeek(date);
                }

                showToast(`${date} のデータを保存し、グラフを更新しました`);
                renderAll();
            });

            // Form dynamic alert preview when entering dinner vs bed
            const dinnerInput = document.getElementById('inputDinner');
            const bedInput = document.getElementById('inputBed');

            function checkFormDiff() {
                const dinner = parseFloat(dinnerInput.value);
                const bed = parseFloat(bedInput.value);
                const alertEl = document.getElementById('formAlert');

                if (!isNaN(dinner) && !isNaN(bed)) {
                    if (bed > dinner) {
                        const diff = (bed - dinner).toFixed(2);
                        alertEl.className = 'p-3 rounded-xl text-xs flex items-center gap-2 bg-rose-50 text-rose-700 border border-rose-200';
                        alertEl.innerHTML = `<i data-lucide="alert-circle" class="w-4 h-4 text-rose-600"></i> 就寝直前が夕食より <strong>+${diff}kg</strong> 大きいです！（グラフで赤色表示）`;
                        alertEl.classList.remove('hidden');
                    } else {
                        const diff = (dinner - bed).toFixed(2);
                        alertEl.className = 'p-3 rounded-xl text-xs flex items-center gap-2 bg-emerald-50 text-emerald-700 border border-emerald-200';
                        alertEl.innerHTML = `<i data-lucide="check-circle-2" class="w-4 h-4 text-emerald-600"></i> 就寝直前が夕食より <strong>-${diff}kg</strong> 減少しています（順調）`;
                        alertEl.classList.remove('hidden');
                    }
                    lucide.createIcons();
                } else {
                    alertEl.classList.add('hidden');
                }
            }

            dinnerInput.addEventListener('input', checkFormDiff);
            bedInput.addEventListener('input', checkFormDiff);

            // Populate form when date input changes
            document.getElementById('inputDate').addEventListener('change', (e) => {
                const selectedDate = e.target.value;
                loadFormDataForDate(selectedDate);
                checkFormDiff();
            });
        }

        // VIEW 1: 56-Point Continuous Timeline Chart (14 days x 4 timepoints)
        function renderTimelineChart(ctx, dates) {
            const labels = [];
            const dataPoints = [];
            const pointColors = [];
            const pointRadius = [];
            const isNightGainSegment = [];

            const timeKeys = [
                { key: 'wake', name: '起床' },
                { key: 'breakfast', name: '朝食' },
                { key: 'dinner', name: '夕食' },
                { key: 'bed', name: '就寝' }
            ];

            dates.forEach(d => {
                const dayLog = weightData[d] || {};
                const shortDate = formatShortDate(d);

                const dinnerVal = dayLog.dinner;
                const bedVal = dayLog.bed;
                const isRedDay = (dinnerVal !== null && dinnerVal !== undefined && bedVal !== null && bedVal !== undefined && bedVal > dinnerVal);

                timeKeys.forEach(tk => {
                    labels.push(`${shortDate} ${tk.name}`);
                    const val = (dayLog[tk.key] !== undefined && dayLog[tk.key] !== null) ? dayLog[tk.key] : null;
                    dataPoints.push(val);

                    if (tk.key === 'bed' && isRedDay) {
                        pointColors.push('#ef4444');
                        pointRadius.push(7);
                    } else if (tk.key === 'wake') {
                        pointColors.push('#3b82f6');
                        pointRadius.push(4);
                    } else if (tk.key === 'breakfast') {
                        pointColors.push('#10b981');
                        pointRadius.push(4);
                    } else if (tk.key === 'dinner') {
                        pointColors.push('#f59e0b');
                        pointRadius.push(4);
                    } else {
                        pointColors.push('#6366f1');
                        pointRadius.push(4);
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
                        borderColor: '#64748b',
                        borderWidth: 2,
                        tension: 0.15,
                        pointBackgroundColor: pointColors,
                        pointBorderColor: pointColors,
                        pointRadius: pointRadius,
                        pointHoverRadius: 8,
                        spanGaps: false, // Breaks the line when measurements are missing (未測定・測り忘れ)
                        segment: {
                            borderColor: ctx => {
                                const p0 = ctx.p0DataIndex;
                                const p1 = ctx.p1DataIndex;
                                if (p0 !== undefined && p1 !== undefined) {
                                    if (p0 % 4 === 2 && p1 % 4 === 3) {
                                        const y0 = ctx.p0.parsed.y;
                                        const y1 = ctx.p1.parsed.y;
                                        if (y1 > y0) {
                                            return '#ef4444'; // Red line for night weight gain
                                        }
                                    }
                                }
                                return '#94a3b8';
                            },
                            borderWidth: ctx => {
                                const p0 = ctx.p0DataIndex;
                                const p1 = ctx.p1DataIndex;
                                if (p0 % 4 === 2 && p1 % 4 === 3 && ctx.p1.parsed.y > ctx.p0.parsed.y) {
                                    return 3.5;
                                }
                                return 2;
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
                                        return `就寝直前: ${val} kg (⚠️ 夕食後より増加)`;
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
                                font: { size: 11 }
                            }
                        },
                        y: {
                            suggestedMin: getMinWeight(dataPoints) - 0.5,
                            suggestedMax: getMaxWeight(dataPoints) + 0.5,
                            ticks: {
                                stepSize: 0.5,
                                callback: value => value.toFixed(1) + ' kg'
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
            const breakfastData = [];
            const dinnerData = [];
            const bedData = [];

            const bedPointColors = [];
            const bedPointRadius = [];

            dates.forEach(d => {
                const log = weightData[d] || {};
                wakeData.push(log.wake ?? null);
                breakfastData.push(log.breakfast ?? null);
                dinnerData.push(log.dinner ?? null);
                bedData.push(log.bed ?? null);

                if (log.dinner !== null && log.dinner !== undefined &&
                    log.bed !== null && log.bed !== undefined && log.bed > log.dinner) {
                    bedPointColors.push('#ef4444');
                    bedPointRadius.push(7);
                } else {
                    bedPointColors.push('#6366f1');
                    bedPointRadius.push(4);
                }
            });

            chartInstance = new Chart(ctx, {
                type: 'line',
                data: {
                    labels: shortLabels,
                    datasets: [
                        {
                            label: '起床直後',
                            data: wakeData,
                            borderColor: '#3b82f6',
                            backgroundColor: '#3b82f6',
                            spanGaps: false,
                            tension: 0.2
                        },
                        {
                            label: '朝食直後',
                            data: breakfastData,
                            borderColor: '#10b981',
                            backgroundColor: '#10b981',
                            spanGaps: false,
                            tension: 0.2
                        },
                        {
                            label: '夕食直後',
                            data: dinnerData,
                            borderColor: '#f59e0b',
                            backgroundColor: '#f59e0b',
                            spanGaps: false,
                            tension: 0.2
                        },
                        {
                            label: '就寝直前 (夕食＜就寝で赤点)',
                            data: bedData,
                            borderColor: '#6366f1',
                            backgroundColor: bedPointColors,
                            pointBorderColor: bedPointColors,
                            pointRadius: bedPointRadius,
                            pointHoverRadius: 8,
                            spanGaps: false,
                            tension: 0.2
                        }
                    ]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: {
                        legend: { position: 'top' },
                        tooltip: {
                            callbacks: {
                                afterLabel: function(context) {
                                    if (context.datasetIndex === 3) {
                                        const d = dates[context.dataIndex];
                                        const log = weightData[d];
                                        if (log && log.dinner !== undefined && log.dinner !== null && log.bed !== undefined && log.bed !== null && log.bed > log.dinner) {
                                            const diff = (log.bed - log.dinner).toFixed(2);
                                            return `⚠️ 夕食直後より +${diff}kg 増加！`;
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
                                callback: v => v.toFixed(1) + ' kg'
                            }
                        }
                    }
                }
            });
        }

        // VIEW 3: Dinner vs Bed Diff Bar Chart
        function renderDiffChart(ctx, dates) {
            const shortLabels = dates.map(d => formatShortDate(d));
            const diffData = [];
            const barColors = [];

            dates.forEach(d => {
                const log = weightData[d] || {};
                if (log.dinner !== null && log.dinner !== undefined && log.bed !== null && log.bed !== undefined) {
                    const diff = parseFloat((log.bed - log.dinner).toFixed(2));
                    diffData.push(diff);
                    if (diff > 0) {
                        barColors.push('#ef4444'); // RED for weight gain
                    } else {
                        barColors.push('#10b981'); // GREEN for decrease or equal
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
                        label: '夕食後 → 就寝前の体重変化 (kg)',
                        data: diffData,
                        backgroundColor: barColors,
                        borderRadius: 6
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
                                    if (val > 0) return `増加: +${val} kg (注意)`;
                                    return `減少: ${val} kg`;
                                }
                            }
                        }
                    },
                    scales: {
                        y: {
                            title: { display: true, text: '体重差 (kg)' },
                            ticks: {
                                stepSize: 0.5,
                                callback: v => (v > 0 ? '+' : '') + v.toFixed(2) + ' kg'
                            }
                        }
                    }
                }
            });
        }

        function getMinWeight(arr) {
            const valid = arr.filter(v => v !== null && !isNaN(v));
            return valid.length > 0 ? Math.min(...valid) : 60;
        }

        function getMaxWeight(arr) {
            const valid = arr.filter(v => v !== null && !isNaN(v));
            return valid.length > 0 ? Math.max(...valid) : 70;
        }

        function renderTable(dates) {
            const tbody = document.getElementById('dataTableBody');
            tbody.innerHTML = '';

            dates.forEach(d => {
                const log = weightData[d] || { wake: null, breakfast: null, dinner: null, bed: null };
                const shortDate = formatShortDate(d);

                const wakeStr = log.wake !== null && log.wake !== undefined ? log.wake.toFixed(2) : '-';
                const breakfastStr = log.breakfast !== null && log.breakfast !== undefined ? log.breakfast.toFixed(2) : '-';
                const dinnerStr = log.dinner !== null && log.dinner !== undefined ? log.dinner.toFixed(2) : '-';
                const bedStr = log.bed !== null && log.bed !== undefined ? log.bed.toFixed(2) : '-';

                let diffStr = '-';
                let isRed = false;
                let badge = '<span class="text-slate-400 text-xs">-</span>';

                if (log.dinner !== null && log.dinner !== undefined && log.bed !== null && log.bed !== undefined) {
                    const diff = parseFloat((log.bed - log.dinner).toFixed(2));
                    if (diff > 0) {
                        diffStr = `<span class="text-rose-600 font-bold">+${diff.toFixed(2)} kg</span>`;
                        isRed = true;
                        badge = `<span class="inline-flex items-center gap-1 px-2 py-0.5 rounded-full text-xs font-semibold bg-rose-100 text-rose-700">
                                    <i data-lucide="trending-up" class="w-3 h-3"></i> 増加
                                 </span>`;
                    } else if (diff < 0) {
                        diffStr = `<span class="text-emerald-600 font-medium">${diff.toFixed(2)} kg</span>`;
                        badge = `<span class="inline-flex items-center gap-1 px-2 py-0.5 rounded-full text-xs font-semibold bg-emerald-100 text-emerald-700">
                                    <i data-lucide="trending-down" class="w-3 h-3"></i> 減少
                                 </span>`;
                    } else {
                        diffStr = `<span class="text-slate-600">0.00 kg</span>`;
                        badge = `<span class="inline-flex items-center gap-1 px-2 py-0.5 rounded-full text-xs font-semibold bg-slate-100 text-slate-600">
                                    キープ
                                 </span>`;
                    }
                }

                const tr = document.createElement('tr');
                tr.className = `hover:bg-indigo-50/50 cursor-pointer transition-colors ${isRed ? 'bg-rose-50/40' : ''}`;
                
                tr.innerHTML = `
                    <td class="py-2.5 px-3 font-medium text-slate-800 flex items-center gap-1.5">
                        ${d} <span class="text-xs text-slate-500 font-normal">(${shortDate.split('(')[1]}</span>
                        ${isRed ? '<i data-lucide="alert-circle" class="w-4 h-4 text-rose-500 inline"></i>' : ''}
                    </td>
                    <td class="py-2.5 px-3 text-slate-700">${wakeStr}</td>
                    <td class="py-2.5 px-3 text-slate-700">${breakfastStr}</td>
                    <td class="py-2.5 px-3 text-slate-700 font-medium">${dinnerStr}</td>
                    <td class="py-2.5 px-3 ${isRed ? 'text-rose-600 font-bold' : 'text-slate-700'}">${bedStr}</td>
                    <td class="py-2.5 px-3">${diffStr}</td>
                    <td class="py-2.5 px-3 text-center">${badge}</td>
                `;

                // Click row to fill form for editing
                tr.addEventListener('click', () => {
                    document.getElementById('inputDate').value = d;
                    loadFormDataForDate(d);
                    
                    // Trigger custom animation/highlight
                    tr.classList.add('bg-indigo-100');
                    setTimeout(() => tr.classList.remove('bg-indigo-100'), 500);

                    // Scroll to form if on mobile
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
                alert("エクスポートするデータがありません");
                return;
            }

            let csvContent = "data:text/csv;charset=utf-8,日付,起床直後(kg),朝食直後(kg),夕食直後(kg),就寝直前(kg),夕食→就寝差分(kg)\n";

            dates.forEach(d => {
                const log = weightData[d];
                const wake = log.wake ?? '';
                const breakfast = log.breakfast ?? '';
                const dinner = log.dinner ?? '';
                const bed = log.bed ?? '';
                let diff = '';
                if (dinner !== '' && bed !== '') {
                    diff = (bed - dinner).toFixed(2);
                }
                csvContent += `${d},${wake},${breakfast},${dinner},${bed},${diff}\n`;
            });

            const encodedUri = encodeURI(csvContent);
            const link = document.createElement("a");
            link.setAttribute("href", encodedUri);
            link.setAttribute("download", `weight_data_${formatDate(new Date())}.csv`);
            document.body.appendChild(link);
            link.click();
            document.body.removeChild(link);
            showToast("CSVをダウンロードしました");
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

        // Initialize application on DOM content loaded
        document.addEventListener('DOMContentLoaded', () => {
            initIcons();
            loadDataFromStorage();
            setDefaultFormDate();
            
            currentStartDate = getStartOfCurrent14Days();

            // Populate initial form data for today
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
                showToast(val ? "APIキーを保存しました" : "APIキー設定を消去しました");
            });
        }

        function ensureApiKey() {
            const key = getStoredApiKey();
            if (!key) {
                document.getElementById('apiKeyModal').classList.remove('hidden');
                alert("Gemini AI機能を利用するには APIキー の設定が必要です。");
                return null;
            }
            return key;
        }

        // ==========================================
        // Gemini API Helpers & Feature Integration
        // ==========================================

        // Exponential backoff fetch helper
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

        // Helper to convert PCM 16-bit audio to WAV Blob
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
            view.setUint16(20, 1, true); // PCM format
            view.setUint16(22, numChannels, true);
            view.setUint32(24, sampleRate, true);
            view.setUint32(28, byteRate, true);
            view.setUint16(32, blockAlign, true);
            view.setUint16(34, 16, true); // 16 bits per sample
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
            
            // Build summary string from current 14 days data
            let logsSummary = [];
            let nightGainsCount = 0;
            let totalDiffs = [];

            dates.forEach(d => {
                const log = weightData[d];
                if (!log) return;
                const wake = log.wake ?? '未測定';
                const bf = log.breakfast ?? '未測定';
                const din = log.dinner ?? '未測定';
                const bed = log.bed ?? '未測定';
                let nightDiffStr = '計算不可';

                if (log.dinner !== null && log.dinner !== undefined && log.bed !== null && log.bed !== undefined) {
                    const diff = log.bed - log.dinner;
                    nightDiffStr = `${diff > 0 ? '+' : ''}${diff.toFixed(2)}kg`;
                    totalDiffs.push(diff);
                    if (diff > 0) nightGainsCount++;
                }

                logsSummary.push(`日付:${d} | 起床:${wake} | 朝食後:${bf} | 夕食後:${din} | 就寝前:${bed} | 夕食→就寝差:${nightDiffStr}`);
            });

            if (logsSummary.length === 0) {
                alert("分析するための体重データが登録されていません。データを入力するかサンプル生成を実行してください。");
                return;
            }

            loadingText.textContent = "Gemini AI (gemini-3-flash-preview) が14日間のバイタルログを解析中...";
            loadingState.classList.remove('hidden');
            resultCard.classList.add('hidden');

            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-3-flash-preview:generateContent?key=${apiKey}`;

            const systemPrompt = "あなたは温かく親身なプロの管理栄養士・ヘルスコーチです。ユーザーの14日間の体重ログ（起床時・朝食後・夕食後・就寝前）を分析し、特に「夕食後より就寝前に体重が増加している日（夜間増加）」に注目して実践的なフィードバックを提示してください。";
            
            const userPrompt = `以下はユーザーの14日間の体重測定記録です。

【測定データ】
${logsSummary.join('\n')}

14日間のうち、夕食後より就寝前が増加した日数は ${nightGainsCount} 日間です。
このデータを元に、以下のJSONフォーマットで回答を返してください。

JSON構造:
{
  "grade": "S", "A", "B", "C" のいずれか（全体的な習慣維持と夕食→就寝の増加頻度で評価）,
  "summary": "14日間の総評メッセージ（50〜80文字程度）",
  "goodPoints": ["評価できる良い点1", "評価できる良い点2"],
  "nightAnalysis": "夕食から就寝までの体重変動（夜間増加）に関する原因分析とアドバイス（100文字程度）",
  "actionTips": ["明日からできる改善アクション1", "改善アクション2", "改善アクション3"],
  "speechText": "音声読み上げ用の短く明るい応援メッセージ（70文字程度）"
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
                    
                    document.getElementById('aiGradeBadge').textContent = data.grade || 'A';
                    document.getElementById('aiSummaryTitle').textContent = data.summary;
                    
                    const goodUl = document.getElementById('aiGoodPoints');
                    goodUl.innerHTML = (data.goodPoints || []).map(p => `<li>${p}</li>`).join('');

                    document.getElementById('aiNightAnalysis').textContent = data.nightAnalysis;

                    const actionUl = document.getElementById('aiActionTips');
                    actionUl.innerHTML = (data.actionTips || []).map(a => `<li>${a}</li>`).join('');

                    latestSpeechText = data.speechText || data.summary;

                    resultCard.classList.remove('hidden');
                    showToast("Gemini AI の14日間分析が完了しました");
                }
            } catch (err) {
                console.error("Gemini Analysis Error:", err);
                alert("Gemini AI の分析呼び出しに失敗しました。APIキーを確認して再試行してください。");
            } finally {
                loadingState.classList.add('hidden');
            }
        }

        async function playSpeechAdvice() {
            if (!latestSpeechText) {
                alert("最初に「AIで14日間を分析する」を実行してください。");
                return;
            }

            const apiKey = ensureApiKey();
            if (!apiKey) return;

            const ttsBtnText = document.getElementById('ttsBtnText');
            const originalText = ttsBtnText.textContent;
            ttsBtnText.textContent = "⏳ 音声生成中...";

            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-tts:generateContent?key=${apiKey}`;

            const payload = {
                contents: [{
                    parts: [{ text: `親しみやすくハキハキとしたトーンで発話してください: ${latestSpeechText}` }]
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

                    ttsBtnText.textContent = "▶ 再生中...";
                    player.onended = () => {
                        ttsBtnText.textContent = originalText;
                    };
                } else {
                    alert("音声データの取得に失敗しました。");
                    ttsBtnText.textContent = originalText;
                }
            } catch (err) {
                console.error("Gemini TTS Error:", err);
                alert("音声読み上げの生成に失敗しました。");
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

            loadingText.textContent = "Gemini 3.1 Flash Image で達成モチベーションバッジ画像を生成中...";
            loadingState.classList.remove('hidden');

            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-3.1-flash-image:generateContent?key=${apiKey}`;

            const promptText = "A cute pastel colorful motivational reward badge icon with a friendly cute weight scale character, soft glowing stars, high resolution modern illustration, text overlay in Japanese reading '14日間達成!'";

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
                    showToast("Gemini 3.1 Flash Image のバッジ画像を生成しました");
                }
            } catch (err) {
                console.error("Gemini Image Gen Error:", err);
                alert("画像バッジの生成に失敗しました。");
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
