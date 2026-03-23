# new-zea
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>紐西蘭 15 天旅遊規劃 - 定位連結版</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css" rel="stylesheet">
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/sortablejs@1.15.0/Sortable.min.js"></script>
    
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@400;500;700;900&display=swap');
        body { font-family: 'Noto+Sans+TC', sans-serif; background-color: #f8fafc; }
        .map-container { height: 380px; border-radius: 2rem; z-index: 10; border: 4px solid white; box-shadow: 0 10px 30px -10px rgba(0,0,0,0.1); }
        .nav-item.active { color: #4f46e5; border-bottom: 4px solid #4f46e5; font-weight: 900; }
        .day-btn.active { background-color: #4f46e5; color: white; transform: scale(1.1); box-shadow: 0 4px 12px rgba(79, 70, 229, 0.4); }
        .card-shadow { transition: all 0.3s ease; }
        .card-shadow:hover { transform: translateY(-3px); box-shadow: 0 15px 20px -5px rgba(0, 0, 0, 0.05); }
        .no-scrollbar::-webkit-scrollbar { display: none; }
    </style>
</head>
<body class="pb-24">

    <!-- Header -->
    <header class="bg-white/90 backdrop-blur-md sticky top-0 z-50 border-b">
        <div class="max-w-7xl mx-auto px-6 h-16 flex justify-between items-center">
            <div class="text-xl font-black text-indigo-600">NZ EXPLORER PRO</div>
            <nav class="hidden md:flex gap-6 h-full font-bold text-slate-500">
                <button onclick="changeTab('itinerary')" class="nav-item h-full flex items-center px-1" id="tab-itinerary">行程管理</button>
                <button onclick="changeTab('restaurant')" class="nav-item h-full flex items-center px-1" id="tab-restaurant">推薦餐廳</button>
                <button onclick="changeTab('activity')" class="nav-item h-full flex items-center px-1" id="tab-activity">活動體驗</button>
                <button onclick="changeTab('room')" class="nav-item h-full flex items-center px-1" id="tab-room">住宿清單</button>
                <button onclick="changeTab('others')" class="nav-item h-full flex items-center px-1" id="tab-others">其他連結</button>
            </nav>
            <button onclick="resetData()" class="text-xs font-bold text-slate-300 hover:text-red-500 transition">重置</button>
        </div>
    </header>

    <main class="max-w-7xl mx-auto p-4 md:p-8">
        
        <!-- 行程區塊 -->
        <div id="section-itinerary" class="section-content">
            <div class="flex overflow-x-auto no-scrollbar gap-3 mb-6 pb-2" id="day-selector"></div>
            <div id="itinerary-map" class="map-container mb-8"></div>
            <div class="flex justify-between items-end mb-6">
                <div>
                    <h2 class="text-3xl font-black text-slate-800" id="current-day-title">Day 1 行程</h2>
                    <p class="text-slate-400 text-sm mt-1">拖曳可排序，有網址的地點會顯示連結圖示</p>
                </div>
                <button onclick="openModal('add', 'itinerary')" class="bg-indigo-600 text-white px-6 py-3 rounded-2xl font-bold shadow-lg hover:bg-indigo-700 transition flex items-center gap-2">
                    <i class="fa-solid fa-plus"></i> 新增地點
                </button>
            </div>
            <div id="itinerary-list" class="space-y-4"></div>
        </div>

        <!-- 通用列表區塊 -->
        <div id="section-generic" class="section-content hidden">
            <div id="restaurant-city-selector" class="flex overflow-x-auto no-scrollbar gap-2 mb-6 hidden"></div>
            <div id="restaurant-map" class="map-container mb-8 hidden"></div>
            <div class="flex justify-between items-center mb-6">
                <h2 class="text-3xl font-black text-slate-800" id="generic-title">資訊清單</h2>
                <button id="add-generic-btn" class="bg-slate-900 text-white px-6 py-3 rounded-2xl font-bold flex items-center gap-2">
                    <i class="fa-solid fa-plus"></i> 新增資料
                </button>
            </div>
            <div id="generic-grid" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6"></div>
        </div>

    </main>

    <!-- 表單彈窗 -->
    <div id="data-modal" class="fixed inset-0 bg-slate-900/60 backdrop-blur-sm z-[100] hidden flex items-center justify-center p-4">
        <div class="bg-white w-full max-w-xl rounded-[2.5rem] p-8 shadow-2xl max-h-[90vh] overflow-y-auto no-scrollbar">
            <div class="flex justify-between items-center mb-8">
                <h3 id="modal-title" class="text-2xl font-black text-slate-800">編輯內容</h3>
                <button onclick="closeModal()" class="w-10 h-10 flex items-center justify-center rounded-full bg-slate-100 text-slate-400"><i class="fa-solid fa-xmark"></i></button>
            </div>
            <form id="main-form" class="space-y-4">
                <input type="hidden" id="f-type">
                <input type="hidden" id="f-id">
                <input type="hidden" id="f-lat"><input type="hidden" id="f-lng">

                <div>
                    <label class="text-xs font-black text-slate-400 uppercase mb-2 block">項目名稱</label>
                    <input id="f-name" type="text" required placeholder="例如：奧克蘭大學" class="w-full p-4 bg-slate-50 border-none rounded-2xl focus:ring-2 focus:ring-indigo-500 outline-none">
                </div>
                <div id="city-field" class="hidden">
                    <label class="text-xs font-black text-slate-400 uppercase mb-2 block">所屬城市</label>
                    <select id="f-city" class="w-full p-4 bg-slate-50 border-none rounded-2xl outline-none">
                        <option value="奧克蘭">奧克蘭</option><option value="羅托路亞">羅托路亞</option>
                        <option value="陶波">陶波</option><option value="威靈頓">威靈頓</option><option value="基督城">基督城</option>
                    </select>
                </div>
                <div>
                    <label class="text-xs font-black text-slate-400 uppercase mb-2 block">Google 地圖定位地址</label>
                    <input id="f-address" type="text" placeholder="輸入地址後系統將自動抓取經緯度" class="w-full p-4 bg-slate-50 border-none rounded-2xl focus:ring-2 focus:ring-indigo-500 outline-none">
                </div>
                <div>
                    <label class="text-xs font-black text-slate-400 uppercase mb-2 block text-indigo-600">外部參考網址 (點擊連結圖示跳轉)</label>
                    <input id="f-url" type="text" placeholder="https://..." class="w-full p-4 bg-indigo-50/50 border-none rounded-2xl focus:ring-2 focus:ring-indigo-500 outline-none">
                </div>
                <div class="grid grid-cols-2 gap-4">
                    <div>
                        <label class="text-xs font-black text-slate-400 uppercase mb-2 block">預估價格</label>
                        <input id="f-price" type="text" placeholder="NTD" class="w-full p-4 bg-slate-50 border-none rounded-2xl outline-none">
                    </div>
                    <div>
                        <label class="text-xs font-black text-slate-400 uppercase mb-2 block">交通方式</label>
                        <input id="f-trans" type="text" placeholder="客運 / 走路" class="w-full p-4 bg-slate-50 border-none rounded-2xl outline-none">
                    </div>
                </div>
                <div>
                    <label class="text-xs font-black text-slate-400 uppercase mb-2 block">簡介或備註</label>
                    <textarea id="f-desc" rows="3" class="w-full p-4 bg-slate-50 border-none rounded-2xl outline-none" placeholder="輸入補充資訊..."></textarea>
                </div>
                <button type="submit" class="w-full bg-indigo-600 text-white py-5 rounded-2xl font-black text-lg shadow-xl shadow-indigo-100 hover:bg-indigo-700 transition">儲存並定位</button>
            </form>
        </div>
    </div>

    <!-- Mobile Nav -->
    <div class="md:hidden fixed bottom-0 w-full bg-white border-t flex justify-around py-4 z-[60]">
        <button onclick="changeTab('itinerary')" class="flex flex-col items-center gap-1 text-slate-400" id="m-tab-itinerary"><i class="fa-solid fa-map-location"></i><span class="text-[10px]">行程</span></button>
        <button onclick="changeTab('restaurant')" class="flex flex-col items-center gap-1 text-slate-400" id="m-tab-restaurant"><i class="fa-solid fa-utensils"></i><span class="text-[10px]">餐廳</span></button>
        <button onclick="changeTab('room')" class="flex flex-col items-center gap-1 text-slate-400" id="m-tab-room"><i class="fa-solid fa-bed"></i><span class="text-[10px]">住宿</span></button>
        <button onclick="changeTab('others')" class="flex flex-col items-center gap-1 text-slate-400" id="m-tab-others"><i class="fa-solid fa-link"></i><span class="text-[10px]">連結</span></button>
    </div>

    <script>
        const NZ_CITIES = {
            "奧克蘭": [-36.8484, 174.7633],
            "羅托路亞": [-38.1368, 176.2497],
            "陶波": [-38.6857, 176.0702],
            "威靈頓": [-41.2865, 174.7762],
            "基督城": [-43.5321, 172.6362]
        };

        const INITIAL_DATA = {
            itinerary: {
                1: [
                    {id: 1, name: "奧克蘭機場", address: "Auckland Airport", trans: "飛機", lat:-37.0082, lng:174.7850, url: "https://www.google.com/maps/search/?api=1&query=Auckland+Airport"},
                    {id: 2, name: "奧克蘭濱海", address: "Viaduct Harbour, Auckland", trans: "客運", lat:-36.8437, lng:174.7627, url: "https://www.google.com/maps/search/?api=1&query=Viaduct+Harbour+Auckland"},
                    {id: 3, name: "奧克蘭大橋觀景", address: "Auckland Harbour Bridge viewpoint", trans: "走路", lat:-36.8286, lng:174.7508, url: "https://www.google.com/maps/search/?api=1&query=Auckland+Harbour+Bridge+viewpoint"}
                ],
                2: [
                    {id: 4, name: "奧克蘭大學", address: "University of Auckland", trans: "公車", lat:-36.8520, lng:174.7690, url: "https://www.google.com/maps/search/?api=1&query=University+of+Auckland"},
                    {id: 5, name: "伊甸山", address: "Mount Eden Auckland", trans: "公車", lat:-36.8780, lng:174.7640, url: "https://www.google.com/maps/search/?api=1&query=Mount+Eden+Auckland"},
                    {id: 6, name: "聖三一大教堂", address: "Holy Trinity Cathedral Auckland", trans: "走路", lat:-36.8596, lng:174.7836, url: "https://www.google.com/maps/search/?api=1&query=Holy+Trinity+Cathedral+Auckland"}
                ],
                3: [
                    {id: 7, name: "奧克蘭美術館", address: "Auckland Art Gallery", trans: "走路", lat:-36.8509, lng:174.7656, url: "https://www.google.com/maps/search/?api=1&query=Auckland+Art+Gallery"},
                    {id: 8, name: "阿爾伯特公園", address: "Albert Park Auckland", trans: "走路", lat:-36.8505, lng:174.7680}
                ],
                4: [
                    {id: 9, name: "天空塔", address: "Sky Tower Auckland", trans: "走路", lat:-36.8485, lng:174.7622, url: "https://www.google.com/maps/search/?api=1&query=Sky+Tower+Auckland"},
                    {id: 10, name: "奧克蘭-羅托路亞", address: "InterCity Bus Auckland", trans: "客運", lat:-36.8475, lng:174.7630}
                ],
                5: [
                    {id: 11, name: "庫伊勞公園", address: "Kuirau Park", trans: "走路", lat:-38.1328, lng:176.2447, url: "https://www.google.com/maps/search/?api=1&query=Kuirau+Park"},
                    {id: 12, name: "羅托路亞博物館", address: "Rotorua Museum", trans: "走路", lat:-38.1350, lng:176.2588, url: "https://www.google.com/maps/search/?api=1&query=Rotorua+Museum"}
                ]
                // ... 其它天數
            },
            restaurant: { '奧克蘭': [], '羅托路亞': [], '陶波': [], '威靈頓': [], '基督城': [] },
            activity: [],
            room: [
                {id: 101, name: "Kiwi International Hotel", address: "411 Queen Street, Auckland", desc: "1/5–1/8 (3晚) | 家庭房", price: "5979", url: "https://www.google.com/maps/search/?api=1&query=Kiwi+International+Hotel+Auckland"},
                {id: 102, name: "Rotorua Thermal Holiday Park", address: "463 Old Taupo Rd, Rotorua", desc: "1/8–1/11 (3晚) | 標準小屋", price: "9351", url: "https://www.google.com/maps/search/?api=1&query=Rotorua+Thermal+Holiday+Park"},
                {id: 103, name: "Taupo Debretts Spa Resort", address: "76 Napier-Taupo Rd, Taupo", desc: "1/11–1/14 (3晚) | 三人小屋", price: "4974", url: "https://www.google.com/maps/search/?api=1&query=Taupo+Debretts+Spa+Resort"},
                {id: 104, name: "Hotel Waterloo & Backpackers", address: "1 Bunny Street, Wellington", desc: "1/14–1/16 (2晚) | 三人房", price: "3852", url: "https://www.google.com/maps/search/?api=1&query=Hotel+Waterloo+Wellington"}
            ],
            others: [
                {id: 201, name: "役男出國前一個月內申請", url: "https://www.gov.tw/News_Content_2_371238", desc: "出國前必辦"},
                {id: 202, name: "入境憑證申請及流程", url: "https://nativecamp.net/zh-tw/blog/19873/", desc: "入境流程詳細教學"},
                {id: 203, name: "奧克蘭攻略", url: "https://timtingtravel.com/new-zealand-auckland/", desc: "奧克蘭旅遊總整理"},
                {id: 204, name: "羅托魯瓦攻略", url: "https://timtingtravel.com/rotorua/", desc: "溫泉、文化、景點"},
                {id: 205, name: "威靈頓攻略", url: "https://yanziaart.com/solotravel_nz_d10_wellington/", desc: "首都一日遊推薦"},
                {id: 206, name: "北島到南島搭船攻略", url: "https://yanziaart.com/solotravel_nz_d11_interislander/", desc: "跨島渡輪教學"},
                {id: 207, name: "基督城攻略", url: "https://yanziaart.com/solotravel_nz_d13_christchurch/", desc: "南島門戶景點"},
                {id: 208, name: "InterCity 長途巴士官網", url: "https://www.intercity.co.nz/", desc: "巴士訂票系統"}
            ]
        };

        // 確保 15 天都有資料結構
        for(let i=1; i<=15; i++) if(!INITIAL_DATA.itinerary[i]) INITIAL_DATA.itinerary[i] = [];

        let state = JSON.parse(localStorage.getItem('nz_final_link_v2')) || {
            tab: 'itinerary',
            day: 1,
            city: '奧克蘭',
            data: INITIAL_DATA
        };

        let mapItinerary = null;
        let mapRestaurant = null;
        let markersItinerary = [];
        let markersRestaurant = [];
        let routeLine = null;

        function initMap(id) {
            if (id === 'itinerary' && !mapItinerary) {
                mapItinerary = L.map('itinerary-map').setView(NZ_CITIES["奧克蘭"], 11);
                L.tileLayer('https://{s}.basemaps.cartocdn.com/rastertiles/voyager/{z}/{x}/{y}{r}.png').addTo(mapItinerary);
            } else if (id === 'restaurant' && !mapRestaurant) {
                mapRestaurant = L.map('restaurant-map').setView(NZ_CITIES["奧克蘭"], 11);
                L.tileLayer('https://{s}.basemaps.cartocdn.com/rastertiles/voyager/{z}/{x}/{y}{r}.png').addTo(mapRestaurant);
            }
        }

        async function fetchCoords(address, name) {
            if (!address) return null;
            try {
                const res = await fetch(`https://nominatim.openstreetmap.org/search?format=json&q=${encodeURIComponent(address || name)}+New+Zealand&limit=1`);
                const data = await res.json();
                if (data.length > 0) return [parseFloat(data[0].lat), parseFloat(data[0].lon)];
            } catch (e) { console.error("Geocoding failed", e); }
            return null;
        }

        async function refreshMaps() {
            if (state.tab === 'itinerary') {
                if (!mapItinerary) initMap('itinerary');
                markersItinerary.forEach(m => mapItinerary.removeLayer(m));
                markersItinerary = [];
                if (routeLine) mapItinerary.removeLayer(routeLine);
                const items = state.data.itinerary[state.day];
                const coords = [];
                items.forEach((item, idx) => {
                    if (item.lat && item.lng) {
                        const pos = [item.lat, item.lng];
                        const m = L.marker(pos).addTo(mapItinerary).bindPopup(`<b>${idx+1}. ${item.name}</b>`);
                        markersItinerary.push(m);
                        coords.push(pos);
                    }
                });
                if (coords.length > 0) {
                    if (coords.length > 1) routeLine = L.polyline(coords, { color: '#4f46e5', weight: 4, opacity: 0.5 }).addTo(mapItinerary);
                    mapItinerary.fitBounds(L.latLngBounds(coords), { padding: [50, 50], maxZoom: 15 });
                }
            } else if (state.tab === 'restaurant') {
                if (!mapRestaurant) initMap('restaurant');
                markersRestaurant.forEach(m => mapRestaurant.removeLayer(m));
                markersRestaurant = [];
                const items = state.data.restaurant[state.city] || [];
                const coords = [];
                items.forEach(item => {
                    if (item.lat && item.lng) {
                        const pos = [item.lat, item.lng];
                        const m = L.marker(pos).addTo(mapRestaurant).bindPopup(`<b>${item.name}</b>`);
                        markersRestaurant.push(m);
                        coords.push(pos);
                    }
                });
                if (coords.length > 0) mapRestaurant.fitBounds(L.latLngBounds(coords), { padding: [50, 50], maxZoom: 15 });
                else mapRestaurant.setView(NZ_CITIES[state.city] || NZ_CITIES["奧克蘭"], 12);
            }
        }

        function changeTab(tab) {
            state.tab = tab;
            document.querySelectorAll('.section-content').forEach(s => s.classList.add('hidden'));
            document.querySelectorAll('.nav-item').forEach(n => n.classList.remove('active'));
            document.querySelectorAll('.md\\:hidden button').forEach(b => b.classList.remove('text-indigo-600'));
            
            if (tab === 'itinerary') {
                document.getElementById('section-itinerary').classList.remove('hidden');
                document.getElementById('tab-itinerary').classList.add('active');
                document.getElementById('m-tab-itinerary').classList.add('text-indigo-600');
                renderDaySelector(); renderItinerary();
            } else {
                document.getElementById('section-generic').classList.remove('hidden');
                document.getElementById(`tab-${tab}`).classList.add('active');
                if(document.getElementById(`m-tab-${tab}`)) document.getElementById(`m-tab-${tab}`).classList.add('text-indigo-600');
                const isRest = tab === 'restaurant';
                document.getElementById('restaurant-city-selector').classList.toggle('hidden', !isRest);
                document.getElementById('restaurant-map').classList.toggle('hidden', !isRest);
                if (isRest) renderCitySelector();
                renderGeneric();
            }
            setTimeout(() => { 
                if(mapItinerary) mapItinerary.invalidateSize(); 
                if(mapRestaurant) mapRestaurant.invalidateSize(); 
                refreshMaps(); 
            }, 300);
            save();
        }

        function renderDaySelector() {
            const container = document.getElementById('day-selector');
            container.innerHTML = '';
            for(let i=1; i<=15; i++) {
                const btn = document.createElement('button');
                btn.className = `day-btn shrink-0 w-12 h-12 rounded-2xl border-2 border-white font-black transition-all ${state.day == i ? 'active' : 'bg-white text-slate-400'}`;
                btn.innerText = i;
                btn.onclick = () => { state.day = i; renderDaySelector(); renderItinerary(); refreshMaps(); };
                container.appendChild(btn);
            }
        }

        function renderCitySelector() {
            const container = document.getElementById('restaurant-city-selector');
            container.innerHTML = '';
            Object.keys(state.data.restaurant).forEach(c => {
                const btn = document.createElement('button');
                btn.className = `shrink-0 px-5 py-2 rounded-full font-bold text-sm ${state.city === c ? 'bg-indigo-600 text-white' : 'bg-white text-slate-500'}`;
                btn.innerText = c;
                btn.onclick = () => { state.city = c; renderCitySelector(); renderGeneric(); refreshMaps(); };
                container.appendChild(btn);
            });
        }

        function renderItinerary() {
            const list = document.getElementById('itinerary-list');
            document.getElementById('current-day-title').innerText = `Day ${state.day} 行程`;
            list.innerHTML = '';
            const items = state.data.itinerary[state.day];
            items.forEach((item, idx) => {
                list.innerHTML += `
                    <div class="bg-white p-5 rounded-[2rem] card-shadow flex items-center gap-4 cursor-move border-l-4 ${item.lat ? 'border-indigo-500' : 'border-slate-100'}" data-id="${item.id}">
                        <div class="w-8 h-8 rounded-full bg-slate-100 flex items-center justify-center font-black text-slate-400 text-xs">${idx+1}</div>
                        <div class="flex-1 min-w-0">
                            <div class="flex items-center gap-2">
                                <h4 class="font-bold text-slate-800 truncate">${item.name}</h4>
                                ${item.url ? `<a href="${item.url}" target="_blank" class="text-indigo-500 hover:text-indigo-700 bg-indigo-50 w-6 h-6 flex items-center justify-center rounded-full transition-colors"><i class="fa-solid fa-arrow-up-right-from-square text-[10px]"></i></a>` : ''}
                            </div>
                            <div class="flex gap-3 mt-1">
                                <span class="text-[10px] text-slate-400"><i class="fa-solid fa-car mr-1"></i>${item.trans || '未定'}</span>
                                <span class="text-[10px] text-slate-400"><i class="fa-solid fa-tag mr-1"></i>$${item.price || '0'}</span>
                            </div>
                        </div>
                        <div class="flex gap-1">
                            <button onclick="editItem('itinerary', ${item.id})" class="p-2 text-slate-200 hover:text-indigo-600"><i class="fa-solid fa-pen"></i></button>
                            <button onclick="deleteItem('itinerary', ${item.id})" class="p-2 text-slate-200 hover:text-red-500"><i class="fa-solid fa-trash"></i></button>
                        </div>
                    </div>
                `;
            });
            new Sortable(list, { animation: 200, onEnd: () => {
                const ids = Array.from(list.children).map(c => parseInt(c.dataset.id));
                state.data.itinerary[state.day] = ids.map(id => state.data.itinerary[state.day].find(i => i.id == id));
                save(); refreshMaps();
            }});
        }

        function renderGeneric() {
            const grid = document.getElementById('generic-grid');
            const title = document.getElementById('generic-title');
            const btn = document.getElementById('add-generic-btn');
            const labels = { restaurant: '推薦餐廳', activity: '活動體驗', room: '住宿清單', others: '重要連結' };
            title.innerText = labels[state.tab];
            btn.onclick = () => openModal('add', state.tab);
            grid.innerHTML = '';
            const items = (state.tab === 'restaurant') ? (state.data.restaurant[state.city] || []) : (state.data[state.tab] || []);
            
            if (items.length === 0) {
                grid.innerHTML = `<div class="col-span-full py-20 text-center text-slate-300 font-bold">目前暫無資料</div>`;
                return;
            }

            items.forEach(item => {
                grid.innerHTML += `
                    <div class="bg-white rounded-[2.5rem] p-6 card-shadow flex flex-col justify-between border border-slate-50">
                        <div>
                            <div class="flex justify-between items-start mb-2">
                                <h4 class="font-black text-lg text-slate-800 leading-tight">${item.name}</h4>
                                ${item.price ? `<span class="text-[10px] font-bold text-indigo-500 bg-indigo-50 px-2 py-1 rounded-lg shrink-0">$${item.price}</span>` : ''}
                            </div>
                            <p class="text-xs text-slate-400 mb-4 line-clamp-3">${item.desc || item.address || '無說明'}</p>
                        </div>
                        <div class="flex flex-col gap-2 mt-2">
                            ${item.url ? `<a href="${item.url}" target="_blank" class="w-full py-3 bg-indigo-600 text-white rounded-xl text-center font-bold text-xs shadow-lg shadow-indigo-100 hover:bg-indigo-700 transition"><i class="fa-solid fa-link mr-1"></i>查看詳細資訊</a>` : ''}
                            <div class="flex gap-2">
                                <button onclick="editItem('${state.tab}', ${item.id})" class="flex-1 py-2 bg-slate-50 text-slate-400 rounded-lg text-[10px] font-bold hover:bg-slate-100">編輯</button>
                                <button onclick="deleteItem('${state.tab}', ${item.id})" class="px-3 bg-slate-50 text-slate-300 rounded-lg hover:text-red-500"><i class="fa-solid fa-trash"></i></button>
                            </div>
                        </div>
                    </div>
                `;
            });
        }

        function openModal(mode, type, id = null) {
            const form = document.getElementById('main-form');
            form.reset();
            document.getElementById('f-type').value = type;
            document.getElementById('f-id').value = id || '';
            document.getElementById('modal-title').innerText = (id ? '編輯' : '新增') + '內容';
            document.getElementById('city-field').classList.toggle('hidden', type !== 'restaurant');
            
            if (id) {
                let item = (type === 'itinerary') ? state.data.itinerary[state.day].find(i => i.id == id) :
                           (type === 'restaurant') ? state.data.restaurant[state.city].find(i => i.id == id) :
                           state.data[type].find(i => i.id == id);
                if (item) {
                    document.getElementById('f-name').value = item.name;
                    document.getElementById('f-address').value = item.address || '';
                    document.getElementById('f-url').value = item.url || '';
                    document.getElementById('f-price').value = item.price || '';
                    document.getElementById('f-trans').value = item.trans || '';
                    document.getElementById('f-desc').value = item.desc || '';
                    if(type === 'restaurant') document.getElementById('f-city').value = state.city;
                }
            }
            document.getElementById('data-modal').classList.remove('hidden');
        }

        function closeModal() { document.getElementById('data-modal').classList.add('hidden'); }

        document.getElementById('main-form').onsubmit = async function(e) {
            e.preventDefault();
            const type = document.getElementById('f-type').value;
            const id = document.getElementById('f-id').value;
            const addr = document.getElementById('f-address').value;
            const name = document.getElementById('f-name').value;
            
            const coords = await fetchCoords(addr, name);
            const data = {
                id: id ? parseInt(id) : Date.now(),
                name: name, 
                address: addr,
                url: document.getElementById('f-url').value,
                price: document.getElementById('f-price').value,
                trans: document.getElementById('f-trans').value,
                desc: document.getElementById('f-desc').value,
                lat: coords ? coords[0] : null, 
                lng: coords ? coords[1] : null
            };

            if (type === 'itinerary') {
                if(id) { const idx = state.data.itinerary[state.day].findIndex(i => i.id == id); state.data.itinerary[state.day][idx] = data; }
                else state.data.itinerary[state.day].push(data);
                renderItinerary();
            } else if (type === 'restaurant') {
                const city = document.getElementById('f-city').value;
                if(id) { const idx = state.data.restaurant[state.city].findIndex(i => i.id == id); state.data.restaurant[state.city][idx] = data; }
                else state.data.restaurant[city].push(data);
                renderGeneric();
            } else {
                if(id) { const idx = state.data[type].findIndex(i => i.id == id); state.data[type][idx] = data; }
                else state.data[type].push(data);
                renderGeneric();
            }
            save(); closeModal(); refreshMaps();
        };

        function deleteItem(type, id) {
            if(!confirm('確定刪除此項目？')) return;
            if(type === 'itinerary') state.data.itinerary[state.day] = state.data.itinerary[state.day].filter(i => i.id != id);
            else if(type === 'restaurant') state.data.restaurant[state.city] = state.data.restaurant[state.city].filter(i => i.id != id);
            else state.data[type] = state.data[type].filter(i => i.id != id);
            save(); changeTab(state.tab);
        }

        function editItem(t, id) { openModal('edit', t, id); }
        function save() { localStorage.setItem('nz_final_link_v2', JSON.stringify(state)); }
        function resetData() { if(confirm('這將清除所有自定義修改，恢復預設資料。確定嗎？')) { localStorage.clear(); location.reload(); } }

        window.onload = () => {
            // 自動修復可能遺失的天數資料
            for(let i=1; i<=15; i++) if(!state.data.itinerary[i]) state.data.itinerary[i] = [];
            changeTab(state.tab);
        };
    </script>
</body>
</html>
