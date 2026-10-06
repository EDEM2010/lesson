# lesson

<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Neo Soccer Arcade 2026</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Google Fonts -->
    <link href="https://fonts.googleapis.com/css2?family=Teko:wght@500;700&family=Montserrat:wght@400;600;800;900&display=swap" rel="stylesheet">
    <style>
        * {
            user-select: none;
            -webkit-user-select: none;
            touch-action: manipulation;
        }
        body {
            font-family: 'Montserrat', sans-serif;
            background-color: #0b0f19;
            color: #ffffff;
            overflow: hidden;
        }
        .font-teko {
            font-family: 'Teko', sans-serif;
        }
        .glass-panel {
            background: rgba(15, 23, 42, 0.75);
            backdrop-filter: blur(16px);
            -webkit-backdrop-filter: blur(16px);
            border: 1px solid rgba(255, 255, 255, 0.1);
        }
        .glass-button {
            background: linear-gradient(135deg, rgba(255, 255, 255, 0.1), rgba(255, 255, 255, 0.03));
            backdrop-filter: blur(8px);
            border: 1px solid rgba(255, 255, 255, 0.15);
            transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1);
        }
        .glass-button:hover {
            background: linear-gradient(135deg, rgba(0, 240, 255, 0.25), rgba(0, 110, 255, 0.15));
            border-color: rgba(0, 240, 255, 0.5);
            transform: translateY(-2px);
            box-shadow: 0 8px 20px rgba(0, 240, 255, 0.2);
        }
        .glass-button:active {
            transform: translateY(1px);
        }
        .neon-text-blue {
            text-shadow: 0 0 10px rgba(0, 240, 255, 0.5), 0 0 20px rgba(0, 240, 255, 0.3);
        }
        .neon-text-gold {
            text-shadow: 0 0 10px rgba(255, 215, 0, 0.5), 0 0 20px rgba(255, 215, 0, 0.3);
        }
        /* Custom scrollbar for menu */
        ::-webkit-scrollbar {
            width: 6px;
        }
        ::-webkit-scrollbar-track {
            background: rgba(0,0,0,0.2);
        }
        ::-webkit-scrollbar-thumb {
            background: rgba(0, 240, 255, 0.4);
            border-radius: 3px;
        }
        canvas {
            display: block;
            touch-action: none;
        }
    </style>
</head>
<body class="h-screen w-screen relative flex items-center justify-center bg-slate-950 overflow-hidden">

    <!-- Sound Effects via Web Audio API -->
    <script>
        class SoundFX {
            constructor() {
                this.ctx = null;
                this.enabled = true;
            }

            init() {
                if (!this.ctx) {
                    const AudioContext = window.AudioContext || window.webkitAudioContext;
                    this.ctx = new AudioContext();
                }
                if (this.ctx.state === 'suspended') {
                    this.ctx.resume();
                }
            }

            playKick(freq = 120, decay = 0.15) {
                if (!this.enabled || !this.ctx) return;
                try {
                    const osc = this.ctx.createOscillator();
                    const gain = this.ctx.createGain();
                    osc.type = 'sine';
                    osc.frequency.setValueAtTime(freq, this.ctx.currentTime);
                    osc.frequency.exponentialRampToValueAtTime(30, this.ctx.currentTime + decay);
                    
                    gain.gain.setValueAtTime(0.8, this.ctx.currentTime);
                    gain.gain.exponentialRampToValueAtTime(0.01, this.ctx.currentTime + decay);
                    
                    osc.connect(gain);
                    gain.connect(this.ctx.destination);
                    osc.start();
                    osc.stop(this.ctx.currentTime + decay);
                } catch(e){}
            }

            playWhistle() {
                if (!this.enabled || !this.ctx) return;
                try {
                    const osc1 = this.ctx.createOscillator();
                    const osc2 = this.ctx.createOscillator();
                    const gain = this.ctx.createGain();

                    osc1.type = 'sine';
                    osc2.type = 'sine';
                    osc1.frequency.setValueAtTime(2400, this.ctx.currentTime);
                    osc2.frequency.setValueAtTime(2800, this.ctx.currentTime);

                    gain.gain.setValueAtTime(0, this.ctx.currentTime);
                    gain.gain.linearRampToValueAtTime(0.3, this.ctx.currentTime + 0.05);
                    gain.gain.exponentialRampToValueAtTime(0.001, this.ctx.currentTime + 0.4);

                    osc1.connect(gain);
                    osc2.connect(gain);
                    gain.connect(this.ctx.destination);

                    osc1.start();
                    osc2.start();
                    osc1.stop(this.ctx.currentTime + 0.4);
                    osc2.stop(this.ctx.currentTime + 0.4);
                } catch(e){}
            }

            playGoalSound() {
                if (!this.enabled || !this.ctx) return;
                try {
                    // Fanfare synth chord
                    const freqs = [261.63, 329.63, 392.00, 523.25];
                    freqs.forEach((f, index) => {
                        const osc = this.ctx.createOscillator();
                        const gain = this.ctx.createGain();
                        osc.type = 'triangle';
                        osc.frequency.setValueAtTime(f, this.ctx.currentTime + index * 0.08);
                        
                        gain.gain.setValueAtTime(0, this.ctx.currentTime + index * 0.08);
                        gain.gain.linearRampToValueAtTime(0.2, this.ctx.currentTime + index * 0.08 + 0.05);
                        gain.gain.exponentialRampToValueAtTime(0.001, this.ctx.currentTime + index * 0.08 + 0.8);
                        
                        osc.connect(gain);
                        gain.connect(this.ctx.destination);
                        osc.start(this.ctx.currentTime + index * 0.08);
                        osc.stop(this.ctx.currentTime + index * 0.08 + 0.8);
                    });
                } catch(e){}
            }

            playPostHit() {
                if (!this.enabled || !this.ctx) return;
                try {
                    const osc = this.ctx.createOscillator();
                    const gain = this.ctx.createGain();
                    osc.type = 'square';
                    osc.frequency.setValueAtTime(800, this.ctx.currentTime);
                    osc.frequency.exponentialRampToValueAtTime(200, this.ctx.currentTime + 0.1);
                    
                    gain.gain.setValueAtTime(0.5, this.ctx.currentTime);
                    gain.gain.exponentialRampToValueAtTime(0.01, this.ctx.currentTime + 0.1);
                    
                    osc.connect(gain);
                    gain.connect(this.ctx.destination);
                    osc.start();
                    osc.stop(this.ctx.currentTime + 0.1);
                } catch(e){}
            }
        }
        const sfx = new SoundFX();
    </script>

    <!-- GAME CONTAINER -->
    <div id="app" class="relative w-full h-full max-w-[1400px] max-h-[850px] md:rounded-3xl shadow-2xl overflow-hidden flex flex-col bg-slate-900 border border-slate-800">
        
        <!-- CANVAS GAME SCREEN -->
        <div class="relative w-full h-full flex-1">
            <canvas id="gameCanvas" class="w-full h-full bg-emerald-900"></canvas>

            <!-- IN-GAME HUD OVERLAY -->
            <div id="hud" class="absolute top-4 left-0 right-0 px-6 flex justify-between items-center pointer-events-none hidden">
                <!-- Team 1 Info -->
                <div class="flex items-center gap-3 bg-slate-900/80 backdrop-blur-md border border-slate-700/60 px-4 py-2 rounded-2xl shadow-xl">
                    <span id="hud-t1-flag" class="text-3xl">🇦🇷</span>
                    <div>
                        <div id="hud-t1-name" class="font-bold text-sm tracking-wider text-slate-200">ARG</div>
                        <div class="text-[10px] text-cyan-400 font-semibold tracking-wider uppercase">ИГРОК 1</div>
                    </div>
                    <div id="hud-t1-score" class="font-teko text-4xl font-bold ml-2 text-white leading-none">0</div>
                </div>

                <!-- Match Time & Status -->
                <div class="flex flex-col items-center bg-slate-900/80 backdrop-blur-md border border-slate-700/60 px-6 py-2 rounded-2xl shadow-xl">
                    <div id="hud-time" class="font-teko text-3xl font-bold text-amber-400 tracking-wider leading-none">02:00</div>
                    <div id="hud-period" class="text-[10px] text-slate-400 font-bold uppercase tracking-widest">1-Й ТАЙМ</div>
                </div>

                <!-- Team 2 Info -->
                <div class="flex items-center gap-3 bg-slate-900/80 backdrop-blur-md border border-slate-700/60 px-4 py-2 rounded-2xl shadow-xl">
                    <div id="hud-t2-score" class="font-teko text-4xl font-bold mr-2 text-white leading-none">0</div>
                    <div class="text-right">
                        <div id="hud-t2-name" class="font-bold text-sm tracking-wider text-slate-200">BRA</div>
                        <div id="hud-t2-type" class="text-[10px] text-rose-400 font-semibold tracking-wider uppercase">ИИ</div>
                    </div>
                    <span id="hud-t2-flag" class="text-3xl">🇧🇷</span>
                </div>
            </div>

            <!-- GOAL ANNOUNCEMENT BANNER -->
            <div id="goal-banner" class="absolute inset-0 flex flex-col items-center justify-center pointer-events-none hidden bg-black/40 backdrop-blur-sm z-30 transition-all duration-300">
                <h1 class="font-teko text-8xl md:text-9xl font-black text-transparent bg-clip-text bg-gradient-to-r from-yellow-300 via-amber-400 to-amber-600 drop-shadow-[0_10px_20px_rgba(255,215,0,0.5)] animate-bounce tracking-widest">
                    ГОООООЛ!
                </h1>
                <div id="goal-scorer" class="text-xl md:text-2xl font-bold text-white tracking-widest uppercase bg-slate-900/80 px-6 py-2 rounded-full border border-amber-400/50 shadow-2xl">
                    АРГЕНТИНА ЗАБИВАЕТ!
                </div>
            </div>

            <!-- PAUSE MENU OVERLAY -->
            <div id="pause-screen" class="absolute inset-0 bg-slate-950/80 backdrop-blur-md flex flex-col items-center justify-center z-40 hidden">
                <div class="glass-panel p-8 rounded-3xl max-w-md w-full mx-4 text-center border border-slate-700 shadow-2xl">
                    <h2 class="font-teko text-5xl font-bold mb-6 tracking-wide text-cyan-400 neon-text-blue">ПАУЗА</h2>
                    <div class="flex flex-col gap-4">
                        <button id="btn-resume" class="glass-button py-3 px-6 rounded-xl font-bold text-lg text-white">ПРОДОЛЖИТЬ</button>
                        <button id="btn-restart" class="glass-button py-3 px-6 rounded-xl font-bold text-lg text-white">РЕСТАРТ МАТЧА</button>
                        <button id="btn-menu" class="glass-button py-3 px-6 rounded-xl font-bold text-lg text-rose-400 hover:text-rose-300">В ГЛАВНОЕ МЕНЮ</button>
                    </div>
                </div>
            </div>

            <!-- GAME OVER SCREEN -->
            <div id="game-over-screen" class="absolute inset-0 bg-slate-950/90 backdrop-blur-lg flex flex-col items-center justify-center z-40 hidden">
                <div class="glass-panel p-8 rounded-3xl max-w-lg w-full mx-4 text-center border border-amber-500/30 shadow-2xl">
                    <div class="text-amber-400 text-5xl mb-2"><i class="fa-solid fa-trophy"></i></div>
                    <h2 class="font-teko text-6xl font-black mb-1 text-white tracking-wider">МАТЧ ЗАВЕРШЕН</h2>
                    <p id="winner-text" class="text-xl font-bold text-amber-300 mb-6">ПОБЕДИЛА АРГЕНТИНА!</p>
                    
                    <div class="flex justify-center items-center gap-6 bg-slate-900/60 p-4 rounded-2xl mb-8 border border-slate-800">
                        <div class="text-center">
                            <span id="final-t1-flag" class="text-4xl block mb-1">🇦🇷</span>
                            <span id="final-t1-name" class="text-xs font-bold text-slate-400">ARG</span>
                        </div>
                        <div id="final-score" class="font-teko text-5xl font-bold text-white px-4">3 - 1</div>
                        <div class="text-center">
                            <span id="final-t2-flag" class="text-4xl block mb-1">🇧🇷</span>
                            <span id="final-t2-name" class="text-xs font-bold text-slate-400">BRA</span>
                        </div>
                    </div>

                    <div class="flex gap-4">
                        <button id="btn-rematch" class="flex-1 glass-button py-3 px-4 rounded-xl font-bold text-white bg-amber-500/20 border-amber-500/40">РЕВАНШ</button>
                        <button id="btn-final-menu" class="flex-1 glass-button py-3 px-4 rounded-xl font-bold text-slate-300">В МЕНЮ</button>
                    </div>
                </div>
            </div>

            <!-- MOBILE VIRTUAL CONTROLS OVERLAY -->
            <div id="mobile-controls" class="absolute inset-0 pointer-events-none hidden md:hidden z-20">
                <!-- Left Joystick Pad -->
                <div id="joystick-zone" class="absolute bottom-6 left-6 w-36 h-36 bg-slate-900/40 border border-white/10 rounded-full pointer-events-auto touch-none flex items-center justify-center">
                    <div id="joystick-knob" class="w-14 h-14 bg-cyan-500/60 border-2 border-cyan-300 rounded-full shadow-lg"></div>
                </div>

                <!-- Right Action Buttons -->
                <div class="absolute bottom-6 right-6 flex gap-3 pointer-events-auto">
                    <button id="btn-mobile-pass" class="w-16 h-16 bg-blue-600/70 border-2 border-blue-400 active:bg-blue-500 rounded-full font-bold text-sm text-white shadow-lg flex items-center justify-center">
                        ПАС
                    </button>
                    <button id="btn-mobile-shoot" class="w-18 h-18 bg-amber-500/80 border-2 border-amber-300 active:bg-amber-400 rounded-full font-black text-base text-white shadow-xl flex items-center justify-center -translate-y-2">
                        УДАР
                    </button>
                </div>
            </div>
        </div>

        <!-- MAIN MENU SCREEN -->
        <div id="main-menu" class="absolute inset-0 z-50 bg-slate-950 flex flex-col justify-between p-6 md:p-10 overflow-y-auto">
            <!-- Background Glow FX -->
            <div class="absolute top-1/4 left-1/2 -translate-x-1/2 -translate-y-1/2 w-[500px] h-[500px] bg-cyan-500/10 rounded-full blur-[120px] pointer-events-none"></div>
            <div class="absolute bottom-10 right-10 w-[400px] h-[400px] bg-indigo-500/10 rounded-full blur-[100px] pointer-events-none"></div>

            <!-- Header -->
            <div class="relative z-10 flex justify-between items-center border-b border-slate-800 pb-4">
                <div class="flex items-center gap-3">
                    <div class="w-10 h-10 rounded-xl bg-gradient-to-tr from-cyan-500 to-blue-600 flex items-center justify-center shadow-lg shadow-cyan-500/30">
                        <i class="fa-solid fa-futbol text-xl text-white"></i>
                    </div>
                    <span class="font-teko text-3xl font-bold tracking-wider text-white">NEO <span class="text-cyan-400">SOCCER</span> '26</span>
                </div>
                <div class="text-xs text-slate-400 bg-slate-900 border border-slate-800 px-3 py-1.5 rounded-full">
                    <i class="fa-solid fa-keyboard mr-1"></i> P1: WASD+Space | P2: Arrows+Enter
                </div>
            </div>

            <!-- Main Selection Grid -->
            <div class="relative z-10 my-auto py-6 grid grid-cols-1 lg:grid-cols-12 gap-8 items-center max-w-6xl mx-auto w-full">
                
                <!-- Team 1 Select -->
                <div class="lg:col-span-4 glass-panel p-6 rounded-3xl border border-slate-800">
                    <h3 class="text-xs font-bold tracking-widest text-cyan-400 uppercase mb-4 flex items-center justify-between">
                        <span>КОМАНДА 1 (P1)</span>
                        <i class="fa-solid fa-gamepad"></i>
                    </h3>
                    <div id="t1-selector" class="grid grid-cols-3 gap-2">
                        <!-- JS populated -->
                    </div>
                </div>

                <!-- Match Settings & Start -->
                <div class="lg:col-span-4 flex flex-col items-center text-center gap-5">
                    <div class="space-y-1">
                        <span class="text-xs font-bold tracking-widest text-slate-400 uppercase">РЕЖИМ МАТЧА</span>
                        <div class="flex bg-slate-900 p-1 rounded-2xl border border-slate-800">
                            <button id="mode-vs-ai" class="px-4 py-2 rounded-xl text-xs font-bold transition-all bg-cyan-500 text-slate-950 shadow-md">ПРОТИВ ИИ</button>
                            <button id="mode-2p" class="px-4 py-2 rounded-xl text-xs font-bold text-slate-400 hover:text-white transition-all">2 ИГРОКА</button>
                        </div>
                    </div>

                    <!-- AI Difficulty -->
                    <div id="ai-diff-container" class="space-y-1 w-full max-w-xs">
                        <span class="text-xs font-bold tracking-widest text-slate-400 uppercase">СЛОЖНОСТЬ ИИ</span>
                        <div class="grid grid-cols-3 gap-1 bg-slate-900 p-1 rounded-2xl border border-slate-800">
                            <button class="diff-btn px-2 py-1.5 rounded-xl text-xs font-bold text-slate-400 hover:text-white" data-diff="easy">ЛЕГКО</button>
                            <button class="diff-btn px-2 py-1.5 rounded-xl text-xs font-bold bg-indigo-600 text-white" data-diff="medium">СРЕДНЕ</button>
                            <button class="diff-btn px-2 py-1.5 rounded-xl text-xs font-bold text-slate-400 hover:text-white" data-diff="hard">СЛОЖНО</button>
                        </div>
                    </div>

                    <!-- Match Duration -->
                    <div class="space-y-1 w-full max-w-xs">
                        <span class="text-xs font-bold tracking-widest text-slate-400 uppercase">ВРЕМЯ МАТЧА</span>
                        <div class="grid grid-cols-3 gap-1 bg-slate-900 p-1 rounded-2xl border border-slate-800">
                            <button class="time-btn px-2 py-1.5 rounded-xl text-xs font-bold text-slate-400 hover:text-white" data-time="60">1 МИН</button>
                            <button class="time-btn px-2 py-1.5 rounded-xl text-xs font-bold bg-indigo-600 text-white" data-time="120">2 МИН</button>
                            <button class="time-btn px-2 py-1.5 rounded-xl text-xs font-bold text-slate-400 hover:text-white" data-time="180">3 МИН</button>
                        </div>
                    </div>

                    <!-- Big Start Match Button -->
                    <button id="btn-start-match" class="w-full mt-2 py-4 px-8 rounded-2xl bg-gradient-to-r from-cyan-400 via-blue-500 to-indigo-600 text-slate-950 font-black text-2xl tracking-wider uppercase shadow-xl shadow-cyan-500/20 hover:scale-105 active:scale-95 transition-all duration-200">
                        НАЧАТЬ МАТЧ <i class="fa-solid fa-play ml-2"></i>
                    </button>
                </div>

                <!-- Team 2 Select -->
                <div class="lg:col-span-4 glass-panel p-6 rounded-3xl border border-slate-800">
                    <h3 class="text-xs font-bold tracking-widest text-rose-400 uppercase mb-4 flex items-center justify-between">
                        <span>КОМАНДА 2 (<span id="t2-label">ИИ</span>)</span>
                        <i id="t2-icon" class="fa-solid fa-robot"></i>
                    </h3>
                    <div id="t2-selector" class="grid grid-cols-3 gap-2">
                        <!-- JS populated -->
                    </div>
                </div>

            </div>

            <!-- Footer -->
            <div class="relative z-10 text-center text-xs text-slate-500 border-t border-slate-800/80 pt-3">
                Neo Soccer 2026 • Управление: WASD + Пробел / Стрелки + Enter • Создано на HTML5 Canvas
            </div>
        </div>

    </div>

    <script>
        // TEAM DATA SETUP
        const TEAMS = [
            { id: 'arg', name: 'Аргентина', short: 'ARG', flag: '🇦🇷', color: '#75AADB', secondary: '#FFFFFF', speed: 4.8, power: 9 },
            { id: 'bra', name: 'Бразилия', short: 'BRA', flag: '🇧🇷', color: '#FFDC02', secondary: '#009C3B', speed: 5.0, power: 8.5 },
            { id: 'fra', name: 'Франция', short: 'FRA', flag: '🇫🇷', color: '#002395', secondary: '#ED2939', speed: 4.9, power: 8.8 },
            { id: 'ger', name: 'Германия', short: 'GER', flag: '🇩🇪', color: '#111111', secondary: '#FFCC00', speed: 4.6, power: 9.2 },
            { id: 'esp', name: 'Испания', short: 'ESP', flag: '🇪🇸', color: '#C60B1E', secondary: '#FFC400', speed: 4.7, power: 8.2 },
            { id: 'eng', name: 'Англия', short: 'ENG', flag: '🏴󠁧󠁢󠁥󠁮󠁧󠁿', color: '#FFFFFF', secondary: '#CE1124', speed: 4.7, power: 8.7 }
        ];

        // GAME STATE
        const state = {
            mode: 'vs-ai', // 'vs-ai' or '2p'
            aiDiff: 'medium', // 'easy', 'medium', 'hard'
            matchTime: 120, // seconds
            team1: TEAMS[0],
            team2: TEAMS[1],
            score1: 0,
            score2: 0,
            timeRemaining: 120,
            isPaused: false,
            isGameOver: false,
            isGoalSlowMo: false,
            slowMoTimer: 0
        };

        // CANVAS SETUP
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');

        function resizeCanvas() {
            const container = canvas.parentElement;
            canvas.width = container.clientWidth;
            canvas.height = container.clientHeight;
        }
        window.addEventListener('resize', resizeCanvas);
        resizeCanvas();

        // KEYBOARD INPUT MANAGER
        const keys = {};
        window.addEventListener('keydown', e => {
            keys[e.code] = true;
            if (e.code === 'KeyP' || e.code === 'Escape') {
                togglePause();
            }
        });
        window.addEventListener('keyup', e => {
            keys[e.code] = false;
        });

        // PHYSICS & GAME OBJECTS
        const PITCH = {
            padding: 50,
            goalWidth: 140,
            goalDepth: 35
        };

        class Ball {
            constructor(x, y) {
                this.x = x;
                this.y = y;
                this.z = 0; // Height for 3d bounce
                this.vx = 0;
                this.vy = 0;
                this.vz = 0;
                this.radius = 10;
                this.friction = 0.982;
                this.gravity = 0.4;
                this.lastKicker = null;
            }

            update() {
                this.x += this.vx;
                this.y += this.vy;
                
                // Height physics
                this.z += this.vz;
                if (this.z > 0) {
                    this.vz -= this.gravity;
                } else {
                    this.z = 0;
                    this.vz = -this.vz * 0.5; // bounce dampening
                    if (Math.abs(this.vz) < 0.5) this.vz = 0;
                }

                // Apply ground friction if on ground
                if (this.z === 0) {
                    this.vx *= this.friction;
                    this.vy *= this.friction;
                }

                // Stop minimal motion
                if (Math.abs(this.vx) < 0.02) this.vx = 0;
                if (Math.abs(this.vy) < 0.02) this.vy = 0;
            }

            draw() {
                const shadowScale = 1 - Math.min(this.z / 150, 0.6);
                
                // Draw Shadow
                ctx.beginPath();
                ctx.ellipse(this.x + 4, this.y + 6, this.radius * shadowScale, (this.radius / 2) * shadowScale, 0, 0, Math.PI * 2);
                ctx.fillStyle = 'rgba(0, 0, 0, 0.35)';
                ctx.fill();

                // Ball Circle at height z
                const renderY = this.y - this.z;
                ctx.beginPath();
                ctx.arc(this.x, renderY, this.radius, 0, Math.PI * 2);
                ctx.fillStyle = '#FFFFFF';
                ctx.fill();
                ctx.lineWidth = 2;
                ctx.strokeStyle = '#000000';
                ctx.stroke();

                // Ball pattern detail
                ctx.beginPath();
                ctx.arc(this.x - 2, renderY - 2, 3, 0, Math.PI * 2);
                ctx.fillStyle = '#111111';
                ctx.fill();
            }
        }

        class Player {
            constructor(x, y, team, role, isHuman = false, playerNum = 1) {
                this.startX = x;
                this.startY = y;
                this.x = x;
                this.y = y;
                this.vx = 0;
                this.vy = 0;
                this.radius = 16;
                this.team = team;
                this.role = role; // 'GK', 'DEF', 'ATT'
                this.isHuman = isHuman;
                this.playerNum = playerNum;
                this.angle = 0;
                this.kickCooldown = 0;
                this.speed = team.speed;
            }

            update(ball, teammates, opponents) {
                if (this.kickCooldown > 0) this.kickCooldown--;

                if (this.isHuman) {
                    this.handleHumanInput();
                } else {
                    this.handleAI(ball, teammates, opponents);
                }

                // Apply velocity & dampening
                this.x += this.vx;
                this.y += this.vy;
                this.vx *= 0.82;
                this.vy *= 0.82;

                // Angle toward movement direction
                if (Math.abs(this.vx) > 0.1 || Math.abs(this.vy) > 0.1) {
                    this.angle = Math.atan2(this.vy, this.vx);
                }
            }

            handleHumanInput() {
                let dx = 0;
                let dy = 0;

                if (this.playerNum === 1) {
                    if (keys['KeyW']) dy -= 1;
                    if (keys['KeyS']) dy += 1;
                    if (keys['KeyA']) dx -= 1;
                    if (keys['KeyD']) dx += 1;

                    // Shoot / Pass
                    if (keys['Space']) this.shoot(gameBall, 1.0);
                    if (keys['KeyF'] || keys['ShiftLeft']) this.shoot(gameBall, 0.65);
                } else {
                    // Player 2 controls
                    if (keys['ArrowUp']) dy -= 1;
                    if (keys['ArrowDown']) dy += 1;
                    if (keys['ArrowLeft']) dx -= 1;
                    if (keys['ArrowRight']) dx += 1;

                    if (keys['Enter']) this.shoot(gameBall, 1.0);
                    if (keys['KeyL'] || keys['KeyK']) this.shoot(gameBall, 0.65);
                }

                // Mobile Virtual Joystick Input (for P1)
                if (this.playerNum === 1 && mobileJoyDir.active) {
                    dx = mobileJoyDir.x;
                    dy = mobileJoyDir.y;
                }

                if (dx !== 0 || dy !== 0) {
                    const len = Math.hypot(dx, dy);
                    this.vx += (dx / len) * (this.speed * 0.28);
                    this.vy += (dy / len) * (this.speed * 0.28);
                }
            }

            handleAI(ball, teammates, opponents) {
                const target = { x: this.startX, y: this.startY };
                const isTeam1 = (this.team.id === state.team1.id);
                const attackingRight = isTeam1;

                // AI Aggressiveness tuning by difficulty
                let aggroDist = 200;
                if (state.aiDiff === 'easy') aggroDist = 140;
                if (state.aiDiff === 'hard') aggroDist = 280;

                const distToBall = Math.hypot(ball.x - this.x, ball.y - this.y);

                if (this.role === 'GK') {
                    // Goalkeeper Logic
                    const goalX = isTeam1 ? PITCH.padding + 10 : canvas.width - PITCH.padding - 10;
                    target.x = goalX;
                    target.y = Math.max(canvas.height / 2 - 60, Math.min(canvas.height / 2 + 60, ball.y));

                    // GK clears ball if very close
                    if (distToBall < 50) {
                        target.x = ball.x;
                        target.y = ball.y;
                        if (distToBall < 25) {
                            this.shoot(ball, 1.1, attackingRight ? 0 : Math.PI);
                        }
                    }
                } else if (this.role === 'ATT' || distToBall < aggroDist) {
                    // Attacker or close player chases ball
                    target.x = ball.x;
                    target.y = ball.y;

                    // If has ball, aim towards opponent's goal and shoot!
                    if (distToBall < 26) {
                        const enemyGoalX = attackingRight ? canvas.width - PITCH.padding : PITCH.padding;
                        const enemyGoalY = canvas.height / 2;
                        const shootAngle = Math.atan2(enemyGoalY - this.y, enemyGoalX - this.x);
                        
                        this.shoot(ball, 0.95, shootAngle + (Math.random() - 0.5) * 0.2);
                    }
                } else {
                    // Defensive positioning
                    target.x = this.startX + (ball.x - canvas.width / 2) * 0.3;
                    target.y = this.startY + (ball.y - canvas.height / 2) * 0.3;
                }

                // Move toward target
                const dx = target.x - this.x;
                const dy = target.y - this.y;
                const dist = Math.hypot(dx, dy);

                if (dist > 5) {
                    this.vx += (dx / dist) * (this.speed * 0.22);
                    this.vy += (dy / dist) * (this.speed * 0.22);
                }
            }

            shoot(ball, powerMult = 1.0, forcedAngle = null) {
                if (this.kickCooldown > 0) return;
                const dist = Math.hypot(ball.x - this.x, ball.y - this.y);

                if (dist < 30) {
                    const angle = forcedAngle !== null ? forcedAngle : this.angle;
                    const basePower = (this.team.power || 8.5) * powerMult;
                    
                    ball.vx = Math.cos(angle) * basePower;
                    ball.vy = Math.sin(angle) * basePower;
                    ball.vz = 2.5 + Math.random() * 2.5; // Give ball air height
                    ball.lastKicker = this;

                    this.kickCooldown = 15; // Cooldown ticks
                    sfx.playKick(140 + powerMult * 60);

                    // Add kick particle effects
                    createKickParticles(ball.x, ball.y);
                }
            }

            draw() {
                // Shadow
                ctx.beginPath();
                ctx.ellipse(this.x, this.y + 12, this.radius, this.radius / 2.5, 0, 0, Math.PI * 2);
                ctx.fillStyle = 'rgba(0,0,0,0.3)';
                ctx.fill();

                // Player Circle Body
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
                ctx.fillStyle = this.team.color;
                ctx.fill();
                ctx.lineWidth = 3;
                ctx.strokeStyle = this.team.secondary;
                ctx.stroke();

                // Direction pointer / visor
                ctx.beginPath();
                ctx.moveTo(this.x, this.y);
                ctx.lineTo(this.x + Math.cos(this.angle) * (this.radius + 4), this.y + Math.sin(this.angle) * (this.radius + 4));
                ctx.strokeStyle = '#FFFFFF';
                ctx.lineWidth = 3;
                ctx.stroke();

                // Indicator if controlled by human
                if (this.isHuman) {
                    ctx.beginPath();
                    ctx.moveTo(this.x, this.y - this.radius - 12);
                    ctx.lineTo(this.x - 6, this.y - this.radius - 20);
                    ctx.lineTo(this.x + 6, this.y - this.radius - 20);
                    ctx.closePath();
                    ctx.fillStyle = this.playerNum === 1 ? '#00f0ff' : '#ff0055';
                    ctx.fill();
                }

                // GK text indicator
                if (this.role === 'GK') {
                    ctx.fillStyle = '#FFFFFF';
                    ctx.font = '900 10px Montserrat';
                    ctx.textAlign = 'center';
                    ctx.fillText('GK', this.x, this.y + 4);
                }
            }
        }

        // PARTICLES SYSTEM
        let particles = [];
        function createKickParticles(x, y) {
            for (let i = 0; i < 8; i++) {
                particles.push({
                    x, y,
                    vx: (Math.random() - 0.5) * 6,
                    vy: (Math.random() - 0.5) * 6,
                    life: 1.0,
                    color: '#00f0ff'
                });
            }
        }

        function createConfetti() {
            for (let i = 0; i < 80; i++) {
                particles.push({
                    x: canvas.width / 2 + (Math.random() - 0.5) * 300,
                    y: canvas.height / 2 + (Math.random() - 0.5) * 200,
                    vx: (Math.random() - 0.5) * 12,
                    vy: (Math.random() - 0.5) * 12 - 4,
                    life: 2.0,
                    color: ['#FFD700', '#00F0FF', '#FF0055', '#FFFFFF', '#00FF66'][Math.floor(Math.random() * 5)]
                });
            }
        }

        function updateAndDrawParticles() {
            for (let i = particles.length - 1; i >= 0; i--) {
                const p = particles[i];
                p.x += p.vx;
                p.y += p.vy;
                p.life -= 0.025;

                ctx.globalAlpha = Math.max(0, p.life);
                ctx.fillStyle = p.color;
                ctx.fillRect(p.x, p.y, 4, 4);
                ctx.globalAlpha = 1.0;

                if (p.life <= 0) particles.splice(i, 1);
            }
        }

        // INSTANTIATE GAME ENTITIES
        let gameBall = new Ball(0, 0);
        let players = [];

        function initMatchPositions() {
            const w = canvas.width;
            const h = canvas.height;
            gameBall = new Ball(w / 2, h / 2);

            players = [
                // Team 1 (Left to Right)
                new Player(PITCH.padding + 30, h / 2, state.team1, 'GK', false),
                new Player(w * 0.28, h / 2 - 80, state.team1, 'DEF', false),
                new Player(w * 0.38, h / 2 + 50, state.team1, 'ATT', true, 1), // Human P1

                // Team 2 (Right to Left)
                new Player(w - PITCH.padding - 30, h / 2, state.team2, 'GK', false),
                new Player(w * 0.72, h / 2 + 80, state.team2, 'DEF', false),
                new Player(w * 0.62, h / 2 - 50, state.team2, 'ATT', state.mode === '2p', 2)  // Human P2 or AI
            ];
        }

        // PITCH DRAWING HELPERS
        function drawPitch() {
            const w = canvas.width;
            const h = canvas.height;
            const pad = PITCH.padding;

            // Pitch Grass Base
            ctx.fillStyle = '#15803d';
            ctx.fillRect(0, 0, w, h);

            // Grass Stripes
            const stripeWidth = (w - pad * 2) / 10;
            for (let i = 0; i < 10; i++) {
                if (i % 2 === 0) {
                    ctx.fillStyle = '#166534';
                    ctx.fillRect(pad + i * stripeWidth, pad, stripeWidth, h - pad * 2);
                }
            }

            // Pitch Lines
            ctx.strokeStyle = 'rgba(255, 255, 255, 0.85)';
            ctx.lineWidth = 3;

            // Outer boundary
            ctx.strokeRect(pad, pad, w - pad * 2, h - pad * 2);

            // Center line
            ctx.beginPath();
            ctx.moveTo(w / 2, pad);
            ctx.lineTo(w / 2, h - pad);
            ctx.stroke();

            // Center circle
            ctx.beginPath();
            ctx.arc(w / 2, h / 2, 70, 0, Math.PI * 2);
            ctx.stroke();

            // Center spot
            ctx.beginPath();
            ctx.arc(w / 2, h / 2, 4, 0, Math.PI * 2);
            ctx.fillStyle = '#FFFFFF';
            ctx.fill();

            // Penalty Areas
            const boxHeight = 220;
            const boxWidth = 110;

            // Left Penalty Box
            ctx.strokeRect(pad, (h - boxHeight) / 2, boxWidth, boxHeight);
            // Right Penalty Box
            ctx.strokeRect(w - pad - boxWidth, (h - boxHeight) / 2, boxWidth, boxHeight);

            // GOALS DRAWING
            const goalTop = (h - PITCH.goalWidth) / 2;

            // Left Goal Net
            ctx.fillStyle = 'rgba(255, 255, 255, 0.25)';
            ctx.fillRect(pad - PITCH.goalDepth, goalTop, PITCH.goalDepth, PITCH.goalWidth);
            ctx.strokeStyle = '#FFFFFF';
            ctx.strokeRect(pad - PITCH.goalDepth, goalTop, PITCH.goalDepth, PITCH.goalWidth);

            // Right Goal Net
            ctx.fillRect(w - pad, goalTop, PITCH.goalDepth, PITCH.goalWidth);
            ctx.strokeRect(w - pad, goalTop, PITCH.goalDepth, PITCH.goalWidth);
        }

        // COLLISION & BOUNDARY CHECKS
        function handleCollisions() {
            const w = canvas.width;
            const h = canvas.height;
            const pad = PITCH.padding;
            const goalTop = (h - PITCH.goalWidth) / 2;
            const goalBottom = goalTop + PITCH.goalWidth;

            // Player-Player Collisions
            for (let i = 0; i < players.length; i++) {
                for (let j = i + 1; j < players.length; j++) {
                    const p1 = players[i];
                    const p2 = players[j];
                    const dx = p2.x - p1.x;
                    const dy = p2.y - p1.y;
                    const dist = Math.hypot(dx, dy);
                    const minDist = p1.radius + p2.radius;

                    if (dist < minDist && dist > 0) {
                        const overlap = minDist - dist;
                        const nx = dx / dist;
                        const ny = dy / dist;

                        p1.x -= nx * overlap * 0.5;
                        p1.y -= ny * overlap * 0.5;
                        p2.x += nx * overlap * 0.5;
                        p2.y += ny * overlap * 0.5;
                    }
                }
            }

            // Player-Ball Collision (Tackling / Dribbling)
            players.forEach(p => {
                const dx = gameBall.x - p.x;
                const dy = gameBall.y - p.y;
                const dist = Math.hypot(dx, dy);
                const minDist = p.radius + gameBall.radius;

                if (dist < minDist) {
                    const nx = dx / dist;
                    const ny = dy / dist;

                    // Push ball slightly forward
                    gameBall.x = p.x + nx * minDist;
                    gameBall.y = p.y + ny * minDist;

                    // Transfer player momentum to ball
                    gameBall.vx += p.vx * 0.6;
                    gameBall.vy += p.vy * 0.6;
                }

                // Keep players in pitch bounds
                p.x = Math.max(pad + p.radius, Math.min(w - pad - p.radius, p.x));
                p.y = Math.max(pad + p.radius, Math.min(h - pad - p.radius, p.y));
            });

            // Ball Pitch Bounds & Goal Checking
            const inGoalY = (gameBall.y >= goalTop && gameBall.y <= goalBottom);

            // Left / Right Walls
            if (gameBall.x - gameBall.radius <= pad) {
                if (inGoalY) {
                    // LEFT GOAL SCORED! (Team 2 scores)
                    if (gameBall.x < pad - 10) triggerGoal(state.team2);
                } else {
                    gameBall.x = pad + gameBall.radius;
                    gameBall.vx = -gameBall.vx * 0.7;
                    sfx.playPostHit();
                }
            } else if (gameBall.x + gameBall.radius >= w - pad) {
                if (inGoalY) {
                    // RIGHT GOAL SCORED! (Team 1 scores)
                    if (gameBall.x > w - pad + 10) triggerGoal(state.team1);
                } else {
                    gameBall.x = w - pad - gameBall.radius;
                    gameBall.vx = -gameBall.vx * 0.7;
                    sfx.playPostHit();
                }
            }

            // Top / Bottom Walls
            if (gameBall.y - gameBall.radius <= pad) {
                gameBall.y = pad + gameBall.radius;
                gameBall.vy = -gameBall.vy * 0.7;
            } else if (gameBall.y + gameBall.radius >= h - pad) {
                gameBall.y = h - pad - gameBall.radius;
                gameBall.vy = -gameBall.vy * 0.7;
            }
        }

        // GOAL & TIMER SYSTEM
        let goalLock = false;

        function triggerGoal(scoringTeam) {
            if (goalLock) return;
            goalLock = true;

            if (scoringTeam.id === state.team1.id) {
                state.score1++;
            } else {
                state.score2++;
            }

            updateHUD();
            sfx.playWhistle();
            sfx.playGoalSound();
            createConfetti();

            // Slow Motion Effect
            state.isGoalSlowMo = true;
            state.slowMoTimer = 60;

            // Display Goal Banner
            const banner = document.getElementById('goal-banner');
            const scorerText = document.getElementById('goal-scorer');
            scorerText.textContent = `${scoringTeam.name.toUpperCase()} ЗАБИВАЕТ!`;
            banner.classList.remove('hidden');

            setTimeout(() => {
                banner.classList.add('hidden');
                initMatchPositions();
                goalLock = false;
                state.isGoalSlowMo = false;
            }, 2500);
        }

        let matchTimerInterval = null;
        function startMatchTimer() {
            if (matchTimerInterval) clearInterval(matchTimerInterval);

            matchTimerInterval = setInterval(() => {
                if (state.isPaused || state.isGameOver || goalLock) return;

                state.timeRemaining--;
                updateHUD();

                if (state.timeRemaining <= 0) {
                    endMatch();
                }
            }, 1000);
        }

        function updateHUD() {
            const minutes = Math.floor(state.timeRemaining / 60).toString().padStart(2, '0');
            const seconds = (state.timeRemaining % 60).toString().padStart(2, '0');
            document.getElementById('hud-time').textContent = `${minutes}:${seconds}`;

            document.getElementById('hud-t1-score').textContent = state.score1;
            document.getElementById('hud-t2-score').textContent = state.score2;
        }

        function endMatch() {
            state.isGameOver = true;
            clearInterval(matchTimerInterval);
            sfx.playWhistle();

            const gameOverScreen = document.getElementById('game-over-screen');
            const winnerText = document.getElementById('winner-text');

            if (state.score1 > state.score2) {
                winnerText.textContent = `ПОБЕДИЛА ${state.team1.name.toUpperCase()}!`;
            } else if (state.score2 > state.score1) {
                winnerText.textContent = `ПОБЕДИЛА ${state.team2.name.toUpperCase()}!`;
            } else {
                winnerText.textContent = 'НИЧЬЯ!';
            }

            document.getElementById('final-t1-flag').textContent = state.team1.flag;
            document.getElementById('final-t1-name').textContent = state.team1.short;
            document.getElementById('final-t2-flag').textContent = state.team2.flag;
            document.getElementById('final-t2-name').textContent = state.team2.short;
            document.getElementById('final-score').textContent = `${state.score1} - ${state.score2}`;

            gameOverScreen.classList.remove('hidden');
        }

        // MAIN GAME LOOP
        function gameLoop() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            drawPitch();

            if (!state.isPaused && !state.isGameOver) {
                // Slow motion handler on goal
                const updates = state.isGoalSlowMo ? 1 : 1; 

                for (let u = 0; u < updates; u++) {
                    gameBall.update();
                    players.forEach(p => p.update(gameBall, players, players));
                    handleCollisions();
                }
            }

            // Render entities
            players.forEach(p => p.draw());
            gameBall.draw();
            updateAndDrawParticles();

            requestAnimationFrame(gameLoop);
        }

        // MENU UI & INTERACTION CONTROLLERS
        function buildTeamSelectors() {
            const t1Box = document.getElementById('t1-selector');
            const t2Box = document.getElementById('t2-selector');

            t1Box.innerHTML = '';
            t2Box.innerHTML = '';

            TEAMS.forEach(team => {
                // Team 1 Item
                const btn1 = document.createElement('button');
                btn1.className = `p-3 rounded-2xl border transition-all flex flex-col items-center gap-1 ${state.team1.id === team.id ? 'bg-cyan-500/20 border-cyan-400 shadow-lg shadow-cyan-500/20 scale-105' : 'bg-slate-900/60 border-slate-800 hover:border-slate-700'}`;
                btn1.innerHTML = `<span class="text-3xl">${team.flag}</span><span class="text-xs font-bold text-slate-200">${team.short}</span>`;
                btn1.onclick = () => {
                    state.team1 = team;
                    buildTeamSelectors();
                };
                t1Box.appendChild(btn1);

                // Team 2 Item
                const btn2 = document.createElement('button');
                btn2.className = `p-3 rounded-2xl border transition-all flex flex-col items-center gap-1 ${state.team2.id === team.id ? 'bg-rose-500/20 border-rose-400 shadow-lg shadow-rose-500/20 scale-105' : 'bg-slate-900/60 border-slate-800 hover:border-slate-700'}`;
                btn2.innerHTML = `<span class="text-3xl">${team.flag}</span><span class="text-xs font-bold text-slate-200">${team.short}</span>`;
                btn2.onclick = () => {
                    state.team2 = team;
                    buildTeamSelectors();
                };
                t2Box.appendChild(btn2);
            });
        }

        // Mode Toggles
        document.getElementById('mode-vs-ai').onclick = function() {
            state.mode = 'vs-ai';
            this.className = 'px-4 py-2 rounded-xl text-xs font-bold transition-all bg-cyan-500 text-slate-950 shadow-md';
            document.getElementById('mode-2p').className = 'px-4 py-2 rounded-xl text-xs font-bold text-slate-400 hover:text-white transition-all';
            document.getElementById('ai-diff-container').classList.remove('hidden');
            document.getElementById('t2-label').textContent = 'ИИ';
            document.getElementById('t2-icon').className = 'fa-solid fa-robot';
        };

        document.getElementById('mode-2p').onclick = function() {
            state.mode = '2p';
            this.className = 'px-4 py-2 rounded-xl text-xs font-bold transition-all bg-cyan-500 text-slate-950 shadow-md';
            document.getElementById('mode-vs-ai').className = 'px-4 py-2 rounded-xl text-xs font-bold text-slate-400 hover:text-white transition-all';
            document.getElementById('ai-diff-container').classList.add('hidden');
            document.getElementById('t2-label').textContent = 'P2';
            document.getElementById('t2-icon').className = 'fa-solid fa-gamepad';
        };

        // Difficulty Buttons
        document.querySelectorAll('.diff-btn').forEach(btn => {
            btn.onclick = function() {
                document.querySelectorAll('.diff-btn').forEach(b => b.className = 'diff-btn px-2 py-1.5 rounded-xl text-xs font-bold text-slate-400 hover:text-white');
                this.className = 'diff-btn px-2 py-1.5 rounded-xl text-xs font-bold bg-indigo-600 text-white';
                state.aiDiff = this.dataset.diff;
            };
        });

        // Time Buttons
        document.querySelectorAll('.time-btn').forEach(btn => {
            btn.onclick = function() {
                document.querySelectorAll('.time-btn').forEach(b => b.className = 'time-btn px-2 py-1.5 rounded-xl text-xs font-bold text-slate-400 hover:text-white');
                this.className = 'time-btn px-2 py-1.5 rounded-xl text-xs font-bold bg-indigo-600 text-white';
                state.matchTime = parseInt(this.dataset.time);
            };
        });

        // Start Match
        document.getElementById('btn-start-match').onclick = () => {
            sfx.init();
            sfx.playWhistle();

            state.score1 = 0;
            state.score2 = 0;
            state.timeRemaining = state.matchTime;
            state.isPaused = false;
            state.isGameOver = false;

            // Update HUD graphics
            document.getElementById('hud-t1-flag').textContent = state.team1.flag;
            document.getElementById('hud-t1-name').textContent = state.team1.short;
            document.getElementById('hud-t2-flag').textContent = state.team2.flag;
            document.getElementById('hud-t2-name').textContent = state.team2.short;
            document.getElementById('hud-t2-type').textContent = state.mode === '2p' ? 'ИГРОК 2' : 'ИИ';

            document.getElementById('main-menu').classList.add('hidden');
            document.getElementById('hud').classList.remove('hidden');

            // Detect mobile & show controls if touch device
            if ('ontouchstart' in window || navigator.maxTouchPoints > 0) {
                document.getElementById('mobile-controls').classList.remove('hidden');
            }

            initMatchPositions();
            startMatchTimer();
        };

        // Pause Menu Handlers
        function togglePause() {
            if (state.isGameOver || document.getElementById('main-menu').classList.contains('hidden') === false) return;

            state.isPaused = !state.isPaused;
            const pauseScreen = document.getElementById('pause-screen');
            if (state.isPaused) {
                pauseScreen.classList.remove('hidden');
            } else {
                pauseScreen.classList.add('hidden');
            }
        }

        document.getElementById('btn-resume').onclick = togglePause;
        document.getElementById('btn-restart').onclick = () => {
            togglePause();
            document.getElementById('btn-start-match').click();
        };
        document.getElementById('btn-menu').onclick = () => {
            togglePause();
            document.getElementById('hud').classList.add('hidden');
            document.getElementById('main-menu').classList.remove('hidden');
            document.getElementById('mobile-controls').classList.add('hidden');
        };

        document.getElementById('btn-rematch').onclick = () => {
            document.getElementById('game-over-screen').classList.add('hidden');
            document.getElementById('btn-start-match').click();
        };
        document.getElementById('btn-final-menu').onclick = () => {
            document.getElementById('game-over-screen').classList.add('hidden');
            document.getElementById('hud').classList.add('hidden');
            document.getElementById('main-menu').classList.remove('hidden');
            document.getElementById('mobile-controls').classList.add('hidden');
        };

        // VIRTUAL JOYSTICK (MOBILE)
        const joystickZone = document.getElementById('joystick-zone');
        const joystickKnob = document.getElementById('joystick-knob');
        let mobileJoyDir = { x: 0, y: 0, active: false };

        if (joystickZone) {
            let joyRect = null;

            joystickZone.addEventListener('touchstart', e => {
                mobileJoyDir.active = true;
                joyRect = joystickZone.getBoundingClientRect();
                updateJoystick(e.touches[0]);
            });

            joystickZone.addEventListener('touchmove', e => {
                if (!mobileJoyDir.active) return;
                updateJoystick(e.touches[0]);
            });

            const resetJoy = () => {
                mobileJoyDir = { x: 0, y: 0, active: false };
                joystickKnob.style.transform = `translate(0px, 0px)`;
            };

            joystickZone.addEventListener('touchend', resetJoy);
            joystickZone.addEventListener('touchcancel', resetJoy);

            function updateJoystick(touch) {
                const centerX = joyRect.left + joyRect.width / 2;
                const centerY = joyRect.top + joyRect.height / 2;
                let dx = touch.clientX - centerX;
                let dy = touch.clientY - centerY;
                const maxDist = joyRect.width / 2 - 10;
                const dist = Math.hypot(dx, dy);

                if (dist > maxDist) {
                    dx = (dx / dist) * maxDist;
                    dy = (dy / dist) * maxDist;
                }

                joystickKnob.style.transform = `translate(${dx}px, ${dy}px)`;
                mobileJoyDir.x = dx / maxDist;
                mobileJoyDir.y = dy / maxDist;
            }

            // Mobile Action Buttons
            document.getElementById('btn-mobile-shoot').addEventListener('touchstart', (e) => {
                e.preventDefault();
                keys['Space'] = true;
                setTimeout(() => keys['Space'] = false, 100);
            });

            document.getElementById('btn-mobile-pass').addEventListener('touchstart', (e) => {
                e.preventDefault();
                keys['KeyF'] = true;
                setTimeout(() => keys['KeyF'] = false, 100);
            });
        }

        // INITIALIZE APP
        window.onload = function() {
            buildTeamSelectors();
            requestAnimationFrame(gameLoop);
        };
    </script>
</body>
</html>
