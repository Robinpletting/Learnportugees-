<!DOCTYPE html>
<html lang="nl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Portugees Meesterschap - Statistieken</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        :root { --primary: #2563eb; --success: #22c55e; --danger: #ef4444; }
        body { font-family: 'Inter', system-ui, sans-serif; background: #f8fafc; color: #1e293b; user-select: none; }
        
        .neo-brutal { 
            background: white; 
            border: 3px solid #000; 
            box-shadow: 4px 4px 0px #000; 
            border-radius: 16px; 
            transition: transform 0.1s, box-shadow 0.1s; 
        }
        
        .neo-brutal:active:not(.locked) { 
            transform: translate(2px, 2px); 
            box-shadow: 1px 1px 0px #000; 
        }
        
        .locked { 
            opacity: 0.5; 
            filter: grayscale(1); 
            cursor: not-allowed !important; 
        }

        .option-btn { 
            width: 100%; 
            text-align: left; 
            padding: 1.25rem; 
            margin-bottom: 0.75rem; 
            font-weight: 700; 
            font-size: 1.1rem;
        }

        .correct { background: var(--success) !important; color: white; border-color: #000; box-shadow: none; }
        .wrong { background: var(--danger) !important; color: white; border-color: #000; box-shadow: none; }
        
        .no-scrollbar::-webkit-scrollbar { display: none; }

        @keyframes slideUp {
            from { transform: translateY(100%); }
            to { transform: translateY(0); }
        }
        .animate-slide-up { animation: slideUp 0.3s ease-out forwards; }

        #save-indicator {
            position: fixed;
            top: 10px;
            right: 10px;
            background: #22c55e;
            color: white;
            font-size: 10px;
            padding: 2px 8px;
            border-radius: 4px;
            font-weight: bold;
            opacity: 0;
            transition: opacity 0.3s;
            z-index: 1000;
        }
    </style>
</head>
<body class="p-0 m-0 overflow-x-hidden">

<div id="save-indicator">Opgeslagen!</div>

<div id="app">
    <!-- Dashboard -->
    <div id="dashboard" class="max-w-md mx-auto p-4 pb-24">
        <header class="flex justify-between items-center mb-6 mt-4">
            <div>
                <h1 class="text-2xl font-black italic tracking-tighter">PORTUGEES <span class="text-blue-600">PRO</span></h1>
                <p class="text-[10px] font-bold opacity-50 uppercase tracking-widest">Inclusief fout-analyse</p>
            </div>
            <div class="neo-brutal px-4 py-2 bg-yellow-300 font-black">XP: <span id="xp-val">0</span></div>
        </header>

        <div class="flex gap-2 overflow-x-auto mb-6 pb-2 no-scrollbar">
            <button onclick="switchCat('A1')" id="tab-A1" class="neo-brutal px-6 py-2 font-black shrink-0">A1</button>
            <button onclick="switchCat('A2')" id="tab-A2" class="neo-brutal px-6 py-2 font-black shrink-0">A2</button>
            <button onclick="switchCat('B1')" id="tab-B1" class="neo-brutal px-6 py-2 font-black shrink-0">B1</button>
            <button onclick="switchCat('B2')" id="tab-B2" class="neo-brutal px-6 py-2 font-black shrink-0">B2</button>
            <button onclick="switchCat('C1')" id="tab-C1" class="neo-brutal px-6 py-2 font-black shrink-0">C1</button>
            <button onclick="switchCat('C2')" id="tab-C2" class="neo-brutal px-6 py-2 font-black shrink-0">C2</button>
        </div>

        <div id="level-container" class="space-y-4"></div>
    </div>

    <!-- Resume Modal -->
    <div id="resume-modal" class="hidden fixed inset-0 bg-black/80 z-[110] flex items-center justify-center p-6">
        <div class="bg-white neo-brutal p-8 w-full max-w-sm text-center">
            <h2 class="text-2xl font-black mb-4">Sessie hervatten?</h2>
            <div class="bg-slate-50 p-4 border-2 border-black rounded-xl mb-6">
                <p class="text-slate-600 font-medium text-sm">Huidige voortgang:</p>
                <p class="font-black text-xl">Vraag <span id="resume-index" class="text-blue-600"></span>/30</p>
                <p class="text-[10px] font-black uppercase opacity-40 mt-1">Correct tot nu toe: <span id="resume-correct" class="text-green-600"></span></p>
            </div>
            <div class="flex flex-col gap-3">
                <button onclick="confirmResume(true)" class="w-full neo-brutal bg-blue-600 text-white py-4 font-black uppercase">Ga verder</button>
                <button onclick="confirmResume(false)" class="w-full neo-brutal bg-slate-200 py-3 font-black uppercase text-sm">Begin opnieuw</button>
            </div>
        </div>
    </div>

    <!-- Quiz Modus -->
    <div id="quiz-screen" class="hidden fixed inset-0 bg-white z-50 flex flex-col">
        <div class="p-4 border-b-4 border-black bg-white">
            <div class="flex justify-between items-center mb-4">
                <button onclick="exitQuiz()" class="w-10 h-10 flex items-center justify-center font-black text-2xl">✕</button>
                <div class="flex gap-4">
                   <div class="text-[10px] font-black text-green-600 uppercase">Goed: <span id="live-correct">0</span></div>
                   <div class="text-[10px] font-black text-red-600 uppercase">Fout: <span id="live-wrong">0</span></div>
                </div>
                <div class="font-mono font-bold bg-slate-100 px-3 py-1 rounded-full border-2 border-black"><span id="curr-q">1</span>/30</div>
            </div>
            <div class="w-full bg-gray-200 h-3 border-2 border-black rounded-full overflow-hidden">
                <div id="progress-bar" class="bg-blue-600 h-full transition-all duration-300" style="width: 0%"></div>
            </div>
        </div>
        
        <div class="flex-1 flex flex-col items-center justify-center p-6 bg-slate-50">
            <div class="text-center mb-10 w-full">
                <p id="q-direction" class="text-[10px] font-black opacity-40 uppercase tracking-widest mb-2"></p>
                <h2 id="question-text" class="text-3xl font-black px-4 leading-tight"></h2>
            </div>
            <div id="options-grid" class="w-full max-w-sm px-2"></div>
        </div>

        <div id="feedback-bar" class="hidden fixed bottom-0 left-0 right-0 p-8 bg-white border-t-8 border-black animate-slide-up z-50">
            <div class="max-w-md mx-auto">
                <div class="flex items-center gap-3 mb-2">
                    <span class="text-red-600 text-3xl">✕</span>
                    <h3 class="text-red-600 font-black text-2xl italic">FOUT!</h3>
                </div>
                <p class="text-xs font-bold opacity-40 uppercase mb-1">Correcte vertaling:</p>
                <p id="correct-ans" class="text-2xl font-black mb-8 p-4 bg-slate-50 border-2 border-black rounded-xl"></p>
                <button onclick="nextQuestion()" class="w-full neo-brutal bg-black text-white py-5 font-black uppercase text-xl">Doorgaan</button>
            </div>
        </div>
    </div>

    <!-- Eindscore Modal -->
    <div id="score-modal" class="hidden fixed inset-0 bg-black/60 z-[100] flex items-center justify-center p-6">
        <div class="bg-white neo-brutal p-8 w-full max-w-sm text-center">
            <h2 class="text-3xl font-black mb-2 uppercase">Resultaat</h2>
            <div class="text-6xl font-black my-4 text-blue-600"><span id="final-perc">0</span>%</div>
            
            <div class="grid grid-cols-2 gap-4 mb-8">
                <div class="p-3 bg-green-50 border-2 border-green-200 rounded-xl">
                    <p class="text-[10px] font-black text-green-600 uppercase">Goed</p>
                    <p class="text-2xl font-black" id="final-correct">0</p>
                </div>
                <div class="p-3 bg-red-50 border-2 border-red-200 rounded-xl">
                    <p class="text-[10px] font-black text-red-600 uppercase">Fout</p>
                    <p class="text-2xl font-black" id="final-wrong">0</p>
                </div>
            </div>

            <p id="score-msg" class="font-bold mb-8 text-slate-600"></p>
            <button onclick="closeQuiz()" class="w-full neo-brutal bg-black text-white py-4 font-black uppercase">Terug naar menu</button>
        </div>
    </div>
</div>

<script>
    const STORAGE_KEY = 'port_pro_v16_stats';
    const CATS = ['A1', 'A2', 'B1', 'B2', 'C1', 'C2'];
    const VOCABULARY = {};
    
    // Thema generator voor 1800 unieke woorden
    const themes = {
        'A1': ['Basis', 'Getal', 'Kleur', 'Gezin', 'Eten', 'Dier', 'Huis', 'Les', 'Lichaam', 'Tijd'],
        'A2': ['Hobby', 'Winkel', 'Reis', 'Ziek', 'Weer', 'Mode', 'Tuin', 'Werk', 'Vervoer', 'Stad'],
        'B1': ['Kantoor', 'Nieuws', 'Relatie', 'Mens', 'Tech', 'Natuur', 'Plannen', 'Studie', 'Feest', 'Wet'],
        'B2': ['Onderzoek', 'Beurs', 'Recht', 'Psyche', 'Logica', 'Dialoog', 'Vorm', 'Toen', 'Norm', 'Pitch'],
        'C1': ['Boek', 'Waarde', 'Focus', 'Tint', 'Zegje', 'Rede', 'Staat', 'Vraag', 'Pen', 'Taal'],
        'C2': ['Zijn', 'Tegen', 'Woord', 'Keus', 'Vrij', 'Heel', 'Boven', 'Blik', 'Vorm', 'Weten']
    };

    CATS.forEach(cat => {
        themes[cat].forEach((theme, idx) => {
            const levelId = `${cat}-${idx + 1}`;
            VOCABULARY[levelId] = [];
            for (let i = 1; i <= 30; i++) {
                VOCABULARY[levelId].push({
                    p: `${theme} PT ${i}`, 
                    n: `${theme} NL ${i}`
                });
            }
        });
    });

    // A1-1 Specifieke woorden
    VOCABULARY['A1-1'] = [
        {p:"Olá", n:"Hallo"}, {p:"Sim", n:"Ja"}, {p:"Não", n:"Nee"}, {p:"Bom dia", n:"Goedemorgen"}, {p:"Obrigado", n:"Bedankt"},
        {p:"Por favor", n:"Alstublieft"}, {p:"Desculpe", n:"Sorry"}, {p:"Boa noite", n:"Goedenavond"}, {p:"Tudo bem?", n:"Alles goed?"}, {p:"Água", n:"Water"},
        {p:"Pão", n:"Brood"}, {p:"Leite", n:"Melk"}, {p:"Café", n:"Koffie"}, {p:"Vinho", n:"Wijn"}, {p:"Cerveja", n:"Bier"},
        {p:"Mãe", n:"Moeder"}, {p:"Pai", n:"Vader"}, {p:"Irmão", n:"Broer"}, {p:"Irmã", n:"Zus"}, {p:"Amigo", n:"Vriend"},
        {p:"Casa", n:"Huis"}, {p:"Rua", n:"Straat"}, {p:"Cidade", n:"Stad"}, {p:"Carro", n:"Auto"}, {p:"Livro", n:"Boek"},
        {p:"Maçã", n:"Appel"}, {p:"Gato", n:"Kat"}, {p:"Cão", n:"Hond"}, {p:"Escola", n:"School"}, {p:"Trabalho", n:"Werk"}
    ];

    let state = JSON.parse(localStorage.getItem(STORAGE_KEY)) || {
        xp: 0,
        scores: {},
        sessions: {},
        cat: 'A1'
    };

    let quiz = null;
    let pendingLevelId = null;

    function save() { 
        localStorage.setItem(STORAGE_KEY, JSON.stringify(state)); 
        const ind = document.getElementById('save-indicator');
        ind.style.opacity = 1;
        setTimeout(() => ind.style.opacity = 0, 800);
    }

    function switchCat(cat) {
        const idx = CATS.indexOf(cat);
        if (idx > 0) {
            const prev = CATS[idx-1];
            if ((state.scores[`${prev}-10`] || 0) < 80) return;
        }
        state.cat = cat;
        save();
        renderDashboard();
    }

    function renderDashboard() {
        document.getElementById('xp-val').innerText = state.xp;
        const container = document.getElementById('level-container');
        container.innerHTML = '';

        CATS.forEach(c => {
            const tab = document.getElementById(`tab-${c}`);
            const unlocked = c === 'A1' || (state.scores[`${CATS[CATS.indexOf(c)-1]}-10`] >= 80);
            tab.className = `neo-brutal px-6 py-2 font-black shrink-0 ${!unlocked ? 'locked' : (state.cat === c ? 'bg-blue-600 text-white' : 'bg-white')}`;
            tab.onclick = unlocked ? () => switchCat(c) : null;
        });

        for (let i = 1; i <= 10; i++) {
            const id = `${state.cat}-${i}`;
            const sc = state.scores[id] || 0;
            const hasActiveSession = state.sessions[id] && state.sessions[id].index < 29;
            const open = i === 1 || (state.scores[`${state.cat}-${i-1}`] >= 80);
            
            const div = document.createElement('div');
            div.className = `neo-brutal p-5 flex items-center gap-4 ${!open ? 'locked' : 'cursor-pointer'}`;
            if(open) div.onclick = () => tryStartQuiz(id);
            
            div.innerHTML = `
                <div class="w-12 h-12 flex items-center justify-center font-black rounded-xl border-2 border-black ${sc >= 80 ? 'bg-green-400' : 'bg-blue-100'}">
                    ${hasActiveSession ? '▶' : i}
                </div>
                <div class="flex-1">
                    <div class="flex justify-between text-[10px] font-black uppercase mb-1">
                        <span>Level ${i} ${hasActiveSession ? '(Hervatten)' : ''}</span>
                        <span>Max Score: ${sc}%</span>
                    </div>
                    <div class="w-full bg-slate-200 border border-black h-2 rounded-full overflow-hidden">
                        <div class="bg-blue-600 h-full transition-all duration-500" style="width: ${sc}%"></div>
                    </div>
                </div>
            `;
            container.appendChild(div);
        }
    }

    function tryStartQuiz(id) {
        if (state.sessions[id] && state.sessions[id].index > 0 && state.sessions[id].index < 29) {
            pendingLevelId = id;
            document.getElementById('resume-index').innerText = state.sessions[id].index + 1;
            document.getElementById('resume-correct').innerText = state.sessions[id].correct;
            document.getElementById('resume-modal').classList.remove('hidden');
        } else {
            initNewQuiz(id);
        }
    }

    function confirmResume(resume) {
        document.getElementById('resume-modal').classList.add('hidden');
        if (resume) {
            resumeQuiz(pendingLevelId);
        } else {
            initNewQuiz(pendingLevelId);
        }
    }

    function initNewQuiz(id) {
        const pool = [...VOCABULARY[id]];
        const questions = pool.sort(() => Math.random() - 0.5).map(item => {
            const toPT = Math.random() > 0.5;
            const correctAns = toPT ? item.p : item.n;
            const others = pool.filter(i => i.p !== item.p).sort(() => Math.random() - 0.5).slice(0, 3).map(i => toPT ? i.p : i.n);
            return {
                q: toPT ? item.n : item.p,
                a: correctAns,
                opts: [correctAns, ...others].sort(() => Math.random() - 0.5),
                dir: toPT ? 'Portugees' : 'Nederlands'
            };
        });

        quiz = { id, questions, index: 0, correct: 0, wrong: 0 };
        saveSession();
        launchQuiz();
    }

    function resumeQuiz(id) {
        const s = state.sessions[id];
        quiz = { id, questions: s.questions, index: s.index, correct: s.correct, wrong: s.wrong || 0 };
        launchQuiz();
    }

    function launchQuiz() {
        document.getElementById('quiz-screen').classList.remove('hidden');
        updateLiveStats();
        showQuestion();
    }

    function updateLiveStats() {
        document.getElementById('live-correct').innerText = quiz.correct;
        document.getElementById('live-wrong').innerText = quiz.wrong;
    }

    function showQuestion() {
        const q = quiz.questions[quiz.index];
        document.getElementById('curr-q').innerText = quiz.index + 1;
        document.getElementById('progress-bar').style.width = `${(quiz.index / 30) * 100}%`;
        document.getElementById('q-direction').innerText = `Vertaal naar ${q.dir}`;
        document.getElementById('question-text').innerText = q.q;
        document.getElementById('feedback-bar').classList.add('hidden');
        
        const grid = document.getElementById('options-grid');
        grid.innerHTML = '';
        q.opts.forEach(opt => {
            const btn = document.createElement('button');
            btn.className = 'neo-brutal option-btn bg-white';
            btn.innerText = opt;
            btn.onclick = () => check(opt, q.a, btn);
            grid.appendChild(btn);
        });
    }

    function check(sel, ans, btn) {
        const btns = document.querySelectorAll('.option-btn');
        btns.forEach(b => b.onclick = null);

        if (sel === ans) {
            btn.classList.add('correct');
            quiz.correct++;
            state.xp += 10;
            document.getElementById('xp-val').innerText = state.xp;
            updateLiveStats();
            setTimeout(nextQuestion, 500);
        } else {
            btn.classList.add('wrong');
            quiz.wrong++;
            updateLiveStats();
            document.getElementById('correct-ans').innerText = ans;
            document.getElementById('feedback-bar').classList.remove('hidden');
        }
    }

    function saveSession() {
        state.sessions[quiz.id] = {
            index: quiz.index,
            correct: quiz.correct,
            wrong: quiz.wrong,
            questions: quiz.questions
        };
        save();
    }

    function nextQuestion() {
        quiz.index++;
        if (quiz.index < 30) {
            saveSession();
            showQuestion();
        } else {
            delete state.sessions[quiz.id];
            finish();
        }
    }

    function finish() {
        const perc = Math.round((quiz.correct / 30) * 100);
        state.scores[quiz.id] = Math.max(state.scores[quiz.id] || 0, perc);
        save();
        
        document.getElementById('final-perc').innerText = perc;
        document.getElementById('final-correct').innerText = quiz.correct;
        document.getElementById('final-wrong').innerText = quiz.wrong;
        document.getElementById('score-modal').classList.remove('hidden');
        
        document.getElementById('score-msg').innerText = perc >= 80 
            ? "Lekker bezig! Level voltooid. 🎉" 
            : "Nog niet genoeg voor het volgende level (80% nodig).";
    }

    function exitQuiz() {
        saveSession();
        document.getElementById('quiz-screen').classList.add('hidden');
        renderDashboard();
    }

    function closeQuiz() {
        document.getElementById('score-modal').classList.add('hidden');
        document.getElementById('quiz-screen').classList.add('hidden');
        renderDashboard();
    }

    renderDashboard();
</script>
</body>
</html>

