<!DOCTYPE html>
<html lang="ja" class="h-full">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>HIRO WORKS 在庫管理システム</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=Noto+Sans+JP:wght@400;500;700&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        body { font-family: 'Noto Sans JP', 'Inter', sans-serif; }
        /* 携帯での文字の縦折り返しを防ぎ、必ず横書きで綺麗に表示するためのスタイル */
        .nowrap-cell {
            white-space: nowrap;
            word-break: normal;
        }
        @media print {
            body * { visibility: hidden; }
            #print-area, #print-area * { visibility: visible; }
            #print-area { position: absolute; left: 0; top: 0; width: 100%; }
            .no-print { display: none !important; }
        }
    </style>
</head>
<body class="bg-emerald-50/40 text-slate-800 h-full flex flex-col">

    <header class="bg-emerald-600 text-white shadow-md no-print shrink-0">
        <div class="max-w-7xl mx-auto px-3 sm:px-4 py-2.5 flex flex-col sm:flex-row justify-between items-center gap-2.5">
            <div class="flex items-center space-x-2.5 w-full sm:w-auto justify-between sm:justify-start">
                <div class="bg-emerald-500 p-2 rounded-xl text-white shadow-lg shadow-emerald-600/30">
                    <i class="fa-solid fa-boxes-stacked text-lg"></i>
                </div>
                <div>
                    <h1 class="text-xs sm:text-sm font-bold tracking-tight">HIRO WORKS 在庫管理</h1>
                    <p class="text-[9px] text-emerald-100">建築シーリング・副資材専門管理</p>
                </div>
            </div>
            <!-- Optimized mobile tab navigation -->
            <nav class="flex space-x-1 bg-emerald-700/60 p-1 rounded-xl text-[10px] sm:text-xs font-medium w-full sm:w-auto justify-center overflow-x-auto">
                <button onclick="switchTab('inventory')" id="tab-inventory" class="px-2.5 py-1.5 rounded-lg transition-all duration-200 bg-emerald-500 text-white shadow whitespace-nowrap nowrap-cell text-xs">
                    <i class="fa-solid fa-warehouse mr-1"></i>在庫一覧
                </button>
                <button onclick="switchTab('movement')" id="tab-movement" class="px-2.5 py-1.5 rounded-lg transition-all duration-200 text-emerald-100 hover:text-white hover:bg-emerald-500 whitespace-nowrap nowrap-cell text-xs">
                    <i class="fa-solid fa-right-left mr-1"></i>入出庫記録
                </button>
                <button onclick="switchTab('categories')" id="tab-categories" class="px-2.5 py-1.5 rounded-lg transition-all duration-200 text-emerald-100 hover:text-white hover:bg-emerald-500 whitespace-nowrap nowrap-cell text-xs">
                    <i class="fa-solid fa-layer-group mr-1"></i>分類・資材登録
                </button>
                <button onclick="switchTab('report')" id="tab-report" class="px-2.5 py-1.5 rounded-lg transition-all duration-200 text-emerald-100 hover:text-white hover:bg-emerald-500 whitespace-nowrap nowrap-cell text-xs">
                    <i class="fa-solid fa-print mr-1"></i>月末レポート
                </button>
            </nav>
        </div>
    </header>

    <main class="flex-1 max-w-7xl w-full mx-auto p-3 sm:p-6 overflow-y-auto">

        <!-- Notification Toast -->
        <div id="toast" class="fixed bottom-5 right-5 z-50 transform translate-y-20 opacity-0 transition-all duration-300 bg-emerald-800 text-white px-5 py-3 rounded-xl shadow-xl flex items-center space-x-3 no-print">
            <i id="toast-icon" class="fa-solid fa-circle-check text-emerald-300"></i>
            <span id="toast-message" class="text-xs sm:text-sm font-medium">操作が完了しました</span>
        </div>

        <!-- ==================== TAB 1: INVENTORY ==================== -->
        <section id="view-inventory" class="space-y-4 sm:space-y-6">
            <!-- Stats cards -->
            <div class="grid grid-cols-1 sm:grid-cols-2 gap-4 max-w-xl">
                <div class="bg-white p-3.5 sm:p-4 rounded-2xl shadow-sm border border-emerald-100 flex items-center justify-between">
                    <div>
                        <p class="text-[11px] font-semibold text-slate-500 uppercase tracking-wider">要発注（在庫少 ≤ 5）</p>
                        <h3 id="stat-low-stock" class="text-xl sm:text-2xl font-bold text-amber-600 mt-0.5">0</h3>
                    </div>
                    <div class="bg-amber-50 text-amber-600 p-2.5 rounded-xl">
                        <i class="fa-solid fa-triangle-exclamation text-lg"></i>
                    </div>
                </div>
                <div class="bg-white p-3.5 sm:p-4 rounded-2xl shadow-sm border border-emerald-100 flex items-center justify-between">
                    <div>
                        <p class="text-[11px] font-semibold text-slate-500 uppercase tracking-wider">登録資材数</p>
                        <h3 id="stat-total-items" class="text-xl sm:text-2xl font-bold text-emerald-700 mt-0.5">0</h3>
                    </div>
                    <div class="bg-emerald-50 text-emerald-600 p-2.5 rounded-xl">
                        <i class="fa-solid fa-boxes-stacked text-lg"></i>
                    </div>
                </div>
            </div>

            <!-- Filter & Controls Toolbar -->
            <div class="bg-white p-4 rounded-2xl shadow-sm border border-emerald-100 flex flex-wrap gap-3 items-center justify-between">
                <div class="flex flex-wrap items-center gap-2 sm:gap-3 flex-1">
                    <div class="relative min-w-[180px] flex-1">
                        <span class="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none text-slate-400">
                            <i class="fa-solid fa-search text-xs"></i>
                        </span>
                        <input type="text" id="search-input" oninput="renderInventory()" placeholder="資材名で検索..." class="w-full pl-9 pr-3 py-2 bg-emerald-50/40 border border-emerald-100 rounded-xl text-xs sm:text-sm focus:outline-none focus:ring-2 focus:ring-emerald-500">
                    </div>
                    <select id="filter-large" onchange="renderInventory()" class="bg-emerald-50/40 border border-emerald-100 text-[11px] sm:text-xs rounded-xl px-2.5 py-2 focus:outline-none focus:ring-2 focus:ring-emerald-500 nowrap-cell">
                        <option value="">大分類 (すべて)</option>
                    </select>
                    <select id="filter-medium" onchange="renderInventory()" class="bg-emerald-50/40 border border-emerald-100 text-[11px] sm:text-xs rounded-xl px-2.5 py-2 focus:outline-none focus:ring-2 focus:ring-emerald-500 nowrap-cell">
                        <option value="">中分類 (すべて)</option>
                    </select>
                </div>
                <div class="flex items-center gap-2 w-full sm:w-auto">
                    <button onclick="openModal('modal-quick-movement')" class="flex-1 sm:flex-none bg-emerald-600 hover:bg-emerald-700 text-white font-medium px-3.5 py-2 rounded-xl text-xs transition shadow-sm shadow-emerald-600/20 flex items-center justify-center gap-1.5 nowrap-cell">
                        <i class="fa-solid fa-right-left"></i> 入出庫処理
                    </button>
                    <button onclick="openModal('modal-add-item')" class="flex-1 sm:flex-none bg-emerald-700 hover:bg-emerald-800 text-white font-medium px-3.5 py-2 rounded-xl text-xs transition shadow-sm flex items-center justify-center gap-1.5 nowrap-cell">
                        <i class="fa-solid fa-plus"></i> 新規登録
                    </button>
                    <button onclick="resetAllData()" title="全データを初期状態にリセット" class="bg-rose-50 hover:bg-rose-100 text-rose-600 border border-rose-200 px-3 py-2 rounded-xl text-xs transition flex items-center gap-1 nowrap-cell">
                        <i class="fa-solid fa-rotate-right"></i> リセット
                    </button>
                </div>
            </div>

            <!-- Inventory Table Column Order: 資材名 -> 現在庫数 -> ステータス -> 操作 -> 大分類 -> 中分類（成分） -> 小分類（形態） -->
            <!-- 大分類から小分類はさらに文字を小さくして横書き表示 -->
            <div class="bg-white rounded-2xl shadow-sm border border-emerald-100 overflow-hidden">
                <div class="overflow-x-auto">
                    <table class="w-full text-left border-collapse min-w-[850px]">
                        <thead>
                            <tr class="bg-emerald-50/80 border-b border-emerald-100 text-[11px] font-semibold text-slate-700 uppercase tracking-wider">
                                <th class="p-3 nowrap-cell">資材名 / メーカー</th>
                                <th class="p-3 text-right nowrap-cell">現在庫数</th>
                                <th class="p-3 text-center nowrap-cell">ステータス</th>
                                <th class="p-3 text-center nowrap-cell">操作</th>
                                <th class="p-3 nowrap-cell text-[9px]">大分類</th>
                                <th class="p-3 nowrap-cell text-[9px]">中分類（成分）</th>
                                <th class="p-3 nowrap-cell text-[9px]">小分類（形態）</th>
                            </tr>
                        </thead>
                        <tbody id="inventory-table-body" class="divide-y divide-emerald-50 text-xs text-slate-700">
                            <!-- Populated by JS -->
                        </tbody>
                    </table>
                </div>
            </div>
        </section>

        <!-- ==================== TAB 2: MOVEMENT ==================== -->
        <section id="view-movement" class="space-y-6 hidden">
            <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                <!-- Entry/Exit Form Card -->
                <div class="bg-white p-5 sm:p-6 rounded-2xl shadow-sm border border-emerald-100 lg:col-span-1 space-y-4">
                    <h2 class="text-sm sm:text-base font-bold text-slate-800 flex items-center gap-2">
                        <i class="fa-solid fa-pen-to-square text-emerald-600"></i> 入出庫の記録
                    </h2>
                    <form id="movement-form" onsubmit="handleMovementSubmit(event)" class="space-y-3.5">
                        <div>
                            <label class="block text-[11px] font-semibold text-slate-600 mb-1">処理区分</label>
                            <div class="grid grid-cols-2 gap-2">
                                <label class="border border-emerald-100 rounded-xl p-2.5 flex items-center justify-center space-x-2 cursor-pointer hover:bg-emerald-50/30 transition has-[:checked]:bg-emerald-50 has-[:checked]:border-emerald-500 has-[:checked]:text-emerald-700">
                                    <input type="radio" name="m-type" value="入庫" checked class="text-emerald-600 focus:ring-emerald-500">
                                    <span class="font-medium text-xs nowrap-cell">入庫（入荷）</span>
                                </label>
                                <label class="border border-emerald-100 rounded-xl p-2.5 flex items-center justify-center space-x-2 cursor-pointer hover:bg-rose-50/30 transition has-[:checked]:bg-rose-50 has-[:checked]:border-rose-500 has-[:checked]:text-rose-700">
                                    <input type="radio" name="m-type" value="出庫" class="text-rose-600 focus:ring-rose-500">
                                    <span class="font-medium text-xs nowrap-cell">出庫（使用）</span>
                                </label>
                            </div>
                        </div>

                        <div>
                            <label class="block text-[11px] font-semibold text-slate-600 mb-1">対象資材</label>
                            <select id="m-item-id" required class="w-full bg-emerald-50/40 border border-emerald-100 rounded-xl p-2.5 text-xs focus:ring-2 focus:ring-emerald-500 focus:outline-none nowrap-cell">
                                <!-- Populated by JS -->
                            </select>
                        </div>

                        <div>
                            <label class="block text-[11px] font-semibold text-slate-600 mb-1">数量</label>
                            <input type="number" id="m-quantity" min="1" value="1" required class="w-full bg-emerald-50/40 border border-emerald-100 rounded-xl p-2.5 text-xs focus:ring-2 focus:ring-emerald-500 focus:outline-none">
                        </div>

                        <div>
                            <label class="block text-[11px] font-semibold text-slate-600 mb-1">使用現場名 <span class="text-rose-500 text-[10px] font-normal">(出庫時のみ必須)</span></label>
                            <input type="text" id="m-site-name" placeholder="例：〇〇ビル改修工事現場" class="w-full bg-emerald-50/40 border border-emerald-100 rounded-xl p-2.5 text-xs focus:ring-2 focus:ring-emerald-500 focus:outline-none">
                        </div>

                        <div>
                            <label class="block text-[11px] font-semibold text-slate-600 mb-1">担当者・備考</label>
                            <input type="text" id="m-memo" placeholder="例：山田、追加発注分など" class="w-full bg-emerald-50/40 border border-emerald-100 rounded-xl p-2.5 text-xs focus:ring-2 focus:ring-emerald-500 focus:outline-none">
                        </div>

                        <button type="submit" class="w-full bg-emerald-600 hover:bg-emerald-700 text-white font-medium py-2.5 rounded-xl shadow-md transition text-xs">
                            記録を登録する
                        </button>
                    </form>
                </div>

                <!-- Movement History Log Table -->
                <div class="bg-white p-5 sm:p-6 rounded-2xl shadow-sm border border-emerald-100 lg:col-span-2 space-y-4">
                    <div class="flex justify-between items-center">
                        <h2 class="text-sm sm:text-base font-bold text-slate-800 flex items-center gap-2">
                            <i class="fa-solid fa-clock-rotate-left text-emerald-600"></i> 最近の入出庫履歴
                        </h2>
                        <span class="text-[11px] text-slate-500">直近の記録一覧</span>
                    </div>
                    <div class="overflow-x-auto">
                        <table class="w-full text-left border-collapse min-w-[550px]">
                            <thead>
                                <tr class="bg-emerald-50/80 border-b border-emerald-100 text-[11px] font-semibold text-slate-700 uppercase tracking-wider">
                                    <th class="p-3 nowrap-cell">日時</th>
                                    <th class="p-3 nowrap-cell">区分</th>
                                    <th class="p-3 nowrap-cell">資材名</th>
                                    <th class="p-3 text-right nowrap-cell">数量</th>
                                    <th class="p-3 nowrap-cell">現場名 / 備考</th>
                                    <th class="p-3 text-center nowrap-cell">取消</th>
                                </tr>
                            </thead>
                            <tbody id="movement-table-body" class="divide-y divide-emerald-50 text-xs text-slate-700">
                                <!-- Populated by JS -->
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>
        </section>

        <!-- ==================== TAB 3: CATEGORIES ==================== -->
        <section id="view-categories" class="space-y-6 hidden">
            <div class="bg-white p-4 rounded-2xl shadow-sm border border-emerald-100 flex justify-between items-center">
                <div>
                    <h3 class="font-bold text-slate-800 text-sm">分類ごとの追加・管理</h3>
                    <p class="text-[11px] text-slate-500">大分類・中分類・小分類の選択肢を自由に追加・削除できます。</p>
                </div>
                <button onclick="resetAllData()" class="bg-rose-50 hover:bg-rose-100 text-rose-600 border border-rose-200 px-3 py-1.5 rounded-xl text-xs transition flex items-center gap-1 nowrap-cell">
                    <i class="fa-solid fa-rotate-right"></i> データリセット
                </button>
            </div>
            <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
                <!-- Large Category Management -->
                <div class="bg-white p-5 rounded-2xl shadow-sm border border-emerald-100 space-y-4">
                    <h3 class="font-bold text-slate-800 flex items-center gap-2 text-xs sm:text-sm">
                        <i class="fa-solid fa-folder text-emerald-600"></i> 大分類の管理
                    </h3>
                    <div class="flex gap-2">
                        <input type="text" id="new-large-input" placeholder="新しい大分類名" class="flex-1 bg-emerald-50/40 border border-emerald-100 rounded-xl px-3 py-2 text-xs focus:outline-none focus:ring-2 focus:ring-emerald-500">
                        <button onclick="addCategory('大分類')" class="bg-emerald-600 hover:bg-emerald-700 text-white px-3 py-2 rounded-xl text-xs font-medium transition nowrap-cell">追加</button>
                    </div>
                    <ul id="list-large-categories" class="divide-y divide-emerald-50 text-xs">
                        <!-- Populated by JS -->
                    </ul>
                </div>

                <!-- Medium Category Management -->
                <div class="bg-white p-5 rounded-2xl shadow-sm border border-emerald-100 space-y-4">
                    <h3 class="font-bold text-slate-800 flex items-center gap-2 text-xs sm:text-sm">
                        <i class="fa-solid fa-folder-open text-emerald-600"></i> 中分類（成分）の管理
                    </h3>
                    <div class="flex gap-2">
                        <input type="text" id="new-medium-input" placeholder="新しい中分類名" class="flex-1 bg-emerald-50/40 border border-emerald-100 rounded-xl px-3 py-2 text-xs focus:outline-none focus:ring-2 focus:ring-emerald-500">
                        <button onclick="addCategory('中分類（成分）')" class="bg-emerald-600 hover:bg-emerald-700 text-white px-3 py-2 rounded-xl text-xs font-medium transition nowrap-cell">追加</button>
                    </div>
                    <ul id="list-medium-categories" class="divide-y divide-emerald-50 text-xs">
                        <!-- Populated by JS -->
                    </ul>
                </div>

                <!-- Small Category Management -->
                <div class="bg-white p-5 rounded-2xl shadow-sm border border-emerald-100 space-y-4">
                    <h3 class="font-bold text-slate-800 flex items-center gap-2 text-xs sm:text-sm">
                        <i class="fa-solid fa-file text-emerald-600"></i> 小分類（形態）の管理
                    </h3>
                    <div class="flex gap-2">
                        <input type="text" id="new-small-input" placeholder="新しい小分類名" class="flex-1 bg-emerald-50/40 border border-emerald-100 rounded-xl px-3 py-2 text-xs focus:outline-none focus:ring-2 focus:ring-emerald-500">
                        <button onclick="addCategory('小分類（形態）')" class="bg-emerald-600 hover:bg-emerald-700 text-white px-3 py-2 rounded-xl text-xs font-medium transition nowrap-cell">追加</button>
                    </div>
                    <ul id="list-small-categories" class="divide-y divide-emerald-50 text-xs">
                        <!-- Populated by JS -->
                    </ul>
                </div>
            </div>
        </section>

        <!-- ==================== TAB 4: REPORT / PRINT ==================== -->
        <section id="view-report" class="space-y-6 hidden">
            <div class="bg-white p-5 sm:p-6 rounded-2xl shadow-sm border border-emerald-100 flex flex-wrap justify-between items-center gap-4 no-print">
                <div>
                    <h2 class="text-sm sm:text-base font-bold text-slate-800 flex items-center gap-2">
                        <i class="fa-solid fa-file-invoice text-emerald-600"></i> 月末レポート（当月出庫実績）
                    </h2>
                    <p class="text-[11px] text-slate-500 mt-0.5">当月出庫されたアイテムの一覧を集計して綺麗に印刷できます。</p>
                </div>
                <button onclick="window.print()" class="bg-emerald-600 hover:bg-emerald-700 text-white font-medium px-4 py-2 rounded-xl text-xs shadow-md transition flex items-center gap-2 nowrap-cell">
                    <i class="fa-solid fa-print"></i> 印刷またはPDF保存
                </button>
            </div>

            <!-- Printable Area -->
            <div id="print-area" class="bg-white p-6 sm:p-8 rounded-2xl shadow-sm border border-emerald-100 space-y-6">
                <div class="flex justify-between items-end border-b border-slate-200 pb-4">
                    <div>
                        <h1 class="text-lg sm:text-xl font-bold text-slate-900">月次出庫実績報告書</h1>
                        <p id="report-date-label" class="text-xs text-slate-500 mt-1">集計月: 2026年10月度</p>
                    </div>
                    <div class="text-right text-[11px] text-slate-500">
                        <p>発行日: <span id="report-print-date"></span></p>
                        <p>HIRO WORKS</p>
                    </div>
                </div>

                <!-- Report Summary Table (Outbound items only) -->
                <div class="space-y-2">
                    <h3 class="text-xs font-bold text-slate-700 uppercase tracking-wider">当月出庫アイテム一覧</h3>
                    <div class="overflow-x-auto">
                        <table class="w-full text-left border-collapse text-xs">
                            <thead>
                                <tr class="bg-slate-100 border-b border-slate-200 font-bold text-slate-700 text-[11px]">
                                    <th class="p-2 border border-slate-200 nowrap-cell">出庫日時</th>
                                    <th class="p-2 border border-slate-200 nowrap-cell">資材名・メーカー</th>
                                    <th class="p-2 border border-slate-200 text-right nowrap-cell">出庫数量</th>
                                    <th class="p-2 border border-slate-200 nowrap-cell">使用現場名</th>
                                    <th class="p-2 border border-slate-200 nowrap-cell">担当・備考</th>
                                </tr>
                            </thead>
                            <tbody id="report-table-body" class="divide-y divide-slate-200">
                                <!-- Populated by JS -->
                            </tbody>
                        </table>
                    </div>
                </div>

                <!-- Signature block for print -->
                <div class="pt-8 grid grid-cols-3 gap-6 text-center text-xs text-slate-600">
                    <div class="border-t border-slate-400 pt-2">管理者印</div>
                    <div class="border-t border-slate-400 pt-2">倉庫確認印</div>
                    <div class="border-t border-slate-400 pt-2">総務確認印</div>
                </div>
            </div>
        </section>

    </main>

    <!-- ==================== MODALS ==================== -->
    <!-- Modal: Add Item -->
    <div id="modal-add-item" class="fixed inset-0 bg-slate-900/50 backdrop-blur-sm z-50 flex items-center justify-center p-4 hidden">
        <div class="bg-white rounded-2xl max-w-md w-full p-6 shadow-2xl space-y-4">
            <div class="flex justify-between items-center">
                <h3 class="text-sm sm:text-base font-bold text-slate-800">新規資材登録</h3>
                <button onclick="closeModal('modal-add-item')" class="text-slate-400 hover:text-slate-600"><i class="fa-solid fa-xmark text-base"></i></button>
            </div>
            <form id="add-item-form" onsubmit="handleAddItemSubmit(event)" class="space-y-3">
                <div>
                    <label class="block text-[11px] font-semibold text-slate-600 mb-1">資材名・メーカー名</label>
                    <input type="text" id="ai-name" required placeholder="例：セメダイン 8000 など" class="w-full bg-emerald-50/40 border border-emerald-100 rounded-xl p-2.5 text-xs focus:ring-2 focus:ring-emerald-500 focus:outline-none">
                </div>
                <div>
                    <label class="block text-[11px] font-semibold text-slate-600 mb-1">大分類</label>
                    <select id="ai-large" required class="w-full bg-emerald-50/40 border border-emerald-100 rounded-xl p-2.5 text-xs focus:ring-2 focus:ring-emerald-500 focus:outline-none nowrap-cell"></select>
                </div>
                <div>
                    <label class="block text-[11px] font-semibold text-slate-600 mb-1">中分類（成分）</label>
                    <select id="ai-medium" required class="w-full bg-emerald-50/40 border border-emerald-100 rounded-xl p-2.5 text-xs focus:ring-2 focus:ring-emerald-500 focus:outline-none nowrap-cell"></select>
                </div>
                <div>
                    <label class="block text-[11px] font-semibold text-slate-600 mb-1">小分類（形態）</label>
                    <select id="ai-small" required class="w-full bg-emerald-50/40 border border-emerald-100 rounded-xl p-2.5 text-xs focus:ring-2 focus:ring-emerald-500 focus:outline-none nowrap-cell"></select>
                </div>
                <div>
                    <label class="block text-[11px] font-semibold text-slate-600 mb-1">初期在庫数量</label>
                    <input type="number" id="ai-stock" min="0" value="10" required class="w-full bg-emerald-50/40 border border-emerald-100 rounded-xl p-2.5 text-xs focus:ring-2 focus:ring-emerald-500 focus:outline-none">
                </div>
                <div class="flex justify-end space-x-2 pt-2">
                    <button type="button" onclick="closeModal('modal-add-item')" class="px-3.5 py-2 rounded-xl text-xs text-slate-600 hover:bg-slate-100 nowrap-cell">キャンセル</button>
                    <button type="submit" class="bg-emerald-600 hover:bg-emerald-700 text-white font-medium px-4 py-2 rounded-xl text-xs shadow nowrap-cell">登録する</button>
                </div>
            </form>
        </div>
    </div>

    <!-- Modal: Quick Movement -->
    <div id="modal-quick-movement" class="fixed inset-0 bg-slate-900/50 backdrop-blur-sm z-50 flex items-center justify-center p-4 hidden">
        <div class="bg-white rounded-2xl max-w-md w-full p-6 shadow-2xl space-y-4">
            <div class="flex justify-between items-center">
                <h3 class="text-sm sm:text-base font-bold text-slate-800">クイック入出庫処理</h3>
                <button onclick="closeModal('modal-quick-movement')" class="text-slate-400 hover:text-slate-600"><i class="fa-solid fa-xmark text-base"></i></button>
            </div>
            <p class="text-[11px] text-slate-500">一覧画面からスピーディーに入出庫を行います。</p>
            <form onsubmit="handleQuickMovementSubmit(event)" class="space-y-3">
                <div>
                    <label class="block text-[11px] font-semibold text-slate-600 mb-1">処理</label>
                    <div class="grid grid-cols-2 gap-2">
                        <label class="border border-emerald-100 rounded-xl p-2.5 flex items-center justify-center space-x-2 cursor-pointer hover:bg-emerald-50/30 has-[:checked]:bg-emerald-50 has-[:checked]:border-emerald-500">
                            <input type="radio" name="qm-type" value="入庫" checked class="text-emerald-600">
                            <span class="font-medium text-xs nowrap-cell">入庫</span>
                        </label>
                        <label class="border border-emerald-100 rounded-xl p-2.5 flex items-center justify-center space-x-2 cursor-pointer hover:bg-rose-50/30 has-[:checked]:bg-rose-50 has-[:checked]:border-rose-500">
                            <input type="radio" name="qm-type" value="出庫" class="text-rose-600">
                            <span class="font-medium text-xs nowrap-cell">出庫</span>
                        </label>
                    </div>
                </div>
                <div>
                    <label class="block text-[11px] font-semibold text-slate-600 mb-1">資材選択</label>
                    <select id="qm-item-id" required class="w-full bg-emerald-50/40 border border-emerald-100 rounded-xl p-2.5 text-xs focus:ring-2 focus:ring-emerald-500 focus:outline-none nowrap-cell"></select>
                </div>
                <div>
                    <label class="block text-[11px] font-semibold text-slate-600 mb-1">数量</label>
                    <input type="number" id="qm-quantity" min="1" value="1" required class="w-full bg-emerald-50/40 border border-emerald-100 rounded-xl p-2.5 text-xs focus:ring-2 focus:ring-emerald-500 focus:outline-none">
                </div>
                <div>
                    <label class="block text-[11px] font-semibold text-slate-600 mb-1">現場名 (出庫時)</label>
                    <input type="text" id="qm-site" placeholder="使用現場名" class="w-full bg-emerald-50/40 border border-emerald-100 rounded-xl p-2.5 text-xs focus:ring-2 focus:ring-emerald-500 focus:outline-none">
                </div>
                <div class="flex justify-end space-x-2 pt-2">
                    <button type="button" onclick="closeModal('modal-quick-movement')" class="px-3.5 py-2 rounded-xl text-xs text-slate-600 hover:bg-slate-100 nowrap-cell">キャンセル</button>
                    <button type="submit" class="bg-emerald-600 hover:bg-emerald-700 text-white font-medium px-4 py-2 rounded-xl text-xs shadow nowrap-cell">実行する</button>
                </div>
            </form>
        </div>
    </div>

    <script>
        // Initial Default Data
        const defaultCategories = {
            "大分類": ["シーリング材", "プライマー", "副資材（ノズル・マスキング等）"],
            "中分類（成分）": ["シリコーン", "変成シリコーン", "ウレタン", "アクリル", "ポリサルファイド"],
            "小分類（形態）": ["1成分形（カートリッジ）", "1成分形（パウチ）", "2成分形（ペール缶）"]
        };

        const defaultItems = [
            { id: 1, name: "セメダイン 8000", large: "シーリング材", medium: "変成シリコーン", small: "1成分形（カートリッジ）", stock: 45 },
            { id: 2, name: "ボンド MPX-1", large: "シーリング材", medium: "変成シリコーン", small: "1成分形（パウチ）", stock: 12 },
            { id: 3, name: "ウレタンシーリング 2液型", large: "シーリング材", medium: "ウレタン", small: "2成分形（ペール缶）", stock: 4 },
            { id: 4, name: "プライマー #40", large: "プライマー", medium: "ウレタン", small: "1成分形（カートリッジ）", stock: 20 },
            { id: 5, name: "先端ノズル（白・細）", large: "副資材（ノズル・マスキング等）", medium: "シリコーン", small: "1成分形（カートリッジ）", stock: 150 },
            { id: 6, name: "マスキングテープ 210", large: "副資材（ノズル・マスキング等）", medium: "アクリル", small: "1成分形（カートリッジ）", stock: 80 }
        ];

        const defaultMovements = [
            { id: 101, date: "2026-10-01 09:30", type: "入庫", itemId: 1, itemName: "セメダイン 8000", quantity: 50, site: "-", memo: "定期入荷" },
            { id: 102, date: "2026-10-03 14:15", type: "出庫", itemId: 1, itemName: "セメダイン 8000", quantity: 5, site: "〇〇ビル改修工事", memo: "山田" },
            { id: 103, date: "2026-10-05 10:00", type: "出庫", itemId: 3, itemName: "ウレタンシーリング 2液型", quantity: 2, site: "△△マンション外壁", memo: "鈴木" }
        ];

        // State Management
        let categories = JSON.parse(localStorage.getItem('inv_categories')) || defaultCategories;
        let items = JSON.parse(localStorage.getItem('inv_items')) || defaultItems;
        let movements = JSON.parse(localStorage.getItem('inv_movements')) || defaultMovements;

        function saveData() {
            localStorage.setItem('inv_categories', JSON.stringify(categories));
            localStorage.setItem('inv_items', JSON.stringify(items));
            localStorage.setItem('inv_movements', JSON.stringify(movements));
        }

        // Reset All Data to Default
        function resetAllData() {
            if(!confirm('すべてのデータ（在庫・履歴・分類）を初期状態にリセットしますか？')) return;
            categories = JSON.parse(JSON.stringify(defaultCategories));
            items = JSON.parse(JSON.stringify(defaultItems));
            movements = JSON.parse(JSON.stringify(defaultMovements));
            saveData();
            renderInventory();
            showToast('データを初期状態にリセットしました');
        }

        // Tab Navigation Switcher
        function switchTab(tabId) {
            ['inventory', 'movement', 'categories', 'report'].forEach(t => {
                document.getElementById(`view-${t}`).classList.add('hidden');
                document.getElementById(`tab-${t}`).className = "px-2.5 py-1.5 rounded-lg transition-all duration-200 text-emerald-100 hover:text-white hover:bg-emerald-500 whitespace-nowrap nowrap-cell text-xs";
            });
            document.getElementById(`view-${tabId}`).classList.remove('hidden');
            document.getElementById(`tab-${tabId}`).className = "px-2.5 py-1.5 rounded-lg transition-all duration-200 bg-emerald-500 text-white shadow whitespace-nowrap nowrap-cell text-xs";
            
            if(tabId === 'inventory') renderInventory();
            if(tabId === 'movement') renderMovementView();
            if(tabId === 'categories') renderCategoriesView();
            if(tabId === 'report') renderReportView();
        }

        // Toast Notifications
        function showToast(message, isError = false) {
            const toast = document.getElementById('toast');
            const msg = document.getElementById('toast-message');
            const icon = document.getElementById('toast-icon');
            msg.textContent = message;
            icon.className = isError ? "fa-solid fa-circle-exclamation text-rose-300" : "fa-solid fa-circle-check text-emerald-300";
            toast.classList.remove('translate-y-20', 'opacity-0');
            setTimeout(() => {
                toast.classList.add('translate-y-20', 'opacity-0');
            }, 3000);
        }

        // Modal Controls
        function openModal(modalId) {
            document.getElementById(modalId).classList.remove('hidden');
            if(modalId === 'modal-add-item') populateCategorySelects('ai');
            if(modalId === 'modal-quick-movement') populateItemSelects('qm-item-id');
        }
        function closeModal(modalId) {
            document.getElementById(modalId).classList.add('hidden');
        }

        function renderInventory() {
            const search = document.getElementById('search-input').value.toLowerCase();
            const fLarge = document.getElementById('filter-large').value;
            const fMedium = document.getElementById('filter-medium').value;

            // Populate filter dropdowns
            populateDropdownOptions('filter-large', categories['大分類'], fLarge, '大分類 (すべて)');
            populateDropdownOptions('filter-medium', categories['中分類（成分）'], fMedium, '中分類 (すべて)');

            const tbody = document.getElementById('inventory-table-body');
            tbody.innerHTML = '';

            let lowStockCount = 0;

            const filtered = items.filter(item => {
                const matchSearch = item.name.toLowerCase().includes(search);
                const matchLarge = !fLarge || item.large === fLarge;
                const matchMedium = !fMedium || item.medium === fMedium;
                return matchSearch && matchLarge && matchMedium;
            });

            filtered.forEach(item => {
                const isLow = item.stock <= 5;
                if(isLow) lowStockCount++;

                const tr = document.createElement('tr');
                tr.className = "hover:bg-emerald-50/20 transition border-b border-emerald-50";
                // 列順: 資材名 -> 現在庫数 -> ステータス -> 操作 -> 大分類 -> 中分類（成分） -> 小分類（形態）
                // カテゴリ部分は「text-[9px]」でさらに小さく横書き表示
                tr.innerHTML = `
                    <td class="p-3 font-semibold text-slate-900 nowrap-cell text-xs">${item.name}</td>
                    <td class="p-3 text-right font-bold text-slate-900 nowrap-cell text-xs">${item.stock}</td>
                    <td class="p-3 text-center nowrap-cell">
                        ${isLow ? '<span class="bg-amber-100 text-amber-800 px-2 py-0.5 rounded text-[10px] font-semibold inline-block nowrap-cell">要発注</span>' : '<span class="bg-emerald-100 text-emerald-800 px-2 py-0.5 rounded text-[10px] font-semibold inline-block nowrap-cell">適正</span>'}
                    </td>
                    <td class="p-3 text-center space-x-1 nowrap-cell">
                        <button onclick="quickAdjustStock(${item.id}, 1)" title="+1 入庫" class="w-7 h-7 rounded-lg bg-emerald-50 text-emerald-600 hover:bg-emerald-100 transition inline-flex items-center justify-center"><i class="fa-solid fa-plus text-[11px]"></i></button>
                        <button onclick="quickAdjustStock(${item.id}, -1)" title="-1 出庫" class="w-7 h-7 rounded-lg bg-rose-50 text-rose-600 hover:bg-rose-100 transition inline-flex items-center justify-center"><i class="fa-solid fa-minus text-[11px]"></i></button>
                        <button onclick="deleteItem(${item.id})" title="削除" class="text-slate-400 hover:text-rose-600 transition p-1"><i class="fa-solid fa-trash text-[11px]"></i></button>
                    </td>
                    <td class="p-3 nowrap-cell"><span class="bg-emerald-50 text-emerald-700 px-1.5 py-0.5 rounded text-[9px] font-medium inline-block nowrap-cell">${item.large}</span></td>
                    <td class="p-3 text-slate-500 text-[9px] nowrap-cell">${item.medium}</td>
                    <td class="p-3 text-slate-500 text-[9px] nowrap-cell">${item.small}</td>
                `;
                tbody.appendChild(tr);
            });

            // Update stats
            document.getElementById('stat-low-stock').textContent = lowStockCount;
            document.getElementById('stat-total-items').textContent = items.length;
        }

        function populateDropdownOptions(elementId, optionsArray, selectedVal, defaultText) {
            const el = document.getElementById(elementId);
            if(!el) return;
            el.innerHTML = `<option value="">${defaultText}</option>`;
            optionsArray.forEach(opt => {
                const selected = opt === selectedVal ? 'selected' : '';
                el.innerHTML += `<option value="${opt}" ${selected}>${opt}</option>`;
            });
        }

        function renderMovementView() {
            populateItemSelects('m-item-id');
            const tbody = document.getElementById('movement-table-body');
            tbody.innerHTML = '';

            movements.slice().reverse().forEach(m => {
                const badgeColor = m.type === '入庫' ? 'bg-emerald-100 text-emerald-800' : 'bg-rose-100 text-rose-800';
                const tr = document.createElement('tr');
                tr.className = "hover:bg-emerald-50/20 transition border-b border-emerald-50";
                tr.innerHTML = `
                    <td class="p-3 text-[10px] text-slate-500 nowrap-cell">${m.date}</td>
                    <td class="p-3 nowrap-cell"><span class="${badgeColor} px-2 py-0.5 rounded text-[10px] font-semibold inline-block nowrap-cell">${m.type}</span></td>
                    <td class="p-3 font-medium text-slate-800 text-xs nowrap-cell">${m.itemName}</td>
                    <td class="p-3 text-right font-bold text-xs nowrap-cell">${m.quantity}</td>
                    <td class="p-3 text-slate-600 text-xs nowrap-cell">${m.site !== '-' ? `<span class="bg-slate-100 text-slate-700 px-2 py-0.5 rounded font-medium mr-1 nowrap-cell inline-block"><i class="fa-solid fa-location-dot"></i> ${m.site}</span>` : ''} ${m.memo || ''}</td>
                    <td class="p-3 text-center nowrap-cell"><button onclick="deleteMovement(${m.id})" class="text-slate-400 hover:text-rose-600 p-1"><i class="fa-solid fa-trash-can text-xs"></i></button></td>
                `;
                tbody.appendChild(tr);
            });
        }

        function populateItemSelects(elementId) {
            const select = document.getElementById(elementId);
            if(!select) return;
            select.innerHTML = '';
            items.forEach(i => {
                select.innerHTML += `<option value="${i.id}">${i.name} (現在庫: ${i.stock})</option>`;
            });
        }

        function populateCategorySelects(prefix) {
            populateDropdownOptions(`${prefix}-large`, categories['大分類'], '', '選択してください');
            populateDropdownOptions(`${prefix}-medium`, categories['中分類（成分）'], '', '選択してください');
            populateDropdownOptions(`${prefix}-small`, categories['小分類（形態）'], '', '選択してください');
        }

        function handleMovementSubmit(e) {
            e.preventDefault();
            const type = document.querySelector('input[name="m-type"]:checked').value;
            const itemId = parseInt(document.getElementById('m-item-id').value);
            const quantity = parseInt(document.getElementById('m-quantity').value);
            const site = document.getElementById('m-site-name').value.trim() || '-';
            const memo = document.getElementById('m-memo').value.trim();

            const item = items.find(i => i.id === itemId);
            if(!item) return;

            if(type === '出庫' && item.stock < quantity) {
                showToast(`在庫が不足しています (現在庫: ${item.stock})`, true);
                return;
            }

            if(type === '入庫') item.stock += quantity;
            else item.stock -= quantity;

            const now = new Date();
            const dateStr = `${now.getFullYear()}-${String(now.getMonth()+1).padStart(2,'0')}-${String(now.getDate()).padStart(2,'0')} ${String(now.getHours()).padStart(2,'0')}:${String(now.getMinutes()).padStart(2,'0')}`;

            movements.push({
                id: Date.now(),
                date: dateStr,
                type: type,
                itemId: item.id,
                itemName: item.name,
                quantity: quantity,
                site: site,
                memo: memo
            });

            saveData();
            renderMovementView();
            showToast(`${type}処理を完了しました (${item.name} x ${quantity})`);
            document.getElementById('movement-form').reset();
        }

        function handleQuickMovementSubmit(e) {
            e.preventDefault();
            const type = document.querySelector('input[name="qm-type"]:checked').value;
            const itemId = parseInt(document.getElementById('qm-item-id').value);
            const quantity = parseInt(document.getElementById('qm-quantity').value);
            const site = document.getElementById('qm-site').value.trim() || '-';

            const item = items.find(i => i.id === itemId);
            if(!item) return;

            if(type === '出庫' && item.stock < quantity) {
                showToast(`在庫が不足しています (現在庫: ${item.stock})`, true);
                return;
            }

            if(type === '入庫') item.stock += quantity;
            else item.stock -= quantity;

            const now = new Date();
            const dateStr = `${now.getFullYear()}-${String(now.getMonth()+1).padStart(2,'0')}-${String(now.getDate()).padStart(2,'0')} ${String(now.getHours()).padStart(2,'0')}:${String(now.getMinutes()).padStart(2,'0')}`;

            movements.push({
                id: Date.now(),
                date: dateStr,
                type: type,
                itemId: item.id,
                itemName: item.name,
                quantity: quantity,
                site: site,
                memo: 'クイック操作'
            });

            saveData();
            closeModal('modal-quick-movement');
            renderInventory();
            showToast(`クイック${type}処理が完了しました`);
        }

        function quickAdjustStock(itemId, delta) {
            const item = items.find(i => i.id === itemId);
            if(!item) return;
            if(delta < 0 && item.stock + delta < 0) {
                showToast('在庫が0未満になるため出庫できません', true);
                return;
            }

            item.stock += delta;
            const type = delta > 0 ? '入庫' : '出庫';
            const now = new Date();
            const dateStr = `${now.getFullYear()}-${String(now.getMonth()+1).padStart(2,'0')}-${String(now.getDate()).padStart(2,'0')} ${String(now.getHours()).padStart(2,'0')}:${String(now.getMinutes()).padStart(2,'0')}`;

            movements.push({
                id: Date.now(),
                date: dateStr,
                type: type,
                itemId: item.id,
                itemName: item.name,
                quantity: Math.abs(delta),
                site: type === '出庫' ? '一般・保守' : '-',
                memo: 'ワンクリック調整'
            });

            saveData();
            renderInventory();
            showToast(`${item.name} を ${type}しました`);
        }

        function handleAddItemSubmit(e) {
            e.preventDefault();
            const newItem = {
                id: Date.now(),
                name: document.getElementById('ai-name').value.trim(),
                large: document.getElementById('ai-large').value,
                medium: document.getElementById('ai-medium').value,
                small: document.getElementById('ai-small').value,
                stock: parseInt(document.getElementById('ai-stock').value)
            };

            items.push(newItem);
            saveData();
            closeModal('modal-add-item');
            renderInventory();
            showToast('新しい資材を登録しました');
            document.getElementById('add-item-form').reset();
        }

        function deleteItem(itemId) {
            if(!confirm('この資材データを削除してもよろしいですか？')) return;
            items = items.filter(i => i.id !== itemId);
            saveData();
            renderInventory();
            showToast('資材を削除しました');
        }

        function deleteMovement(mId) {
            movements = movements.filter(m => m.id !== mId);
            saveData();
            renderMovementView();
            showToast('記録を削除しました');
        }

        function renderCategoriesView() {
            renderCategoryList('大分類', 'list-large-categories', 'new-large-input');
            renderCategoryList('中分類（成分）', 'list-medium-categories', 'new-medium-input');
            renderCategoryList('小分類（形態）', 'list-small-categories', 'new-small-input');
        }

        function renderCategoryList(catKey, elementId, inputId) {
            const ul = document.getElementById(elementId);
            if(!ul) return;
            ul.innerHTML = '';
            categories[catKey].forEach((cat, index) => {
                const li = document.createElement('li');
                li.className = "py-2.5 flex justify-between items-center text-slate-700";
                li.innerHTML = `
                    <span class="font-medium nowrap-cell text-xs">${cat}</span>
                    <button onclick="deleteCategory('${catKey}', ${index})" class="text-slate-400 hover:text-rose-600 transition p-1"><i class="fa-solid fa-trash-can text-xs"></i></button>
                `;
                ul.appendChild(li);
            });
        }

        function addCategory(catKey) {
            const inputId = catKey === '大分類' ? 'new-large-input' : catKey === '中分類（成分）' ? 'new-medium-input' : 'new-small-input';
            const val = document.getElementById(inputId).value.trim();
            if(!val) return;
            if(categories[catKey].includes(val)) {
                showToast('すでに同じ分類が存在します', true);
                return;
            }
            categories[catKey].push(val);
            saveData();
            document.getElementById(inputId).value = '';
            renderCategoriesView();
            showToast('分類を追加しました');
        }

        function deleteCategory(catKey, index) {
            if(categories[catKey].length <= 1) {
                showToast('最低1つの分類が必要です', true);
                return;
            }
            if(!confirm('この分類を削除しますか？')) return;
            categories[catKey].splice(index, 1);
            saveData();
            renderCategoriesView();
            showToast('分類を削除しました');
        }

        function renderReportView() {
            const now = new Date();
            const year = now.getFullYear();
            const month = String(now.getMonth() + 1).padStart(2, '0');
            document.getElementById('report-date-label').textContent = `集計月: ${year}年 ${month}月度`;
            document.getElementById('report-print-date').textContent = `${year}/${month}/${String(now.getDate()).padStart(2, '0')}`;

            const tbody = document.getElementById('report-table-body');
            tbody.innerHTML = '';

            // 出庫（type === '出庫'）された履歴のみを抽出
            const outboundMovements = movements.filter(m => m.type === '出庫');

            if (outboundMovements.length === 0) {
                tbody.innerHTML = `<tr><td colspan="5" class="p-4 text-center text-slate-400">当月の出庫実績はありません</td></tr>`;
                return;
            }

            outboundMovements.slice().reverse().forEach(m => {
                const tr = document.createElement('tr');
                tr.innerHTML = `
                    <td class="p-2 border border-slate-200 text-[10px] text-slate-600 nowrap-cell">${m.date}</td>
                    <td class="p-2 border border-slate-200 font-semibold nowrap-cell">${m.itemName}</td>
                    <td class="p-2 border border-slate-200 text-right font-bold nowrap-cell">${m.quantity}</td>
                    <td class="p-2 border border-slate-200 nowrap-cell"><span class="bg-slate-100 text-slate-700 px-2 py-0.5 rounded font-medium">${m.site !== '-' ? m.site : '一般・保守'}</span></td>
                    <td class="p-2 border border-slate-200 text-slate-600 text-xs nowrap-cell">${m.memo || ''}</td>
                `;
                tbody.appendChild(tr);
            });
        }

        // Initialize on load
        window.onload = function() {
            renderInventory();
        };
    </script>
</body>
</html>