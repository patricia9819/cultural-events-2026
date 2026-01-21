<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>115年文化局活動管考系統 (V11雲端版)</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/alpinejs@3.x.x/dist/cdn.min.js" defer></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <style>
        body { font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif; background-color: #f8fafc; }
        .calendar-grid { display: grid; grid-template-columns: repeat(7, minmax(0, 1fr)); gap: 1px; background-color: #cbd5e1; border: 1px solid #cbd5e1; }
        .calendar-cell { background-color: white; min-height: 100px; position: relative; cursor: pointer; transition: all 0.1s; overflow: hidden; }
        .calendar-cell:hover { background-color: #f8fafc; }
        .calendar-cell.active { background-color: #eff6ff; box-shadow: inset 0 0 0 2px #3b82f6; }
        .event-bar { font-size: 0.75rem; padding: 1px 4px; margin-bottom: 2px; border-radius: 3px; white-space: normal; word-break: break-word; line-height: 1.25; font-weight: 600; border-left-width: 3px; }
        .sidebar-scroll::-webkit-scrollbar { width: 6px; }
        .sidebar-scroll::-webkit-scrollbar-track { background: #f1f5f9; }
        .sidebar-scroll::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 3px; }
        [x-cloak] { display: none !important; }
        .loading-overlay { position: fixed; inset: 0; background: rgba(255,255,255,0.7); display: flex; justify-content: center; align-items: center; z-index: 100; backdrop-filter: blur(2px); }
        @media (max-width: 640px) { .calendar-cell { min-height: 70px; } .event-bar { font-size: 10px; padding: 1px 2px; line-height: 1.1; } }
    </style>
</head>
<body x-data="calendarApp()" x-init="init()" class="text-gray-800 pb-24">

    <div x-show="isLoading" class="loading-overlay" x-transition>
        <div class="bg-white p-4 rounded-xl shadow-2xl flex flex-col items-center">
            <i class="fas fa-spinner fa-spin text-4xl text-blue-600 mb-2"></i>
            <span class="font-bold text-gray-700">資料雲端同步中...</span>
        </div>
    </div>

    <header class="bg-white shadow-sm sticky top-0 z-40 border-b border-gray-200">
        <div class="max-w-[1600px] mx-auto px-4 py-3 flex justify-between items-center">
            <h1 class="text-lg font-bold text-gray-800 flex items-center">
                <i class="fas fa-cloud text-blue-600 mr-2"></i>
                <span class="hidden sm:inline">文化局活動管考 (V11雲端版)</span>
                <span class="sm:hidden">管考系統</span>
            </h1>
            <div class="flex items-center gap-2">
                <span class="text-xs text-green-600 font-bold bg-green-50 px-2 py-1 rounded border border-green-200 flex items-center">
                    <i class="fas fa-wifi mr-1"></i> 連線中
                </span>
                <button @click="resetToToday()" class="bg-blue-50 text-blue-600 px-3 py-1 rounded-full text-sm font-bold hover:bg-blue-100 transition border border-blue-200">今天</button>
            </div>
        </div>
        
        <div class="max-w-[1600px] mx-auto px-4 py-2 flex justify-center items-center bg-gray-50 border-b">
            <button @click="changeMonth(-1)" class="p-2 hover:bg-gray-200 rounded-md text-gray-600"><i class="fas fa-chevron-left"></i></button>
            <span class="px-4 font-bold text-lg min-w-[120px] text-center" x-text="formatMonthYear()"></span>
            <button @click="changeMonth(1)" class="p-2 hover:bg-gray-200 rounded-md text-gray-600"><i class="fas fa-chevron-right"></i></button>
        </div>

        <div class="max-w-[1600px] mx-auto px-4 py-2 flex gap-3 overflow-x-auto text-xs whitespace-nowrap scrollbar-hide bg-white items-center h-10 border-b">
            <span class="flex items-center font-bold text-purple-700"><span class="w-2.5 h-2.5 rounded-full bg-purple-600 mr-1"></span>議會</span>
            <span class="flex items-center font-bold text-red-700"><span class="w-2.5 h-2.5 rounded-full bg-red-600 mr-1"></span>市府</span>
            <div class="w-px h-4 bg-gray-300 mx-1"></div>
            <span class="flex items-center"><span class="w-2.5 h-2.5 rounded-full bg-cyan-500 mr-1"></span>市美</span>
            <span class="flex items-center"><span class="w-2.5 h-2.5 rounded-full bg-rose-500 mr-1"></span>表藝</span>
            <span class="flex items-center"><span class="w-2.5 h-2.5 rounded-full bg-blue-500 mr-1"></span>資源</span>
        </div>
    </header>

    <main class="max-w-[1600px] mx-auto px-4 py-6">
        <div class="grid grid-cols-1 xl:grid-cols-4 gap-6 items-start">
            <div class="xl:col-span-3 space-y-6">
                <div class="bg-white rounded-xl shadow-md overflow-hidden border border-gray-200">
                    <div class="grid grid-cols-7 bg-gray-50 text-center py-2 border-b">
                        <div class="text-red-500 font-bold text-sm">日</div><div class="text-gray-500 font-bold text-sm">一</div><div class="text-gray-500 font-bold text-sm">二</div><div class="text-gray-500 font-bold text-sm">三</div><div class="text-gray-500 font-bold text-sm">四</div><div class="text-gray-500 font-bold text-sm">五</div><div class="text-green-600 font-bold text-sm">六</div>
                    </div>
                    <div class="calendar-grid">
                        <template x-for="day in daysInMonth" :key="day.dateStr">
                            <div class="calendar-cell p-1" :class="{'bg-gray-50 text-gray-400': !day.isCurrentMonth, 'active': isSelected(day.dateStr), 'bg-yellow-50': isToday(day.dateStr) && !isSelected(day.dateStr)}" @click="selectDate(day.dateStr)">
                                <div class="flex justify-between items-start px-1 mb-1"><span class="text-xs font-bold" x-text="day.dayNum"></span></div>
                                <div class="flex flex-col gap-0.5">
                                    <template x-for="evt in getStartedEventsForDay(day.dateStr, 'council').slice(0, 1)"><div class="event-bar bg-purple-100 text-purple-900 border-purple-600" x-text="getEventDisplayText(evt)"></div></template>
                                    <template x-for="evt in getStartedEventsForDay(day.dateStr, 'city').slice(0, 1)"><div class="event-bar bg-red-100 text-red-900 border-red-600 font-bold" x-text="getEventDisplayText(evt)"></div></template>
                                    <template x-for="evt in getStartedEventsForDay(day.dateStr, 'bureau').slice(0, 3)"><div class="event-bar" :class="getUnitTheme(evt.unit).bar" x-text="getEventDisplayText(evt)"></div></template>
                                </div>
                            </div>
                        </template>
                    </div>
                </div>

                <div class="grid grid-cols-1 lg:grid-cols-3 gap-4">
                    <div class="bg-white rounded-xl shadow-md border-t-4 border-purple-600 flex flex-col min-h-[250px]">
                        <div class="p-3 border-b bg-purple-50 flex justify-between items-center"><h2 class="font-bold text-purple-900 text-lg flex items-center"><i class="fas fa-gavel mr-2"></i>議會行程</h2></div>
                        <div class="p-3 flex-1 overflow-y-auto max-h-[400px]">
                            <template x-if="selectedDayEvents.council.length === 0"><div class="text-center text-gray-400 py-8 text-sm"><p>無行程</p></div></template>
                            <div class="space-y-3"><template x-for="evt in selectedDayEvents.council" :key="evt.id"><div class="bg-white border border-purple-100 rounded-lg p-3 shadow-sm relative group"><div class="flex justify-between items-start"><div><div class="flex items-center gap-2 text-xs text-gray-500 mb-1"><span x-text="formatDateRange(evt.start, evt.end)"></span></div><h3 class="font-bold text-gray-800 text-sm" x-text="evt.title"></h3></div><button @click.stop="editEvent(evt)" class="text-gray-300 hover:text-purple-600 p-1"><i class="fas fa-pen"></i></button></div></div></template></div>
                        </div>
                    </div>
                    <div class="bg-white rounded-xl shadow-md border-t-4 border-red-600 flex flex-col min-h-[250px]">
                        <div class="p-3 border-b bg-red-50 flex justify-between items-center"><h2 class="font-bold text-red-900 text-lg flex items-center"><i class="fas fa-building mr-2"></i>市府重大</h2></div>
                        <div class="p-3 flex-1 overflow-y-auto max-h-[400px]">
                            <template x-if="selectedDayEvents.city.length === 0"><div class="text-center text-gray-400 py-8 text-sm"><p>無行程</p><button @click="openModal('city')" class="mt-1 text-red-600 hover:underline">新增</button></div></template>
                            <div class="space-y-3"><template x-for="evt in selectedDayEvents.city" :key="evt.id"><div class="bg-white border border-red-100 rounded-lg p-3 shadow-sm relative group border-l-4 border-l-red-500"><div class="flex justify-between items-start"><div><div class="flex items-center gap-2 text-xs text-gray-500 mb-1"><span x-text="formatDateRange(evt.start, evt.end)"></span></div><h3 class="font-bold text-gray-800 text-sm" x-text="evt.title"></h3></div><button @click.stop="editEvent(evt)" class="text-gray-300 hover:text-red-600 p-1"><i class="fas fa-pen"></i></button></div></div></template></div>
                        </div>
                    </div>
                    <div class="bg-white rounded-xl shadow-md border-t-4 border-blue-600 flex flex-col min-h-[250px]">
                        <div class="p-3 border-b bg-blue-50 flex justify-between items-center"><h2 class="font-bold text-gray-800 text-lg flex items-center"><i class="fas fa-palette mr-2"></i>文化局</h2></div>
                        <div class="p-3 flex-1 overflow-y-auto max-h-[400px]">
                            <template x-if="selectedDayEvents.bureau.length === 0"><div class="text-center text-gray-400 py-8 text-sm"><p>無活動</p><button @click="openModal('bureau')" class="mt-1 text-blue-600 hover:underline">新增</button></div></template>
                            <div class="space-y-3"><template x-for="evt in selectedDayEvents.bureau" :key="evt.id"><div class="bg-white border rounded-lg p-3 shadow-sm relative group border-l-4" :class="[getUnitTheme(evt.unit).cardBorder, evt.highlight ? 'bg-orange-50' : 'bg-white']"><div class="flex justify-between items-start"><div class="flex-1"><div class="flex items-center gap-2 text-xs text-gray-500 mb-1 flex-wrap"><span class="px-1.5 py-0.5 rounded font-bold" :class="getUnitTheme(evt.unit).badge" x-text="evt.unit"></span><span x-text="formatDateRange(evt.start, evt.end)"></span></div><h3 class="font-bold text-gray-800 text-sm leading-snug" x-text="evt.title"></h3></div><button @click.stop="editEvent(evt)" class="text-gray-300 hover:text-blue-600 p-1"><i class="fas fa-pen"></i></button></div></div></template></div>
                        </div>
                    </div>
                </div>
            </div>

            <div class="xl:col-span-1 bg-white rounded-xl shadow-lg border border-gray-200 sticky top-24 h-[calc(100vh-8rem)] flex flex-col">
                <div class="p-4 border-b bg-orange-50 rounded-t-xl"><h2 class="font-bold text-orange-900 text-lg flex items-center"><i class="fas fa-star text-orange-600 mr-2"></i>年度亮點</h2></div>
                <div class="flex-1 overflow-y-auto p-2 sidebar-scroll space-y-2">
                    <template x-for="evt in highlights" :key="evt.id">
                        <div @click="jumpToEvent(evt)" class="bg-white p-3 rounded-lg border hover:border-orange-400 hover:shadow-md cursor-pointer transition group relative" :class="evt.status === 'pending' ? 'border-l-4 border-l-red-400' : 'border-l-4 border-l-orange-500'">
                            <div class="flex justify-between items-start mb-1"><span class="text-xs font-bold text-gray-500" x-text="formatDateShort(evt.start)"></span><span class="text-[10px] px-1.5 py-0.5 rounded font-bold" :class="getUnitTheme(evt.unit).badge" x-text="evt.unit"></span></div>
                            <h3 class="font-bold text-gray-800 text-sm leading-tight group-hover:text-orange-700" x-text="evt.title"></h3>
                        </div>
                    </template>
                </div>
            </div>
        </div>
    </main>

    <button @click="openModal()" class="fixed bottom-6 right-4 sm:right-8 bg-blue-600 hover:bg-blue-700 text-white w-14 h-14 rounded-full shadow-2xl flex items-center justify-center text-2xl z-40 transition transform active:scale-95 border-2 border-white"><i class="fas fa-plus"></i></button>
    <div x-show="showModal" x-cloak class="fixed inset-0 bg-black bg-opacity-60 z-50 flex items-center justify-center p-4 backdrop-blur-sm">
        <div class="bg-white rounded-xl shadow-2xl w-full max-w-lg overflow-hidden flex flex-col max-h-[90vh]" @click.away="showModal = false">
             <div class="bg-gray-50 px-5 py-3 border-b flex justify-between items-center"><h3 class="font-bold text-lg text-gray-800" x-text="isEditing ? '編輯' : '新增'"></h3><button @click="showModal = false"><i class="fas fa-times"></i></button></div>
             <div class="p-5 space-y-4 overflow-y-auto">
                <div><label class="block text-sm font-bold text-gray-700 mb-1">歸類</label><div class="grid grid-cols-3 gap-2"><button type="button" @click="form.category='council'" :class="form.category==='council'?'bg-purple-600 text-white':'bg-white border'" class="py-2 border rounded font-bold text-sm">議會</button><button type="button" @click="form.category='city'" :class="form.category==='city'?'bg-red-600 text-white':'bg-white border'" class="py-2 border rounded font-bold text-sm">市府</button><button type="button" @click="form.category='bureau'" :class="form.category==='bureau'?'bg-blue-600 text-white':'bg-white border'" class="py-2 border rounded font-bold text-sm">文化局</button></div></div>
                <div><label class="block text-sm font-bold text-gray-700">名稱</label><input type="text" x-model="form.title" class="w-full border p-2 rounded"></div>
                <div class="grid grid-cols-2 gap-4"><div><label class="block text-sm font-bold text-gray-700">開始</label><input type="date" x-model="form.start" class="w-full border p-2 rounded"></div><div><label class="block text-sm font-bold text-gray-700">結束</label><input type="date" x-model="form.end" class="w-full border p-2 rounded"></div></div>
                <div class="grid grid-cols-2 gap-4"><div><label class="block text-sm font-bold text-gray-700">單位</label><select x-model="form.unit" class="w-full border p-2 rounded"><option value="文化局">🏛️ 文化局</option><option value="研究">🪻 研究科</option><option value="表藝">💗 表藝科</option><option value="市美">💠 市美館</option><option value="港藝">🌊 港藝中心</option><option value="大墩">🔸 大墩中心</option><option value="葫蘆墩">🌿 葫蘆墩</option><option value="屯藝">🍋 屯藝中心</option><option value="纖維">🧵 纖維館</option><option value="文資">🧱 文資處</option><option value="市圖">📚 市圖書館</option><option value="視覺">🎨 視覺科</option><option value="資源">🤝 資源科</option><option value="秘書">📎 秘書室</option><option value="議會">🟣 議會</option><option value="市府">🔴 市府</option></select></div><div><label class="block text-sm font-bold text-gray-700">狀態</label><select x-model="form.status" class="w-full border p-2 rounded"><option value="confirmed">✅ 已確認</option><option value="pending">⚠ 暫定</option></select></div></div>
                <div class="flex items-center gap-3 pt-2 p-3 bg-orange-50 rounded" x-show="form.category === 'bureau'"><input type="checkbox" id="highlight" x-model="form.highlight"><label for="highlight" class="text-sm font-bold">標記為重大亮點</label></div>
             </div>
             <div class="bg-gray-50 px-5 py-4 border-t flex gap-3"><button x-show="isEditing" @click="deleteEvent()" class="bg-red-100 text-red-700 px-4 py-2 rounded font-bold">刪除</button><div class="flex-1"></div><button @click="showModal = false" class="bg-gray-200 px-4 py-2 rounded font-bold">取消</button><button @click="saveEvent()" class="bg-blue-600 text-white px-6 py-2 rounded font-bold">儲存</button></div>
        </div>
    </div>

    <script>
        function calendarApp() {
            return {
                // ⚠️ 請務必將下方網址換成您部署後的 Google Apps Script 網址 ⚠️
                API_URL: 'https://script.google.com/macros/s/AKfycbxgA-VXlhcxv0MG4qcIqJkC2zY4pIe6IWCVRlzkfdGDXq98zw81YqM5s0X5gtvzSlg1Mw/exec', 
                
                currentDate: new Date(2026, 0, 20),
                selectedDate: '2026-01-20',
                showModal: false, isEditing: false, isLoading: false,
                events: [],
                form: { id: null, title: '', start: '', end: '', category: 'bureau', unit: '文化局', status: 'confirmed', highlight: false },

                getUnitTheme(unit) {
                    const themes = {
                        '研究': { bar: 'bg-violet-100 text-violet-900 border-violet-500', badge: 'bg-violet-100 text-violet-800', cardBorder: 'border-violet-500' },
                        '表藝': { bar: 'bg-rose-100 text-rose-900 border-rose-500', badge: 'bg-rose-100 text-rose-800', cardBorder: 'border-rose-500' },
                        '市美': { bar: 'bg-cyan-100 text-cyan-900 border-cyan-500', badge: 'bg-cyan-100 text-cyan-800', cardBorder: 'border-cyan-500' },
                        '港藝': { bar: 'bg-indigo-100 text-indigo-900 border-indigo-500', badge: 'bg-indigo-100 text-indigo-800', cardBorder: 'border-indigo-500' },
                        '大墩': { bar: 'bg-amber-100 text-amber-900 border-amber-500', badge: 'bg-amber-100 text-amber-800', cardBorder: 'border-amber-500' },
                        '葫蘆墩': { bar: 'bg-emerald-100 text-emerald-900 border-emerald-500', badge: 'bg-emerald-100 text-emerald-800', cardBorder: 'border-emerald-500' },
                        '屯藝': { bar: 'bg-lime-100 text-lime-900 border-lime-500', badge: 'bg-lime-100 text-lime-800', cardBorder: 'border-lime-500' },
                        '纖維': { bar: 'bg-teal-100 text-teal-900 border-teal-500', badge: 'bg-teal-100 text-teal-800', cardBorder: 'border-teal-500' },
                        '文資': { bar: 'bg-stone-100 text-stone-900 border-stone-500', badge: 'bg-stone-100 text-stone-800', cardBorder: 'border-stone-500' },
                        '市圖': { bar: 'bg-sky-100 text-sky-900 border-sky-500', badge: 'bg-sky-100 text-sky-800', cardBorder: 'border-sky-500' },
                        '資源': { bar: 'bg-blue-100 text-blue-900 border-blue-500', badge: 'bg-blue-100 text-blue-800', cardBorder: 'border-blue-500' },
                        '視覺': { bar: 'bg-pink-100 text-pink-900 border-pink-500', badge: 'bg-pink-100 text-pink-800', cardBorder: 'border-pink-500' },
                        '秘書': { bar: 'bg-gray-100 text-gray-900 border-gray-500', badge: 'bg-gray-100 text-gray-800', cardBorder: 'border-gray-500' },
                        '議會': { bar: 'bg-purple-100 text-purple-900 border-purple-500', badge: 'bg-purple-100 text-purple-800', cardBorder: 'border-purple-500' },
                        '市府': { bar: 'bg-red-100 text-red-900 border-red-500', badge: 'bg-red-100 text-red-800', cardBorder: 'border-red-500' },
                        '文化局': { bar: 'bg-gray-100 text-gray-900 border-gray-500', badge: 'bg-gray-100 text-gray-800', cardBorder: 'border-gray-500' }
                    }; return themes[unit] || themes['文化局'];
                },

                init() {
                    if (this.API_URL === 'https://script.google.com/macros/s/AKfycbxgA-VXlhcxv0MG4qcIqJkC2zY4pIe6IWCVRlzkfdGDXq98zw81YqM5s0X5gtvzSlg1Mw/exec') {
                        alert("請先設定 Google Apps Script 網址，否則無法同步！");
                        // 載入預設資料作為展示
                        this.loadLocalMockData();
                    } else {
                        this.fetchData();
                    }
                    if (window.innerWidth < 640) this.resetToToday();
                },

                // 從 Google Sheet 讀取資料
                async fetchData() {
                    this.isLoading = true;
                    try {
                        const res = await fetch(this.API_URL);
                        const data = await res.json();
                        this.events = data;
                        console.log("資料同步成功", data);
                    } catch (error) {
                        console.error("讀取失敗", error);
                        alert("讀取資料失敗，請檢查網路或 API 網址");
                    } finally {
                        this.isLoading = false;
                    }
                },

                // 寫入資料到 Google Sheet
                async saveData() {
                    this.isLoading = true;
                    try {
                        // 使用 fetch POST
                        await fetch(this.API_URL, {
                            method: 'POST',
                            mode: 'no-cors', // Google Apps Script POST 需要 no-cors
                            headers: { 'Content-Type': 'application/json' },
                            body: JSON.stringify(this.events)
                        });
                        
                        // 因為 no-cors 不會回傳結果，我們假設成功並重新讀取確保一致
                        // 等待 1 秒讓 Google Sheet 寫入完成
                        setTimeout(() => {
                            this.fetchData(); 
                            this.showModal = false;
                            alert("儲存成功！");
                        }, 1000);
                        
                    } catch (error) {
                        console.error("儲存失敗", error);
                        alert("儲存失敗");
                        this.isLoading = false;
                    }
                },

                loadLocalMockData() {
                    // 這裡放原本的靜態資料，當作未連線時的範本
                    this.events = [
                        { id: 101, title: '範例資料 (未連線)', start: '2026-01-20', end: '2026-01-20', category: 'bureau', unit: '文化局', status: 'confirmed', highlight: false }
                    ];
                },

                // --- 行事曆核心邏輯維持不變 ---
                get highlights() { return this.events.filter(e => e.category === 'bureau' && e.highlight).sort((a, b) => new Date(a.start) - new Date(b.start)); },
                jumpToEvent(evt) { const [y, m, d] = evt.start.split('-'); this.currentDate = new Date(parseInt(y), parseInt(m) - 1, 1); this.selectedDate = evt.start; },
                get daysInMonth() {
                    const year = this.currentDate.getFullYear(); const month = this.currentDate.getMonth();
                    const firstDay = new Date(year, month, 1); const lastDay = new Date(year, month + 1, 0); const days = []; const startDayOfWeek = firstDay.getDay();
                    for (let i = 0; i < startDayOfWeek; i++) { const d = new Date(year, month, -i); const y = d.getFullYear(); const m = String(d.getMonth() + 1).padStart(2, '0'); const dayVal = String(d.getDate()).padStart(2, '0'); days.unshift({ dayNum: d.getDate(), dateStr: `${y}-${m}-${dayVal}`, isCurrentMonth: false }); }
                    for (let i = 1; i <= lastDay.getDate(); i++) { const d = new Date(year, month, i); const y = d.getFullYear(); const m = String(d.getMonth() + 1).padStart(2, '0'); const dayVal = String(d.getDate()).padStart(2, '0'); days.push({ dayNum: i, dateStr: `${y}-${m}-${dayVal}`, isCurrentMonth: true }); }
                    for (let i = 1; i <= (42 - days.length); i++) { const d = new Date(year, month + 1, i); const y = d.getFullYear(); const m = String(d.getMonth() + 1).padStart(2, '0'); const dayVal = String(d.getDate()).padStart(2, '0'); days.push({ dayNum: i, dateStr: `${y}-${m}-${dayVal}`, isCurrentMonth: false }); }
                    return days;
                },
                changeMonth(step) { this.currentDate = new Date(this.currentDate.getFullYear(), this.currentDate.getMonth() + step, 1); },
                resetToToday() { const now = new Date(); const localISOTime = new Date(now.getTime() - now.getTimezoneOffset() * 60000).toISOString().slice(0, 10); this.selectedDate = localISOTime; this.currentDate = new Date(2026, 0, 1); },
                formatMonthYear() { return `${this.currentDate.getFullYear()}年 ${this.currentDate.getMonth() + 1}月`; },
                isToday(dateStr) { return false; }, isSelected(dateStr) { return this.selectedDate === dateStr; }, selectDate(dateStr) { this.selectedDate = dateStr; },
                getStartedEventsForDay(dateStr, categoryFilter = null) { return this.events.filter(e => (categoryFilter ? e.category === categoryFilter : true) && e.start === dateStr); },
                sortEvents(events) { return events.sort((a, b) => { const aStart = a.start === this.selectedDate; const bStart = b.start === this.selectedDate; if (aStart && !bStart) return -1; if (!aStart && bStart) return 1; return 0; }); },
                getEventsForDay(dateStr, categoryFilter = null) { const list = this.events.filter(e => (categoryFilter ? e.category === categoryFilter : true) && dateStr >= e.start && dateStr <= e.end); return this.sortEvents(list); },
                get selectedDayEvents() { return { council: this.getEventsForDay(this.selectedDate, 'council'), bureau: this.getEventsForDay(this.selectedDate, 'bureau'), city: this.getEventsForDay(this.selectedDate, 'city') }; },
                get selectedDateDisplay() { if(!this.selectedDate) return ''; const [y, m, d] = this.selectedDate.split('-'); return `${m}/${d}`; },
                formatDateRange(start, end) { if (start === end) return start.substring(5); return `${start.substring(5)}~${end.substring(5)}`; },
                formatDateShort(dateStr) { return dateStr.substring(5).replace('-', '/'); },
                getEventDisplayText(evt) { return evt.start === evt.end ? evt.title : `${evt.start.substring(5).replace('-','/')}~${evt.end.substring(5).replace('-','/')} ${evt.title}`; },
                
                openModal(category = 'bureau') { this.isEditing = false; this.form = { id: Date.now(), title: '', start: this.selectedDate, end: this.selectedDate, category: category, unit: category === 'city' ? '市府' : '文化局', status: 'confirmed', highlight: false }; this.showModal = true; },
                editEvent(evt) { this.isEditing = true; this.form = JSON.parse(JSON.stringify(evt)); this.showModal = true; },
                
                // 改用 saveData 存入 Google Sheet
                saveEvent() { if (!this.form.title) { alert('請輸入活動名稱'); return; } const newData = JSON.parse(JSON.stringify(this.form)); if (this.isEditing) { const index = this.events.findIndex(e => e.id === this.form.id); if (index !== -1) this.events[index] = newData; } else { this.events.push(newData); } this.saveData(); },
                deleteEvent() { if (confirm('確定要刪除？')) { this.events = this.events.filter(e => e.id !== this.form.id); this.saveData(); } },
            }
        }
    </script>
</body>
</html>
