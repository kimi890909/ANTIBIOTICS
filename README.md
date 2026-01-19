<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>義大醫療體系抗生素使用智能決策系統 (離線版)</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/lucide@latest"></script>
    
    <style>
        /* 基礎字體設定 */
        @import url('https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@400;500;700&family=Noto+Serif+TC:wght@700&display=swap');
        
        body { 
            font-family: 'Noto Sans TC', sans-serif; 
            background-color: #F3F4F6; /* 淺灰色背景，比圖片背景更穩定 */
            color: #1F2937;
        }
        
        /* 模擬卡片效果 */
        .ink-card {
            background-color: white;
            border-radius: 1rem; /* 圓角 */
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1); /* 陰影 */
            border: 1px solid #E5E7EB;
        }

        /* 輸入框美化 */
        input, select {
            border: 1px solid #D1D5DB;
            background-color: #F9FAFB;
        }
        input:focus, select:focus {
            outline: none;
            border-color: #1E4C7C;
            box-shadow: 0 0 0 3px rgba(30, 76, 124, 0.1);
        }
        
        .serif-font { font-family: 'Noto Serif TC', serif; }
    </style>
</head>
<body class="min-h-screen flex flex-col p-4">

    <header class="mb-6">
        <div class="max-w-5xl mx-auto bg-white p-4 rounded-xl shadow-sm border border-gray-200 flex items-center justify-between">
            <div class="flex items-center gap-4">
                <div class="bg-[#1E4C7C] p-3 rounded-lg text-white">
                    <i data-lucide="activity" class="w-6 h-6"></i>
                </div>
                <div>
                    <h1 class="text-xl font-bold text-[#1E4C7C] serif-font">義大醫療體系抗生素智能決策系統</h1>
                    <p class="text-xs text-gray-500">E-Da AI-Powered Antibiotic Prescribing System</p>
                </div>
            </div>
            <div class="hidden md:flex items-center gap-2 text-xs bg-orange-50 text-orange-600 px-3 py-1 rounded-full border border-orange-200">
                <i data-lucide="database" class="w-3 h-3"></i> 離線存檔版
            </div>
        </div>
    </header>

    <main class="flex-grow w-full max-w-5xl mx-auto grid grid-cols-1 lg:grid-cols-12 gap-6 items-start">
        
        <div class="lg:col-span-4 space-y-6">
            
            <div class="ink-card p-6 border-t-4 border-[#1E4C7C]">
                <h2 class="font-bold text-lg mb-4 flex items-center gap-2 text-[#1E4C7C]">
                    <i data-lucide="user" class="w-5 h-5"></i> 病人參數
                </h2>
                
                <div class="space-y-4">
                    <div class="grid grid-cols-2 gap-3">
                        <div>
                            <label class="block text-xs font-bold text-gray-500 mb-1">性別</label>
                            <select id="p-sex" class="w-full rounded p-2" onchange="calculate()">
                                <option value="M">男性</option>
                                <option value="F">女性</option>
                            </select>
                        </div>
                        <div>
                            <label class="block text-xs font-bold text-gray-500 mb-1">年齡</label>
                            <input type="number" id="p-age" class="w-full rounded p-2" placeholder="歲" oninput="calculate()">
                        </div>
                    </div>

                    <div class="grid grid-cols-2 gap-3">
                        <div>
                            <label class="block text-xs font-bold text-gray-500 mb-1">身高 (cm)</label>
                            <input type="number" id="p-ht" class="w-full rounded p-2" placeholder="cm" oninput="calculate()">
                        </div>
                        <div>
                            <label class="block text-xs font-bold text-gray-500 mb-1">體重 (kg)</label>
                            <input type="number" id="p-wt" class="w-full rounded p-2" placeholder="kg" oninput="calculate()">
                        </div>
                    </div>

                    <div>
                        <label class="block text-xs font-bold text-gray-500 mb-1">血清肌酸酐 (SCr)</label>
                        <input type="number" id="p-scr" step="0.01" class="w-full rounded p-2" placeholder="mg/dL" oninput="calculate()">
                    </div>

                    <div id="result-panel" class="hidden bg-blue-50 rounded-lg p-4 border border-blue-100 space-y-2">
                        <div class="flex justify-between items-end border-b border-blue-200 pb-2">
                            <span class="text-xs text-blue-800 font-bold">eGFR (C-G)</span>
                            <span id="disp-ccr" class="text-xl font-bold text-blue-900">--</span>
                        </div>
                        <div class="grid grid-cols-2 gap-2 text-xs text-gray-600">
                            <div>BMI: <span id="disp-bmi" class="font-bold text-gray-800">--</span></div>
                            <div>IBW: <span id="disp-ibw" class="font-bold text-gray-800">--</span></div>
                        </div>
                    </div>
                </div>
            </div>

            <div class="ink-card p-6 border-t-4 border-[#B5493D]">
                <h2 class="font-bold text-lg mb-4 flex items-center gap-2 text-[#B5493D]">
                    <i data-lucide="settings" class="w-5 h-5"></i> 特殊情境
                </h2>
                
                <div class="flex items-center justify-between p-3 bg-gray-50 border rounded-lg mb-4">
                    <span class="text-sm font-bold text-gray-700">Loading Dose (起始劑量)</span>
                    <input type="checkbox" id="is-loading" class="w-5 h-5 accent-[#B5493D]">
                </div>

                <div class="grid grid-cols-4 gap-2">
                    <label class="cursor-pointer">
                        <input type="radio" name="dialysis" value="None" checked class="peer sr-only">
                        <div class="p-2 text-center text-xs rounded border hover:bg-gray-100 peer-checked:bg-gray-800 peer-checked:text-white transition">無</div>
                    </label>
                    <label class="cursor-pointer">
                        <input type="radio" name="dialysis" value="HD" class="peer sr-only">
                        <div class="p-2 text-center text-xs rounded border hover:bg-gray-100 peer-checked:bg-gray-800 peer-checked:text-white transition">HD</div>
                    </label>
                    <label class="cursor-pointer">
                        <input type="radio" name="dialysis" value="CVVH" class="peer sr-only">
                        <div class="p-2 text-center text-xs rounded border hover:bg-gray-100 peer-checked:bg-gray-800 peer-checked:text-white transition">CVVH</div>
                    </label>
                    <label class="cursor-pointer">
                        <input type="radio" name="dialysis" value="PD" class="peer sr-only">
                        <div class="p-2 text-center text-xs rounded border hover:bg-gray-100 peer-checked:bg-gray-800 peer-checked:text-white transition">PD</div>
                    </label>
                </div>
            </div>
        </div>

        <div class="lg:col-span-8 space-y-6">
            <div class="ink-card p-6">
                <label class="block text-sm font-bold text-[#1E4C7C] mb-2">選擇抗生素</label>
                <div class="flex gap-2">
                    <select id="drug-select" class="w-full p-3 rounded-lg border-2 border-blue-100 text-lg font-bold text-[#1E4C7C]">
                        <option value="">-- 請先載入資料 --</option>
                    </select>
                </div>
                <button onclick="queryDrug()" class="mt-4 w-full bg-[#1E4C7C] hover:bg-[#15385e] text-white font-bold py-3 rounded-lg shadow-md transition">
                    查詢建議劑量
                </button>
            </div>

            <div id="results-area" class="space-y-4">
                <div class="flex flex-col items-center justify-center h-40 border-2 border-dashed border-gray-300 rounded-lg text-gray-400">
                    <i data-lucide="search" class="w-8 h-8 mb-2 opacity-50"></i>
                    <span>請輸入病人參數並選擇藥物</span>
                </div>
            </div>
        </div>

    </main>

    <script>
        // 初始化圖示
        lucide.createIcons();

        // 1. 內建資料庫 (直接寫在程式碼內，保證讀取得到)
        const DB = [
            { DrugName: "Vancomycin 1g/vial", Category: "CCR", CCR_Min: 90, CCR_Max: 999, Dose: 1000, Unit: "MG", Freq: "Q12H", Remarks: "Monitor Trough Level", Is_MgKg: "N", NonObese_Ref: "實際體重", Obese_Ref: "實際體重" },
            { DrugName: "Vancomycin 1g/vial", Category: "CCR", CCR_Min: 50, CCR_Max: 89, Dose: 1000, Unit: "MG", Freq: "Q24H", Remarks: "Monitor Trough Level", Is_MgKg: "N", NonObese_Ref: "實際體重", Obese_Ref: "實際體重" },
            { DrugName: "Vancomycin 1g/vial", Category: "CCR", CCR_Min: 10, CCR_Max: 49, Dose: 500, Unit: "MG", Freq: "Q24H", Remarks: "Monitor Trough Level", Is_MgKg: "N", NonObese_Ref: "實際體重", Obese_Ref: "實際體重" },
            { DrugName: "Vancomycin 1g/vial", Category: "HD", CCR_Min: 0, CCR_Max: 999, Dose: 1000, Unit: "MG", Freq: "STAT", Remarks: "After HD, monitor trough", Is_MgKg: "N", NonObese_Ref: "實際體重", Obese_Ref: "實際體重" },
            { DrugName: "Cefepime 2g/vial", Category: "CCR", CCR_Min: 60, CCR_Max: 999, Dose: 2000, Unit: "MG", Freq: "Q12H", Remarks: "", Is_MgKg: "N", NonObese_Ref: "實際體重", Obese_Ref: "實際體重" },
            { DrugName: "Cefepime 2g/vial", Category: "CCR", CCR_Min: 30, CCR_Max: 59, Dose: 2000, Unit: "MG", Freq: "Q24H", Remarks: "", Is_MgKg: "N", NonObese_Ref: "實際體重", Obese_Ref: "實際體重" },
            { DrugName: "Cefepime 2g/vial", Category: "HD", CCR_Min: 0, CCR_Max: 999, Dose: 1000, Unit: "MG", Freq: "Q24H", Remarks: "Give dose after HD", Is_MgKg: "N", NonObese_Ref: "實際體重", Obese_Ref: "實際體重" },
            { DrugName: "Levofloxacin 500mg/vial", Category: "CCR", CCR_Min: 50, CCR_Max: 999, Dose: 750, Unit: "MG", Freq: "Q24H", Remarks: "", Is_MgKg: "N", NonObese_Ref: "實際體重", Obese_Ref: "實際體重" },
            { DrugName: "Levofloxacin 500mg/vial", Category: "CCR", CCR_Min: 20, CCR_Max: 49, Dose: 750, Unit: "MG", Freq: "Q48H", Remarks: "", Is_MgKg: "N", NonObese_Ref: "實際體重", Obese_Ref: "實際體重" },
            { DrugName: "Meropenem 500mg/vial", Category: "CCR", CCR_Min: 50, CCR_Max: 999, Dose: 1000, Unit: "MG", Freq: "Q8H", Remarks: "", Is_MgKg: "N", NonObese_Ref: "實際體重", Obese_Ref: "實際體重" },
            { DrugName: "Meropenem 500mg/v
