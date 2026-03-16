<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Load Profile Interactive Dashboard</title>
    <!-- โหลด Tailwind CSS สำหรับจัดการความสวยงาม (UI) -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- โหลด Chart.js สำหรับสร้างกราฟ -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <!-- โหลด PapaParse สำหรับอ่านไฟล์ CSV -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.4.1/papaparse.min.js"></script>
    <!-- โหลด Moment.js สำหรับจัดการวันที่ในกราฟ -->
    <script src="https://cdn.jsdelivr.net/npm/moment@2.29.4/moment.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chartjs-adapter-moment@1.0.1/dist/chartjs-adapter-moment.min.js"></script>
    
    <!-- โหลด Plugin สำหรับทำ Zoom และ Pan กราฟ -->
    <script src="https://cdn.jsdelivr.net/npm/hammerjs@2.0.8"></script>
    <script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-zoom@2.0.1/dist/chartjs-plugin-zoom.min.js"></script>

    <!-- โหลด Flatpickr สำหรับช่องเลือกวันที่แบบ 24 ชม. และพิมพ์ได้ -->
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/flatpickr/dist/flatpickr.min.css">
    <script src="https://cdn.jsdelivr.net/npm/flatpickr"></script>
    <script src="https://npmcdn.com/flatpickr/dist/l10n/th.js"></script>
    
    <style>
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background-color: #f8fafc; }
        .chart-container { position: relative; height: 380px; width: 100%; }
        /* ตกแต่ง Scrollbar */
        ::-webkit-scrollbar { width: 8px; height: 8px; }
        ::-webkit-scrollbar-track { background: #f1f1f1; }
        ::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 4px; }
        ::-webkit-scrollbar-thumb:hover { background: #94a3b8; }
        #toast { transition: opacity 0.3s ease-in-out; }
    </style>
</head>
<body class="text-gray-900 bg-gray-50">

    <!-- แถบเมนูด้านบน -->
    <nav class="bg-gray-900 text-white shadow-md p-4 flex justify-between items-center sticky top-0 z-40 border-b-4 border-purple-600">
        <div class="flex items-center space-x-3 cursor-pointer" onclick="showHome()">
            <svg class="w-7 h-7 text-purple-400" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 19v-6a2 2 0 00-2-2H5a2 2 0 00-2 2v6a2 2 0 002 2h2a2 2 0 002-2zm0 0V9a2 2 0 012-2h2a2 2 0 012 2v10m-6 0a2 2 0 002 2h2a2 2 0 002-2m0 0V5a2 2 0 012-2h2a2 2 0 012 2v14a2 2 0 01-2 2h-2a2 2 0 01-2-2z"></path></svg>
            <h1 class="text-xl font-bold tracking-wide hidden sm:block">Load Profile</h1>
        </div>
        <div class="flex items-center space-x-2">
            
            <button onclick="showHome()" class="bg-gray-800 text-gray-200 px-3 py-2.5 rounded-lg shadow-sm hover:bg-gray-700 hover:text-white transition text-sm font-bold flex items-center space-x-1 sm:space-x-2">
                <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 12l2-2m0 0l7-7 7 7M5 10v10a1 1 0 001 1h3m10-11l2 2m-2-2v10a1 1 0 01-1 1h-3m-6 0a1 1 0 001-1v-4a1 1 0 011-1h2a1 1 0 011 1v4a1 1 0 001 1m-6 0h6"></path></svg>
                <span class="hidden sm:inline">หน้าแรก</span>
            </button>

            <!-- Dropdown สำหรับเลือกข้อมูลที่บันทึกไว้ -->
            <select id="profileSelector" onchange="switchProfile(this.value)" class="hidden bg-white text-gray-800 text-sm rounded-lg px-3 py-2.5 border border-gray-300 shadow-sm focus:outline-none focus:ring-2 focus:ring-purple-500 font-medium max-w-[150px] sm:max-w-xs cursor-pointer">
                <option value="">-- ข้อมูลที่บันทึก --</option>
            </select>
            
            <button id="deleteProfileBtn" onclick="openDeleteModal()" class="hidden bg-gray-800 text-gray-200 px-3 sm:px-4 py-2.5 rounded-lg shadow-sm hover:bg-rose-600 hover:text-white transition text-sm font-bold items-center space-x-1 sm:space-x-2">
                <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"></path></svg>
                <span class="hidden lg:inline">ลบข้อมูลนี้</span>
            </button>

            <label for="csvFileInput" class="bg-white text-purple-700 px-3 sm:px-5 py-2.5 rounded-lg shadow-sm cursor-pointer hover:bg-purple-50 hover:shadow transition text-sm font-bold flex items-center space-x-1 sm:space-x-2">
                <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4v16m8-8H4"></path></svg>
                <span class="hidden sm:inline">เพิ่มข้อมูลใหม่</span>
                <span class="sm:hidden">เพิ่ม</span>
            </label>
            <input type="file" id="csvFileInput" accept=".csv" class="hidden">
        </div>
    </nav>

    <!-- พื้นที่เนื้อหาหลัก -->
    <main class="max-w-7xl mx-auto p-4 md:p-6">
        
        <!-- กล่องข้อความแจ้งเตือน (Toast) -->
        <div id="toast" class="hidden fixed top-20 right-4 bg-purple-600 text-white px-6 py-3 rounded-lg shadow-xl z-50 flex items-center space-x-2">
            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"></path></svg>
            <span id="toastMessage" class="font-medium">แจ้งเตือน</span>
        </div>

        <!-- แท็บนำทาง (Tab Navigation) -->
        <div class="flex space-x-1 bg-gray-200 p-1.5 rounded-xl mb-6 w-fit mx-auto md:mx-0 shadow-inner">
            <button onclick="switchTab('dashboard')" id="tabBtnDashboard" class="px-4 sm:px-6 py-2 rounded-lg bg-white text-purple-700 font-bold shadow-sm transition">📊 แดชบอร์ดวิเคราะห์</button>
            <button onclick="switchTab('calculator')" id="tabBtnCalculator" class="px-4 sm:px-6 py-2 rounded-lg text-gray-600 hover:text-gray-900 font-semibold transition">🧮 เครื่องคิดเลขค่าไฟ</button>
        </div>

        <!-- ==========================================
             VIEW 1: แดชบอร์ด (Dashboard View)
        =========================================== -->
        <div id="tabDashboardView">
            
            <!-- หน้าแรก (Home / Empty State) -->
            <div id="emptyState" class="space-y-8 mt-4 md:mt-8">
                <!-- แบนเนอร์เพิ่มข้อมูลใหม่ -->
                <div class="bg-white rounded-2xl shadow-sm border border-gray-200 p-8 md:p-12 text-center">
                    <div class="bg-purple-50 w-20 h-20 rounded-full flex items-center justify-center mx-auto mb-6">
                        <svg class="w-10 h-10 text-purple-600" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 17v-2m3 2v-4m3 4v-6m2 10H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"></path></svg>
                    </div>
                    <h2 class="text-2xl font-bold text-gray-800 mb-2">ยินดีต้อนรับสู่ Load Profile</h2>
                    <p class="text-gray-500 max-w-md mx-auto mb-6">คลิกเลือกข้อมูลที่เคยบันทึกไว้ด้านล่าง เพื่อดู Dashboard<br>หรืออัปโหลดไฟล์ข้อมูล CSV ใหม่เพื่อเริ่มต้นวิเคราะห์</p>
                    
                    <label for="csvFileInputHome" class="inline-flex bg-purple-600 text-white px-8 py-3.5 rounded-xl shadow-md cursor-pointer hover:bg-purple-700 hover:shadow-lg transition font-bold items-center space-x-2">
                        <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-8l-4-4m0 0L8 8m4-4v12"></path></svg>
                        <span>อัปโหลดไฟล์ CSV ใหม่</span>
                    </label>
                    <input type="file" id="csvFileInputHome" accept=".csv" class="hidden">
                </div>

                <!-- กล่องรายการข้อมูลที่เซฟไว้ (จะแสดงเมื่อมีข้อมูลในระบบ) -->
                <div id="savedProfilesContainer" class="hidden">
                    <div class="flex items-center mb-4 space-x-2 justify-between w-full">
                        <div class="flex items-center space-x-2">
                            <svg class="w-6 h-6 text-purple-600" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 8h14M5 8a2 2 0 110-4h14a2 2 0 110 4M5 8v10a2 2 0 002 2h10a2 2 0 002-2V8m-9 4h4"></path></svg>
                            <h3 class="text-xl font-bold text-gray-900">ข้อมูลที่บันทึกไว้ล่าสุด</h3>
                        </div>
                        <button onclick="openTrashModal()" class="text-sm font-semibold text-gray-500 hover:text-rose-600 transition flex items-center space-x-1 bg-white px-3 py-1.5 rounded-lg border border-gray-200 shadow-sm">
                            <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"></path></svg>
                            <span>ถังขยะ</span>
                        </button>
                    </div>
                    <div id="savedProfilesGrid" class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-4">
                        <!-- การ์ดข้อมูลจะถูกสร้างและนำมาใส่ตรงนี้ด้วย Javascript -->
                    </div>
                </div>
            </div>

            <!-- พื้นที่ Dashboard (จะแสดงเมื่อเลือกข้อมูลแล้ว) -->
            <div id="dashboardContent" class="hidden space-y-6">
                
                <!-- ส่วนตัวกรองข้อมูล (Filter) -->
                <div class="bg-white p-5 rounded-2xl shadow-sm border border-gray-200 flex flex-col lg:flex-row items-end lg:items-center space-y-4 lg:space-y-0 lg:space-x-4">
                    <div class="w-full lg:w-auto">
                        <label class="block text-xs font-bold text-gray-500 mb-1.5">ตั้งแต่ วันที่-เวลา</label>
                        <input type="text" id="startDate" placeholder="คลิกเลือกหรือพิมพ์เวลา..." class="w-full border border-gray-300 rounded-lg px-3 py-2 text-sm focus:outline-none focus:border-purple-500 focus:ring-1 focus:ring-purple-500 bg-white">
                    </div>
                    <div class="w-full lg:w-auto">
                        <label class="block text-xs font-bold text-gray-500 mb-1.5">ถึง วันที่-เวลา</label>
                        <input type="text" id="endDate" placeholder="คลิกเลือกหรือพิมพ์เวลา..." class="w-full border border-gray-300 rounded-lg px-3 py-2 text-sm focus:outline-none focus:border-purple-500 focus:ring-1 focus:ring-purple-500 bg-white">
                    </div>
                    <div class="w-full lg:w-auto flex space-x-2">
                        <button id="applyFilterBtn" class="w-full lg:w-auto bg-purple-600 hover:bg-purple-700 text-white px-6 py-2 rounded-lg shadow-sm text-sm font-semibold transition">
                            กรองข้อมูล
                        </button>
                        <button id="resetFilterBtn" class="w-full lg:w-auto bg-gray-900 hover:bg-black text-white px-6 py-2 rounded-lg text-sm font-semibold transition border border-gray-800">
                            แสดงทั้งหมด
                        </button>
                    </div>
                    <div class="flex-grow text-right text-sm text-gray-500 mt-4 lg:mt-0 bg-gray-50 px-4 py-2 rounded-lg inline-block">
                        ข้อมูลทั้งหมด: <span id="dataCount" class="font-bold text-purple-700 text-lg">0</span> รายการ
                    </div>
                </div>

                <!-- แถบแนะนำการซูมกราฟและปุ่มรีเซ็ต -->
                <div class="flex flex-col sm:flex-row justify-between items-center bg-purple-50 text-purple-900 p-3 rounded-xl text-sm font-medium border border-purple-100 shadow-sm">
                    <div class="flex items-center space-x-2 mb-2 sm:mb-0">
                        <svg class="w-5 h-5 text-purple-600" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0zM10 7v3m0 0v3m0-3h3m-3 0H7"></path></svg>
                        <span>💡 <b>การควบคุมกราฟ:</b> เลื่อนลูกกลิ้งเมาส์ (Scroll) เพื่อซูม และคลิกเมาส์ค้างเพื่อเลื่อนซ้าย-ขวา</span>
                    </div>
                    <button onclick="resetAllZooms()" class="bg-gray-900 hover:bg-black text-white px-4 py-1.5 rounded-lg shadow transition whitespace-nowrap">
                        รีเซ็ตมุมมองกราฟ
                    </button>
                </div>

                <!-- กล่องสรุปสถิติ (Summary Cards) -->
                <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
                    <div class="bg-white p-6 rounded-2xl shadow-sm border border-gray-200 relative overflow-hidden group">
                        <div class="absolute top-0 right-0 w-16 h-16 bg-purple-100 rounded-bl-full -mr-8 -mt-8 transition-transform group-hover:scale-110"></div>
                        <p class="text-xs font-bold text-gray-500 uppercase tracking-wider relative z-10">พลังงานไฟฟ้ารวม</p>
                        <h3 class="text-3xl font-bold text-gray-900 mt-2 relative z-10"><span id="statTotalEnergy">0.00</span> <span class="text-base font-medium text-gray-500">kWh</span></h3>
                        <p class="text-xs text-gray-400 mt-2 relative z-10 font-medium bg-gray-50 inline-block px-2 py-1 rounded">สูตร: ∑ (kW ÷ 4)</p>
                    </div>
                    <div class="bg-white p-6 rounded-2xl shadow-sm border border-gray-200 relative overflow-hidden group cursor-pointer hover:border-purple-400 hover:shadow-md transition-all" onclick="togglePeakInfo('power')" title="คลิกเพื่อดูเวลาที่เกิด Peak">
                        <div class="absolute top-0 right-0 w-16 h-16 bg-gray-200 rounded-bl-full -mr-8 -mt-8 transition-transform group-hover:scale-110"></div>
                        <div class="flex justify-between items-start relative z-10">
                            <p class="text-xs font-bold text-gray-500 uppercase tracking-wider">กำลังไฟฟ้าสูงสุด (Peak)</p>
                            <svg class="w-4 h-4 text-gray-400 group-hover:text-purple-600 transition-colors" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 15l-2 5L9 9l11 4-5 2zm0 0l5 5M7.188 2.239l.777 2.897M5.136 7.965l-2.898-.777M13.95 4.05l-2.122 2.122m-5.657 5.656l-2.12 2.122"></path></svg>
                        </div>
                        <h3 class="text-3xl font-bold text-gray-900 mt-2 relative z-10"><span id="statMaxPower">0.00</span> <span class="text-base font-medium text-gray-500">kW</span></h3>
                        <p class="text-xs text-gray-400 mt-2 relative z-10 font-medium bg-gray-50 inline-block px-2 py-1 rounded">จากคอลัมน์ CH4</p>
                        <div id="maxPowerTime" class="hidden mt-3 relative z-10">
                            <div class="bg-purple-50 border border-purple-100 rounded-lg p-2.5 text-sm font-bold text-purple-700 flex items-center shadow-inner">
                                <span id="maxPowerTimeText">⏱️ -</span>
                            </div>
                        </div>
                    </div>
                    <div class="bg-white p-6 rounded-2xl shadow-sm border border-gray-200 relative overflow-hidden group cursor-pointer hover:border-purple-400 hover:shadow-md transition-all" onclick="togglePeakInfo('current')" title="คลิกเพื่อดูเวลาที่เกิด Peak">
                        <div class="absolute top-0 right-0 w-16 h-16 bg-gray-800 rounded-bl-full -mr-8 -mt-8 transition-transform group-hover:scale-110"></div>
                        <div class="flex justify-between items-start relative z-10">
                            <p class="text-xs font-bold text-gray-500 uppercase tracking-wider">กระแสไฟฟ้าสูงสุด</p>
                            <svg class="w-4 h-4 text-gray-400 group-hover:text-purple-600 transition-colors" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 15l-2 5L9 9l11 4-5 2zm0 0l5 5M7.188 2.239l.777 2.897M5.136 7.965l-2.898-.777M13.95 4.05l-2.122 2.122m-5.657 5.656l-2.12 2.122"></path></svg>
                        </div>
                        <h3 class="text-3xl font-bold text-gray-900 mt-2 relative z-10"><span id="statMaxCurrent">0.00</span> <span class="text-base font-medium text-gray-500">A</span></h3>
                        <p class="text-xs text-gray-400 mt-2 relative z-10 font-medium bg-gray-50 inline-block px-2 py-1 rounded">สูตร: (kW × 1000) ÷ V</p>
                        <div id="maxCurrentTime" class="hidden mt-3 relative z-10">
                            <div class="bg-purple-50 border border-purple-100 rounded-lg p-2.5 text-sm font-bold text-purple-700 flex items-center shadow-inner">
                                <span id="maxCurrentTimeText">⏱️ -</span>
                            </div>
                        </div>
                    </div>
                    <div class="bg-white p-6 rounded-2xl shadow-sm border border-gray-200 relative overflow-hidden group">
                        <div class="absolute top-0 right-0 w-16 h-16 bg-purple-50 rounded-bl-full -mr-8 -mt-8 transition-transform group-hover:scale-110"></div>
                        <p class="text-xs font-bold text-gray-500 uppercase tracking-wider relative z-10">แรงดันไฟฟ้าเฉลี่ย</p>
                        <h3 class="text-3xl font-bold text-gray-900 mt-2 relative z-10"><span id="statAvgVoltage">0.00</span> <span class="text-base font-medium text-gray-500">V</span></h3>
                        <p class="text-xs text-gray-400 mt-2 relative z-10 font-medium bg-gray-50 inline-block px-2 py-1 rounded">จากคอลัมน์ CH3</p>
                    </div>
                </div>

                <!-- พื้นที่แสดงกราฟรวม (Combined Chart) -->
                <div class="bg-white p-5 rounded-2xl shadow-sm border border-gray-200 mt-6">
                    <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center mb-2 space-y-2 sm:space-y-0">
                        <h3 class="text-lg font-bold text-gray-900">กราฟข้อมูลโหลดโปรไฟล์รวม (แท่งกราฟเปรียบเทียบ)</h3>
                    </div>
                    <p class="text-sm text-purple-600 font-semibold mb-4 bg-purple-50 inline-block px-3 py-1.5 rounded-lg border border-purple-100">
                        👆 <b>ทริคการใช้งาน:</b> คุณสามารถ "คลิก" ที่ชื่อข้อมูลด้านบนกราฟ (เช่น คำว่า กำลังไฟฟ้า) เพื่อเลือกเปิดหรือปิดการแสดงผลได้ตามต้องการ!
                    </p>
                    <div class="chart-container" style="height: 480px;">
                        <canvas id="mainChart"></canvas>
                    </div>
                </div>

                <!-- ตารางข้อมูล (Data Table) -->
                <div class="bg-white p-5 rounded-2xl shadow-sm border border-gray-200 mt-6">
                    <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center mb-4 space-y-3 sm:space-y-0">
                        <h3 class="text-lg font-bold text-gray-900">ตารางข้อมูลรายละเอียด (Data Table)</h3>
                        <button onclick="exportToCSV()" class="bg-emerald-600 hover:bg-emerald-700 text-white px-4 py-2 rounded-lg text-sm font-semibold shadow-sm flex items-center space-x-2 transition">
                            <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4"></path></svg>
                            <span>ดาวน์โหลดข้อมูล (CSV)</span>
                        </button>
                    </div>
                    <div class="overflow-x-auto max-h-96 rounded-lg border border-gray-200">
                        <table class="w-full text-sm text-left text-gray-600">
                            <thead class="text-xs text-white uppercase bg-gray-900 sticky top-0 z-10 shadow-sm border-b-2 border-purple-600">
                                <tr>
                                    <th scope="col" class="px-6 py-3 whitespace-nowrap">วัน-เวลา</th>
                                    <th scope="col" class="px-6 py-3 whitespace-nowrap text-center">สถานะ</th>
                                    <th scope="col" class="px-6 py-3 whitespace-nowrap text-right">แรงดันไฟฟ้า (V)</th>
                                    <th scope="col" class="px-6 py-3 whitespace-nowrap text-right">กำลังไฟฟ้า (kW)</th>
                                    <th scope="col" class="px-6 py-3 whitespace-nowrap text-right text-purple-300">พลังงาน (kWh)</th>
                                    <th scope="col" class="px-6 py-3 whitespace-nowrap text-right text-gray-300">กระแสไฟฟ้า (A)</th>
                                </tr>
                            </thead>
                            <tbody id="dataTableBody">
                                <!-- ข้อมูลจะถูกเติมเข้ามาตรงนี้ผ่าน JavaScript -->
                            </tbody>
                        </table>
                    </div>
                    <div class="mt-3 text-xs text-gray-400 text-right">
                        *แสดงข้อมูลตามช่วงเวลาที่กรอง (จำกัดการแสดงผลสูงสุด 5,000 แถวแรกเพื่อประสิทธิภาพของบราวเซอร์)
                    </div>
                </div>

            </div>
        </div>

        <!-- ==========================================
             VIEW 2: เครื่องคิดเลขค่าไฟ (Calculator View)
        =========================================== -->
        <div id="tabCalculatorView" class="hidden space-y-6">
            <div class="bg-white p-6 md:p-8 rounded-2xl shadow-sm border border-gray-200">
                <h2 class="text-2xl font-bold text-gray-900 mb-2 flex items-center">
                    <span class="bg-emerald-100 text-emerald-600 p-2 rounded-xl mr-3 shadow-inner">⚡</span>
                    โปรแกรมประเมินค่าไฟเครื่องใช้ไฟฟ้า
                </h2>
                <p class="text-gray-500 mb-8 ml-14">เพิ่มรายการเครื่องใช้ไฟฟ้า ระบุกำลังไฟและชั่วโมงที่ใช้งานเพื่อประเมินค่าไฟรวม</p>

                <div class="grid grid-cols-1 lg:grid-cols-2 gap-8 lg:gap-12">
                    
                    <!-- ส่วนกรอกข้อมูล -->
                    <div class="space-y-4">
                        
                        <!-- อัตราค่าไฟรวม -->
                        <div class="bg-emerald-50 p-4 rounded-xl border border-emerald-200">
                            <label class="block text-sm font-bold text-gray-700 mb-2 flex items-center justify-between">
                                <span>อัตราค่าไฟปัจจุบัน (รวม Ft/ภาษี แล้ว)</span>
                                <span class="text-xs font-normal text-emerald-600 bg-emerald-100 px-2 py-1 rounded">อัปเดตล่าสุด</span>
                            </label>
                            <div class="flex items-center space-x-3">
                                <input type="number" id="calcRate" value="4.18" step="0.01" min="0" class="w-full md:w-1/2 border border-emerald-300 rounded-lg px-4 py-2.5 focus:outline-none focus:border-emerald-500 focus:ring-1 focus:ring-emerald-500 text-emerald-700 font-bold text-lg bg-white shadow-sm">
                                <span class="text-gray-600 font-semibold whitespace-nowrap">บาท / หน่วย</span>
                            </div>
                        </div>

                        <!-- รายการเครื่องใช้ไฟฟ้า -->
                        <div id="applianceList" class="space-y-4 max-h-[400px] overflow-y-auto pr-2 pb-2">
                            <!-- แถวเริ่มต้น 1 แถว -->
                            <div class="appliance-item bg-gray-50 p-4 rounded-xl border border-gray-200 relative shadow-sm transition-all">
                                <button onclick="removeAppliance(this)" class="absolute top-3 right-3 text-gray-400 hover:text-rose-500 transition" title="ลบรายการนี้">
                                    <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"></path></svg>
                                </button>
                                <div class="grid grid-cols-1 md:grid-cols-2 gap-4 mr-6">
                                    <div class="md:col-span-2">
                                        <label class="block text-xs font-bold text-gray-500 mb-1">ชื่ออุปกรณ์ (ใส่หรือไม่ใส่ก็ได้)</label>
                                        <input type="text" class="w-full border border-gray-300 rounded-lg px-3 py-2 text-sm focus:outline-none focus:border-emerald-500 focus:ring-1 focus:ring-emerald-500" placeholder="เช่น แอร์ห้องนอน, ทีวี 55 นิ้ว">
                                    </div>
                                    <div>
                                        <label class="block text-xs font-bold text-gray-500 mb-1">กำลังไฟฟ้า</label>
                                        <div class="flex shadow-sm rounded-lg">
                                            <input type="number" class="app-power w-full border border-gray-300 rounded-l-lg px-3 py-2 focus:outline-none focus:border-emerald-500 focus:ring-1 focus:ring-emerald-500 font-mono text-sm" value="1000" min="0">
                                            <select class="app-unit bg-white border border-l-0 border-gray-300 rounded-r-lg px-2 py-2 text-gray-700 font-bold focus:outline-none cursor-pointer text-sm hover:bg-gray-50 transition">
                                                <option value="W">W</option>
                                                <option value="kW" selected>kW</option>
                                            </select>
                                        </div>
                                    </div>
                                    <div>
                                        <label class="block text-xs font-bold text-gray-500 mb-1">เปิดใช้งาน (ชม./วัน)</label>
                                        <input type="number" class="app-hours w-full border border-gray-300 rounded-lg px-3 py-2 focus:outline-none focus:border-emerald-500 focus:ring-1 focus:ring-emerald-500 font-mono text-sm" value="1" min="0" max="24">
                                    </div>
                                    <div>
                                        <label class="block text-xs font-bold text-gray-500 mb-1">ใช้งาน (วัน/เดือน)</label>
                                        <input type="number" class="app-days w-full border border-gray-300 rounded-lg px-3 py-2 focus:outline-none focus:border-emerald-500 focus:ring-1 focus:ring-emerald-500 font-mono text-sm" value="30" min="1" max="31">
                                    </div>
                                </div>
                            </div>
                        </div>

                        <!-- ปุ่มเพิ่มรายการ -->
                        <button onclick="addAppliance()" class="w-full bg-white hover:bg-gray-50 text-emerald-600 font-bold py-3 rounded-xl border-2 border-emerald-200 border-dashed flex justify-center items-center space-x-2 transition shadow-sm">
                            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4v16m8-8H4"></path></svg>
                            <span>เพิ่มเครื่องใช้ไฟฟ้า</span>
                        </button>

                    </div>

                    <!-- ส่วนแสดงผลลัพธ์ -->
                    <div class="bg-gray-900 p-8 rounded-2xl border border-gray-800 flex flex-col justify-center text-white relative overflow-hidden shadow-lg">
                        <!-- BG Decoration -->
                        <div class="absolute -top-24 -right-24 w-48 h-48 bg-emerald-500 rounded-full opacity-10 blur-3xl"></div>
                        <div class="absolute -bottom-24 -left-24 w-48 h-48 bg-purple-500 rounded-full opacity-10 blur-3xl"></div>
                        
                        <h3 class="text-xl font-bold text-gray-200 mb-6 flex items-center relative z-10">
                            <svg class="w-6 h-6 mr-2 text-emerald-400" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 17v-2m3 2v-4m3 4v-6m2 10H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"></path></svg>
                            สรุปค่าใช้จ่ายประเมิน (รวมทุกอุปกรณ์)
                        </h3>
                        
                        <div class="space-y-6 relative z-10">
                            <div class="flex justify-between items-end border-b border-gray-700 pb-4">
                                <span class="text-gray-400 font-medium">กินไฟวันละ:</span>
                                <div class="text-right">
                                    <span class="text-2xl font-bold text-white"><span id="resUnitsDay">0.00</span></span>
                                    <span class="text-sm text-gray-500 font-normal ml-1">หน่วย (kWh)</span>
                                </div>
                            </div>
                            
                            <div class="flex justify-between items-end border-b border-gray-700 pb-4">
                                <span class="text-gray-400 font-medium">กินไฟเดือนละ:</span>
                                <div class="text-right">
                                    <span class="text-3xl font-bold text-emerald-400"><span id="resUnitsMonth">0.00</span></span>
                                    <span class="text-sm text-emerald-500/70 font-normal ml-1">หน่วย (kWh)</span>
                                </div>
                            </div>
                            
                            <div class="pt-4">
                                <div class="flex justify-between items-center mb-2">
                                    <span class="text-gray-300 font-bold text-lg">คิดเป็นเงินประมาณ:</span>
                                </div>
                                <div class="bg-gray-800/80 p-4 rounded-xl border border-gray-700 flex justify-between items-center">
                                    <span class="text-5xl font-bold text-rose-500 tracking-tight"><span id="resCostMonth">0.00</span></span>
                                    <span class="text-gray-400 font-bold text-lg">บาท / เดือน</span>
                                </div>
                            </div>
                        </div>
                    </div>

                </div>
            </div>
        </div>

        <!-- Modal: ตั้งชื่อข้อมูล (Save Profile) -->
        <div id="saveModal" class="hidden fixed inset-0 bg-gray-900 bg-opacity-60 z-50 flex items-center justify-center p-4 backdrop-blur-sm">
            <div class="bg-white rounded-2xl shadow-2xl max-w-md w-full p-6 transform transition-all">
                <h3 class="text-xl font-bold text-gray-900 mb-2">💾 บันทึกข้อมูลโหลดโปรไฟล์</h3>
                <p class="text-sm text-gray-500 mb-5">ตั้งชื่อข้อมูลนี้เพื่อเก็บไว้ดูในภายหลัง (เช่น ชื่อลูกค้า, ชื่อโครงการ)</p>
                
                <input type="text" id="profileNameInput" placeholder="ระบุชื่อข้อมูล..." class="w-full border border-gray-300 rounded-lg px-4 py-3 text-gray-800 focus:outline-none focus:border-purple-500 focus:ring-2 focus:ring-purple-200 mb-2 transition">
                <p id="nameError" class="text-xs text-rose-500 hidden mb-4">กรุณาระบุชื่อข้อมูลก่อนบันทึก</p>
                
                <div class="flex justify-end space-x-3 mt-5">
                    <button onclick="closeSaveModal()" class="px-5 py-2.5 rounded-lg text-gray-600 bg-gray-100 hover:bg-gray-200 font-semibold transition">ยกเลิก</button>
                    <button onclick="confirmSaveProfile()" class="px-5 py-2.5 rounded-lg text-white bg-purple-600 hover:bg-purple-700 font-semibold shadow-md transition flex items-center space-x-2">
                        <span>บันทึกข้อมูล</span>
                    </button>
                </div>
            </div>
        </div>

        <!-- Modal: ยืนยันการลบ (Delete Profile) -->
        <div id="deleteModal" class="hidden fixed inset-0 bg-gray-900 bg-opacity-60 z-50 flex items-center justify-center p-4 backdrop-blur-sm">
            <div class="bg-white rounded-2xl shadow-2xl max-w-sm w-full p-6 text-center transform transition-all">
                <div class="w-16 h-16 bg-rose-100 text-rose-600 rounded-full flex items-center justify-center mx-auto mb-4 shadow-inner">
                    <svg class="w-8 h-8" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z"></path></svg>
                </div>
                <h3 class="text-xl font-bold text-gray-900 mb-2">ยืนยันการลบข้อมูล?</h3>
                <p class="text-sm text-gray-500 mb-6">ระบบจะย้าย <b id="deleteProfileName" class="text-gray-800 text-base"></b> ไปไว้ที่ถังขยะเป็นเวลา 30 วันก่อนลบทิ้งถาวร</p>
                <div class="flex justify-center space-x-3">
                    <button onclick="closeDeleteModal()" class="px-5 py-2.5 rounded-lg text-gray-600 bg-gray-100 hover:bg-gray-200 font-semibold transition w-full">ยกเลิก</button>
                    <button onclick="confirmDeleteProfile()" class="px-5 py-2.5 rounded-lg text-white bg-rose-600 hover:bg-rose-700 font-semibold shadow-md transition w-full">ย้ายไปถังขยะ</button>
                </div>
            </div>
        </div>

        <!-- Modal: ถังขยะ (Trash) -->
        <div id="trashModal" class="hidden fixed inset-0 bg-gray-900 bg-opacity-60 z-50 flex items-center justify-center p-4 backdrop-blur-sm">
            <div class="bg-white rounded-2xl shadow-2xl max-w-2xl w-full p-6 transform transition-all max-h-[90vh] flex flex-col">
                <div class="flex justify-between items-center mb-4">
                    <h3 class="text-xl font-bold text-gray-900 flex items-center">
                        <svg class="w-6 h-6 text-rose-500 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"></path></svg>
                        ถังขยะ (เก็บไว้ 30 วัน)
                    </h3>
                    <button onclick="closeTrashModal()" class="text-gray-400 hover:text-gray-700 transition">
                        <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path></svg>
                    </button>
                </div>
                <div id="trashContainer" class="overflow-y-auto flex-grow pr-2 space-y-3">
                    <!-- รายการถังขยะจะถูกเติมด้วย JS -->
                </div>
                <div class="mt-5 text-right">
                    <button onclick="closeTrashModal()" class="px-5 py-2.5 rounded-lg text-gray-600 bg-gray-100 hover:bg-gray-200 font-semibold transition">ปิดหน้าต่าง</button>
                </div>
            </div>
        </div>

    </main>

    <script>
        // ==========================================
        // ส่วนควบคุมแท็บ (Tab Navigation)
        // ==========================================
        function switchTab(tabId) {
            const tabDashBtn = document.getElementById('tabBtnDashboard');
            const tabCalcBtn = document.getElementById('tabBtnCalculator');
            const viewDash = document.getElementById('tabDashboardView');
            const viewCalc = document.getElementById('tabCalculatorView');

            if (tabId === 'dashboard') {
                tabDashBtn.className = "px-4 sm:px-6 py-2 rounded-lg bg-white text-purple-700 font-bold shadow-sm transition";
                tabCalcBtn.className = "px-4 sm:px-6 py-2 rounded-lg text-gray-600 hover:text-gray-900 font-semibold transition";
                viewDash.classList.remove('hidden');
                viewCalc.classList.add('hidden');
            } else {
                tabCalcBtn.className = "px-4 sm:px-6 py-2 rounded-lg bg-white text-purple-700 font-bold shadow-sm transition";
                tabDashBtn.className = "px-4 sm:px-6 py-2 rounded-lg text-gray-600 hover:text-gray-900 font-semibold transition";
                viewDash.classList.add('hidden');
                viewCalc.classList.remove('hidden');
                calculateCost(); // อัปเดตตัวเลขเมื่อเปิดแท็บนี้
            }
        }

        // ==========================================
        // ลอจิกสำหรับเครื่องคิดเลขค่าไฟ (Calculator)
        // ==========================================
        
        // ฟังก์ชันเพิ่มรายการอุปกรณ์
        function addAppliance() {
            const container = document.getElementById('applianceList');
            const template = `
                <button onclick="removeAppliance(this)" class="absolute top-3 right-3 text-gray-400 hover:text-rose-500 transition" title="ลบรายการนี้">
                    <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"></path></svg>
                </button>
                <div class="grid grid-cols-1 md:grid-cols-2 gap-4 mr-6">
                    <div class="md:col-span-2">
                        <label class="block text-xs font-bold text-gray-500 mb-1">ชื่ออุปกรณ์ (ใส่หรือไม่ใส่ก็ได้)</label>
                        <input type="text" class="w-full border border-gray-300 rounded-lg px-3 py-2 text-sm focus:outline-none focus:border-emerald-500 focus:ring-1 focus:ring-emerald-500" placeholder="เช่น พัดลม, เตารีด">
                    </div>
                    <div>
                        <label class="block text-xs font-bold text-gray-500 mb-1">กำลังไฟฟ้า</label>
                        <div class="flex shadow-sm rounded-lg">
                            <input type="number" class="app-power w-full border border-gray-300 rounded-l-lg px-3 py-2 focus:outline-none focus:border-emerald-500 focus:ring-1 focus:ring-emerald-500 font-mono text-sm" value="0" min="0">
                            <select class="app-unit bg-white border border-l-0 border-gray-300 rounded-r-lg px-2 py-2 text-gray-700 font-bold focus:outline-none cursor-pointer text-sm hover:bg-gray-50 transition">
                                <option value="W" selected>W</option>
                                <option value="kW">kW</option>
                            </select>
                        </div>
                    </div>
                    <div>
                        <label class="block text-xs font-bold text-gray-500 mb-1">เปิดใช้งาน (ชม./วัน)</label>
                        <input type="number" class="app-hours w-full border border-gray-300 rounded-lg px-3 py-2 focus:outline-none focus:border-emerald-500 focus:ring-1 focus:ring-emerald-500 font-mono text-sm" value="1" min="0" max="24">
                    </div>
                    <div>
                        <label class="block text-xs font-bold text-gray-500 mb-1">ใช้งาน (วัน/เดือน)</label>
                        <input type="number" class="app-days w-full border border-gray-300 rounded-lg px-3 py-2 focus:outline-none focus:border-emerald-500 focus:ring-1 focus:ring-emerald-500 font-mono text-sm" value="30" min="1" max="31">
                    </div>
                </div>
            `;
            const div = document.createElement('div');
            div.className = 'appliance-item bg-gray-50 p-4 rounded-xl border border-gray-200 relative shadow-sm transition-all';
            div.innerHTML = template;
            container.appendChild(div);
            
            // เลื่อนหน้าจอลงไปที่รายการใหม่
            container.scrollTop = container.scrollHeight;
            calculateCost();
        }

        // ฟังก์ชันลบรายการอุปกรณ์
        function removeAppliance(btn) {
            const container = document.getElementById('applianceList');
            if (container.querySelectorAll('.appliance-item').length <= 1) {
                showToast('ต้องมีเครื่องใช้ไฟฟ้าอย่างน้อย 1 รายการ', true);
                return;
            }
            const item = btn.closest('.appliance-item');
            item.remove();
            calculateCost();
        }

        // คำนวณค่าไฟรวมทั้งหมด
        function calculateCost() {
            let totalUnitsPerDay = 0;
            let totalUnitsPerMonth = 0;
            const rate = parseFloat(document.getElementById('calcRate').value) || 0;

            // วนลูปอ่านค่าจากทุกรายการเครื่องใช้ไฟฟ้า
            document.querySelectorAll('.appliance-item').forEach(item => {
                const power = parseFloat(item.querySelector('.app-power').value) || 0;
                const unit = item.querySelector('.app-unit').value;
                const hours = parseFloat(item.querySelector('.app-hours').value) || 0;
                const days = parseFloat(item.querySelector('.app-days').value) || 0;

                // แปลงเป็นกิโลวัตต์เสมอ
                let kW = unit === 'W' ? power / 1000 : power;
                
                let daily = kW * hours;
                let monthly = daily * days;

                totalUnitsPerDay += daily;
                totalUnitsPerMonth += monthly;
            });

            const costPerMonth = totalUnitsPerMonth * rate;

            // อัปเดตหน้าจอ
            document.getElementById('resUnitsDay').textContent = totalUnitsPerDay.toLocaleString('en-US', {minimumFractionDigits: 2, maximumFractionDigits: 2});
            document.getElementById('resUnitsMonth').textContent = totalUnitsPerMonth.toLocaleString('en-US', {minimumFractionDigits: 2, maximumFractionDigits: 2});
            document.getElementById('resCostMonth').textContent = costPerMonth.toLocaleString('en-US', {minimumFractionDigits: 2, maximumFractionDigits: 2});
        }

        // ผูก Event ให้คำนวณใหม่ทุกครั้งที่มีการพิมพ์หรือเปลี่ยนค่าในฟอร์มเครื่องคิดเลข
        document.getElementById('calcRate').addEventListener('input', calculateCost);
        document.getElementById('applianceList').addEventListener('input', calculateCost);
        document.getElementById('applianceList').addEventListener('change', calculateCost);

        // ==========================================
        // ลอจิกสำหรับแดชบอร์ด (Load Profile) เดิม
        // ==========================================
        let masterData = [];
        let filteredData = [];
        let mainChartObj = null;
        let peakPowerObj = null;
        let peakCurrentObj = null;
        let fpStart = null;
        let fpEnd = null;

        let savedProfilesDB = JSON.parse(localStorage.getItem('loadProfileDB')) || {};
        let trashDB = JSON.parse(localStorage.getItem('loadProfileTrashDB')) || {};
        let currentProfileId = null;
        let pendingCSVText = "";
        let profileToDelete = null;

        const formatDate = (date) => {
            const d = date.getDate().toString().padStart(2, '0');
            const m = (date.getMonth() + 1).toString().padStart(2, '0');
            const y = date.getFullYear();
            const H = date.getHours().toString().padStart(2, '0');
            const M = date.getMinutes().toString().padStart(2, '0');
            return `${d}/${m}/${y} ${H}:${M}`;
        };

        const fileInput = document.getElementById('csvFileInput');
        const fileInputHome = document.getElementById('csvFileInputHome');
        const emptyState = document.getElementById('emptyState');
        const dashboardContent = document.getElementById('dashboardContent');
        
        const fpConfig = {
            enableTime: true, dateFormat: "Y-m-d H:i", time_24hr: true, allowInput: true, locale: "th"
        };
        fpStart = flatpickr("#startDate", fpConfig);
        fpEnd = flatpickr("#endDate", fpConfig);

        document.addEventListener('DOMContentLoaded', () => {
            cleanupTrash(); 
            updateProfileDropdown();
            showHome(); 
            calculateCost(); // เริ่มการทำงานเครื่องคิดเลขตอนเปิดเว็บ
        });

        function showHome() {
            switchTab('dashboard'); // เผื่ออยู่แท็บอื่นให้เด้งกลับมาแท็บหลัก
            dashboardContent.style.display = 'none';
            emptyState.style.display = 'block';
            
            currentProfileId = null;
            document.getElementById('profileSelector').value = "";
            document.getElementById('deleteProfileBtn').classList.add('hidden');
        }

        // ---------------- ระบบจัดการ Profile ----------------

        function updateProfileDropdown() {
            const selector = document.getElementById('profileSelector');
            const deleteBtn = document.getElementById('deleteProfileBtn');
            const profilesContainer = document.getElementById('savedProfilesContainer');
            const profilesGrid = document.getElementById('savedProfilesGrid');
            
            const profileIds = Object.keys(savedProfilesDB);
            selector.innerHTML = '<option value="">-- สลับดูข้อมูลที่บันทึก --</option>';
            profilesGrid.innerHTML = '';
            
            if (profileIds.length > 0) {
                selector.classList.remove('hidden');
                profilesContainer.classList.remove('hidden');
                
                const sortedProfiles = profileIds.map(id => ({id, ...savedProfilesDB[id]})).sort((a, b) => {
                    if (a.isStarred && !b.isStarred) return -1;
                    if (!a.isStarred && b.isStarred) return 1;
                    return b.timestamp - a.timestamp;
                });
                
                let gridHtml = '';
                sortedProfiles.forEach(p => {
                    const option = document.createElement('option');
                    option.value = p.id;
                    option.textContent = (p.isStarred ? '⭐ ' : '') + p.name;
                    selector.appendChild(option);

                    const saveDate = new Date(p.timestamp).toLocaleDateString('th-TH', { year: 'numeric', month: 'short', day: 'numeric', hour: '2-digit', minute: '2-digit' });
                    const starIconColor = p.isStarred ? 'text-yellow-400 hover:text-yellow-500' : 'text-gray-300 hover:text-yellow-400';
                    const starIconFill = p.isStarred ? 'currentColor' : 'none';
                    
                    gridHtml += `
                        <div class="bg-white p-5 rounded-2xl shadow-sm border ${p.isStarred ? 'border-yellow-300 bg-yellow-50/30' : 'border-gray-200'} hover:border-purple-400 hover:shadow-md transition cursor-pointer group flex flex-col justify-between h-full" onclick="switchProfile('${p.id}')">
                            <div>
                                <div class="flex justify-between items-start mb-3">
                                    <div class="flex items-center space-x-2">
                                        <div class="bg-purple-100 p-2 rounded-lg text-purple-600 group-hover:bg-purple-600 group-hover:text-white transition">
                                            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 19v-6a2 2 0 00-2-2H5a2 2 0 00-2 2v6a2 2 0 002 2h2a2 2 0 002-2zm0 0V9a2 2 0 012-2h2a2 2 0 012 2v10m-6 0a2 2 0 002 2h2a2 2 0 002-2m0 0V5a2 2 0 012-2h2a2 2 0 012 2v14a2 2 0 01-2 2h-2a2 2 0 01-2-2z"></path></svg>
                                        </div>
                                        <button onclick="event.stopPropagation(); toggleStarProfile('${p.id}')" class="${starIconColor} p-1 transition" title="ติดดาว/เอาดาวออก">
                                            <svg class="w-6 h-6" fill="${starIconFill}" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M11.049 2.927c.3-.921 1.603-.921 1.902 0l1.519 4.674a1 1 0 00.95.69h4.915c.969 0 1.371 1.24.588 1.81l-3.976 2.888a1 1 0 00-.363 1.118l1.518 4.674c.3.922-.755 1.688-1.538 1.118l-3.976-2.888a1 1 0 00-1.176 0l-3.976 2.888c-.783.57-1.838-.197-1.538-1.118l1.518-4.674a1 1 0 00-.363-1.118l-3.976-2.888c-.784-.57-.38-1.81.588-1.81h4.914a1 1 0 00.951-.69l1.519-4.674z"></path></svg>
                                        </button>
                                    </div>
                                    <button onclick="event.stopPropagation(); openDeleteModal('${p.id}')" class="text-gray-300 hover:text-rose-500 transition" title="ลบข้อมูลชุดนี้">
                                        <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"></path></svg>
                                    </button>
                                </div>
                                <h4 class="font-bold text-gray-900 text-lg group-hover:text-purple-700 transition line-clamp-2 leading-tight" title="${p.name}">${p.name}</h4>
                            </div>
                            <p class="text-xs text-gray-400 mt-4 flex items-center">
                                <svg class="w-3.5 h-3.5 mr-1" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z"></path></svg> 
                                เซฟเมื่อ: ${saveDate}
                            </p>
                        </div>
                    `;
                });
                profilesGrid.innerHTML = gridHtml;
            } else {
                selector.classList.add('hidden');
                deleteBtn.classList.add('hidden');
                profilesContainer.classList.add('hidden');
            }
        }

        function toggleStarProfile(id) {
            if (savedProfilesDB[id]) {
                savedProfilesDB[id].isStarred = !savedProfilesDB[id].isStarred;
                localStorage.setItem('loadProfileDB', JSON.stringify(savedProfilesDB));
                updateProfileDropdown();
                if (savedProfilesDB[id].isStarred) { showToast(`⭐ ติดดาวให้ข้อมูล "${savedProfilesDB[id].name}" แล้ว`, false); } 
                else { showToast(`เอาดาวออกจากข้อมูล "${savedProfilesDB[id].name}" แล้ว`, false); }
            }
        }

        function switchProfile(id) {
            if (!id || !savedProfilesDB[id]) { showHome(); return; }
            currentProfileId = id;
            document.getElementById('profileSelector').value = id;
            document.getElementById('deleteProfileBtn').classList.remove('hidden');
            document.getElementById('deleteProfileBtn').classList.add('flex');
            showToast(`เปิดข้อมูล: ${savedProfilesDB[id].name}`, false);
            processCSV(savedProfilesDB[id].csv); 
        }

        function openSaveModal() {
            const inputEl = document.getElementById('profileNameInput');
            const errorEl = document.getElementById('nameError');
            inputEl.value = ''; inputEl.classList.remove('border-rose-500', 'ring-rose-200');
            errorEl.textContent = 'กรุณาระบุชื่อข้อมูลก่อนบันทึก'; errorEl.classList.add('hidden');
            document.getElementById('saveModal').classList.remove('hidden'); inputEl.focus();
        }

        function closeSaveModal() {
            document.getElementById('saveModal').classList.add('hidden');
            document.getElementById('csvFileInput').value = ''; 
            document.getElementById('csvFileInputHome').value = ''; 
            pendingCSVText = "";
        }

        function confirmSaveProfile() {
            const inputEl = document.getElementById('profileNameInput');
            const nameInput = inputEl.value.trim();
            const errorEl = document.getElementById('nameError');
            
            if (!nameInput) {
                inputEl.classList.add('border-rose-500', 'ring-rose-200');
                errorEl.textContent = 'กรุณาระบุชื่อข้อมูลก่อนบันทึก'; errorEl.classList.remove('hidden');
                return;
            }

            const isDuplicate = Object.values(savedProfilesDB).some(profile => profile.name.toLowerCase() === nameInput.toLowerCase());
            if (isDuplicate) {
                inputEl.classList.add('border-rose-500', 'ring-rose-200');
                errorEl.textContent = 'ชื่อนี้มีอยู่แล้ว กรุณาเปลี่ยนชื่อใหม่'; errorEl.classList.remove('hidden');
                return;
            }
            
            inputEl.classList.remove('border-rose-500', 'ring-rose-200'); errorEl.classList.add('hidden');
            const newId = 'profile_' + Date.now();
            try {
                savedProfilesDB[newId] = { name: nameInput, timestamp: Date.now(), csv: pendingCSVText };
                localStorage.setItem('loadProfileDB', JSON.stringify(savedProfilesDB));
                closeSaveModal(); updateProfileDropdown(); switchProfile(newId); 
                showToast(`💾 บันทึกและเปิด "${nameInput}" เรียบร้อยแล้ว`);
            } catch (err) {
                showToast("พื้นที่จัดเก็บในเครื่องเต็ม! ระบบจะแสดงผลอย่างเดียวโดยไม่บันทึกถาวร", true);
                closeSaveModal(); processCSV(pendingCSVText);
            }
        }

        function openDeleteModal(id = null) {
            profileToDelete = id || currentProfileId;
            if (!profileToDelete || !savedProfilesDB[profileToDelete]) return;
            document.getElementById('deleteProfileName').textContent = `"${savedProfilesDB[profileToDelete].name}"`;
            document.getElementById('deleteModal').classList.remove('hidden');
        }

        function closeDeleteModal() { document.getElementById('deleteModal').classList.add('hidden'); profileToDelete = null; }

        function confirmDeleteProfile() {
            if (profileToDelete && savedProfilesDB[profileToDelete]) {
                const name = savedProfilesDB[profileToDelete].name;
                savedProfilesDB[profileToDelete].deletedAt = Date.now();
                trashDB[profileToDelete] = savedProfilesDB[profileToDelete];
                localStorage.setItem('loadProfileTrashDB', JSON.stringify(trashDB));
                delete savedProfilesDB[profileToDelete]; localStorage.setItem('loadProfileDB', JSON.stringify(savedProfilesDB)); 
                updateProfileDropdown(); closeDeleteModal(); showToast(`🗑️ ย้าย "${name}" ไปที่ถังขยะแล้ว`);
                if (currentProfileId === profileToDelete) { showHome(); }
                profileToDelete = null;
            }
        }

        function cleanupTrash() {
            const now = Date.now(); const THIRTY_DAYS = 30 * 24 * 60 * 60 * 1000; let changed = false;
            for (let id in trashDB) { if (now - trashDB[id].deletedAt > THIRTY_DAYS) { delete trashDB[id]; changed = true; } }
            if (changed) localStorage.setItem('loadProfileTrashDB', JSON.stringify(trashDB));
        }

        function openTrashModal() { renderTrashItems(); document.getElementById('trashModal').classList.remove('hidden'); }
        function closeTrashModal() { document.getElementById('trashModal').classList.add('hidden'); }

        function renderTrashItems() {
            const container = document.getElementById('trashContainer'); container.innerHTML = '';
            const trashIds = Object.keys(trashDB);
            if (trashIds.length === 0) { container.innerHTML = '<div class="text-center py-8 text-gray-500">ถังขยะว่างเปล่า</div>'; return; }
            const now = Date.now(); const THIRTY_DAYS = 30 * 24 * 60 * 60 * 1000;
            const sortedTrash = trashIds.map(id => ({id, ...trashDB[id]})).sort((a, b) => b.deletedAt - a.deletedAt);
            let html = '';
            sortedTrash.forEach(item => {
                const daysLeft = Math.ceil((THIRTY_DAYS - (now - item.deletedAt)) / (1000 * 60 * 60 * 24));
                const deleteDateStr = new Date(item.deletedAt).toLocaleDateString('th-TH', { year: 'numeric', month: 'short', day: 'numeric', hour: '2-digit', minute: '2-digit' });
                html += `
                    <div class="bg-gray-50 p-4 rounded-xl border border-gray-200 flex flex-col sm:flex-row justify-between items-start sm:items-center space-y-3 sm:space-y-0">
                        <div>
                            <h4 class="font-bold text-gray-900">${item.name}</h4>
                            <p class="text-xs text-gray-500 mt-1">ลบเมื่อ: ${deleteDateStr} <span class="text-rose-500 font-medium ml-2">(ลบถาวรใน ${daysLeft} วัน)</span></p>
                        </div>
                        <div class="flex space-x-2 w-full sm:w-auto mt-2 sm:mt-0">
                            <button onclick="restoreProfile('${item.id}')" class="flex-1 sm:flex-none px-3 py-1.5 bg-emerald-100 text-emerald-700 hover:bg-emerald-200 rounded-lg text-sm font-semibold transition">กู้คืน</button>
                            <button onclick="hardDeleteProfile('${item.id}')" class="flex-1 sm:flex-none px-3 py-1.5 bg-rose-100 text-rose-700 hover:bg-rose-200 rounded-lg text-sm font-semibold transition">ลบถาวร</button>
                        </div>
                    </div>
                `;
            });
            container.innerHTML = html;
        }

        function restoreProfile(id) {
            if (trashDB[id]) {
                const profile = trashDB[id]; const name = profile.name; delete profile.deletedAt; 
                savedProfilesDB[id] = profile; localStorage.setItem('loadProfileDB', JSON.stringify(savedProfilesDB));
                delete trashDB[id]; localStorage.setItem('loadProfileTrashDB', JSON.stringify(trashDB));
                updateProfileDropdown(); renderTrashItems(); showToast(`✅ กู้คืนข้อมูล "${name}" เรียบร้อยแล้ว`);
            }
        }

        function hardDeleteProfile(id) {
            if (trashDB[id]) {
                if(confirm(`คุณแน่ใจหรือไม่ที่จะลบ "${trashDB[id].name}" อย่างถาวร? (ไม่สามารถกู้คืนได้อีก)`)) {
                    delete trashDB[id]; localStorage.setItem('loadProfileTrashDB', JSON.stringify(trashDB));
                    renderTrashItems(); showToast(`🗑️ ลบข้อมูลถาวรแล้ว`);
                }
            }
        }

        fileInput.addEventListener('change', handleFileUpload);
        fileInputHome.addEventListener('change', handleFileUpload);
        document.getElementById('applyFilterBtn').addEventListener('click', applyFilters);
        document.getElementById('resetFilterBtn').addEventListener('click', resetFilters);

        function showToast(message, isError = false) {
            const toast = document.getElementById('toast');
            document.getElementById('toastMessage').textContent = message;
            toast.className = `hidden fixed top-20 right-4 text-white px-6 py-3 rounded-lg shadow-xl z-50 flex items-center space-x-2 transition-opacity duration-300 ${isError ? 'bg-gray-900' : 'bg-purple-600'}`;
            toast.style.display = 'flex'; setTimeout(() => toast.style.opacity = '1', 10);
            setTimeout(() => { toast.style.opacity = '0'; setTimeout(() => toast.style.display = 'none', 300); }, 3000);
        }

        function handleFileUpload(event) {
            const file = event.target.files[0]; if (!file) return;
            const reader = new FileReader();
            reader.onload = function(e) { pendingCSVText = e.target.result; openSaveModal(); };
            reader.onerror = function() { showToast("ไม่สามารถอ่านไฟล์ได้ โปรดลองอีกครั้ง", true); };
            reader.readAsText(file);
        }

        function processCSV(csvText) {
            const lines = csvText.split('\n'); let headerIndex = -1;
            for (let i = 0; i < lines.length; i++) {
                if (lines[i].includes("CH1") && lines[i].includes("CH2") && lines[i].includes("CH3") && lines[i].includes("No.")) { headerIndex = i; break; }
            }
            if (headerIndex === -1) { showToast("รูปแบบไฟล์ไม่ถูกต้อง ไม่พบหัวตารางที่ต้องการ", true); return; }
            const tableCSV = lines.slice(headerIndex).join('\n');

            Papa.parse(tableCSV, {
                header: true, skipEmptyLines: true,
                complete: function(results) {
                    masterData = [];
                    results.data.forEach(row => {
                        if (!row.CH1 || row.CH1.trim() === "") return;
                        try {
                            const dateStr = row.CH1.trim(); const parts = dateStr.split(' '); if(parts.length < 2) return;
                            const dateParts = parts[0].split('-'); const timeStr = parts[1];
                            const isoDateStr = `${dateParts[2]}-${dateParts[1]}-${dateParts[0]}T${timeStr}`;
                            const timestamp = new Date(isoDateStr); if (isNaN(timestamp.getTime())) return;
                            const status = (row.CH2 || "").trim();
                            let voltage = parseFloat((row.CH3 || "0").replace(/[^\d.-]/g, ''));
                            let powerKW = parseFloat((row.CH4 || "0").replace(/[^\d.-]/g, ''));
                            if (isNaN(voltage)) voltage = 0; if (isNaN(powerKW)) powerKW = 0;
                            const energyKWh = powerKW / 4;
                            let currentA = 0; if (voltage > 0) { currentA = (powerKW * 1000) / voltage; }
                            masterData.push({ timestamp: timestamp, status: status, voltage: voltage, power: powerKW, energy: energyKWh, current: currentA });
                        } catch (err) {}
                    });

                    if (masterData.length > 0) {
                        masterData.sort((a, b) => a.timestamp - b.timestamp);
                        setupDatePickers(); filteredData = [...masterData]; updateDashboard();
                        
                        // เมื่ออัปโหลดแล้วให้เด้งไปหน้า Dashboard เสมอ
                        switchTab('dashboard'); 
                        emptyState.style.display = 'none'; dashboardContent.style.display = 'block';
                        showToast(`วิเคราะห์ข้อมูลสำเร็จทั้งหมด ${masterData.length.toLocaleString()} รายการ`);
                    } else { showToast("ไฟล์นี้ไม่มีข้อมูลให้อ่าน", true); }
                }
            });
        }

        function setupDatePickers() {
            if (masterData.length === 0) return;
            const firstDate = masterData[0].timestamp; const lastDate = masterData[masterData.length - 1].timestamp;
            fpStart.set("minDate", firstDate); fpStart.set("maxDate", lastDate); fpStart.setDate(firstDate);
            fpEnd.set("minDate", firstDate); fpEnd.set("maxDate", lastDate); fpEnd.setDate(lastDate);
        }

        function applyFilters() {
            const startSelected = fpStart.selectedDates[0]; const endSelected = fpEnd.selectedDates[0];
            if (!startSelected || !endSelected) { showToast("กรุณาระบุช่วงเวลาให้ครบ", true); return; }
            const startTime = startSelected.getTime(); const endTime = endSelected.getTime();
            filteredData = masterData.filter(d => { const t = d.timestamp.getTime(); return t >= startTime && t <= endTime; });
            updateDashboard(); showToast(`พบข้อมูล ${filteredData.length.toLocaleString()} รายการในช่วงที่เลือก`);
        }

        function resetFilters() { setupDatePickers(); filteredData = [...masterData]; updateDashboard(); showToast("แสดงข้อมูลทั้งหมดแล้ว"); }

        function updateDashboard() {
            document.getElementById('dataCount').textContent = filteredData.length.toLocaleString();
            calculateAndRenderStats(); renderCharts(); renderTable();
        }

        function calculateAndRenderStats() {
            if (filteredData.length === 0) {
                document.getElementById('statTotalEnergy').textContent = "0.00"; document.getElementById('statMaxPower').textContent = "0.00";
                document.getElementById('statMaxCurrent').textContent = "0.00"; document.getElementById('statAvgVoltage').textContent = "0.00"; return;
            }
            let totalEnergy = 0; let maxPower = -1; let maxCurrent = -1; let totalVoltage = 0; let voltageCount = 0;
            peakPowerObj = null; peakCurrentObj = null;

            filteredData.forEach(d => {
                totalEnergy += d.energy;
                if (d.power > maxPower) { maxPower = d.power; peakPowerObj = d; }
                if (d.current > maxCurrent) { maxCurrent = d.current; peakCurrentObj = d; }
                if (d.voltage > 0) { totalVoltage += d.voltage; voltageCount++; }
            });

            const avgVoltage = voltageCount > 0 ? (totalVoltage / voltageCount) : 0;
            document.getElementById('statTotalEnergy').textContent = totalEnergy.toLocaleString('en-US', {minimumFractionDigits: 2, maximumFractionDigits: 2});
            document.getElementById('statMaxPower').textContent = maxPower > -1 ? maxPower.toLocaleString('en-US', {minimumFractionDigits: 2, maximumFractionDigits: 2}) : "0.00";
            document.getElementById('statMaxCurrent').textContent = maxCurrent > -1 ? maxCurrent.toLocaleString('en-US', {minimumFractionDigits: 2, maximumFractionDigits: 2}) : "0.00";
            document.getElementById('statAvgVoltage').textContent = avgVoltage.toLocaleString('en-US', {minimumFractionDigits: 2, maximumFractionDigits: 2});
            document.getElementById('maxPowerTime').classList.add('hidden'); document.getElementById('maxCurrentTime').classList.add('hidden');

            if (peakPowerObj) { document.getElementById('maxPowerTimeText').textContent = `⏱️ ${formatDate(peakPowerObj.timestamp)}`; }
            if (peakCurrentObj) { document.getElementById('maxCurrentTimeText').textContent = `⏱️ ${formatDate(peakCurrentObj.timestamp)}`; }
        }

        function togglePeakInfo(type) {
            const elId = type === 'power' ? 'maxPowerTime' : 'maxCurrentTime'; const el = document.getElementById(elId);
            if (el.classList.contains('hidden')) { el.classList.remove('hidden');
                if (type === 'power' && peakPowerObj) { showToast(`⚡ กำลังไฟฟ้าสูงสุด ${peakPowerObj.power.toFixed(3)} kW เกิดขึ้นเมื่อ: ${formatDate(peakPowerObj.timestamp)}`, false); } 
                else if (type === 'current' && peakCurrentObj) { showToast(`🔥 กระแสไฟฟ้าสูงสุด ${peakCurrentObj.current.toFixed(2)} A เกิดขึ้นเมื่อ: ${formatDate(peakCurrentObj.timestamp)}`, false); }
            } else { el.classList.add('hidden'); }
        }

        function resetAllZooms() { if (mainChartObj) mainChartObj.resetZoom(); }

        function renderTable() {
            const tbody = document.getElementById('dataTableBody'); tbody.innerHTML = ''; const displayData = filteredData.slice(0, 5000); 
            if (displayData.length === 0) { tbody.innerHTML = `<tr><td colspan="6" class="px-6 py-8 text-center text-gray-500 bg-white">ไม่พบข้อมูล</td></tr>`; return; }

            let rowsHtml = '';
            displayData.forEach((row, index) => {
                const bgClass = index % 2 === 0 ? 'bg-white' : 'bg-gray-50';
                let statusHtml = '<span class="text-gray-300">-</span>';
                if (row.status) { statusHtml = `<span class="bg-gray-200 text-gray-800 px-2.5 py-0.5 rounded text-xs font-bold">${row.status}</span>`; }
                rowsHtml += `
                    <tr class="border-b border-gray-100 ${bgClass} hover:bg-purple-50 transition-colors">
                        <td class="px-6 py-2.5 font-medium text-gray-900 whitespace-nowrap">${formatDate(row.timestamp)}</td>
                        <td class="px-6 py-2.5 text-center">${statusHtml}</td>
                        <td class="px-6 py-2.5 text-right font-mono text-gray-600">${row.voltage.toFixed(2)}</td>
                        <td class="px-6 py-2.5 text-right font-mono text-gray-600">${row.power.toFixed(3)}</td>
                        <td class="px-6 py-2.5 text-right font-mono text-purple-600 font-bold">${row.energy.toFixed(3)}</td>
                        <td class="px-6 py-2.5 text-right font-mono text-gray-900 font-bold">${row.current.toFixed(2)}</td>
                    </tr>
                `;
            });
            tbody.innerHTML = rowsHtml;
        }

        function exportToCSV() {
            if (filteredData.length === 0) { showToast("ไม่มีข้อมูลสำหรับดาวน์โหลด", true); return; }
            let csvContent = "\uFEFF"; csvContent += "วัน-เวลา,สถานะ,แรงดันไฟฟ้า (V),กำลังไฟฟ้า (kW),พลังงาน (kWh),กระแสไฟฟ้า (A)\n";
            filteredData.forEach(row => {
                csvContent += `${formatDate(row.timestamp)},${row.status ? `"${row.status}"` : "-"},${row.voltage.toFixed(2)},${row.power.toFixed(3)},${row.energy.toFixed(3)},${row.current.toFixed(2)}\n`;
            });
            const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' }); const url = URL.createObjectURL(blob);
            const link = document.createElement("a"); link.setAttribute("href", url);
            const now = new Date(); const dateStr = `${now.getFullYear()}${String(now.getMonth()+1).padStart(2,'0')}${String(now.getDate()).padStart(2,'0')}`;
            link.setAttribute("download", `LoadProfile_Export_${dateStr}.csv`);
            document.body.appendChild(link); link.click(); document.body.removeChild(link);
            showToast("ดาวน์โหลดไฟล์ข้อมูลสำเร็จ!");
        }

        function renderCharts() {
            const labels = filteredData.map(d => d.timestamp);
            const powerData = filteredData.map(d => d.power); const energyData = filteredData.map(d => d.energy);
            const voltageData = filteredData.map(d => d.voltage); const currentData = filteredData.map(d => d.current);

            const timeScaleOptions = {
                type: 'time', time: { tooltipFormat: 'DD/MM/YYYY HH:mm', displayFormats: { hour: 'DD MMM HH:mm', day: 'DD MMM' } },
                grid: { display: false }, border: { display: false }, ticks: { maxRotation: 0, autoSkip: true, maxTicksLimit: 6, color: '#6b7280', font: { size: 11 } }
            };

            const modernTooltip = {
                backgroundColor: 'rgba(17, 24, 39, 0.95)', titleColor: '#ffffff', bodyColor: '#e5e7eb',
                titleFont: { size: 13, weight: 'bold' }, bodyFont: { size: 12 }, padding: 12, cornerRadius: 8, boxPadding: 6, usePointStyle: true, borderColor: 'rgba(255, 255, 255, 0.15)', borderWidth: 1
            };

            const getCleanYScale = (title, position = 'left', extra = {}) => ({
                type: 'linear', display: true, position: position, title: { display: true, text: title, color: '#374151', font: { size: 12, weight: 'bold' } },
                grid: { color: '#f3f4f6', borderDash: [5, 5], drawBorder: false }, border: { display: false }, ticks: { color: '#6b7280', font: { size: 11 } }, ...extra
            });

            Chart.defaults.font.family = "'Segoe UI', Tahoma, Geneva, Verdana, sans-serif";
            if (mainChartObj) mainChartObj.destroy();
            const ctxMain = document.getElementById('mainChart').getContext('2d');
            mainChartObj = new Chart(ctxMain, {
                type: 'bar',
                data: {
                    labels: labels,
                    datasets: [
                        { label: 'กำลังไฟฟ้า (kW)', data: powerData, borderColor: '#2563eb', backgroundColor: 'rgba(37, 99, 235, 0.85)', hoverBackgroundColor: 'rgba(37, 99, 235, 1)', borderWidth: 1, yAxisID: 'y', borderRadius: 4, order: 1 },
                        { label: 'พลังงาน (kWh)', data: energyData, borderColor: '#8b5cf6', backgroundColor: 'rgba(139, 92, 246, 0.85)', hoverBackgroundColor: 'rgba(139, 92, 246, 1)', borderWidth: 1, yAxisID: 'y', borderRadius: 4, order: 2 },
                        { label: 'แรงดันไฟฟ้า (V)', data: voltageData, borderColor: '#f59e0b', backgroundColor: 'rgba(245, 158, 11, 0.85)', hoverBackgroundColor: 'rgba(245, 158, 11, 1)', borderWidth: 1, yAxisID: 'yVoltage', borderRadius: 4, order: 3 },
                        { label: 'กระแสไฟฟ้า (A)', data: currentData, borderColor: '#dc2626', backgroundColor: 'rgba(220, 38, 38, 0.85)', hoverBackgroundColor: 'rgba(220, 38, 38, 1)', borderWidth: 1, yAxisID: 'yCurrent', borderRadius: 4, order: 4 }
                    ]
                },
                options: {
                    animation: false, normalized: true, responsive: true, maintainAspectRatio: false, interaction: { mode: 'index', intersect: false },
                    plugins: { tooltip: modernTooltip, legend: { position: 'top', labels: { usePointStyle: true, boxWidth: 10, font: { size: 13, weight: 'bold' }, padding: 20 } }, zoom: { pan: { enabled: true, mode: 'x' }, zoom: { wheel: { enabled: true }, pinch: { enabled: true }, mode: 'x' } } },
                    scales: {
                        x: timeScaleOptions,
                        y: getCleanYScale('Power & Energy (kW / kWh)', 'left', { beginAtZero: true, title: {display:true, text:'kW / kWh', color: '#4b5563'}, ticks: { color: '#6b7280', font: { size: 11 } } }),
                        yVoltage: getCleanYScale('Voltage (V)', 'right', { suggestedMin: 215, suggestedMax: 235, grid: { display: false }, title: {display:true, text:'Voltage (V)', color: '#4b5563'}, ticks: { color: '#6b7280', font: { size: 11 } } }),
                        yCurrent: getCleanYScale('Current (A)', 'right', { beginAtZero: true, grid: { display: false }, title: {display:true, text:'Current (A)', color: '#4b5563'}, ticks: { color: '#6b7280', font: { size: 11 } } })
                    }
                }
            });
        }
    </script>
</body>
</html>
