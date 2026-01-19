<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>義大醫療體系抗生素使用智能決策系統 V01</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/lucide@latest"></script>
    
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Ma+Shan+Zheng&family=Noto+Sans+TC:wght@300;400;500;700&family=Noto+Serif+TC:wght@400;600;700&display=swap');
        
        :root {
            --color-ink-primary: #1E4C7C;
            --color-ink-secondary: #2A7E68;
            --color-paper: #F4F1EA;
        }
        
        body { 
            font-family: 'Noto Sans TC', sans-serif; 
            background-color: var(--color-paper);
            background-image: url("data:image/svg+xml,%3Csvg width='200' height='200' viewBox='0 0 200 200' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='noise'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.6' numOctaves='3' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='200' height='200' filter='url(%23noise)' opacity='0.08'/%3E%3C/svg%3E");
            color: #2C3E50;
        }

        .serif-font { font-family: 'Noto Serif TC', serif; }
        .ink-card {
            background: rgba(255,255,255,0.85);
            backdrop-filter: blur(12px);
            border: 1px solid rgba(255,255,255,0.4);
            border-radius: 1rem;
            box-shadow: 0 8px 32px 0 rgba(31,38,135,0.07);
        }

        /* 輸入框樣式 */
        input, select {
            background-color: rgba(255,255,255,0.6);
            border: 1px solid #D1D5DB;
        }
        input:focus, select:focus {
            border-color: var(--color-ink-primary);
            outline: none;
            box-shadow: 0 0 0 3px rgba(30, 76, 124, 0.1);
        }
        
        /* 載入動畫 */
        .loader {
            border: 2px solid rgba(30, 76, 124, 0.1);
            border-top: 2px solid var(--color-ink-primary);
            border-radius: 50%;
            width: 16px; height: 16px;
            animation: spin 1s linear infinite;
        }
        @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
    </style>
</head>
<body class="min-h-screen flex flex-col p-4">

    <header class="mb-6 mt-4">
        <div class="max-w-4xl mx-auto flex items-center justify-between">
            <div class="flex items-center gap-4">
                <div class="bg-[#1E4C7C] p-2.5 rounded-lg text-white shadow-lg">
                    <i data-lucide="activity" class="w-6 h-6"></i>
                </div>
                <div>
                    <h1 class="text-xl md:text-2xl font-bold tracking-wide serif-font text-[#1E4C7C]">義大醫療體系抗生素智能決策系統</h1>
                    <p class="text-xs text-gray-500 tracking-widest mt-0.5">E-Da AI-Powered Antibiotic Prescribing System</p>
                </div>
            </div>
            <div id="db-status" class="flex items-center gap-2 text-xs bg-gray-100 px-3 py-1.5 rounded-full border border-gray-200">
                <span class="loader"></span>
                <span class="text-gray-500 font-medium">資料載入中...</span>
            </div>
        </div>
    </header>

    <main class="flex-grow max-w-4xl mx-auto w-full grid grid-cols-1 lg:grid-cols-12 gap-6 items-start">
        
        <div class="lg:col-span-4 space-y-6">
            <div class="ink-card p-6 border-t-4 border-[#1E4C7C]">
                <h2 class="font-bold text-[#2C3E50] flex items-center gap-2 text-lg serif-font mb-4">
                    <i data-lucide="user-circle" class="w-5 h-5 text-[#1E4C7C]"></i> 病人參數
                </h2>
                
                <div class="space-y-4">
                    <div class="grid grid-cols-2 gap-4">
                        <div>
                            <label class="block text-xs font-bold text-gray-500 mb-1">性別</label>
                            <select id="p-sex" class="w-full rounded-lg p-2.5 text-sm" onchange="app.calculateMetrics()">
                                <option value="M">男性 (Male)</option>
                                <option value="F">女性 (Female)</option>
                            </select>
                        </div>
                        <div>
                            <label class="block text-xs font-bold text-gray-500 mb-1">年齡</label>
                            <input type="number" id="p-age" class="w-full rounded-lg p-2.5 text-sm" placeholder="Age" oninput="app.calculateMetrics()">
                        </div>
                    </div>

                    <div class="grid grid-cols-2 gap-4">
                        <div>
                            <label class="block text-xs font-bold text-gray-500 mb-1">身高 (cm)</label>
                            <input type="number" id="p-ht" class="w-full rounded-lg p-2.5 text-sm" placeholder="cm" oninput="app.calculateMetrics()">
                        </div>
                        <div>
                            <label class="block text-xs font-bold text-gray-500 mb-1">體重 (kg)</label>
                            <input type="number" id="p-wt" class="w-full rounded-lg p-2.5 text-sm" placeholder="kg" oninput="app.calculateMetrics()">
                        </div>
                    </div>

                    <div>
                        <label class="block text-xs font-bold text-gray-500 mb-1">血清肌酸酐 (SCr)</label>
                        <div class="relative">
                            <input type="number" id="p-scr" step="0.01" class="w-full rounded-lg p-2.5 text-sm pr-12" placeholder="mg/dL" oninput="app.calculateMetrics()">
                            <span class="absolute right-3 top-2.5 text-xs text-gray-400">mg/dL</span>
                        </div>
                    </div>

                    <div id="metrics-panel" class="hidden bg-[#F0F7FF] rounded-xl border border-[#D0E3F5] p-4 space-y-2 mt-4">
                        <div class="flex justify-between items-end border-b border-[#D0E3F5] pb-2">
                            <span class="text-xs text-[#1E4C7C] font-bold">eGFR (C-G)</span>
                            <span id="disp-ccr" class="text-xl font-bold text-[#1E4C7C]">--</span>
                        </div>
                        <div class="grid grid-cols-2 gap-2 text-xs text-gray-600">
                            <div>BMI: <span id="disp-bmi" class="font-bold text-gray-800">--</span></div>
                            <div>IBW: <span id="disp-ibw" class="font-bold text-gray-800">--</span></div>
                        </div>
                    </div>
                </div>
            </div>

            <div class="ink-card p-6 border-t-4 border-[#B5493D]">
                <h2 class="font-bold text-[#2C3E50] flex items-center gap-2 text-lg serif-font mb-4">
                    <i data-lucide="settings-2" class="w-5 h-5 text-[#B5493D]"></i> 情境設定
                </h2>
                <div class="flex items-center justify-between p-3 bg-white border rounded-lg mb-4">
                    <span class="text-sm font-bold text-gray-600">Loading Dose (起始劑量)</span>
                    <input type="checkbox" id="is-loading-dose" class="w-5 h-5 accent-[#B5493D]">
                </div>
                <div class="grid grid-cols-4 gap-2">
                    <label class="cursor-pointer"><input type="radio" name="dialysis" value="None" checked class="peer sr-only"><div class="p-2 text-center text-xs rounded border peer-checked:bg-[#2C3E50] peer-checked:text-white">無</div></label>
                    <label class="cursor-pointer"><input type="radio" name="dialysis" value="HD" class="peer sr-only"><div class="p-2 text-center text-xs rounded border peer-checked:bg-[#2C3E50] peer-checked:text-white">HD</div></label>
                    <label class="cursor-pointer"><input type="radio" name="dialysis" value="CVVH" class="peer sr-only"><div class="p-2 text-center text-xs rounded border peer-checked:bg-[#2C3E50] peer-checked:text-white">CVVH</div></label>
                    <label class="cursor-pointer"><input type="radio" name="dialysis" value="PD" class="peer sr-only"><div class="p-2 text-center text-xs rounded border peer-checked:bg-[#2C3E50] peer-checked:text-white">PD</div></label>
                </div>
            </div>
        </div>

        <div class="lg:col-span-8 space-y-6">
            <div class="ink-card p-6">
                <label class="block text-sm font-bold text-[#2C3E50] mb-2 serif-font">選擇抗生素 (Antibiotics)</label>
                <div class="relative">
                    <select id="drug-select" class="w-full p-3 rounded-xl border-2 border-gray-200 text-lg font-bold text-[#1E4C7C]" onchange="app.performQuery()">
                        <option value="">資料載入中...</option>
                    </select>
                </div>
                <button onclick="app.performQuery()" class="w-full mt-4 py-3 bg-[#1E4C7C] text-white rounded-xl font-bold shadow-lg hover:bg-[#15385e] transition">
                    查詢建議劑量
                </button>
            </div>

            <div id="results-area" class="space-y-4">
                <div class="flex flex-col items-center justify-center h-48 border-2 border-dashed border-gray-300 rounded-xl bg-white/50 text-gray-400">
                    <i data-lucide="search" class="w-8 h-8 mb-2 opacity-50"></i>
                    <span class="text-sm">請輸入病人資料並選擇藥物</span>
                </div>
            </div>
        </div>

    </main>

    <script id="csv-data" type="text/plain">
DrugName,Category,CCR_Min,CCR_Max,Wt_Min,Wt_Max,Dose,Unit,Freq,Route,Remarks,Is_MgKg,Obese_Ref,NonObese_Ref
Vancomycin 1g/vial,CCR,90,999,0,999,1000,MG,Q12H,IVD,"Monitor Trough Level",N,實際體重,實際體重
Vancomycin 1g/vial,CCR,50,89,0,999,1000,MG,Q24H,IVD,"Monitor Trough Level",N,實際體重,實際體重
Vancomycin 1g/vial,CCR,10,49,0,999,500,MG,Q24H,IVD,"Monitor Trough Level",N,實際體重,實際體重
Vancomycin 1g/vial,HD,0,999,0,999,1000,MG,STAT,IVD,"After HD, monitor trough",N,實際體重,實際體重
Cefepime 2g/vial,CCR,60,999,0,999,2000,MG,Q12H,IVD,,N,實際體重,實際體重
Cefepime 2g/vial,CCR,30,59,0,999,2000,MG,Q24H,IVD,,N,實際體重,實際體重
Cefepime 2g/vial,HD,0,999,0,999,1000,MG,Q24H,IVD,"Give dose after HD",N,實際體重,實際體重
Levofloxacin 500mg/vial,CCR,50,999,0,999,750,MG,Q24H,IVD,,N,實際體重,實際體重
Levofloxacin 500mg/vial,CCR,20,49,0,999,750,MG,Q48H,IVD,,N,實際體重,實際體重
Meropenem 500mg/vial,CCR,50,999,0,999,1000,MG,Q8H,IVD,,N,實際體重,實際體重
Meropenem 500mg/vial,CCR,26,49,0,999,1000,MG,Q12H,IVD,,N,實際體重,實際體重
Meropenem 500mg/vial,CCR,10,25,0,999,500,MG,Q12H,IVD,,N,實際體重,實際體重
Teicoplanin 200mg/vial,Loading dose,0,999,0,999,6,MG,Q12H,IVF,"Initial 3 doses",Y,校正體重,校正體重
Teicoplanin 200mg/vial,CCR,50,999,0,999,6,MG,QD,IVF,"Maintenance",Y,校正體重,校正體重
Colistin 2MIU/vial,CCR,80,999,0,999,150,MG,Q12H,IVD,"Base ~ 2.5-5 mg/kg CBA",N,校正體重,校正體重
    </script>

    <script>
        const app = (function() {
            // 原作者的 Google API (如果他關掉了，系統會自動切換用上面的 CSV)
            const API_URL = "https://script.google.com/macros/s/AKfycbyIlj9JfsGuXMH6ogAONxX_kV_t66OhJMeqcmgwwVbLFi-OF-vvevQGOLLsF-1SKn9RiQ/exec";
            
            let allDrugs = [];
            let currentMetrics = null;
            const els = {};

            function init() {
                // 綁定所有 ID
                document.querySelectorAll('[id]').forEach(el => els[el.id] = el);
                lucide.createIcons();
                fetchData();
            }

            async function fetchData() {
                const statusBadge = els['db-status'];
                try {
                    // 嘗試連線原作者資料庫
                    const response = await fetch(API_URL);
                    if (!response.ok) throw new Error("Network");
                    const jsonData = await response.json();
                    allDrugs = normalizeData(jsonData);
                    statusBadge.innerHTML = `<i data-lucide="cloud" class="w-3 h-3"></i> 雲端同步中`;
                    statusBadge.className = "flex items-center gap-2 text-xs bg-[#E3F2FD] text-[#1E4C7C] px-3 py-1.5 rounded-full border border-[#BBDEFB]";
                } catch (error) {
                    // 失敗則使用內建資料 (存檔模式)
                    console.log("切換至離線模式");
                    const csvContent = document.getElementById('csv-data').textContent;
                    allDrugs = parseCSV(csvContent);
                    statusBadge.innerHTML = `<i data-lucide="database" class="w-3 h-3"></i> 離線資料庫`;
                    statusBadge.className = "flex items-center gap-2 text-xs bg-[#FFF3E0] text-[#E65100] px-3 py-1.5 rounded-full border border-[#FFE0B2]";
                }
                updateDropdown();
            }

            function normalizeData(rawData) {
                return rawData.map(row => ({
                    ...row,
                    CCR_Min: parseFloat(row.CCR_Min) || 0,
                    CCR_Max: parseFloat(row.CCR_Max) || 999,
                    Wt_Min: parseFloat(row.Wt_Min) || 0,
                    Wt_Max: parseFloat(row.Wt_Max) || 999,
                    Dose: parseFloat(row.Dose) || 0
                }));
            }

            function parseCSV(csvText) {
                const lines = csvText.trim().split('\n');
                const headers = lines[0].split(',').map(h => h.trim());
                const data = [];
                for (let i = 1; i < lines.length; i++) {
                    // 簡單的 CSV 解析，處理引號
                    const rowMatch = lines[i].match(/(".*?"|[^",\s]+)(?=\s*,|\s*$)/g);
                    if (!rowMatch) continue;
                    const rowValues = rowMatch.map(val => val.replace(/^"|"$/g, '').trim());
                    if (rowValues.length < headers.length) continue;
                    const obj = {};
                    headers.forEach((h, index) => obj[h] = rowValues[index]);
                    data.push(obj);
                }
                return normalizeData(data);
            }

            function updateDropdown() {
                const select = els['drug-select'];
                select.innerHTML = '<option value="">-- 請選擇藥品 --</option>';
                const uniqueNames = Array.from(new Set(allDrugs.map(d => d.DrugName))).sort();
                uniqueNames.forEach(name => {
                    const opt = document.createElement('option');
                    opt.value = name; opt.textContent = name;
                    select.appendChild(opt);
                });
            }

            function calculateMetrics() {
                const sex = els['p-sex'].value;
                const age = parseFloat(els['p-age'].value);
                const ht = parseFloat(els['p-ht'].value);
                const wt = parseFloat(els['p-wt'].value);
                const scr = parseFloat(els['p-scr'].value);
                const panel = els['metrics-panel'];

                if (!age || !ht || !wt || !scr) {
                    panel.classList.add('hidden');
                    currentMetrics = null;
                    return;
                }
                panel.classList.remove('hidden');

                const htM = ht / 100;
                const bmi = wt / (htM * htM);
                
                // IBW 計算
                let htInch = ht / 2.54;
                if (htInch < 60) htInch = 60;
                let ibw = (sex === 'M') ? 50 + 2.3 * (htInch - 60) : 45.5 + 2.3 * (htInch - 60);
                
                // AdjBW 計算
                const adjbw = ibw + 0.4 * (wt - ibw);
                
                // eGFR (Cockcroft-Gault)
                let ccrWeight = wt; // 預設實際體重
                let label = "ActBW";
                
                if (bmi >= 18.5 && bmi < 25) { ccrWeight = ibw; label = "IBW"; }
                else if (bmi >= 25) { ccrWeight = adjbw; label = "AdjBW"; }

                let ccr = ((140 - age) * ccrWeight) / (72 * scr);
                if (sex === 'F') ccr *= 0.85;

                els['disp-ccr'].textContent = ccr.toFixed(1) + " ml/min";
                els['disp-bmi'].textContent = bmi.toFixed(1);
                els['disp-ibw'].textContent = ibw.toFixed(1);

                currentMetrics = { bmi, abw: wt, ibw, adjbw, ccr, isObese: bmi >= 30 };
            }

            function performQuery() {
                if (!currentMetrics || !els['drug-select'].value) return;
                
                const drugName = els['drug-select'].value;
                const isLoading = els['is-loading-dose'].checked;
                const dialysis = document.querySelector('input[name="dialysis"]:checked').value;
                
                const candidates = allDrugs.filter(d => d.DrugName === drugName);
                let filtered = [];
                let logic = "";

                // 1. Loading Dose
                if (isLoading) {
                    filtered = candidates.filter(d => d.Category.toLowerCase().includes("loading"));
                    logic = "Loading Dose";
                }
                
                // 2. 透析
                if (filtered.length === 0 && dialysis !== "None") {
                    filtered = candidates.filter(d => d.Category === dialysis);
                    logic = `透析 (${dialysis})`;
                }
                
                // 3. 一般腎功能
                if (filtered.length === 0) {
                    filtered = candidates.filter(d => d.Category === "CCR" && currentMetrics.ccr >= d.CCR_Min && currentMetrics.ccr < d.CCR_Max);
                    logic = `eGFR ${currentMetrics.ccr.toFixed(0)}`;
                }

                renderResults(filtered, logic);
            }

            function renderResults(rows, logic) {
                const container = els['results-area'];
                container.innerHTML = "";
                
                if (rows.length === 0) {
                    container.innerHTML = `<div class="p-4 bg-red-50 text-red-600 rounded-lg border border-red-200">查無建議劑量 (Logic: ${logic})</div>`;
                    return;
                }

                rows.forEach(row => {
                    // 判斷體重標準
                    let refWt = currentMetrics.abw;
                    if (row.NonObese_Ref.includes("理想")) refWt = currentMetrics.ibw;
                    if (row.NonObese_Ref.includes("校正")) refWt = currentMetrics.adjbw;
                    if (currentMetrics.isObese && row.Obese_Ref.includes("校正")) refWt = currentMetrics.adjbw;

                    let displayDose = `${row.Dose} ${row.Unit}`;
                    if (row.Is_MgKg === 'Y') {
                        const totalDose = row.Dose * refWt;
                        displayDose = `${totalDose.toFixed(0)} ${row.Unit} <span class="text-sm text-gray-400">(${row.Dose} mg/kg)</span>`;
                    }

                    const card = document.createElement('div');
                    card.className = "ink-card p-6 border-l-4 border-[#2A7E68] mb-4";
                    card.innerHTML = `
                        <div class="flex justify-between items-start mb-4">
                            <h3 class="font-bold text-xl text-[#2C3E50] serif-font">${row.DrugName}</h3>
                            <span class="bg-[#E8F5E9] text-[#2E7D32] text-xs px-2 py-1 rounded border border-[#C8E6C9] font-bold">${row.Category}</span>
                        </div>
                        <div class="grid grid-cols-2 gap-4 mb-3">
                            <div class="bg-gray-50 p-3 rounded border">
                                <span class="block text-xs text-gray-400 font-bold uppercase">Dose</span>
                                <span class="text-xl font-bold text-[#1E4C7C]">${displayDose}</span>
                            </div>
                            <div class="bg-gray-50 p-3 rounded border">
                                <span class="block text-xs text-gray-400 font-bold uppercase">Frequency</span>
                                <span class="text-xl font-bold text-[#2C3E50]">${row.Freq}</span>
                            </div>
                        </div>
                        ${row.Remarks ? `<div class="text-sm text-[#B5493D] bg-[#FFF8E1] p-2 rounded border border-[#FFECB3]"><i data-lucide="info" class="w-3 h-3 inline mr-1"></i>${row.Remarks}</div>` : ''}
                    `;
                    container.appendChild(card);
                    lucide.createIcons();
                });
            }

            return { init, calculateMetrics, performQuery };
        })();

        window.addEventListener('DOMContentLoaded', app.init);
    </script>
</body>
</html>
