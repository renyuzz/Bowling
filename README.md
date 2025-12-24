<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>A比B少才好玩 - 手機版</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- SortableJS for drag and drop -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/Sortable/1.15.0/Sortable.min.js"></script>
    <!-- FontAwesome for icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@400;500;700&display=swap');
        
        body {
            font-family: 'Noto Sans TC', sans-serif;
            background-color: #f8fafc;
            overscroll-behavior-y: none; 
            -webkit-tap-highlight-color: transparent;
        }

        /* 手機版標頭高度調整 */
        .game-header-cell {
            display: flex;
            flex-direction: column;
            justify-content: flex-start;
            align-items: center;
            padding: 6px 2px;
            line-height: 1.1;
            min-height: 130px; 
            transition: background-color 0.2s;
        }

        /* 獲獎顏色標記 - 提高對比度 */
        .is-a-winner { background-color: #fbbf24 !important; color: #78350f !important; font-weight: 800; border-color: #f59e0b !important; }
        .is-b-winner { background-color: #3b82f6 !important; color: #ffffff !important; font-weight: 800; border-color: #2563eb !important; }
        .is-both-winner { background-color: #a855f7 !important; color: #ffffff !important; font-weight: 800; border-color: #9333ea !important; }

        /* 隱藏原生數字箭頭 */
        input[type=number]::-webkit-inner-spin-button, 
        input[type=number]::-webkit-outer-spin-button { 
            -webkit-appearance: none; 
            margin: 0; 
        }

        .header-select {
            background: #ffffff;
            border: 1px solid #e2e8f0;
            border-radius: 6px;
            font-size: 11px;
            padding: 0 4px;
            height: 24px;
            margin: 4px 0;
            cursor: pointer;
            width: 95%;
            appearance: none;
            text-align: center;
        }

        .winner-tag {
            font-size: 10px;
            display: flex;
            align-items: center;
            justify-content: center;
            width: 100%;
            overflow: hidden;
            text-overflow: ellipsis;
            white-space: nowrap;
            margin-top: 2px;
            border-radius: 4px;
            min-height: 18px;
            font-weight: 500;
        }

        .score-display {
            font-size: 12px;
            font-weight: 800;
            color: #0f172a;
            margin: 2px 0;
        }
        
        .tag-title {
            font-size: 9px;
            font-weight: bold;
            color: #94a3b8;
            margin-top: 6px;
            border-bottom: 1.5px solid #f1f5f9;
            width: 85%;
            text-align: center;
            padding-bottom: 2px;
        }

        /* 手機輸入框優化 */
        .score-input {
            font-size: 14px !important; /* 防止 iOS 自動放大 */
            height: 36px;
            border-radius: 6px;
            transition: all 0.2s;
        }

        .score-input:focus {
            background-color: #fff;
            border-color: #3b82f6;
            box-shadow: 0 0 0 2px rgba(59, 130, 246, 0.1);
        }

        /* 自定義捲軸 */
        .custom-scroll::-webkit-scrollbar { width: 4px; }
        .custom-scroll::-webkit-scrollbar-track { background: transparent; }
        .custom-scroll::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 10px; }
    </style>
</head>
<body class="text-gray-800 h-screen flex flex-col overflow-hidden">

    <!-- Header -->
    <header class="bg-white shadow-md z-30 flex-shrink-0">
        <div class="px-4 py-2.5 flex justify-between items-center border-b border-gray-100">
            <div class="flex items-center gap-2">
                <div class="bg-yellow-400 p-1.5 rounded-lg">
                    <i class="fa-solid fa-trophy text-white text-sm"></i>
                </div>
                <h1 class="font-black text-gray-800 tracking-tight text-base">A比B少才好玩</h1>
            </div>
            <div class="flex items-center gap-2">
                <div class="text-right">
                    <div class="text-[9px] text-gray-400 font-bold leading-none">算錢用的</div>
                    <div class="text-base font-black text-blue-600">$<span id="total-money">0</span></div>
                </div>
            </div>
        </div>

        <!-- 獎金摘要 (手機版橫向排列) -->
        <div class="bg-slate-50 px-4 py-2 flex justify-around border-b border-gray-200">
             <div class="flex flex-col items-center">
                 <span class="text-[9px] font-bold text-gray-400">A標單局獎金</span>
                 <span class="text-sm font-bold text-amber-600">$<span id="game-a-money">0</span></span>
             </div>
             <div class="h-6 w-px bg-gray-200"></div>
             <div class="flex flex-col items-center">
                 <span class="text-[9px] font-bold text-gray-400">B標單局獎金</span>
                 <span class="text-sm font-bold text-blue-600">$<span id="game-b-money">0</span></span>
             </div>
        </div>
    </header>

    <!-- Main Content -->
    <main class="flex-grow overflow-hidden bg-slate-50 relative w-full flex flex-col">
        
        <!-- 表格固定標頭區 -->
        <div class="bg-white shadow-sm z-20 flex-shrink-0">
            <div id="dynamic-table-header" class="grid grid-cols-[30px_65px_1fr_1fr_1fr_1fr_1fr_30px] md:grid-cols-[40px_100px_1fr_1fr_1fr_1fr_1fr_40px] gap-1 p-1 text-[10px] font-bold text-gray-500 text-center items-stretch pr-2">
                <div class="flex items-center justify-center">#</div>
                <div class="flex items-center"></div>
                <div id="g1-header-placeholder"></div>
                <div id="g2-header-placeholder"></div>
                <div id="g3-header-placeholder"></div>
                <div id="g4-header-placeholder"></div>
                <div id="g5-header-placeholder"></div>
                <div class="flex items-center justify-center text-gray-300"><i class="fa-solid fa-trash-can"></i></div>
            </div>
        </div>

        <!-- 選手列表 (可捲動) -->
        <div class="flex-grow overflow-y-auto custom-scroll px-1 pb-20">
            <div id="player-list" class="divide-y divide-gray-100"></div>
            <div id="empty-state" class="hidden flex flex-col items-center justify-center py-12 text-gray-400">
                <i class="fa-solid fa-user-plus text-3xl mb-2 opacity-20"></i>
                <p class="text-sm">點擊下方按鈕新增選手</p>
            </div>
        </div>

        <!-- 底部懸浮按鈕區 -->
        <div class="absolute bottom-0 left-0 right-0 p-4 bg-gradient-to-t from-slate-50 via-slate-50/90 to-transparent flex justify-between items-center pointer-events-none">
            <button onclick="clearAllPlayers()" class="pointer-events-auto bg-white/90 backdrop-blur border border-red-200 text-red-500 w-12 h-12 rounded-full shadow-lg flex items-center justify-center active:scale-90 transition-transform">
                <i class="fa-solid fa-trash-arrow-up"></i>
            </button>
            <button onclick="addPlayer()" class="pointer-events-auto bg-blue-600 text-white px-6 py-3 rounded-full shadow-xl flex items-center gap-2 font-bold active:scale-95 transition-transform">
                <i class="fa-solid fa-plus"></i>
                <span>新增選手</span>
            </button>
        </div>
    </main>

    <!-- 清除確認視窗 -->
    <div id="confirm-modal" class="fixed inset-0 bg-slate-900/60 hidden items-center justify-center z-50 p-6 backdrop-blur-sm">
        <div class="bg-white rounded-2xl p-6 w-full max-w-xs shadow-2xl text-center">
            <div class="w-16 h-16 bg-red-100 text-red-600 rounded-full flex items-center justify-center mx-auto mb-4">
                <i class="fa-solid fa-exclamation-triangle text-2xl"></i>
            </div>
            <h3 class="font-black text-gray-800 text-lg mb-2">確定清空？</h3>
            <p class="text-gray-500 text-sm mb-6">這將移除所有選手數據與倍率設定。</p>
            <div class="flex gap-3">
                <button onclick="closeModal()" class="flex-1 bg-gray-100 text-gray-600 font-bold py-3 rounded-xl active:bg-gray-200">取消</button>
                <button onclick="confirmClear()" class="flex-1 bg-red-600 text-white font-bold py-3 rounded-xl shadow-lg shadow-red-200 active:bg-red-700">確定</button>
            </div>
        </div>
    </div>

    <script>
        const COST_PER_GAME = 30;
        const PRIZE_A_PER_GAME = 10;
        const PRIZE_B_PER_GAME = 20;
        const MAX_SCORE = 300;
        const GAME_NAMES = ["一", "二", "三", "四", "五"];

        let players = [
            { id: 'p1', name: '選手 A', scores: [210, 190, 150, 200, 230] },
            { id: 'p2', name: '選手 B', scores: [180, 240, 190, 170, 185] },
            { id: 'p3', name: '選手 C', scores: [250, 160, 230, 220, 210] }
        ];

        let gameMultipliers = { 1: 0, 2: 0, 3: 0, 4: 0, 5: 0 };

        document.addEventListener('DOMContentLoaded', () => {
            initHeaderUI();
            renderPlayerList();
            initSortable();
            updateHeaderLabels();
        });

        function initHeaderUI() {
            for (let g = 1; g <= 5; g++) {
                const placeholder = document.getElementById(`g${g}-header-placeholder`);
                if (placeholder) {
                    placeholder.outerHTML = `
                        <div class="game-header-cell border-x border-gray-50">
                            <div class="text-[9px] font-black text-slate-400">第${GAME_NAMES[g-1]}局</div>
                            <select id="select-multiplier-${g}" onchange="updateMultiplier(${g}, this.value)" class="header-select">
                                <option value="0" selected>還沒打完不要急</option>
                                <option value="1">x1.0</option>
                                <option value="0.9">x0.9</option>
                                <option value="0.85">x0.85</option>
                                <option value="0.8">x0.8</option>
                                <option value="0.75">x0.75</option>
                                <option value="0.7">x0.7</option>
                                <option value="0.65">x0.65</option>
                                <option value="0.6">x0.6</option>
                                <option value="0.5">x0.5</option>
                                <option value="0.4">x0.4</option>
                            </select>
                            
                            <div class="tag-title text-amber-500/50">A</div>
                            <div id="h-a-base-${g}" class="score-display">--</div>
                            <div id="h-a-name-${g}" class="winner-tag text-amber-700 bg-amber-100">--</div>
                            
                            <div class="tag-title text-blue-500/50">B</div>
                            <div id="h-b-target-${g}" class="score-display">--</div>
                            <div id="h-b-name-${g}" class="winner-tag text-blue-700 bg-blue-100">--</div>
                        </div>
                    `;
                }
            }
        }

        function initSortable() {
            const list = document.getElementById('player-list');
            if (!list) return;
            new Sortable(list, {
                handle: '.drag-handle',
                animation: 150,
                onEnd: () => {
                    const newPlayers = [];
                    Array.from(list.children).forEach(row => {
                        const p = players.find(p => p.id === row.getAttribute('data-id'));
                        if (p) newPlayers.push(p);
                    });
                    players = newPlayers;
                    updateHeaderLabels();
                }
            });
        }

        function addPlayer() {
            players.push({ id: 'p-'+Date.now(), name: '', scores: [0,0,0,0,0] });
            renderPlayerList();
            updateHeaderLabels();
            // 自動滾動到底部
            const scrollContainer = document.querySelector('.overflow-y-auto');
            scrollContainer.scrollTo({ top: scrollContainer.scrollHeight, behavior: 'smooth' });
        }

        function deletePlayer(index) {
            players.splice(index, 1);
            renderPlayerList();
            updateHeaderLabels();
        }

        function updateScore(pIdx, gIdx, val) {
            let score = Math.min(parseInt(val) || 0, MAX_SCORE);
            players[pIdx].scores[gIdx] = score;
            const input = document.getElementById(`input-${pIdx}-${gIdx}`);
            if (input) input.value = score === 0 ? '' : score;
            updateHeaderLabels();
        }

        function updateMultiplier(gameNum, val) {
            gameMultipliers[gameNum] = parseFloat(val);
            updateHeaderLabels();
        }

        function updateHeaderLabels() {
            const count = players.length;
            document.getElementById('total-money').innerText = count * COST_PER_GAME * 5;
            document.getElementById('game-a-money').innerText = count * PRIZE_A_PER_GAME;
            document.getElementById('game-b-money').innerText = count * PRIZE_B_PER_GAME;
            refreshAllBadges();
        }

        function calculateAllWinners() {
            let winnerMap = {};
            let globalHistory = {};
            players.forEach(p => globalHistory[p.id] = { wonA: false, wonB: false });

            for (let g = 1; g <= 5; g++) {
                const gIdx = g - 1;
                const multiplier = gameMultipliers[g];
                let currentRank = players.map(p => ({ 
                    id: p.id, name: p.name || '?', score: p.scores[gIdx] 
                }))
                .filter(r => r.score > 0)
                .sort((a, b) => b.score - a.score);

                if (currentRank.length === 0) {
                    winnerMap[g] = { aIds: [], bIds: [], baseScore: 0, targetB: 0, aAwardNames: '' };
                    continue;
                }

                let awardA_Ids = [];
                let aProvider = currentRank.find(r => !globalHistory[r.id].wonA);
                if (aProvider) {
                    const targetAScore = aProvider.score;
                    currentRank.filter(r => r.score === targetAScore && !globalHistory[r.id].wonA).forEach(match => {
                        awardA_Ids.push(match.id);
                        globalHistory[match.id].wonA = true;
                    });
                }

                let bBaseProvider = currentRank.find(r => !globalHistory[r.id].wonB);
                let baseScore = bBaseProvider ? bBaseProvider.score : currentRank[0].score;
                let targetB = multiplier > 0 ? Math.floor(baseScore * multiplier) : 0;

                let awardB_Ids = [];
                if (multiplier > 0) {
                    let bCandidates = currentRank.filter(r => {
                        if (globalHistory[r.id].wonB) return false;
                        if (multiplier !== 1.0 && awardA_Ids.includes(r.id)) return false;
                        return true;
                    });

                    if (bCandidates.length > 0) {
                        const diffs = bCandidates.map(c => ({ id: c.id, score: c.score, diff: Math.abs(c.score - targetB) }));
                        const minDiff = Math.min(...diffs.map(d => d.diff));
                        const finalists = diffs.filter(d => d.diff === minDiff);
                        const lowestScore = Math.min(...finalists.map(f => f.score));
                        finalists.filter(f => f.score === lowestScore).forEach(match => {
                            awardB_Ids.push(match.id);
                            globalHistory[match.id].wonB = true;
                        });
                    }
                }
                const aAwardNames = awardA_Ids.map(id => players.find(p => p.id === id)?.name).join(',');
                winnerMap[g] = { aIds: awardA_Ids, bIds: awardB_Ids, baseScore, targetB, aAwardNames };
            }
            return winnerMap;
        }

        function refreshAllBadges() {
            const winners = calculateAllWinners();
            document.querySelectorAll('.score-input').forEach(el => el.classList.remove('is-a-winner', 'is-b-winner', 'is-both-winner'));

            for (let g = 1; g <= 5; g++) {
                const data = winners[g];
                const gIdx = g - 1;
                const mVal = gameMultipliers[g];
                const namesB = data.bIds.map(id => players.find(p => p.id === id)?.name).filter(n => n).join(',');

                if (document.getElementById(`h-a-base-${g}`)) {
                    document.getElementById(`h-a-base-${g}`).innerText = data.baseScore || '--';
                    document.getElementById(`h-b-target-${g}`).innerText = mVal === 0 ? '--' : (data.targetB || '--');
                    document.getElementById(`h-a-name-${g}`).innerText = data.aAwardNames || (players.length > 0 ? '無' : '--');
                    document.getElementById(`h-b-name-${g}`).innerText = mVal === 0 ? '--' : (namesB || (players.length > 0 ? '無' : '--'));
                }

                players.forEach((p, pIdx) => {
                    const input = document.getElementById(`input-${pIdx}-${gIdx}`);
                    if (!input) return;
                    const isA = data.aIds.includes(p.id);
                    const isB = data.bIds.includes(p.id);
                    if (isA && isB) input.classList.add('is-both-winner');
                    else if (isA) input.classList.add('is-a-winner');
                    else if (isB) input.classList.add('is-b-winner');
                });
            }
        }

        function renderPlayerList() {
            const list = document.getElementById('player-list');
            list.innerHTML = '';
            if (players.length === 0) { document.getElementById('empty-state').classList.remove('hidden'); return; }
            document.getElementById('empty-state').classList.add('hidden');

            players.forEach((p, idx) => {
                const row = document.createElement('div');
                row.className = 'grid grid-cols-[30px_65px_1fr_1fr_1fr_1fr_1fr_30px] md:grid-cols-[40px_100px_1fr_1fr_1fr_1fr_1fr_40px] gap-1 p-1.5 items-center bg-white mb-1 rounded-lg border border-gray-100 shadow-sm';
                row.setAttribute('data-id', p.id);
                row.innerHTML = `
                    <div class="drag-handle text-gray-200 py-2 active:text-gray-400"><i class="fa-solid fa-grip-vertical"></i></div>
                    <div class="px-0.5"><input type="text" value="${p.name}" oninput="players[${idx}].name=this.value; updateHeaderLabels()" class="w-full border-none bg-slate-50 rounded px-1.5 py-2 text-xs font-bold" placeholder="誰"></div>
                    ${p.scores.map((s, g) => `<div><input type="number" inputmode="numeric" id="input-${idx}-${g}" value="${s||''}" oninput="updateScore(${idx}, ${g}, this.value)" class="score-input w-full text-center bg-slate-50 border border-slate-100 py-1 text-sm outline-none"></div>`).join('')}
                    <div class="text-center"><button onclick="deletePlayer(${idx})" class="w-8 h-8 flex items-center justify-center text-gray-300 active:text-red-500"><i class="fa-solid fa-circle-xmark"></i></button></div>
                `;
                list.appendChild(row);
            });
            refreshAllBadges();
        }

        function closeModal() { document.getElementById('confirm-modal').classList.add('hidden'); }
        function clearAllPlayers() { document.getElementById('confirm-modal').classList.remove('hidden'); }
        
        function confirmClear() { 
            players = []; 
            for(let g=1; g<=5; g++) {
                gameMultipliers[g] = 0;
                const sel = document.getElementById(`select-multiplier-${g}`);
                if(sel) sel.value = "0";
            }
            renderPlayerList(); 
            updateHeaderLabels(); 
            closeModal(); 
        }
    </script>
</body>
</html>
