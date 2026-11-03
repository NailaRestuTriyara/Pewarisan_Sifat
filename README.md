<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Game Microteaching Kelas</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Nunito:wght@400;700;900&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Nunito', sans-serif;
            margin: 0;
            overflow: hidden;
            background-color: #000;
            user-select: none;
        }

        /* Video Background */
        #bg-video {
            position: absolute;
            top: 0;
            left: 0;
            width: 100vw;
            height: 100vh;
            object-fit: cover;
            transform: scaleX(-1); /* Efek Cermin */
            z-index: 0;
        }

        /* UI Overlays */
        #ui-layer {
            position: absolute;
            inset: 0;
            z-index: 10;
            display: flex;
            pointer-events: none; /* Klik tembus ke elemen yang interaktif */
        }

        .interactive {
            pointer-events: auto;
        }

        /* Transisi halus untuk zona jawaban */
        .zone {
            transition: all 0.5s ease-in-out;
        }

        .glass-panel {
            background: rgba(15, 23, 42, 0.75);
            backdrop-filter: blur(12px);
            -webkit-backdrop-filter: blur(12px);
            border: 1px solid rgba(255, 255, 255, 0.2);
            box-shadow: 0 10px 40px rgba(0, 0, 0, 0.5);
        }

        .text-shadow {
            text-shadow: 0 4px 12px rgba(0,0,0,0.9);
        }

        ::-webkit-scrollbar { width: 8px; }
        ::-webkit-scrollbar-track { background: rgba(0,0,0,0.2); border-radius: 10px; }
        ::-webkit-scrollbar-thumb { background: rgba(255,255,255,0.4); border-radius: 10px; }
    </style>
</head>
<body class="text-white">

    <!-- Kamera Background -->
    <video id="bg-video" autoplay playsinline muted></video>

    <!-- Layer Permainan -->
    <div id="ui-layer">
        
        <!-- Zona KIRI (BENAR) -->
        <div id="zone-benar" class="zone w-1/2 h-full bg-green-500/20 flex flex-col items-center justify-center relative border-r-2 border-white/20">
            <h2 class="text-5xl md:text-7xl font-black text-green-300 text-shadow mb-4 tracking-wider uppercase">BENAR</h2>
            <!-- Ikon Centang (Sembunyi default) -->
            <div id="icon-benar" class="hidden text-8xl md:text-[10rem] text-green-400 drop-shadow-[0_0_20px_rgba(74,222,128,0.8)] transition-transform scale-0 duration-500">
                ✔️
            </div>
        </div>

        <!-- Zona KANAN (SALAH) -->
        <div id="zone-salah" class="zone w-1/2 h-full bg-red-500/20 flex flex-col items-center justify-center relative">
            <h2 class="text-5xl md:text-7xl font-black text-red-300 text-shadow mb-4 tracking-wider uppercase">SALAH</h2>
            <!-- Ikon Silang (Sembunyi default) -->
            <div id="icon-salah" class="hidden text-8xl md:text-[10rem] text-red-400 drop-shadow-[0_0_20px_rgba(248,113,113,0.8)] transition-transform scale-0 duration-500">
                ❌
            </div>
        </div>

    </div>

    <!-- UI Mengambang (Soal & Kontrol Guru) -->
    <div class="absolute inset-0 z-20 pointer-events-none flex flex-col justify-between p-6 md:p-10">
        
        <!-- HEADER / SOAL (Atas Tengah) -->
        <div class="w-full flex justify-center mt-4">
            <div class="interactive glass-panel rounded-3xl p-6 md:p-8 max-w-4xl w-full text-center border-t-4 border-blue-400 transform transition-all">
                <div class="text-blue-300 font-bold mb-2 tracking-widest text-sm md:text-base uppercase" id="q-counter">
                    Soal 1 / 7
                </div>
                <h1 id="question-text" class="text-2xl md:text-4xl lg:text-5xl font-extrabold leading-tight text-white drop-shadow-md">
                    Memuat pertanyaan...
                </h1>
            </div>
        </div>

        <!-- KONTROL GURU (Bawah Tengah) -->
        <div class="w-full flex justify-center mb-4">
            <div class="interactive glass-panel rounded-full p-3 md:p-4 flex gap-4 md:gap-6 items-center border border-white/10 shadow-2xl">
                
                <button onclick="openEditModal()" class="p-3 md:p-4 rounded-full bg-white/10 hover:bg-white/20 transition text-gray-200 hover:text-white" title="Edit Soal">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z" />
                    </svg>
                </button>

                <div class="w-px h-10 bg-white/20"></div>

                <button id="btn-reveal" onclick="revealAnswer()" class="bg-gradient-to-r from-yellow-400 to-yellow-600 hover:from-yellow-500 hover:to-yellow-700 text-black font-black text-lg md:text-xl py-3 px-8 md:px-12 rounded-full shadow-[0_0_15px_rgba(250,204,21,0.6)] transition transform hover:scale-105 active:scale-95">
                    TAMPILKAN JAWABAN
                </button>

                <button id="btn-next" onclick="nextQuestion()" class="hidden bg-gradient-to-r from-blue-500 to-indigo-600 hover:from-blue-600 hover:to-indigo-700 text-white font-black text-lg md:text-xl py-3 px-8 md:px-12 rounded-full shadow-[0_0_15px_rgba(59,130,246,0.6)] transition transform hover:scale-105 active:scale-95">
                    SOAL SELANJUTNYA ➔
                </button>

            </div>
        </div>
    </div>

    <!-- MODAL EDIT SOAL -->
    <div id="edit-modal" class="absolute inset-0 z-50 hidden flex items-center justify-center p-4">
        <div class="absolute inset-0 bg-black/80 backdrop-blur-md" onclick="closeEditModal()"></div>
        
        <div class="relative w-full max-w-4xl bg-slate-900 border border-slate-700 rounded-3xl shadow-2xl flex flex-col max-h-[90vh]">
            <div class="p-6 border-b border-slate-700 flex justify-between items-center bg-slate-800 rounded-t-3xl">
                <h2 class="text-3xl font-black text-white">Bank Soal</h2>
                <button onclick="closeEditModal()" class="text-slate-400 hover:text-white p-2 text-3xl font-bold transition">&times;</button>
            </div>
            
            <div class="p-6 overflow-y-auto flex-grow bg-slate-900" id="edit-list">
                <!-- List Soal -->
            </div>

            <div class="p-6 border-t border-slate-700 flex gap-4 bg-slate-800 rounded-b-3xl">
                <button onclick="addQuestion()" class="flex-1 bg-slate-700 border border-slate-600 hover:bg-slate-600 text-white font-bold py-4 rounded-xl transition">
                    + Tambah Soal Baru
                </button>
                <button onclick="saveQuestions()" class="flex-1 bg-green-500 hover:bg-green-600 text-white font-bold py-4 rounded-xl shadow-lg transition">
                    Simpan & Terapkan
                </button>
            </div>
        </div>
    </div>

    <script>
        // ==========================================
        // DATA SOAL (Default Biologi Genetik)
        // ==========================================
        const defaultQuestions = [
            { text: "Hereditas adalah proses pewarisan informasi genetik dari induk kepada keturunannya.", answer: true },
            { text: "Kromosom adalah bagian dari gen yang membawa informasi genetik tertentu.", answer: false },
            { text: "Fenotipe adalah karakteristik atau sifat yang dapat diamati pada individu.", answer: true },
            { text: "Homozigot adalah kondisi ketika pasangan alel berbeda.", answer: false },
            { text: "Alel adalah bentuk alternatif dari suatu gen.", answer: true },
            { text: "Genotipe adalah sifat fisik yang terlihat dari luar secara langsung.", answer: false },
            { text: "DNA merupakan materi genetik yang menyimpan informasi biologis organisme.", answer: true }
        ];

        let questions = JSON.parse(localStorage.getItem('microteaching_class_q')) || [...defaultQuestions];
        let currentQIndex = 0;
        let isRevealed = false;

        // ==========================================
        // SETUP KAMERA
        // ==========================================
        async function setupCamera() {
            const video = document.getElementById('bg-video');
            try {
                const stream = await navigator.mediaDevices.getUserMedia({ 
                    video: { facingMode: "user", width: { ideal: 1280 }, height: { ideal: 720 } } 
                });
                video.srcObject = stream;
            } catch (err) {
                console.error("Gagal mengakses kamera:", err);
                // Background akan tetap hitam jika kamera gagal
            }
        }

        // ==========================================
        // LOGIKA PERMAINAN
        // ==========================================
        function initGame() {
            currentQIndex = 0;
            renderQuestion();
        }

        function renderQuestion() {
            if (questions.length === 0) {
                document.getElementById('question-text').innerText = "Belum ada soal. Silakan edit soal terlebih dahulu.";
                document.getElementById('q-counter').innerText = "Bank Soal Kosong";
                return;
            }

            // Reset UI
            isRevealed = false;
            resetZones();
            
            // Toggle Tombol
            document.getElementById('btn-reveal').classList.remove('hidden');
            document.getElementById('btn-next').classList.add('hidden');

            // Set Teks
            const q = questions[currentQIndex];
            document.getElementById('question-text').innerText = q.text;
            document.getElementById('q-counter').innerText = `Soal ${currentQIndex + 1} DARI ${questions.length}`;
        }

        function revealAnswer() {
            if (questions.length === 0 || isRevealed) return;
            isRevealed = true;

            const isTrue = questions[currentQIndex].answer;
            const zoneBenar = document.getElementById('zone-benar');
            const zoneSalah = document.getElementById('zone-salah');
            const iconBenar = document.getElementById('icon-benar');
            const iconSalah = document.getElementById('icon-salah');

            // Sembunyikan tombol Tampilkan, Tampilkan tombol Selanjutnya
            document.getElementById('btn-reveal').classList.add('hidden');
            document.getElementById('btn-next').classList.remove('hidden');

            if (isTrue) {
                // JAWABAN BENAR: Kiri bersinar, Kanan gelap
                zoneBenar.className = "zone w-1/2 h-full bg-green-500/60 backdrop-blur-sm flex flex-col items-center justify-center relative";
                zoneSalah.className = "zone w-1/2 h-full bg-black/70 flex flex-col items-center justify-center relative";
                
                iconBenar.classList.remove('hidden');
                setTimeout(() => iconBenar.classList.remove('scale-0'), 50); // Animasi Pop
            } else {
                // JAWABAN SALAH: Kanan bersinar, Kiri gelap
                zoneSalah.className = "zone w-1/2 h-full bg-red-500/60 backdrop-blur-sm flex flex-col items-center justify-center relative";
                zoneBenar.className = "zone w-1/2 h-full bg-black/70 flex flex-col items-center justify-center relative border-r-2 border-white/10";
                
                iconSalah.classList.remove('hidden');
                setTimeout(() => iconSalah.classList.remove('scale-0'), 50); // Animasi Pop
            }
        }

        function nextQuestion() {
            currentQIndex++;
            if (currentQIndex >= questions.length) {
                // Bisa diubah ke layar selesai, tapi untuk demo kita loop saja
                currentQIndex = 0; 
            }
            renderQuestion();
        }

        function resetZones() {
            document.getElementById('zone-benar').className = "zone w-1/2 h-full bg-green-500/20 flex flex-col items-center justify-center relative border-r-2 border-white/20";
            document.getElementById('zone-salah').className = "zone w-1/2 h-full bg-red-500/20 flex flex-col items-center justify-center relative";
            
            const iconBenar = document.getElementById('icon-benar');
            const iconSalah = document.getElementById('icon-salah');
            
            iconBenar.classList.add('scale-0');
            iconSalah.classList.add('scale-0');
            
            setTimeout(() => {
                iconBenar.classList.add('hidden');
                iconSalah.classList.add('hidden');
            }, 500); // Tunggu animasi selesai sebelum display: none
        }

        // ==========================================
        // MODAL EDIT (Bank Soal)
        // ==========================================
        function openEditModal() {
            renderEditList();
            document.getElementById('edit-modal').classList.remove('hidden');
        }

        function closeEditModal() {
            document.getElementById('edit-modal').classList.add('hidden');
        }

        function renderEditList() {
            const listContainer = document.getElementById('edit-list');
            listContainer.innerHTML = ''; 
            
            questions.forEach((q, index) => {
                const item = document.createElement('div');
                item.className = 'bg-slate-800 p-5 rounded-2xl mb-4 border border-slate-600 shadow-sm';
                item.innerHTML = `
                    <div class="flex justify-between items-center mb-3">
                        <span class="text-sm text-slate-300 font-bold uppercase tracking-wider bg-slate-700 px-3 py-1 rounded-full">Soal ${index + 1}</span>
                        <button onclick="deleteQuestion(${index})" class="text-red-400 hover:text-white hover:bg-red-500 px-3 py-1 bg-red-400/10 rounded-lg text-sm transition font-bold">Hapus</button>
                    </div>
                    <textarea class="w-full bg-slate-900 border border-slate-600 rounded-xl p-4 text-white text-lg focus:outline-none focus:border-blue-500 focus:ring-1 focus:ring-blue-500 h-24 mb-4 transition" data-index="${index}">${q.text}</textarea>
                    <div class="flex items-center gap-4 bg-slate-900 p-3 rounded-xl border border-slate-700">
                        <label class="text-slate-300 font-bold">Kunci Jawaban:</label>
                        <select class="bg-slate-800 border border-slate-600 text-white font-bold rounded-lg p-2 flex-grow focus:outline-none focus:border-blue-500 cursor-pointer" data-ans="${index}">
                            <option value="true" ${q.answer ? 'selected' : ''}>BENAR</option>
                            <option value="false" ${!q.answer ? 'selected' : ''}>SALAH</option>
                        </select>
                    </div>
                `;
                listContainer.appendChild(item);
            });
        }

        function addQuestion() {
            questions.push({ text: "Tulis soal baru di sini...", answer: true });
            renderEditList();
            const listContainer = document.getElementById('edit-list');
            setTimeout(() => { listContainer.scrollTop = listContainer.scrollHeight; }, 100);
        }

        function deleteQuestion(index) {
            questions.splice(index, 1);
            renderEditList();
        }

        function saveQuestions() {
            const texts = document.querySelectorAll('textarea[data-index]');
            const answers = document.querySelectorAll('select[data-ans]');
            
            let temp = [];
            texts.forEach((ta, i) => {
                let val = ta.value.trim();
                if (val !== "") {
                    temp.push({ text: val, answer: answers[i].value === 'true' });
                }
            });
            
            questions = temp;
            localStorage.setItem('microteaching_class_q', JSON.stringify(questions));
            closeEditModal();
            
            // Restart game dengan soal baru
            initGame();
        }

        // ==========================================
        // MULAI APLIKASI
        // ==========================================
        window.onload = () => {
            setupCamera();
            initGame();
        };

    </script>
</body>
</html>
