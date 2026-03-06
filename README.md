
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>แดชบอร์ดผลการประเมิน</title>
    <!-- Tailwind CSS สำหรับตกแต่ง UI -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- PapaParse สำหรับอ่านไฟล์ CSV จาก Google Sheet -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.4.1/papaparse.min.js"></script>
    <!-- Chart.js สำหรับสร้างกราฟ -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <!-- Font Awesome สำหรับไอคอน -->
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <!-- Google Fonts: Sarabun -->
    <link href="https://fonts.googleapis.com/css2?family=Sarabun:wght@300;400;500;600;700&display=swap" rel="stylesheet">
    
    <style>
        body {
            font-family: 'Sarabun', sans-serif;
            background-color: #f3f4f6;
        }
        /* Custom scrollbar for table */
        .table-container::-webkit-scrollbar {
            height: 8px;
            width: 8px;
        }
        .table-container::-webkit-scrollbar-track {
            background: #f1f1f1; 
            border-radius: 4px;
        }
        .table-container::-webkit-scrollbar-thumb {
            background: #cbd5e1; 
            border-radius: 4px;
        }
        .table-container::-webkit-scrollbar-thumb:hover {
            background: #94a3b8; 
        }
    </style>
</head>
<body class="text-gray-800 antialiased">

    <!-- Navbar -->
    <nav class="bg-blue-700 text-white shadow-lg">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
            <div class="flex items-center justify-between h-16">
                <div class="flex items-center">
                    <i class="fas fa-chart-line text-2xl mr-3"></i>
                    <span class="font-bold text-xl">แดชบอร์ดสรุปผลการประเมิน</span>
                </div>
                <div>
                    <a href="https://docs.google.com/spreadsheets/d/1ot6yKRcdkuSmwmtiu_-Rj2sCxwIQ5aKwxmJblI_z0BY/edit?usp=sharing" target="_blank" class="text-blue-200 hover:text-white text-sm transition-colors flex items-center">
                        <i class="fas fa-external-link-alt mr-2"></i> เปิด Google Sheet
                    </a>
                </div>
            </div>
        </div>
    </nav>

    <!-- Main Content -->
    <main class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8">
        
        <!-- Loading State -->
        <div id="loadingState" class="flex flex-col items-center justify-center py-20">
            <div class="animate-spin rounded-full h-16 w-16 border-t-4 border-b-4 border-blue-600 mb-4"></div>
            <p class="text-lg text-gray-600 font-medium">กำลังโหลดและคำนวณข้อมูลจาก Google Sheet...</p>
        </div>

        <!-- Error State -->
        <div id="errorState" class="hidden bg-red-100 border-l-4 border-red-500 text-red-700 p-4 mb-6 rounded shadow-sm" role="alert">
            <p class="font-bold">เกิดข้อผิดพลาดในการดึงข้อมูล</p>
            <p id="errorMessage" class="text-sm mt-1">โปรดตรวจสอบว่าลิงก์ถูกต้อง และเปิดสิทธิ์การแชร์เป็น "ทุกคนที่มีลิงก์" (Anyone with the link) แล้ว</p>
        </div>

        <!-- Dashboard Content (Hidden until loaded) -->
        <div id="dashboardContent" class="hidden space-y-6">
            
            <!-- Stats Row -->
            <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
                <!-- Stat Card 1 -->
                <div class="bg-white rounded-xl shadow-sm p-6 border border-gray-100 flex items-center">
                    <div class="p-4 rounded-full bg-blue-100 text-blue-600 mr-4">
                        <i class="fas fa-users text-xl"></i>
                    </div>
                    <div>
                        <p class="text-sm text-gray-500 font-medium">ผู้ตอบประเมิน</p>
                        <p id="totalResponses" class="text-2xl font-bold text-gray-800">0</p>
                    </div>
                </div>
                <!-- Stat Card 2 -->
                <div class="bg-white rounded-xl shadow-sm p-6 border border-gray-100 flex items-center">
                    <div class="p-4 rounded-full bg-yellow-100 text-yellow-600 mr-4">
                        <i class="fas fa-star text-xl"></i>
                    </div>
                    <div>
                        <p class="text-sm text-gray-500 font-medium">ค่าเฉลี่ยรวมทั้งหมด</p>
                        <p id="overallMean" class="text-2xl font-bold text-gray-800">-</p>
                    </div>
                </div>
                <!-- Stat Card 3 -->
                <div class="bg-white rounded-xl shadow-sm p-6 border border-gray-100 flex items-center">
                    <div class="p-4 rounded-full bg-green-100 text-green-600 mr-4">
                        <i class="fas fa-clipboard-list text-xl"></i>
                    </div>
                    <div>
                        <p class="text-sm text-gray-500 font-medium">หัวข้อที่ประเมิน</p>
                        <p id="totalColumns" class="text-2xl font-bold text-gray-800">0</p>
                    </div>
                </div>
                <!-- Stat Card 4 -->
                <div class="bg-white rounded-xl shadow-sm p-6 border border-gray-100 flex items-center">
                    <div class="p-4 rounded-full bg-purple-100 text-purple-600 mr-4">
                        <i class="fas fa-clock text-xl"></i>
                    </div>
                    <div>
                        <p class="text-sm text-gray-500 font-medium">อัปเดตล่าสุด</p>
                        <p id="lastUpdate" class="text-lg font-bold text-gray-800">-</p>
                    </div>
                </div>
            </div>

            <!-- Statistical Summary Section -->
            <div class="bg-white rounded-xl shadow-sm border border-gray-100 overflow-hidden">
                <div class="px-6 py-4 border-b border-gray-200 bg-blue-50">
                    <h2 class="text-lg font-bold text-blue-800 flex items-center">
                        <i class="fas fa-calculator mr-2"></i> สรุปผลสถิติการประเมิน (ค่าเฉลี่ย และ S.D.)
                    </h2>
                </div>
                <div class="table-container overflow-x-auto">
                    <table class="min-w-full divide-y divide-gray-200" id="statsTable">
                        <thead class="bg-gray-100">
                            <tr>
                                <th class="px-6 py-3 text-left text-xs font-bold text-gray-600 uppercase tracking-wider">หัวข้อการประเมิน</th>
                                <th class="px-6 py-3 text-center text-xs font-bold text-gray-600 uppercase tracking-wider">N (คน)</th>
                                <th class="px-6 py-3 text-center text-xs font-bold text-gray-600 uppercase tracking-wider">ค่าเฉลี่ย (Mean)</th>
                                <th class="px-6 py-3 text-center text-xs font-bold text-gray-600 uppercase tracking-wider">ส่วนเบี่ยงเบนมาตรฐาน (S.D.)</th>
                                <th class="px-6 py-3 text-center text-xs font-bold text-gray-600 uppercase tracking-wider">การแปลผล</th>
                            </tr>
                        </thead>
                        <tbody class="bg-white divide-y divide-gray-200" id="statsBody">
                            <!-- Stats Rows will be populated via JS -->
                        </tbody>
                    </table>
                    <div id="noStatsMessage" class="hidden text-center py-8 text-gray-500">
                        ไม่พบข้อมูลที่เป็นตัวเลขหรือระดับคะแนน (1-5) สำหรับคำนวณสถิติ
                    </div>
                </div>
            </div>

            <!-- Charts Section -->
            <div class="bg-white rounded-xl shadow-sm border border-gray-100 p-6">
                <div class="flex flex-col sm:flex-row justify-between items-center mb-6">
                    <h2 class="text-lg font-bold text-gray-800 flex items-center">
                        <i class="fas fa-chart-pie mr-2 text-blue-600"></i> วิเคราะห์สัดส่วนข้อมูล
                    </h2>
                    <div class="mt-4 sm:mt-0 w-full sm:w-1/3">
                        <label for="columnSelect" class="block text-sm font-medium text-gray-700 mb-1">เลือกหัวข้อเพื่อดูกราฟ:</label>
                        <select id="columnSelect" class="mt-1 block w-full pl-3 pr-10 py-2 text-base border-gray-300 focus:outline-none focus:ring-blue-500 focus:border-blue-500 sm:text-sm rounded-md bg-gray-50 border">
                            <!-- Options will be populated via JS -->
                        </select>
                    </div>
                </div>

                <div class="grid grid-cols-1 lg:grid-cols-2 gap-8">
                    <!-- Bar Chart -->
                    <div class="relative h-80 w-full flex justify-center items-center">
                        <canvas id="barChart"></canvas>
                    </div>
                    <!-- Doughnut Chart -->
                    <div class="relative h-80 w-full flex justify-center items-center">
                        <canvas id="doughnutChart"></canvas>
                    </div>
                </div>
            </div>

            <!-- Data Table Section -->
            <div class="bg-white rounded-xl shadow-sm border border-gray-100 overflow-hidden mt-6">
                <div class="px-6 py-4 border-b border-gray-200 bg-gray-50">
                    <h2 class="text-lg font-bold text-gray-800 flex items-center">
                        <i class="fas fa-table mr-2 text-gray-600"></i> ข้อมูลดิบทั้งหมด (Raw Data)
                    </h2>
                </div>
                <div class="table-container overflow-x-auto max-h-[400px]">
                    <table class="min-w-full divide-y divide-gray-200" id="dataTable">
                        <thead class="bg-gray-100 sticky top-0 z-10">
                            <tr id="tableHead">
                                <!-- Headers will be populated via JS -->
                            </tr>
                        </thead>
                        <tbody class="bg-white divide-y divide-gray-200" id="tableBody">
                            <!-- Rows will be populated via JS -->
                        </tbody>
                    </table>
                </div>
            </div>
            
        </div>
    </main>

    <footer class="text-center text-gray-500 text-sm pb-8 pt-4">
        สร้างด้วยระบบอัตโนมัติ • อิงสูตรคำนวณสถิติพื้นฐานการวิจัย • ดึงข้อมูลสดจาก Google Sheets
    </footer>

    <!-- JavaScript Logic -->
    <script>
        // กำหนด Sheet ID จากลิงก์
        const sheetId = '1ot6yKRcdkuSmwmtiu_-Rj2sCxwIQ5aKwxmJblI_z0BY';
        // URL สำหรับโหลดข้อมูลรูปแบบ CSV
        const csvUrl = `https://docs.google.com/spreadsheets/d/${sheetId}/gviz/tq?tqx=out:csv`;

        // ตัวแปรเก็บข้อมูล
        let globalData = [];
        let globalHeaders = [];
        let calculatedStats = [];
        
        // ตัวแปรเก็บ Instance ของ Chart
        let barChartInstance = null;
        let doughnutChartInstance = null;

        // สีสำหรับกราฟ
        const chartColors = [
            'rgba(59, 130, 246, 0.8)', // blue
            'rgba(16, 185, 129, 0.8)', // green
            'rgba(245, 158, 11, 0.8)', // yellow
            'rgba(239, 68, 68, 0.8)',  // red
            'rgba(139, 92, 246, 0.8)', // purple
            'rgba(236, 72, 153, 0.8)', // pink
            'rgba(20, 184, 166, 0.8)', // teal
            'rgba(249, 115, 22, 0.8)', // orange
        ];

        // ฟังก์ชันเริ่มต้นตอนโหลดหน้า
        document.addEventListener('DOMContentLoaded', () => {
            fetchData();

            // จับ Event เมื่อเปลี่ยน Dropdown
            document.getElementById('columnSelect').addEventListener('change', function(e) {
                updateCharts(e.target.value);
            });
        });

        // ดึงข้อมูลด้วย PapaParse
        function fetchData() {
            Papa.parse(csvUrl, {
                download: true,
                header: true,
                skipEmptyLines: true,
                complete: function(results) {
                    if (results.errors && results.errors.length > 0 && results.data.length === 0) {
                        showError("ไม่สามารถอ่านรูปแบบไฟล์ได้");
                        return;
                    }

                    // กรองข้อมูลเฉพาะแถวที่มีข้อมูลจริงๆ
                    globalData = results.data.filter(row => {
                        return Object.values(row).some(val => val !== null && val !== '');
                    });

                    globalHeaders = results.meta.fields;

                    if (globalData.length > 0) {
                        processData();
                        renderDashboard();
                    } else {
                        showError("พบไฟล์แต่ไม่มีข้อมูลในตาราง หรือไม่ได้เปิดสิทธิ์การแชร์แบบสาธารณะ");
                    }
                },
                error: function(error) {
                    showError("ไม่สามารถเชื่อมต่อกับ Google Sheet ได้ โปรดตรวจสอบการตั้งค่าแชร์ลิงก์");
                    console.error("PapaParse Error:", error);
                }
            });
        }

        // แสดงข้อผิดพลาด
        function showError(message) {
            document.getElementById('loadingState').classList.add('hidden');
            document.getElementById('errorState').classList.remove('hidden');
            if (message) {
                document.getElementById('errorMessage').innerText = message;
            }
        }

        // ฟังก์ชันช่วยแปลงข้อความเป็นตัวเลข (รองรับ Likert Scale ภาษาไทย)
        function parseToNumber(val) {
            if (val === null || val === undefined || val === '') return null;
            
            // ถ้าเป็นตัวเลขอยู่แล้ว
            if (!isNaN(val) && val.trim() !== '') return Number(val);
            
            // ถ้าขึ้นต้นด้วยตัวเลข เช่น "5 - ดีมาก" หรือ "5. มากที่สุด"
            const match = String(val).match(/^(\d+(\.\d+)?)/);
            if (match) return Number(match[1]);

            // แปลงข้อความระดับความพึงพอใจเป็นคะแนน
            const textVal = String(val).trim().toLowerCase();
            const likertMap = {
                'มากที่สุด': 5, 'ดีมาก': 5, 'เห็นด้วยอย่างยิ่ง': 5, 'ยอดเยี่ยม': 5,
                'มาก': 4, 'ดี': 4, 'เห็นด้วย': 4,
                'ปานกลาง': 3, 'พอใช้': 3, 'เฉยๆ': 3, 'เห็นด้วยปานกลาง': 3,
                'น้อย': 2, 'ต้องปรับปรุง': 2, 'ไม่เห็นด้วย': 2,
                'น้อยที่สุด': 1, 'แย่': 1, 'ไม่เห็นด้วยอย่างยิ่ง': 1, 'ควรปรับปรุงอย่างยิ่ง': 1
            };
            
            // ค้นหาคำที่ตรงกันบางส่วนก็ได้
            for (const [key, num] of Object.entries(likertMap)) {
                if (textVal.includes(key)) return num;
            }

            return null; // ถ้าแปลงไม่ได้ ให้ข้ามไป
        }

        // เกณฑ์การแปลผล (ช่วงคะแนน)
        function interpretMean(mean) {
            if (mean >= 4.50) return { text: "มากที่สุด / ดีเยี่ยม", color: "text-green-600 bg-green-100" };
            if (mean >= 3.50) return { text: "มาก / ดี", color: "text-blue-600 bg-blue-100" };
            if (mean >= 2.50) return { text: "ปานกลาง / พอใช้", color: "text-yellow-600 bg-yellow-100" };
            if (mean >= 1.50) return { text: "น้อย / ควรปรับปรุง", color: "text-orange-600 bg-orange-100" };
            return { text: "น้อยที่สุด / ต้องปรับปรุงด่วน", color: "text-red-600 bg-red-100" };
        }

        // ประมวลผลและคำนวณสถิติ
        function processData() {
            calculatedStats = [];
            
            globalHeaders.forEach(header => {
                // ข้ามคอลัมน์ที่เป็นประทับเวลา หรือข้อมูลทั่วไปที่ไม่ควรเอามาหาค่าเฉลี่ย
                const skipKeywords = ['timestamp', 'ประทับเวลา', 'เวลา', 'ชื่อ', 'นามสกุล', 'เพศ', 'อายุ', 'อีเมล', 'เบอร์โทร', 'ข้อเสนอแนะ'];
                if (skipKeywords.some(kw => header.toLowerCase().includes(kw))) return;

                const numericValues = [];
                globalData.forEach(row => {
                    const num = parseToNumber(row[header]);
                    if (num !== null) numericValues.push(num);
                });

                // ตรวจสอบว่าคอลัมน์นี้เป็นข้อมูลที่ประเมินได้หรือไม่ (ต้องมีข้อมูลตัวเลขอย่างน้อย 30% ของผู้ตอบทั้งหมด)
                if (numericValues.length > 0 && numericValues.length >= globalData.length * 0.3) {
                    const n = numericValues.length;
                    const sum = numericValues.reduce((a, b) => a + b, 0);
                    const mean = sum / n;
                    
                    // คำนวณ Standard Deviation (S.D.) - แบบกลุ่มตัวอย่าง (Sample)
                    let variance = 0;
                    if (n > 1) {
                        variance = numericValues.reduce((a, b) => a + Math.pow(b - mean, 2), 0) / (n - 1);
                    }
                    const sd = Math.sqrt(variance);

                    calculatedStats.push({
                        header: header,
                        mean: mean,
                        sd: sd,
                        n: n
                    });
                }
            });
        }

        // แสดงแดชบอร์ด
        function renderDashboard() {
            // ซ่อน Loading, แสดง Dashboard
            document.getElementById('loadingState').classList.add('hidden');
            document.getElementById('dashboardContent').classList.remove('hidden');

            // 1. อัปเดตการ์ดสรุปผล
            document.getElementById('totalResponses').innerText = globalData.length.toLocaleString();
            
            // คำนวณค่าเฉลี่ยรวมทั้งหมด (Overall Mean)
            if (calculatedStats.length > 0) {
                const totalMeanSum = calculatedStats.reduce((sum, stat) => sum + stat.mean, 0);
                const overallMean = totalMeanSum / calculatedStats.length;
                document.getElementById('overallMean').innerText = overallMean.toFixed(2);
                document.getElementById('totalColumns').innerText = calculatedStats.length.toLocaleString(); // จำนวนข้อที่ประเมิน
            } else {
                document.getElementById('overallMean').innerText = "N/A";
                document.getElementById('totalColumns').innerText = "0";
            }
            
            // หาวันที่อัปเดตล่าสุด
            let timeCol = globalHeaders.find(h => h.toLowerCase().includes('timestamp') || h.includes('ประทับเวลา') || h.includes('เวลา'));
            if (timeCol && globalData.length > 0) {
                const lastRow = globalData[globalData.length - 1];
                document.getElementById('lastUpdate').innerText = lastRow[timeCol] ? lastRow[timeCol].split(' ')[0] : '-'; // เอาแค่วันที่
            } else {
                const now = new Date();
                document.getElementById('lastUpdate').innerText = now.toLocaleDateString('th-TH');
            }

            // 2. แสดงตารางสถิติ
            const statsBody = document.getElementById('statsBody');
            statsBody.innerHTML = '';
            
            if (calculatedStats.length > 0) {
                document.getElementById('noStatsMessage').classList.add('hidden');
                document.getElementById('statsTable').classList.remove('hidden');
                
                calculatedStats.forEach((stat) => {
                    const tr = document.createElement('tr');
                    tr.className = 'hover:bg-blue-50 transition-colors';
                    
                    const interpretation = interpretMean(stat.mean);

                    tr.innerHTML = `
                        <td class="px-6 py-4 text-sm text-gray-800 font-medium">${stat.header}</td>
                        <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-600 text-center">${stat.n}</td>
                        <td class="px-6 py-4 whitespace-nowrap text-sm text-blue-600 font-bold text-center">${stat.mean.toFixed(2)}</td>
                        <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-600 text-center">${stat.sd.toFixed(2)}</td>
                        <td class="px-6 py-4 whitespace-nowrap text-center">
                            <span class="px-3 py-1 inline-flex text-xs leading-5 font-semibold rounded-full ${interpretation.color}">
                                ${interpretation.text}
                            </span>
                        </td>
                    `;
                    statsBody.appendChild(tr);
                });
            } else {
                document.getElementById('statsTable').classList.add('hidden');
                document.getElementById('noStatsMessage').classList.remove('hidden');
            }

            // 3. สร้างตัวเลือก Dropdown สำหรับกราฟ
            const select = document.getElementById('columnSelect');
            select.innerHTML = ''; 
            
            let defaultSelected = false;

            globalHeaders.forEach(header => {
                if (header.toLowerCase().includes('timestamp') || header.includes('ประทับเวลา')) return;
                
                const option = document.createElement('option');
                option.value = header;
                option.textContent = header;
                select.appendChild(option);

                if (!defaultSelected) {
                    option.selected = true;
                    defaultSelected = true;
                }
            });

            // สร้างกราฟครั้งแรก
            if (select.value) {
                updateCharts(select.value);
            }

            // 4. สร้างตารางข้อมูลดิบ
            renderTable();
        }

        // อัปเดตกราฟ
        function updateCharts(columnName) {
            // นับความถี่ของข้อมูลในคอลัมน์ที่เลือก
            const counts = {};
            globalData.forEach(row => {
                let val = row[columnName];
                if (val === null || val === undefined || val === '') val = 'ไม่ระบุ';
                
                if (counts[val]) {
                    counts[val]++;
                } else {
                    counts[val] = 1;
                }
            });

            const sortedKeys = Object.keys(counts).sort((a, b) => counts[b] - counts[a]);
            const labels = sortedKeys;
            const data = sortedKeys.map(k => counts[k]);

            // ลบกราฟเก่าถ้ามี
            if (barChartInstance) barChartInstance.destroy();
            if (doughnutChartInstance) doughnutChartInstance.destroy();

            const bgColors = labels.map((_, i) => chartColors[i % chartColors.length]);

            // สร้าง Bar Chart
            const ctxBar = document.getElementById('barChart').getContext('2d');
            barChartInstance = new Chart(ctxBar, {
                type: 'bar',
                data: {
                    labels: labels,
                    datasets: [{
                        label: 'จำนวน (คน/รายการ)',
                        data: data,
                        backgroundColor: bgColors,
                        borderWidth: 1,
                        borderRadius: 4
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: {
                        legend: { display: false },
                        title: {
                            display: true,
                            text: 'กราฟแท่งแสดงจำนวน',
                            font: { family: 'Sarabun', size: 16 }
                        }
                    },
                    scales: {
                        y: { beginAtZero: true, ticks: { precision: 0 } }
                    }
                }
            });

            // สร้าง Doughnut Chart
            const ctxDoughnut = document.getElementById('doughnutChart').getContext('2d');
            doughnutChartInstance = new Chart(ctxDoughnut, {
                type: 'doughnut',
                data: {
                    labels: labels,
                    datasets: [{
                        data: data,
                        backgroundColor: bgColors,
                        borderWidth: 0,
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: {
                        legend: {
                            position: 'bottom',
                            labels: { font: { family: 'Sarabun' }, padding: 20 }
                        },
                        title: {
                            display: true,
                            text: 'สัดส่วนข้อมูล',
                            font: { family: 'Sarabun', size: 16 }
                        }
                    }
                }
            });
        }

        // สร้างตารางข้อมูลดิบ
        function renderTable() {
            const thead = document.getElementById('tableHead');
            const tbody = document.getElementById('tableBody');
            
            thead.innerHTML = '';
            tbody.innerHTML = '';

            const thNo = document.createElement('th');
            thNo.className = 'px-6 py-3 text-left text-xs font-bold text-gray-600 uppercase tracking-wider whitespace-nowrap bg-gray-100';
            thNo.textContent = 'ลำดับ';
            thead.appendChild(thNo);

            globalHeaders.forEach(header => {
                const th = document.createElement('th');
                th.className = 'px-6 py-3 text-left text-xs font-bold text-gray-600 uppercase tracking-wider whitespace-nowrap bg-gray-100';
                th.textContent = header;
                thead.appendChild(th);
            });

            globalData.forEach((row, index) => {
                const tr = document.createElement('tr');
                tr.className = index % 2 === 0 ? 'bg-white hover:bg-gray-50' : 'bg-gray-50 hover:bg-gray-100';

                const tdNo = document.createElement('td');
                tdNo.className = 'px-6 py-4 whitespace-nowrap text-sm text-gray-500 font-medium';
                tdNo.textContent = index + 1;
                tr.appendChild(tdNo);

                globalHeaders.forEach(header => {
                    const td = document.createElement('td');
                    td.className = 'px-6 py-4 text-sm text-gray-700 min-w-[150px]';
                    td.textContent = row[header] !== null && row[header] !== undefined ? row[header] : '-';
                    tr.appendChild(td);
                });

                tbody.appendChild(tr);
            });
        }
    </script>
</body>
</html>
