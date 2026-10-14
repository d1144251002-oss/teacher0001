<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>教師控制台 - ClassPoll</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <!-- Firebase JS SDK (雲端即時同步) -->
    <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-database-compat.js"></script>
</head>
<body class="bg-slate-100 font-sans min-h-screen text-slate-800">

    <nav class="bg-indigo-600 text-white shadow-md">
        <div class="max-w-6xl mx-auto px-4 py-3 flex justify-between items-center">
            <div class="flex items-center space-x-2">
                <i class="fa-solid fa-chalkboard-user text-2xl"></i>
                <span class="font-bold text-xl tracking-wide">ClassPoll 教師控制台</span>
            </div>
            <div class="text-sm bg-indigo-700 px-3 py-1 rounded-full flex items-center gap-1">
                <span class="w-2 h-2 rounded-full bg-emerald-400 animate-pulse"></span> 雲端主控中
            </div>
        </div>
    </nav>

    <div class="max-w-4xl mx-auto p-4 md:p-6 space-y-6">

        <!-- 教室代碼切換控制 -->
        <div class="bg-indigo-900 text-white p-6 rounded-2xl shadow-md flex flex-wrap justify-between items-center gap-4">
            <div>
                <span class="text-indigo-300 text-sm font-medium">當前雲端教室代碼</span>
                <div class="flex items-center gap-2 mt-1">
                    <input type="text" id="teacherRoomCodeInput" value="888" class="bg-indigo-800 text-amber-300 font-mono font-black text-2xl px-3 py-1 rounded border border-indigo-600 w-32 uppercase text-center focus:outline-none">
                    <button onclick="switchRoom()" class="bg-amber-500 hover:bg-amber-600 text-slate-900 font-bold text-xs px-4 py-2 rounded shadow transition">
                        切換代碼
                    </button>
                </div>
            </div>
            <div class="flex items-center gap-3">
                <button onclick="generateBotAnswers()" class="bg-indigo-700 hover:bg-indigo-600 text-indigo-100 text-sm px-4 py-2 rounded-lg transition border border-indigo-500">
                    <i class="fa-solid fa-robot mr-1"></i> 模擬 5 位學生回答
                </button>
                <button onclick="clearCurrentAnswers()" class="bg-rose-600 hover:bg-rose-700 text-white text-sm px-4 py-2 rounded-lg transition">
                    <i class="fa-solid fa-trash mr-1"></i> 清除回答紀錄
                </button>
            </div>
        </div>

        <!-- 出題區域 -->
        <div class="bg-white p-6 rounded-2xl shadow-md">
            <h3 class="text-lg font-bold mb-4 text-slate-800 border-b pb-2">
                <i class="fa-solid fa-paper-plane text-indigo-600 mr-2"></i>發布新題目
            </h3>
            <div class="space-y-4">
                <div>
                    <label class="block text-sm font-medium text-slate-700 mb-1">題目內容</label>
                    <input type="text" id="questionInput" value="請問下列何者是台灣最高的山？" class="w-full px-4 py-2 border border-slate-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:outline-none">
                </div>

                <div class="grid grid-cols-1 md:grid-cols-2 gap-3">
                    <div>
                        <label class="block text-xs text-slate-500 mb-1">選項 A</label>
                        <input type="text" id="optA" value="玉山" class="w-full px-3 py-2 border rounded-lg text-sm">
                    </div>
                    <div>
                        <label class="block text-xs text-slate-500 mb-1">選項 B</label>
                        <input type="text" id="optB" value="雪山" class="w-full px-3 py-2 border rounded-lg text-sm">
                    </div>
                    <div>
                        <label class="block text-xs text-slate-500 mb-1">選項 C</label>
                        <input type="text" id="optC" value="陽明山" class="w-full px-3 py-2 border rounded-lg text-sm">
                    </div>
                    <div>
                        <label class="block text-xs text-slate-500 mb-1">選項 D</label>
                        <input type="text" id="optD" value="阿里山" class="w-full px-3 py-2 border rounded-lg text-sm">
                    </div>
                </div>

                <button onclick="publishQuestion()" class="w-full bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-3 rounded-lg shadow-md transition">
                    發布題目至雲端
                </button>
            </div>
        </div>

        <!-- 即時圖表 -->
        <div class="bg-white p-6 rounded-2xl shadow-md">
            <div class="flex justify-between items-center mb-4 border-b pb-2">
                <h3 class="text-lg font-bold text-slate-800">
                    <i class="fa-solid fa-chart-column text-indigo-600 mr-2"></i>即時統計圖表
                </h3>
                <span class="text-sm bg-slate-100 text-slate-600 px-3 py-1 rounded-full font-bold">
                    已回答人數: <span id="totalRespondedCount" class="text-indigo-600 text-base">0</span> 人
                </span>
            </div>

            <div class="w-full max-w-md mx-auto my-4">
                <canvas id="pollChart"></canvas>
            </div>

            <div class="mt-6">
                <h4 class="text-sm font-semibold text-slate-600 mb-2">學生提交明細（實時）：</h4>
                <div id="studentResponsesList" class="max-h-40 overflow-y-auto space-y-1 bg-slate-50 p-3 rounded-lg border text-xs text-slate-600">
                    <div class="text-slate-400 text-center">尚無回答數據</div>
                </div>
            </div>
        </div>

    </div>

    <script>
        // 免費雲端 Firebase 配置
        const firebaseConfig = {
            databaseURL: "https://quiz-app-demo-default-rtdb.firebaseio.com/"
        };
        firebase.initializeApp(firebaseConfig);
        const db = firebase.database();

        let currentRoom = '888';
        let chartInstance = null;
        let roomRef = null;
        let currentQuestion = null;

        function switchRoom() {
            const newRoom = document.getElementById('teacherRoomCodeInput').value.trim().toUpperCase();
            if (!newRoom) {
                alert('請輸入教室代碼！');
                return;
            }
            currentRoom = newRoom;
            listenToRoom(currentRoom);
            alert(`已切換至教室代碼: ${currentRoom}`);
        }

        function listenToRoom(roomCode) {
            if (roomRef) roomRef.off();
            roomRef = db.ref(`rooms/${roomCode}`);

            roomRef.on('value', (snapshot) => {
                const data = snapshot.val() || {};
                currentQuestion = data.activeQuestion || null;
                const answers = data.answers || {};
                updateDashboard(answers);
            });
        }

        function publishQuestion() {
            const questionText = document.getElementById('questionInput').value.trim();
            const options = [
                document.getElementById('optA').value.trim(),
                document.getElementById('optB').value.trim(),
                document.getElementById('optC').value.trim(),
                document.getElementById('optD').value.trim()
            ].filter(opt => opt !== '');

            if (!questionText || options.length === 0) {
                alert('請輸入題目與選項！');
                return;
            }

            const questionData = {
                id: Date.now(),
                question: questionText,
                options: options
            };

            // 推播至雲端並清空回答
            db.ref(`rooms/${currentRoom}`).set({
                activeQuestion: questionData,
                answers: {}
            });

            alert(`題目已成功發布至教室 [${currentRoom}]！`);
        }

        function updateDashboard(answers) {
            const respondedStudents = Object.keys(answers);
            document.getElementById('totalRespondedCount').innerText = respondedStudents.length;

            const listEl = document.getElementById('studentResponsesList');
            if (respondedStudents.length === 0) {
                listEl.innerHTML = `<div class="text-slate-400 text-center">尚無回答數據</div>`;
            } else {
                listEl.innerHTML = respondedStudents.map(s => 
                    `<div class="flex justify-between border-b pb-1">
                        <span class="font-medium">${s}</span>
                        <span class="text-indigo-600 font-bold">${answers[s]}</span>
                     </div>`
                ).join('');
            }

            if (currentQuestion && chartInstance) {
                const counts = currentQuestion.options.map(opt => {
                    return Object.values(answers).filter(ans => ans === opt).length;
                });

                chartInstance.data.labels = currentQuestion.options;
                chartInstance.data.datasets[0].data = counts;
                chartInstance.update();
            }
        }

        function initChart() {
            const ctx = document.getElementById('pollChart').getContext('2d');
            chartInstance = new Chart(ctx, {
                type: 'bar',
                data: {
                    labels: ['選項 A', '選項 B', '選項 C', '選項 D'],
                    datasets: [{
                        label: '得票數',
                        data: [0, 0, 0, 0],
                        backgroundColor: 'rgba(99, 102, 241, 0.75)',
                        borderColor: 'rgba(99, 102, 241, 1)',
                        borderWidth: 1.5,
                        borderRadius: 6
                    }]
                },
                options: {
                    responsive: true,
                    scales: { y: { beginAtZero: true, ticks: { stepSize: 1 } } }
                }
            });
        }

        function generateBotAnswers() {
            if (!currentQuestion) {
                alert('請先發布題目再產生模擬學生！');
                return;
            }

            const botNames = ['小明', '小華', '阿強', '美美', '小東', '莉莉', '大衛'];
            for (let i = 0; i < 5; i++) {
                const randomName = botNames[Math.floor(Math.random() * botNames.length)] + '_' + Math.floor(Math.random() * 90 + 10);
                const randomOpt = currentQuestion.options[Math.floor(Math.random() * currentQuestion.options.length)];
                db.ref(`rooms/${currentRoom}/answers/${randomName}`).set(randomOpt);
            }
        }

        function clearCurrentAnswers() {
            if (confirm('
