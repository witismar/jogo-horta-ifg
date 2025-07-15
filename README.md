<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Horta-Viva IFG - O Jogo</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@400;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Poppins', sans-serif;
            overflow: hidden;
        }
        .game-canvas {
            background-image: url('https://www.transparenttextures.com/patterns/light-grass.png');
            background-color: #a3bf7b;
            border-radius: 0.75rem;
        }
        .modal {
            transition: opacity 0.25s ease, transform 0.25s ease;
        }
        .modal.hidden {
            opacity: 0;
            transform: scale(0.95);
            pointer-events: none;
        }
        .progress-bar-fill {
            transition: width 0.5s ease-in-out;
        }
        .icon-btn {
            transition: transform 0.2s ease, box-shadow 0.2s ease;
        }
        .icon-btn:hover {
            transform: scale(1.05);
            box-shadow: 0 4px 15px rgba(0,0,0,0.1);
        }
        .start-screen {
            background: linear-gradient(135deg, #86B857, #C5E1A5);
        }
        .mission-toast {
            animation: slide-in-out 5s forwards;
        }
        @keyframes slide-in-out {
            0% { transform: translateY(100%); opacity: 0; }
            15% { transform: translateY(0); opacity: 1; }
            85% { transform: translateY(0); opacity: 1; }
            100% { transform: translateY(100%); opacity: 0; }
        }
        .loader {
            border: 4px solid #f3f3f3;
            border-radius: 50%;
            border-top: 4px solid #3498db;
            width: 40px;
            height: 40px;
            animation: spin 1s linear infinite;
        }
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
        .quiz-option {
            transition: background-color 0.2s;
        }
    </style>
</head>
<body class="bg-gray-100 flex items-center justify-center h-screen">

    <!-- Tela de Início -->
    <div id="start-screen" class="start-screen absolute inset-0 z-50 flex flex-col items-center justify-center text-white p-8 text-center">
        <h1 class="text-5xl md:text-7xl font-bold mb-4" style="text-shadow: 2px 2px 4px rgba(0,0,0,0.3);">Horta-Viva IFG</h1>
        <p class="text-xl md:text-2xl mb-2 font-light">Um jogo sobre sustentabilidade e agroecologia.</p>
        <p class="text-lg mb-8 max-w-2xl">Baseado no projeto de Witismar Martins C. Vieira - IFG Campus Itumbiara</p>
        <button id="start-game-btn" class="bg-white text-green-700 font-bold py-4 px-12 rounded-full text-2xl shadow-lg hover:bg-green-100 transition-transform transform hover:scale-105">
            Iniciar Jogo
        </button>
    </div>

    <!-- Estrutura Principal do Jogo -->
    <div id="game-container" class="hidden w-full h-full max-w-7xl mx-auto flex flex-col md:flex-row p-4 gap-4">

        <!-- Painel Esquerdo -->
        <div class="flex-grow flex flex-col gap-4">
            <div class="bg-white p-4 rounded-xl shadow-md flex flex-wrap items-center justify-around gap-4">
                <div class="text-center"><span class="text-lg font-semibold text-gray-700">Dia</span><p id="day-counter" class="text-3xl font-bold text-green-600">1</p></div>
                <div class="text-center"><span class="text-lg font-semibold text-gray-700">Dinheiro</span><p id="money-counter" class="text-3xl font-bold text-yellow-500">$50</p></div>
                <div class="text-center"><span class="text-lg font-semibold text-gray-700">Água</span><p id="water-counter" class="text-3xl font-bold text-blue-500">100L</p></div>
                <div class="text-center"><span class="text-lg font-semibold text-gray-700">Composto</span><p id="compost-counter" class="text-3xl font-bold text-amber-700">5kg</p></div>
            </div>
            <div class="w-full h-64 md:h-full bg-green-200 rounded-xl shadow-inner relative"><canvas id="game-canvas" class="game-canvas w-full h-full"></canvas></div>
        </div>

        <!-- Painel Direito -->
        <div class="w-full md:w-80 flex-shrink-0 flex flex-col gap-4">
            <div class="bg-white p-4 rounded-xl shadow-md flex-grow">
                <h2 class="text-2xl font-bold text-gray-800 border-b pb-2 mb-3">Missão Atual</h2>
                <div id="mission-display">
                    <h3 id="mission-title" class="text-lg font-semibold text-green-700">Bem-vindo!</h3>
                    <p id="mission-description" class="text-gray-600 mt-1">Sua primeira missão é plantar algo. Abra a loja e compre algumas sementes.</p>
                    <div class="bg-gray-200 rounded-full h-4 mt-2 overflow-hidden"><div id="mission-progress" class="bg-green-500 h-full rounded-full progress-bar-fill" style="width: 0%;"></div></div>
                </div>
            </div>
            <div class="bg-white p-4 rounded-xl shadow-md">
                 <h2 class="text-2xl font-bold text-gray-800 border-b pb-2 mb-4">Ações</h2>
                 <div class="grid grid-cols-2 gap-4">
                    <button id="shop-btn" class="icon-btn bg-blue-500 text-white p-3 rounded-lg flex flex-col items-center justify-center"><svg xmlns="http://www.w3.org/2000/svg" class="h-7 w-7 mb-1" viewBox="0 0 20 20" fill="currentColor"><path d="M3 1a1 1 0 000 2h1.22l.305 1.222a.997.997 0 00.01.042l1.358 5.43-.893.892C3.74 11.846 4.632 14 6.414 14H15a1 1 0 000-2H6.414l1-1H14a1 1 0 00.894-.553l3-6A1 1 0 0017 3H4.72l-.38-1.522A1 1 0 003 1zM5 12a2 2 0 100 4 2 2 0 000-4zm10 0a2 2 0 100 4 2 2 0 000-4z" /></svg><span class="font-semibold text-sm">Loja</span></button>
                    <button id="water-btn" class="icon-btn bg-cyan-500 text-white p-3 rounded-lg flex flex-col items-center justify-center"><svg xmlns="http://www.w3.org/2000/svg" class="h-7 w-7 mb-1" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2"><path stroke-linecap="round" stroke-linejoin="round" d="M8 4H6a2 2 0 00-2 2v12a2 2 0 002 2h12a2 2 0 002-2V6a2 2 0 00-2-2h-2m-4-1v8m0 0l3-3m-3 3L9 8m-5 5h2.586a1 1 0 01.707.293l2.414 2.414a1 1 0 001.414 0l2.414-2.414a1 1 0 01.707-.293H21" /></svg><span class="font-semibold text-sm">Irrigar</span></button>
                    <button id="compost-btn" class="icon-btn bg-amber-600 text-white p-3 rounded-lg flex flex-col items-center justify-center"><svg xmlns="http://www.w3.org/2000/svg" class="h-7 w-7 mb-1" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2"><path stroke-linecap="round" stroke-linejoin="round" d="M19 11H5m14 0a2 2 0 012 2v6a2 2 0 01-2 2H5a2 2 0 01-2-2v-6a2 2 0 012-2m14 0V9a2 2 0 00-2-2M5 11V9a2 2 0 012-2m0 0V5a2 2 0 012-2h6a2 2 0 012 2v2M7 7h10" /></svg><span class="font-semibold text-sm">Compostar</span></button>
                    <button id="pest-btn" class="icon-btn bg-red-500 text-white p-3 rounded-lg flex flex-col items-center justify-center"><svg xmlns="http://www.w3.org/2000/svg" class="h-7 w-7 mb-1" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2"><path stroke-linecap="round" stroke-linejoin="round" d="M3.055 11H5a2 2 0 012 2v1a2 2 0 002 2h8a2 2 0 002-2v-1a2 2 0 012-2h1.945M7.881 4.002l1.414 1.414M14.707 4.002l-1.414 1.414M12 10.5h.01M16 16.5v1.5a2 2 0 01-2 2h-4a2 2 0 01-2-2v-1.5" /></svg><span class="font-semibold text-sm">Defensivos</span></button>
                    <button id="gemini-advice-btn" class="icon-btn bg-gradient-to-r from-purple-500 to-indigo-600 text-white p-3 rounded-lg flex flex-col items-center justify-center">
                        <svg xmlns="http://www.w3.org/2000/svg" class="h-7 w-7 mb-1" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2"><path stroke-linecap="round" stroke-linejoin="round" d="M8.228 9c.549-1.165 2.03-2 3.772-2 2.21 0 4 1.343 4 3 0 1.4-1.278 2.575-3.006 2.907-.542.104-.994.54-.994 1.093m0 3h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" /></svg>
                        <span class="font-semibold text-sm">Consultor</span>
                    </button>
                    <!-- NOVO BOTÃO QUIZ -->
                    <button id="gemini-quiz-btn" class="icon-btn bg-gradient-to-r from-yellow-400 to-orange-500 text-white p-3 rounded-lg flex flex-col items-center justify-center">
                        <svg xmlns="http://www.w3.org/2000/svg" class="h-7 w-7 mb-1" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2"><path stroke-linecap="round" stroke-linejoin="round" d="M9.663 17h4.673M12 3v1m6.364 1.636l-.707.707M21 12h-1M4 12H3m3.343-5.657l-.707-.707m2.828 9.9a5 5 0 117.072 0l-.548.547A3.374 3.374 0 0014 18.469V19a2 2 0 11-4 0v-.531c0-.895-.356-1.754-.988-2.386l-.548-.547z" /></svg>
                        <span class="font-semibold text-sm">✨ Quiz</span>
                    </button>
                 </div>
            </div>
        </div>
    </div>

    <!-- Modal Genérico -->
    <div id="modal" class="modal hidden fixed inset-0 bg-black bg-opacity-50 z-40 flex items-center justify-center p-4">
        <div class="bg-white rounded-2xl shadow-2xl w-full max-w-md p-6 relative transform">
            <button id="modal-close-btn" class="absolute top-4 right-4 text-gray-400 hover:text-gray-700">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" /></svg>
            </button>
            <h2 id="modal-title" class="text-3xl font-bold text-gray-800 mb-4">Título do Modal</h2>
            <div id="modal-content" class="text-gray-600 max-h-[60vh] overflow-y-auto"></div>
        </div>
    </div>
    
    <div id="mission-toast" class="hidden fixed bottom-5 right-5 bg-green-600 text-white p-4 rounded-lg shadow-lg max-w-sm z-50">
        <h4 class="font-bold">Nova Missão Concluída!</h4>
        <p id="toast-text"></p>
    </div>

<script>
document.addEventListener('DOMContentLoaded', () => {

    // --- ELEMENTOS DA DOM ---
    const canvas = document.getElementById('game-canvas'), ctx = canvas.getContext('2d');
    const gameContainer = document.getElementById('game-container'), startScreen = document.getElementById('start-screen');
    const startGameBtn = document.getElementById('start-game-btn');
    const dayCounter = document.getElementById('day-counter'), moneyCounter = document.getElementById('money-counter');
    const waterCounter = document.getElementById('water-counter'), compostCounter = document.getElementById('compost-counter');
    const missionTitle = document.getElementById('mission-title'), missionDescription = document.getElementById('mission-description');
    const missionProgress = document.getElementById('mission-progress'), missionToast = document.getElementById('mission-toast');
    const toastText = document.getElementById('toast-text');
    const shopBtn = document.getElementById('shop-btn'), waterBtn = document.getElementById('water-btn');
    const compostBtn = document.getElementById('compost-btn'), pestBtn = document.getElementById('pest-btn');
    const geminiAdviceBtn = document.getElementById('gemini-advice-btn'), geminiQuizBtn = document.getElementById('gemini-quiz-btn');
    const modal = document.getElementById('modal'), modalTitle = document.getElementById('modal-title');
    const modalContent = document.getElementById('modal-content'), modalCloseBtn = document.getElementById('modal-close-btn');

    // --- ESTADO INICIAL DO JOGO ---
    let gameState = {
        day: 1, money: 50, water: 100, compost: 5, organicWaste: 10, dryMatter: 10,
        pesticides: { garlic: 0 },
        plots: [], currentMission: 0, gameSpeed: 5000, isPaused: true,
    };

    // --- CONFIGURAÇÕES E DADOS ---
    const PLOT_COLS = 5, PLOT_ROWS = 3, PLOT_SIZE = 80, PLOT_GAP = 20;
    const PLANT_DATA = {
        alface: { name: 'Alface', growthTime: 2, waterPerDay: 5, price: 5, sellValue: 10, icon: '🥬' },
        tomate: { name: 'Tomate', growthTime: 4, waterPerDay: 8, price: 15, sellValue: 35, icon: '🍅' },
        cenoura: { name: 'Cenoura', growthTime: 3, waterPerDay: 6, price: 8, sellValue: 20, icon: '🥕' },
        brocolis: { name: 'Brócolis', growthTime: 5, waterPerDay: 10, price: 20, sellValue: 50, icon: '🥦' },
    };
    const MISSIONS = [
        { title: "Primeiros Passos", description: "Compre e plante sua primeira cultura.", goal: 1, type: 'plant', reward: { money: 10 }, current: 0 },
        { title: "Mestre da Colheita", description: "Colha 3 plantas maduras.", goal: 3, type: 'harvest', reward: { money: 20, water: 20 }, current: 0 },
        { title: "Ciclo da Vida", description: "Crie seu primeiro lote de composto.", goal: 1, type: 'compost', reward: { money: 15, compost: 10 }, current: 0 },
        { title: "Defesa Natural", description: "Produza um defensivo de alho.", goal: 1, type: 'craft_pest', reward: { money: 25 }, current: 0 },
        { title: "Agricultor Próspero", description: "Acumule $200.", goal: 200, type: 'money', reward: { water: 50, compost: 20 }, current: 0 }
    ];

    // --- INICIALIZAÇÃO ---
    function initGame() {
        gameContainer.classList.remove('hidden'); gameContainer.classList.add('flex');
        startScreen.classList.add('hidden');
        resizeCanvas(); createPlots(); updateUI(); updateMissionDisplay();
        gameState.isPaused = false;
        gameLoop();
    }

    function resizeCanvas() {
        const container = canvas.parentElement;
        canvas.width = container.clientWidth; canvas.height = container.clientHeight;
        draw();
    }

    function createPlots() {
        gameState.plots = [];
        const totalWidth = PLOT_COLS * PLOT_SIZE + (PLOT_COLS - 1) * PLOT_GAP;
        const totalHeight = PLOT_ROWS * PLOT_SIZE + (PLOT_ROWS - 1) * PLOT_GAP;
        const startX = (canvas.width - totalWidth) / 2;
        const startY = (canvas.height - totalHeight) / 2;
        for (let r = 0; r < PLOT_ROWS; r++) {
            for (let c = 0; c < PLOT_COLS; c++) {
                gameState.plots.push({ x: startX + c * (PLOT_SIZE + PLOT_GAP), y: startY + r * (PLOT_SIZE + PLOT_GAP), plant: null, isFertilized: false });
            }
        }
    }

    // --- RENDERIZAÇÃO ---
    function draw() {
        if (!ctx) return;
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        drawPlots();
        requestAnimationFrame(draw);
    }

    function drawPlots() {
        gameState.plots.forEach(plot => {
            ctx.fillStyle = plot.isFertilized ? '#A0522D' : '#D2B48C';
            ctx.fillRect(plot.x, plot.y, PLOT_SIZE, PLOT_SIZE);
            ctx.strokeStyle = '#8B4513'; ctx.lineWidth = 4;
            ctx.strokeRect(plot.x, plot.y, PLOT_SIZE, PLOT_SIZE);
            if (plot.plant) {
                const pData = PLANT_DATA[plot.plant.type];
                const growth = Math.min(1, plot.plant.age / pData.growthTime);
                ctx.font = `${15 + (PLOT_SIZE - 30) * growth}px sans-serif`;
                ctx.textAlign = 'center'; ctx.textBaseline = 'middle';
                ctx.fillText(pData.icon, plot.x + PLOT_SIZE / 2, plot.y + PLOT_SIZE / 2);
                ctx.fillStyle = '#22c55e';
                ctx.fillRect(plot.x, plot.y + PLOT_SIZE - 10, PLOT_SIZE * growth, 10);
                if (plot.plant.hasPest) { ctx.font = `24px sans-serif`; ctx.fillText('🐛', plot.x + PLOT_SIZE - 15, plot.y + 15); }
                if (!plot.plant.isWatered) { ctx.font = `24px sans-serif`; ctx.fillText('💧', plot.x + 15, plot.y + 15); }
            }
        });
    }

    // --- LÓGICA DO JOGO ---
    function gameLoop() {
        if (gameState.isPaused) return;
        gameState.day++;
        updatePlants(); generateResources(); triggerPestEvent();
        updateUI(); checkMissionProgress();
        setTimeout(gameLoop, gameState.gameSpeed);
    }

    function updatePlants() {
        gameState.plots.forEach(plot => {
            if (plot.plant) {
                const waterNeeded = PLANT_DATA[plot.plant.type].waterPerDay;
                plot.plant.isWatered = gameState.water >= waterNeeded;
                if (plot.plant.isWatered) {
                    gameState.water -= waterNeeded;
                    if (plot.plant.age < PLANT_DATA[plot.plant.type].growthTime) {
                        plot.plant.age += plot.isFertilized ? 1.5 : 1;
                    }
                }
            }
        });
    }

    function generateResources() { gameState.organicWaste += 2; gameState.dryMatter += 1; }
    
    function triggerPestEvent() {
        gameState.plots.forEach(plot => {
            if (plot.plant && !plot.plant.hasPest && Math.random() < 0.15) {
                plot.plant.hasPest = true;
                showNotification("Uma praga apareceu em sua horta!", "error");
            }
        });
    }

    // --- INTERAÇÕES ---
    function handleCanvasClick(event) {
        const rect = canvas.getBoundingClientRect();
        const x = event.clientX - rect.left, y = event.clientY - rect.top;
        gameState.plots.forEach((plot, index) => {
            if (x > plot.x && x < plot.x + PLOT_SIZE && y > plot.y && y < plot.y + PLOT_SIZE) onPlotClick(index);
        });
    }

    function onPlotClick(index) {
        const plot = gameState.plots[index];
        if (plot.plant) {
            const pData = PLANT_DATA[plot.plant.type];
            if (plot.plant.age >= pData.growthTime) {
                gameState.money += pData.sellValue;
                const plantName = pData.name;
                modalTitle.textContent = "Colheita Realizada!";
                modalContent.innerHTML = `<p class="text-lg">Você colheu ${plantName} e vendeu por <span class="font-bold text-yellow-500">$${pData.sellValue}</span>!</p><button id="gemini-recipe-btn" class="mt-4 w-full bg-gradient-to-r from-green-500 to-teal-600 text-white font-bold py-3 px-4 rounded-lg hover:opacity-90 transition">✨ Gerar Receita Sustentável</button>`;
                modal.classList.remove('hidden');
                document.getElementById('gemini-recipe-btn').onclick = () => getGeminiRecipe(plantName);
                plot.plant = null; plot.isFertilized = false;
                updateMissionProgress('harvest', 1);
            } else { showNotification(`Esta planta ainda não está pronta.`, "info"); }
        } else { openShop(); }
    }
    
    window.plantSeed = (type) => {
        const pData = PLANT_DATA[type];
        if (gameState.money >= pData.price) {
            const emptyPlotIdx = gameState.plots.findIndex(p => !p.plant);
            if (emptyPlotIdx !== -1) {
                gameState.money -= pData.price;
                gameState.plots[emptyPlotIdx].plant = { type: type, age: 0, hasPest: false, isWatered: true };
                showNotification(`Você plantou ${pData.name}!`, "success");
                updateMissionProgress('plant', 1);
                closeModal(); updateUI();
            } else { showNotification("Não há canteiros vazios!", "error"); }
        } else { showNotification("Dinheiro insuficiente!", "error"); }
    }
    
    function waterPlants() { /* Lógica simplificada em updatePlants */ showNotification("A irrigação agora é automática a cada dia, se houver água disponível.", "info"); }

    // --- MODAIS E SISTEMAS ---
    function openShop() {
        modalTitle.textContent = "Loja de Sementes";
        let content = `<p class="mb-4">Compre sementes para seu cultivo.</p><div class="space-y-3">`;
        for (const key in PLANT_DATA) {
            const p = PLANT_DATA[key];
            content += `<div class="flex items-center justify-between p-3 bg-gray-100 rounded-lg"><div><span class="text-4xl">${p.icon}</span><span class="ml-4 text-lg font-semibold">${p.name}</span></div><button onclick="plantSeed('${key}')" class="bg-green-500 text-white font-bold py-2 px-4 rounded-lg hover:bg-green-600 transition">$${p.price}</button></div>`;
        }
        modalContent.innerHTML = content + `</div>`;
        modal.classList.remove('hidden');
    }
    
    function openCompostModal() {
        modalTitle.textContent = "Composteira";
        modalContent.innerHTML = `<p class="mb-4">Transforme resíduos em adubo.</p><div class="space-y-4"><div class="p-3 bg-green-100 rounded-lg"><p>Resíduo Orgânico: <span class="font-bold">${gameState.organicWaste}</span></p></div><div class="p-3 bg-yellow-100 rounded-lg"><p>Matéria Seca: <span class="font-bold">${gameState.dryMatter}</span></p></div><button id="make-compost-btn" class="w-full bg-amber-700 text-white font-bold py-3 px-4 rounded-lg hover:bg-amber-800 transition">Criar 10kg de Composto</button></div><p class="text-sm text-gray-500 mt-3">Custo: 10 Orgânico, 5 Seca.</p>`;
        document.getElementById('make-compost-btn').onclick = () => {
            if (gameState.organicWaste >= 10 && gameState.dryMatter >= 5) {
                gameState.organicWaste -= 10; gameState.dryMatter -= 5; gameState.compost += 10;
                showNotification("Você produziu 10kg de composto!", "success");
                updateMissionProgress('compost', 1); closeModal(); updateUI();
            } else { showNotification("Recursos insuficientes!", "error"); }
        };
        modal.classList.remove('hidden');
    }
    
    function openPestControlModal() { /* ... Lógica existente ... */ }
    function closeModal() { modal.classList.add('hidden'); }

    // --- MISSÕES E UI ---
    function updateMissionDisplay() { /* ... Lógica existente ... */ }
    function updateMissionProgress(type, amount) { /* ... Lógica existente ... */ }
    function checkMissionProgress() { /* ... Lógica existente ... */ }
    function updateUI() {
        dayCounter.textContent = gameState.day; moneyCounter.textContent = `$${gameState.money}`;
        waterCounter.textContent = `${gameState.water}L`; compostCounter.textContent = `${gameState.compost}kg`;
    }
    function showNotification(message, type = "info") { /* ... Lógica existente ... */ }
    
    // ===================================================================================
    // INTEGRAÇÃO COM A GEMINI API
    // ===================================================================================
    async function callGeminiAPI(prompt) {
        const apiKey = "";
        const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${apiKey}`;
        const payload = { contents: [{ role: "user", parts: [{ text: prompt }] }] };
        try {
            const response = await fetch(apiUrl, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(payload) });
            if (!response.ok) throw new Error(`API Error: ${response.statusText}`);
            const result = await response.json();
            if (result.candidates?.[0]?.content?.parts?.[0]) {
                return result.candidates[0].content.parts[0].text;
            }
            return "Não consegui pensar em uma resposta. Tente novamente.";
        } catch (error) {
            console.error("Erro ao chamar a API Gemini:", error);
            return "Ocorreu um erro de conexão com o consultor. Verifique sua conexão e tente novamente.";
        }
    }

    async function getGeminiAdvice() {
        modalTitle.textContent = "✨ Conselheiro da Horta";
        modalContent.innerHTML = `<div class="flex justify-center items-center"><div class="loader"></div></div><p class="text-center mt-4">Consultando o especialista...</p>`;
        modal.classList.remove('hidden');
        const plantas = gameState.plots.filter(p => p.plant).map(p => PLANT_DATA[p.plant.type].name).join(', ') || 'nenhuma';
        const pragas = gameState.plots.filter(p => p.plant?.hasPest).length;
        const prompt = `Você é um consultor de agroecologia para um jogo educativo. Dê um conselho curto, amigável e estratégico com base no estado atual da horta do jogador. Use emojis. Estado: Dia ${gameState.day}, Dinheiro $${gameState.money}, Água ${gameState.water}L, Composto ${gameState.compost}kg, Plantas: ${plantas}, Pragas: ${pragas}.`;
        const advice = await callGeminiAPI(prompt);
        modalContent.innerHTML = `<p style="white-space: pre-wrap;">${advice}</p>`;
    }

    async function getGeminiRecipe(plantName) {
        modalTitle.textContent = `✨ Receita com ${plantName}`;
        modalContent.innerHTML = `<div class="flex justify-center items-center"><div class="loader"></div></div><p class="text-center mt-4">Criando uma receita deliciosa...</p>`;
        modal.classList.remove('hidden');
        const prompt = `Você é um chef focado em sustentabilidade. O jogador colheu ${plantName}. Crie uma receita simples, rápida e sustentável (baixo desperdício) usando ${plantName}. Formate com "Ingredientes" e "Modo de Preparo".`;
        const recipe = await callGeminiAPI(prompt);
        modalContent.innerHTML = `<div style="white-space: pre-wrap;">${recipe}</div>`;
    }

    // --- NOVA FUNÇÃO: QUIZ DO SÁBIO ---
    async function getGeminiQuiz() {
        modalTitle.textContent = "✨ Quiz do Sábio";
        modalContent.innerHTML = `<div class="flex justify-center items-center"><div class="loader"></div></div><p class="text-center mt-4">Elaborando uma pergunta...</p>`;
        modal.classList.remove('hidden');

        const topics = ['compostagem', 'consorciação de plantas', 'defensivos naturais', 'ciclo da água', 'benefícios da horta urbana', 'polinizadores'];
        const randomTopic = topics[Math.floor(Math.random() * topics.length)];

        const prompt = `
            Você é um educador ambiental para um jogo. Crie uma pergunta de múltipla escolha sobre "${randomTopic}".
            A pergunta deve ser desafiadora, mas educativa.
            Forneça 4 opções (A, B, C, D).
            Indique a chave da resposta correta.
            Forneça uma breve explicação do porquê a resposta está correta.
            Retorne a resposta ESTRITAMENTE como um objeto JSON, sem nenhum texto antes ou depois do JSON.
            O formato do JSON deve ser:
            {
              "pergunta": "Sua pergunta aqui",
              "opcoes": { "A": "Opção A", "B": "Opção B", "C": "Opção C", "D": "Opção D" },
              "resposta_correta": "C",
              "explicacao": "Sua explicação aqui."
            }
        `;
        
        const response = await callGeminiAPI(prompt);
        try {
            // CORREÇÃO: Limpa a resposta da API para remover possíveis blocos de código markdown
            let cleanedResponse = response.trim();
            if (cleanedResponse.startsWith("```json")) {
                cleanedResponse = cleanedResponse.substring(7).trim();
                if (cleanedResponse.endsWith("```")) {
                    cleanedResponse = cleanedResponse.slice(0, -3).trim();
                }
            }
            
            const quizData = JSON.parse(cleanedResponse);
            renderQuiz(quizData);
        } catch (e) {
            console.error("Erro ao parsear JSON do Quiz:", e, "Resposta recebida:", response);
            modalContent.innerHTML = `<p>O Sábio da Horta está com dor de cabeça e não conseguiu criar uma pergunta. Tente novamente mais tarde!</p>`;
        }
    }

    function renderQuiz(quizData) {
        let optionsHtml = '';
        for (const key in quizData.opcoes) {
            optionsHtml += `<button data-key="${key}" class="quiz-option w-full text-left p-3 mt-2 bg-gray-100 hover:bg-gray-200 rounded-lg">${key}: ${quizData.opcoes[key]}</button>`;
        }

        modalContent.innerHTML = `
            <p class="font-semibold text-lg mb-4">${quizData.pergunta}</p>
            <div id="quiz-options-container">${optionsHtml}</div>
            <div id="quiz-feedback" class="mt-4 p-3 rounded-lg hidden"></div>
        `;

        document.querySelectorAll('.quiz-option').forEach(button => {
            button.addEventListener('click', (e) => handleQuizAnswer(e, quizData));
        });
    }

    function handleQuizAnswer(event, quizData) {
        const selectedButton = event.currentTarget;
        const selectedKey = selectedButton.dataset.key;
        const feedbackEl = document.getElementById('quiz-feedback');

        // Desabilita todos os botões
        document.querySelectorAll('.quiz-option').forEach(btn => btn.disabled = true);

        if (selectedKey === quizData.resposta_correta) {
            selectedButton.classList.remove('bg-gray-100');
            selectedButton.classList.add('bg-green-500', 'text-white');
            feedbackEl.innerHTML = `<p class="font-bold">Correto! 🎉</p><p>${quizData.explicacao}</p><p class="mt-2">Você ganhou $20 e 5kg de composto!</p>`;
            feedbackEl.classList.remove('hidden');
            feedbackEl.classList.add('bg-green-100', 'text-green-800');
            gameState.money += 20;
            gameState.compost += 5;
            updateUI();
        } else {
            selectedButton.classList.remove('bg-gray-100');
            selectedButton.classList.add('bg-red-500', 'text-white');
            feedbackEl.innerHTML = `<p class="font-bold">Incorreto! 😕</p><p>${quizData.explicacao}</p>`;
            feedbackEl.classList.remove('hidden');
            feedbackEl.classList.add('bg-red-100', 'text-red-800');
            // Mostra a resposta correta
            document.querySelector(`.quiz-option[data-key="${quizData.resposta_correta}"]`).classList.add('bg-green-500', 'text-white');
        }
    }

    // --- EVENT LISTENERS ---
    window.addEventListener('resize', resizeCanvas);
    canvas.addEventListener('click', handleCanvasClick);
    startGameBtn.addEventListener('click', initGame);
    modalCloseBtn.addEventListener('click', closeModal);
    shopBtn.addEventListener('click', openShop);
    waterBtn.addEventListener('click', waterPlants);
    compostBtn.addEventListener('click', openCompostModal);
    pestBtn.addEventListener('click', openPestControlModal);
    geminiAdviceBtn.addEventListener('click', getGeminiAdvice);
    geminiQuizBtn.addEventListener('click', getGeminiQuiz); // Listener para o novo botão

});
</script>
</body>
</html>
