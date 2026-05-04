<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>PowFit Motion</title>
    
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        darkblue: '#0b1120',
                        panel: '#1e293b',
                        primary: '#3b82f6',
                        primaryHover: '#2563eb',
                        accent: '#60a5fa'
                    }
                }
            }
        }
    </script>

    <!-- FontAwesome & Google Fonts -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;500;600;700&display=swap" rel="stylesheet">

    <style>
        body {
            font-family: 'Poppins', sans-serif;
            background-color: #0b1120; /* darkblue */
            color: #f8fafc;
            -webkit-tap-highlight-color: transparent;
        }

        /* Custom Scrollbar */
        ::-webkit-scrollbar {
            width: 6px;
        }
        ::-webkit-scrollbar-track {
            background: #0b1120;
        }
        ::-webkit-scrollbar-thumb {
            background: #3b82f6;
            border-radius: 10px;
        }

        /* Animations */
        .fade-in { animation: fadeIn 0.3s ease-in-out; }
        .slide-up { animation: slideUp 0.3s ease-out; }

        @keyframes fadeIn {
            from { opacity: 0; }
            to { opacity: 1; }
        }
        @keyframes slideUp {
            from { transform: translateY(20px); opacity: 0; }
            to { transform: translateY(0); opacity: 1; }
        }

        /* Modal Backdrop blur */
        .modal-backdrop {
            backdrop-filter: blur(4px);
        }

        /* Loader */
        .loader {
            border: 3px solid rgba(255,255,255,0.1);
            border-left-color: #3b82f6;
            border-radius: 50%;
            width: 24px;
            height: 24px;
            animation: spin 1s linear infinite;
        }
        @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
    </style>
</head>
<body class="min-h-screen bg-darkblue flex flex-col relative">

    <!-- Notificações Toast -->
    <div id="toast-container" class="fixed top-20 left-1/2 transform -translate-x-1/2 z-[100] flex flex-col gap-2 pointer-events-none w-11/12 max-w-md"></div>

    <!-- TELA DE LOGIN (Fixado) -->
    <div id="login-screen" class="fixed inset-0 z-[100] flex flex-col items-center justify-center bg-darkblue px-6 fade-in">
        <div class="mb-8 text-center">
            <div class="w-24 h-24 bg-panel rounded-full flex items-center justify-center mx-auto mb-4 border-2 border-primary shadow-[0_0_15px_rgba(59,130,246,0.5)]">
                <i class="fas fa-dumbbell text-4xl text-primary"></i>
            </div>
            <h1 class="text-3xl font-bold text-white tracking-wide">PowFit<span class="text-primary">Motion</span></h1>
            <p class="text-gray-400 mt-2 text-sm">Seus treinos e evoluções sempre com você.</p>
        </div>
        
        <button id="btn-google-login" class="flex items-center gap-3 bg-white text-gray-800 px-6 py-3 rounded-full font-semibold shadow-lg hover:bg-gray-100 transition-all active:scale-95 w-full max-w-sm justify-center">
            <img src="https://www.svgrepo.com/show/475656/google-color.svg" alt="Google" class="w-6 h-6">
            Entrar com o Google
        </button>
        <div id="login-loading" class="hidden mt-4 text-primary"><div class="loader"></div></div>
    </div>

    <!-- APP PRINCIPAL -->
    <div id="main-app" class="hidden flex-col w-full min-h-screen">
        
        <!-- CABEÇALHO FIXO -->
        <header class="fixed top-0 left-0 right-0 bg-panel border-b border-gray-800 p-4 flex items-center justify-between z-50 shadow-md shadow-black/40">
            <div class="flex items-center gap-2">
                <i class="fas fa-dumbbell text-primary text-xl"></i>
                <h1 class="font-bold text-xl tracking-wide">PowFit<span class="text-primary">Motion</span></h1>
            </div>
            <div class="flex items-center gap-3">
                <img id="user-avatar" src="" alt="Usuário" class="w-9 h-9 rounded-full border border-primary object-cover hidden">
                <button id="btn-logout" class="text-gray-400 hover:text-red-400 p-2 transition-colors">
                    <i class="fas fa-sign-out-alt text-lg"></i>
                </button>
            </div>
        </header>

        <!-- CONTEÚDO SCROLLÁVEL -->
        <main id="app-content" class="w-full max-w-3xl mx-auto pt-20 pb-10 px-4">
            
            <!-- VIEW: Categorias & Pesquisa -->
            <div id="view-categories" class="slide-up">
                
                <!-- Barra de Pesquisa Global -->
                <div class="mb-6 relative">
                    <div class="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none">
                        <i class="fas fa-search text-gray-500"></i>
                    </div>
                    <input type="text" id="global-search-input" class="w-full bg-panel border border-gray-700 rounded-xl py-3 pl-10 pr-4 text-white placeholder-gray-500 focus:outline-none focus:border-primary focus:ring-1 focus:ring-primary transition shadow-inner" placeholder="Buscar exercício por nome...">
                    <button id="btn-clear-search" class="absolute inset-y-0 right-0 pr-3 flex items-center text-gray-500 hover:text-white hidden">
                        <i class="fas fa-times"></i>
                    </button>
                </div>

                <!-- Resultados da Pesquisa -->
                <div id="search-results-container" class="hidden mb-6">
                    <h2 class="text-lg font-semibold mb-3 border-l-4 border-accent pl-2 text-accent">Resultados da Busca</h2>
                    <div id="search-results-list" class="flex flex-col gap-3">
                        <!-- Gerado via JS -->
                    </div>
                </div>

                <!-- Container Principal de Categorias -->
                <div id="categories-container">
                    <h2 class="text-xl font-semibold mb-4 border-l-4 border-primary pl-2">Categorias</h2>
                    <div id="categories-grid" class="grid grid-cols-2 gap-4 sm:grid-cols-3">
                        <!-- Gerado via JS -->
                    </div>
                </div>
            </div>

            <!-- VIEW: Lista de Exercícios -->
            <div id="view-exercises" class="hidden slide-up flex-col mt-2">
                <div class="flex items-center justify-between mb-4 pb-2 border-b border-gray-800">
                    <button id="btn-back-categories" class="text-gray-400 hover:text-white py-2 flex items-center gap-2 transition">
                        <i class="fas fa-arrow-left"></i> Voltar
                    </button>
                    <h2 id="current-category-title" class="text-lg font-bold text-primary truncate px-2">CATEGORIA</h2>
                </div>
                
                <div class="flex justify-between items-center mb-4">
                    <p class="text-sm text-gray-400" id="exercise-count">0 exercícios</p>
                    <button id="btn-open-add-modal" class="bg-primary/20 text-accent px-4 py-2 rounded-lg text-sm font-medium hover:bg-primary/30 flex items-center gap-2 transition active:scale-95 shadow-sm">
                        <i class="fas fa-plus"></i> Novo Exer.
                    </button>
                </div>

                <div id="exercises-list" class="flex flex-col gap-3">
                    <!-- Gerado via JS -->
                </div>
            </div>

        </main>
    </div>

    <!-- MODAL: Ver Exercício e Adicionar Mídia -->
    <div id="exercise-modal" class="fixed inset-0 z-[120] bg-black/80 modal-backdrop hidden flex-col justify-end sm:justify-center items-center pb-0 sm:pb-10 transition-opacity">
        <div class="bg-panel w-full sm:w-[90%] sm:max-w-md rounded-t-2xl sm:rounded-2xl overflow-hidden flex flex-col max-h-[90vh] slide-up border border-gray-800 shadow-2xl">
            <!-- Cabeçalho do Modal -->
            <div class="flex justify-between items-center p-4 border-b border-gray-800 bg-darkblue/50">
                <div class="flex flex-col truncate pr-2">
                    <h3 id="modal-exercise-title" class="font-bold text-lg truncate">Nome do Exercício</h3>
                    <span id="modal-exercise-category-label" class="text-xs text-primary font-medium">Categoria</span>
                </div>
                <button id="btn-close-exercise-modal" class="text-gray-400 hover:text-white p-2 w-10 h-10 flex items-center justify-center rounded-full hover:bg-gray-800 transition shrink-0">
                    <i class="fas fa-times text-xl"></i>
                </button>
            </div>
            
            <!-- Conteúdo do Modal -->
            <div class="p-4 flex-1 overflow-y-auto flex flex-col items-center">
                
                <!-- Área da Imagem/Gif -->
                <div id="media-container" class="w-full aspect-square bg-darkblue rounded-xl flex items-center justify-center overflow-hidden border border-gray-800 mb-4 relative group">
                    <div id="media-placeholder" class="text-center text-gray-500 flex flex-col items-center p-4">
                        <i class="fas fa-image text-5xl mb-2 opacity-50"></i>
                        <p class="text-sm mt-2">Nenhuma imagem/GIF adicionada.</p>
                    </div>
                    
                    <!-- Imagem Mostrada -->
                    <img id="exercise-media-img" src="" alt="Exercício" class="hidden w-full h-full object-contain">
                    
                    <!-- Botão de Editar Sobreposto (visível apenas quando há imagem salva) -->
                    <button id="btn-edit-media" class="absolute top-3 right-3 bg-black/60 text-white w-10 h-10 rounded-full flex items-center justify-center hover:bg-black/80 transition hidden shadow-lg border border-white/20 active:scale-95">
                        <i class="fas fa-pen"></i>
                    </button>

                    <!-- Tela de Carregamento (Sem Storage, direto pro Firestore) -->
                    <div id="media-loading" class="absolute inset-0 bg-black/80 hidden flex-col items-center justify-center backdrop-blur-md z-10">
                        <div class="loader mb-3"></div>
                        <span id="media-loading-text" class="text-sm font-medium text-white text-center px-4">Salvando no Banco de Dados...</span>
                    </div>
                </div>

                <!-- Input File Escondido -->
                <input type="file" id="media-upload" accept="image/*, image/gif" class="hidden">
                
                <!-- Área de Ações -->
                <div class="w-full">
                    <!-- Botão Adicionar Inicial -->
                    <button id="btn-trigger-upload" class="w-full bg-primary hover:bg-primaryHover text-white py-3 rounded-xl font-semibold flex items-center justify-center gap-2 transition active:scale-95 shadow-lg shadow-primary/20">
                        <i id="btn-upload-icon" class="fas fa-camera"></i> <span id="upload-btn-text">Adicionar Imagem / Gif</span>
                    </button>

                    <!-- Botões de Salvar e Cancelar (Pré-visualização) -->
                    <div id="preview-actions" class="hidden flex w-full gap-3">
                        <button id="btn-cancel-media" class="flex-1 bg-gray-700 hover:bg-gray-600 text-white py-3 rounded-xl font-semibold flex items-center justify-center gap-2 transition active:scale-95">
                            <i class="fas fa-times"></i> Cancelar
                        </button>
                        <button id="btn-save-media" class="flex-1 bg-green-600 hover:bg-green-500 text-white py-3 rounded-xl font-semibold flex items-center justify-center gap-2 transition active:scale-95 shadow-lg shadow-green-600/20">
                            <i class="fas fa-check"></i> Salvar
                        </button>
                    </div>
                </div>

            </div>
        </div>
    </div>

    <!-- MODAL: Adicionar Novo Exercício Personalizado -->
    <div id="add-modal" class="fixed inset-0 z-[120] bg-black/80 modal-backdrop hidden flex-col justify-end sm:justify-center items-center transition-opacity">
        <div class="bg-panel w-full sm:w-[90%] sm:max-w-md rounded-t-2xl sm:rounded-2xl p-5 slide-up border border-gray-800">
            <div class="flex justify-between items-center mb-5">
                <h3 class="font-bold text-lg">Criar Exercício</h3>
                <button id="btn-close-add-modal" class="text-gray-400 hover:text-white p-1">
                    <i class="fas fa-times text-xl"></i>
                </button>
            </div>
            
            <div class="space-y-4">
                <div>
                    <label class="block text-sm text-gray-400 mb-1">Nome do Exercício</label>
                    <input type="text" id="new-exercise-name" class="w-full bg-darkblue border border-gray-700 rounded-lg px-4 py-3 text-white focus:outline-none focus:border-primary focus:ring-1 focus:ring-primary transition" placeholder="Ex: Supino Inclinado Máquina">
                </div>
                <div>
                    <label class="block text-sm text-gray-400 mb-1">Categoria</label>
                    <select id="new-exercise-category" class="w-full bg-darkblue border border-gray-700 rounded-lg px-4 py-3 text-white focus:outline-none focus:border-primary transition appearance-none">
                        <!-- Gerado via JS -->
                    </select>
                </div>
                <button id="btn-save-exercise" class="w-full bg-primary hover:bg-primaryHover text-white py-3 rounded-xl font-semibold mt-4 flex items-center justify-center gap-2 transition active:scale-95">
                    <i class="fas fa-save"></i> Guardar Exercício
                </button>
            </div>
        </div>
    </div>

    <!-- MODAL: Segurança para Editar Imagem -->
    <div id="security-modal" class="fixed inset-0 z-[130] bg-black/80 modal-backdrop hidden flex-col justify-end sm:justify-center items-center transition-opacity">
        <div class="bg-panel w-full sm:w-[90%] sm:max-w-md rounded-t-2xl sm:rounded-2xl p-5 slide-up border border-gray-800 shadow-2xl">
            <div class="flex justify-between items-center mb-4">
                <h3 class="font-bold text-lg text-white"><i class="fas fa-shield-alt text-primary mr-2"></i>Segurança</h3>
                <button id="btn-close-security" class="text-gray-400 hover:text-white p-1 rounded-full hover:bg-gray-800 transition">
                    <i class="fas fa-times text-xl"></i>
                </button>
            </div>
            <p class="text-sm text-gray-400 mb-4">Para editar uma imagem já salva, por favor confirme sua identidade digitando a sua senha secreta.</p>
            
            <div class="space-y-4">
                <div>
                    <!-- type mudou para password para esconder os caracteres, mas a senha é o e-mail -->
                    <input type="password" id="security-email-input" class="w-full bg-darkblue border border-gray-700 rounded-lg px-4 py-3 text-white focus:outline-none focus:border-primary focus:ring-1 focus:ring-primary transition" placeholder="Digite sua senha secreta...">
                </div>
                
                <div class="flex gap-3">
                    <button id="btn-cancel-security" class="flex-1 bg-gray-700 hover:bg-gray-600 text-white py-3 rounded-xl font-semibold transition active:scale-95">
                        Cancelar
                    </button>
                    <button id="btn-confirm-security" class="flex-1 bg-primary hover:bg-primaryHover text-white py-3 rounded-xl font-semibold transition active:scale-95 shadow-lg shadow-primary/20">
                        Confirmar
                    </button>
                </div>
            </div>
        </div>
    </div>

    <!-- Firebase SDK Imports (Sem Storage) -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, GoogleAuthProvider, signInWithPopup, onAuthStateChanged, signOut, signInWithCustomToken } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, collection, doc, setDoc, onSnapshot } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // ==========================================
        // DADOS PADRÃO
        // ==========================================
        const defaultData = {
            "PEITO": [
                "Supino Reto", "Supino Inclinado", "Supino Inclinado com Halteres", 
                "Supino Fechado com Halteres", "Crucifixo no Cross", "Cross Over Alto", 
                "Cross Over Baixo", "Crucifixo Reto", "Crucifixo Inclinado com Halteres", 
                "Crucifixo na Máquina", "Peck Fly", "Peck Fly Unilateral", "Pullover", 
                "Flexão de Braço", "Flexão com Pés Elevados", "Flexão Explosiva"
            ],
            "COSTAS": [
                "Puxada Alta", "Puxada Alta Supinada", "Pulldown", "Remada Aberta", 
                "Remada Baixa", "Remada Curvada", "Remada Curvada Supinada", 
                "Remada Cavalinho (T-Bar)", "Serrote", "Facepull (puxada de cima para baixo)"
            ],
            "PERNAS": [
                "Agachamento Livre", "Agachamento Taça", "Agachamento no Smith", 
                "Agachamento com passada lateral", "Squat", "Hack Machine", "Leg 45°", 
                "Leg 90°", "Agachamento Sumô", "Agachamento Sissy (Livre)", "Afundo", 
                "Recuo", "Avanço", "Passada", "Búlgaro", "Step-up", "Levantamento Terra", 
                "Levantamento Terra Romeno", "Terra Sumô", "Stiff", "Bom Dia", "Mesa Flexora", 
                "Cadeira Flexora", "Elevação Pélvica no Banco", "Elevação Pélvica no Chão", 
                "Elevação Pélvica Unilateral no Chão", "Extensão de Quadril (Glúteo Máximo)", 
                "Extensão Cruzada (Glúteo Médio)", "Coice", "Cachorrinho", "Cadeira Extensora", 
                "Adução", "Abdução", "Abdução Inclinada", "Flexão Nórdica", "Flexão Nórdica Invertida", 
                "Panturrilha Livre", "Panturrilha no Leg Press", "Panturrilha Banco", 
                "Panturrilha Squat", "Panturrilha Unilateral"
            ],
            "BRAÇOS (BÍCEPS)": [
                "Rosca Direta", "Rosca Alternada", "Rosca 21", "Rosca Scott Barra W", 
                "Rosca Scott Unilateral", "Rosca Scott com Halteres", "Rosca Martelo", 
                "Rosca Cross", "Rosca Inversa", "Rosca 45°"
            ],
            "BRAÇOS (TRÍCEPS)": [
                "Triceps Pulley Unilateral", "Triceps Pulley Barra", "Triceps Pulley Corda", 
                "Triceps Pulley Pegada Inversa", "Triceps Francês na Corda", "Triceps Francês com Halter", 
                "Triceps Francês Unilateral", "Triceps Cruzado Polia Dupla", "Triceps Coice Unilateral", 
                "Triceps Arremesso", "Triceps Testa", "Mergulho no Banco"
            ],
            "OMBROS": [
                "Elevação Frontal", "Elevação Frontal no Cross", "Elevação Lateral", 
                "Elevação Lateral Unilateral Cross", "Elevação Lateral Sentado", 
                "Desenvolvimento com Halteres", "Desenvolvimento com Barra", "Arnold Press", 
                "Elevação Borboleta", "Crucifixo Inverso Sentado com Halteres", 
                "Crucifixo Inverso na Polia", "Crucifixo Inverso Unilateral na Polia", 
                "Facepull (puxada reta)", "Remada Alta", "Encolhimento (Trapézio)"
            ],
            "ABDÔMEN": [
                "Infra com Elevação de Perna", "Abdominal Supra", "Abdominal Remador", 
                "Abdominal Bicicleta", "Abdominal Twister com Peso", "Prancha", 
                "Prancha Lateral", "Trituração de Cabos em Pé", "Isometria na parede"
            ],
            "CARDIO": [
                "Bicicleta 10 Minutos", "Bicicleta 15 Minutos", "Bicicleta 20 Minutos", 
                "Esteira 10 Minutos", "Esteira 15 Minutos", "Esteira 20 Minutos", "Pular Corda"
            ]
        };

        const categoryIcons = {
            "PEITO": "fa-child",
            "COSTAS": "fa-walking",
            "PERNAS": "fa-running",
            "BRAÇOS (BÍCEPS)": "fa-hand-rock",
            "BRAÇOS (TRÍCEPS)": "fa-hand-paper",
            "OMBROS": "fa-universal-access",
            "ABDÔMEN": "fa-street-view",
            "CARDIO": "fa-heartbeat"
        };

        const categoryList = Object.keys(defaultData);

        // ==========================================
        // FIREBASE SETUP
        // ==========================================
        let fbConfig;
        if (typeof __firebase_config !== 'undefined') {
            fbConfig = JSON.parse(__firebase_config);
        } else {
            fbConfig = {
                apiKey: "AIzaSyCe1yD1neDzp5bHOrLgClGijpS37RX5U9U",
                authDomain: "powfit-motion.firebaseapp.com",
                projectId: "powfit-motion",
                storageBucket: "powfit-motion.firebasestorage.app",
                messagingSenderId: "431389771359",
                appId: "1:431389771359:web:3061c0d6f4441c37747dea"
            };
        }

        const appFirebase = initializeApp(fbConfig);
        const auth = getAuth(appFirebase);
        const db = getFirestore(appFirebase);
        const systemAppId = typeof __app_id !== 'undefined' ? __app_id : 'powfit-motion';

        // ==========================================
        // ESTADO GLOBAL
        // ==========================================
        let currentUser = null;
        let userDataListener = null;
        let customExercisesData = {};
        
        let currentView = 'categories';
        let activeCategory = '';
        let activeExerciseSlug = '';
        let activeExerciseName = '';
        let modalActiveCategoryContext = ''; 
        
        let pendingMediaFile = null; 

        // ==========================================
        // UTILITÁRIOS & COMPRESSÃO PARA FIRESTORE (BASE64)
        // ==========================================
        function createSlug(text) {
            return text.toString().toLowerCase()
                .normalize('NFD').replace(/[\u0300-\u036f]/g, "") 
                .replace(/\s+/g, '-') 
                .replace(/[^\w\-]+/g, '') 
                .replace(/\-\-+/g, '-') 
                .replace(/^-+/, '').replace(/-+$/, '');
        }

        function showToast(message, type = 'info', duration = 3000) {
            const container = document.getElementById('toast-container');
            const toast = document.createElement('div');
            
            const colors = { success: 'bg-green-600', error: 'bg-red-600', info: 'bg-blue-600', warning: 'bg-yellow-600' };
            const icons = { success: 'fa-check-circle', error: 'fa-exclamation-circle', info: 'fa-info-circle', warning: 'fa-exclamation-triangle' };

            toast.className = `${colors[type]} text-white px-4 py-3 rounded-lg shadow-lg flex items-center gap-3 slide-up text-sm font-medium w-full`;
            toast.innerHTML = `<i class="fas ${icons[type]} text-lg"></i> <span>${message}</span>`;
            
            container.appendChild(toast);
            
            setTimeout(() => {
                toast.style.opacity = '0';
                toast.style.transition = 'opacity 0.3s ease';
                setTimeout(() => toast.remove(), 300);
            }, duration);
        }

        function getAllExercises() {
            let allExs = [];
            categoryList.forEach(cat => {
                defaultData[cat].forEach(name => {
                    allExs.push({ name, slug: createSlug(name), category: cat, isCustom: false });
                });
            });

            Object.keys(customExercisesData).forEach(slug => {
                const data = customExercisesData[slug];
                if (data.isCustom) {
                    allExs.push({ name: data.name, slug: slug, category: data.category, isCustom: true });
                }
            });

            return allExs;
        }

        // Lê arquivo direto para Base64 (para arquivos pequenos)
        function fileToBase64(file) {
            return new Promise((resolve, reject) => {
                const reader = new FileReader();
                reader.readAsDataURL(file);
                reader.onload = () => resolve(reader.result);
                reader.onerror = error => reject(error);
            });
        }

        // Comprime arquivo e retorna Base64 (para arquivos grandes caberem no Firestore)
        function compressToBase64(file, maxDimension = 800, quality = 0.7) {
            return new Promise((resolve, reject) => {
                const reader = new FileReader();
                reader.readAsDataURL(file);
                reader.onload = event => {
                    const img = new Image();
                    img.src = event.target.result;
                    img.onload = () => {
                        let width = img.width;
                        let height = img.height;
                        if (width > maxDimension || height > maxDimension) {
                            if (width > height) {
                                height = Math.round((height *= maxDimension / width));
                                width = maxDimension;
                            } else {
                                width = Math.round((width *= maxDimension / height));
                                height = maxDimension;
                            }
                        }
                        const canvas = document.createElement('canvas');
                        canvas.width = width;
                        canvas.height = height;
                        const ctx = canvas.getContext('2d');
                        ctx.drawImage(img, 0, 0, width, height);
                        
                        // Retorna como JPEG reduzido
                        resolve(canvas.toDataURL('image/jpeg', quality));
                    };
                    img.onerror = () => resolve(event.target.result); //Fallback
                };
                reader.onerror = () => reject(reader.error);
            });
        }

        // ==========================================
        // LÓGICA DE INTERFACE
        // ==========================================
        const DOM = {
            loginScreen: document.getElementById('login-screen'),
            mainApp: document.getElementById('main-app'),
            viewCategories: document.getElementById('view-categories'),
            viewExercises: document.getElementById('view-exercises'),
            categoriesGrid: document.getElementById('categories-grid'),
            exercisesList: document.getElementById('exercises-list'),
            currentCategoryTitle: document.getElementById('current-category-title'),
            exerciseCount: document.getElementById('exercise-count'),
            userAvatar: document.getElementById('user-avatar'),
            
            searchInput: document.getElementById('global-search-input'),
            btnClearSearch: document.getElementById('btn-clear-search'),
            searchResultsContainer: document.getElementById('search-results-container'),
            searchResultsList: document.getElementById('search-results-list'),
            categoriesContainer: document.getElementById('categories-container'),

            exModal: document.getElementById('exercise-modal'),
            exModalTitle: document.getElementById('modal-exercise-title'),
            exModalCatLabel: document.getElementById('modal-exercise-category-label'),
            mediaPlaceholder: document.getElementById('media-placeholder'),
            mediaImg: document.getElementById('exercise-media-img'),
            mediaInput: document.getElementById('media-upload'),
            btnTriggerUpload: document.getElementById('btn-trigger-upload'),
            
            previewActions: document.getElementById('preview-actions'),
            btnSaveMedia: document.getElementById('btn-save-media'),
            btnCancelMedia: document.getElementById('btn-cancel-media'),
            btnEditMedia: document.getElementById('btn-edit-media'),
            
            mediaLoading: document.getElementById('media-loading'),
            mediaLoadingText: document.getElementById('media-loading-text'),
            
            addModal: document.getElementById('add-modal'),
            newExName: document.getElementById('new-exercise-name'),
            newExCat: document.getElementById('new-exercise-category'),
            
            // Novos elementos de Segurança
            securityModal: document.getElementById('security-modal'),
            securityInput: document.getElementById('security-email-input'),
            btnConfirmSecurity: document.getElementById('btn-confirm-security'),
            btnCancelSecurity: document.getElementById('btn-cancel-security'),
            btnCloseSecurity: document.getElementById('btn-close-security')
        };

        function switchView(view) {
            currentView = view;
            DOM.viewCategories.classList.add('hidden');
            DOM.viewExercises.classList.add('hidden');
            
            if (view === 'categories') {
                DOM.viewCategories.classList.remove('hidden');
                if(DOM.searchInput.value.trim() !== '') {
                    handleSearch(); 
                }
            } else if (view === 'exercises') {
                DOM.viewExercises.classList.remove('hidden');
                renderExercisesList();
            }
            
            window.scrollTo({ top: 0, behavior: 'smooth' });
        }

        function handleSearch() {
            const query = DOM.searchInput.value.toLowerCase().trim();
            
            if (query === '') {
                DOM.btnClearSearch.classList.add('hidden');
                DOM.searchResultsContainer.classList.add('hidden');
                DOM.categoriesContainer.classList.remove('hidden');
                return;
            }

            DOM.btnClearSearch.classList.remove('hidden');
            DOM.categoriesContainer.classList.add('hidden');
            DOM.searchResultsContainer.classList.remove('hidden');
            
            const allExercises = getAllExercises();
            const uniqueMap = new Map();
            allExercises.forEach(ex => {
                if (ex.name.toLowerCase().includes(query)) {
                    if (!uniqueMap.has(ex.slug) || ex.isCustom) {
                        uniqueMap.set(ex.slug, ex);
                    }
                }
            });

            const results = Array.from(uniqueMap.values());
            renderSearchResults(results);
        }

        function renderSearchResults(results) {
            DOM.searchResultsList.innerHTML = '';
            
            if (results.length === 0) {
                DOM.searchResultsList.innerHTML = `<div class="text-center py-8 text-gray-500 text-sm">Nenhum exercício encontrado.</div>`;
                return;
            }

            results.sort((a, b) => a.name.localeCompare(b.name));

            results.forEach(ex => {
                const item = document.createElement('div');
                const hasMedia = customExercisesData[ex.slug] && customExercisesData[ex.slug].media;
                
                item.className = "bg-panel border border-gray-800 rounded-xl p-4 flex items-center justify-between cursor-pointer hover:bg-gray-800 transition active:scale-[0.98]";
                item.innerHTML = `
                    <div class="flex items-center gap-4 overflow-hidden w-full">
                        <div class="w-12 h-12 rounded-lg bg-darkblue flex items-center justify-center shrink-0 border border-gray-700 shadow-inner">
                            ${hasMedia 
                                ? `<i class="fas fa-image text-accent text-lg"></i>` 
                                : `<i class="fas fa-dumbbell text-gray-600 text-lg"></i>`}
                        </div>
                        <div class="flex flex-col truncate flex-1">
                           <span class="font-semibold text-[15px] sm:text-base text-gray-100 truncate">${ex.name}</span>
                           <div class="flex items-center gap-2 mt-0.5">
                               <span class="text-[11px] text-gray-400"><i class="fas ${categoryIcons[ex.category]} text-gray-500 mr-1"></i>${ex.category}</span>
                               ${ex.isCustom ? `<span class="text-[10px] bg-primary/20 text-accent px-1.5 py-0.5 rounded">Criado</span>` : ''}
                           </div>
                        </div>
                    </div>
                    <i class="fas fa-chevron-right text-gray-600 text-sm shrink-0 ml-2"></i>
                `;
                
                item.onclick = () => openExerciseModal(ex.name, ex.slug, ex.category);
                DOM.searchResultsList.appendChild(item);
            });
        }

        DOM.searchInput.addEventListener('input', handleSearch);
        DOM.btnClearSearch.addEventListener('click', () => {
            DOM.searchInput.value = '';
            handleSearch();
        });

        function renderCategories() {
            DOM.categoriesGrid.innerHTML = '';
            
            categoryList.forEach(cat => {
                const card = document.createElement('div');
                card.className = "bg-panel border border-gray-800 rounded-2xl p-5 flex flex-col items-center justify-center text-center gap-3 cursor-pointer hover:border-primary hover:bg-gray-800 transition shadow-sm active:scale-95";
                
                const defCount = defaultData[cat].length;
                let customCount = 0;
                Object.values(customExercisesData).forEach(ex => {
                    if (ex.category === cat && ex.isCustom) customCount++;
                });

                card.innerHTML = `
                    <div class="w-14 h-14 rounded-full bg-darkblue flex items-center justify-center border border-gray-700 text-accent">
                        <i class="fas ${categoryIcons[cat] || 'fa-dumbbell'} text-2xl"></i>
                    </div>
                    <div>
                        <h3 class="font-bold text-[15px] tracking-wide text-white leading-tight">${cat}</h3>
                        <p class="text-[13px] text-gray-400 mt-1">${defCount + customCount} exs</p>
                    </div>
                `;
                
                card.onclick = () => {
                    activeCategory = cat;
                    DOM.currentCategoryTitle.innerText = cat;
                    switchView('exercises');
                };
                
                DOM.categoriesGrid.appendChild(card);
            });

            DOM.newExCat.innerHTML = categoryList.map(cat => `<option value="${cat}">${cat}</option>`).join('');
            if(DOM.searchInput.value.trim() !== '') handleSearch();
        }

        function renderExercisesList() {
            DOM.exercisesList.innerHTML = '';
            
            let exs = [...defaultData[activeCategory]].map(name => ({
                name, slug: createSlug(name), isCustom: false, category: activeCategory
            }));

            Object.keys(customExercisesData).forEach(slug => {
                const data = customExercisesData[slug];
                if (data.category === activeCategory && data.isCustom) {
                    exs.push({ name: data.name, slug: slug, isCustom: true, category: activeCategory });
                }
            });

            exs.sort((a, b) => a.name.localeCompare(b.name));
            DOM.exerciseCount.innerText = `${exs.length} exercícios`;

            if (exs.length === 0) {
                DOM.exercisesList.innerHTML = `<div class="text-center py-8 text-gray-500 text-sm">Nenhum exercício nesta categoria.</div>`;
                return;
            }

            exs.forEach(ex => {
                const item = document.createElement('div');
                const hasMedia = customExercisesData[ex.slug] && customExercisesData[ex.slug].media;
                
                item.className = "bg-panel border border-gray-800 rounded-xl p-4 flex items-center justify-between cursor-pointer hover:bg-gray-800 transition active:scale-[0.98]";
                item.innerHTML = `
                    <div class="flex items-center gap-4 overflow-hidden">
                        <div class="w-12 h-12 rounded-lg bg-darkblue flex items-center justify-center shrink-0 border border-gray-700 shadow-inner">
                            ${hasMedia 
                                ? `<i class="fas fa-image text-accent text-lg"></i>` 
                                : `<i class="fas fa-dumbbell text-gray-600 text-lg"></i>`}
                        </div>
                        <div class="flex flex-col truncate">
                           <span class="font-semibold text-[15px] sm:text-base text-gray-100 truncate">${ex.name}</span>
                           ${ex.isCustom ? `<span class="text-[11px] text-accent mt-0.5">Criado por você</span>` : ''}
                        </div>
                    </div>
                    <i class="fas fa-chevron-right text-gray-600 text-sm shrink-0 ml-2"></i>
                `;
                
                item.onclick = () => openExerciseModal(ex.name, ex.slug, ex.category);
                DOM.exercisesList.appendChild(item);
            });
        }

        // ==========================================
        // MODALS & UPLOAD PARA O FIRESTORE COM BOTÃO DE SALVAR
        // ==========================================
        function setupModalState(hasSavedMedia, mediaUrl = '') {
            if (hasSavedMedia) {
                DOM.mediaPlaceholder.classList.add('hidden');
                DOM.mediaImg.src = mediaUrl; 
                DOM.mediaImg.classList.remove('hidden');
                
                DOM.btnTriggerUpload.classList.add('hidden');
                DOM.btnEditMedia.classList.remove('hidden');
            } else {
                DOM.mediaPlaceholder.classList.remove('hidden');
                DOM.mediaImg.src = '';
                DOM.mediaImg.classList.add('hidden');
                
                DOM.btnTriggerUpload.classList.remove('hidden');
                DOM.btnEditMedia.classList.add('hidden');
            }
            
            DOM.previewActions.classList.add('hidden');
            DOM.previewActions.classList.remove('flex');
            
            pendingMediaFile = null;
            DOM.mediaInput.value = '';
        }

        function openExerciseModal(name, slug, category) {
            activeExerciseName = name;
            activeExerciseSlug = slug;
            modalActiveCategoryContext = category || activeCategory; 
            
            DOM.exModalTitle.innerText = name;
            DOM.exModalCatLabel.innerText = modalActiveCategoryContext;
            DOM.exModal.classList.remove('hidden');
            DOM.exModal.classList.add('flex');
            
            const data = customExercisesData[slug];
            setupModalState(!!(data && data.media), data?.media || '');
        }

        function closeExerciseModal() {
            DOM.exModal.classList.add('hidden');
            DOM.exModal.classList.remove('flex');
            pendingMediaFile = null;
            DOM.mediaInput.value = '';
        }

        DOM.btnTriggerUpload.onclick = () => DOM.mediaInput.click();
        
        // === FLUXO DE SEGURANÇA PARA EDIÇÃO ===
        DOM.btnEditMedia.onclick = () => {
            DOM.securityInput.value = ''; // Limpa o campo
            DOM.securityModal.classList.remove('hidden');
            DOM.securityModal.classList.add('flex');
        };

        const closeSecurityModal = () => {
            DOM.securityModal.classList.add('hidden');
            DOM.securityModal.classList.remove('flex');
        };

        DOM.btnCancelSecurity.onclick = closeSecurityModal;
        DOM.btnCloseSecurity.onclick = closeSecurityModal;

        DOM.btnConfirmSecurity.onclick = () => {
            const typedEmail = DOM.securityInput.value.trim().toLowerCase();
            const actualEmail = currentUser?.email?.toLowerCase();

            if (typedEmail === actualEmail && actualEmail !== undefined) {
                // Senha correta! Fecha a modal de segurança e abre a galeria
                closeSecurityModal();
                DOM.mediaInput.click();
            } else {
                // Senha incorreta!
                showToast("Senha secreta incorreta.", "error");
            }
        };
        // ======================================

        DOM.mediaInput.onchange = (e) => {
            const file = e.target.files[0];
            if (!file) return;

            pendingMediaFile = file;
            const localFileUrl = URL.createObjectURL(file);
            
            DOM.mediaPlaceholder.classList.add('hidden');
            DOM.mediaImg.src = localFileUrl;
            DOM.mediaImg.classList.remove('hidden');
            
            DOM.btnTriggerUpload.classList.add('hidden');
            DOM.btnEditMedia.classList.add('hidden'); 
            DOM.previewActions.classList.remove('hidden');
            DOM.previewActions.classList.add('flex');
        };

        DOM.btnCancelMedia.onclick = () => {
            const data = customExercisesData[activeExerciseSlug];
            setupModalState(!!(data && data.media), data?.media || '');
        };

        DOM.btnSaveMedia.onclick = async () => {
            if (!pendingMediaFile || !currentUser) return;

            DOM.mediaLoading.classList.remove('hidden');
            DOM.mediaLoading.style.display = 'flex';
            DOM.mediaLoadingText.innerHTML = "Otimizando arquivo...<br><span class='text-xs text-gray-300 font-normal'>Salvando no Banco de Dados...</span>";

            try {
                let base64Data;
                const MAX_SIZE = 700 * 1024; // 700KB limite seguro do Firestore

                // Se o arquivo for maior que 700KB, comprimimos para texto base64 reduzido
                if (pendingMediaFile.size > MAX_SIZE) {
                    if (pendingMediaFile.type === 'image/gif') {
                        showToast("Aviso: Como você não usa Storage, GIFs muito grandes são convertidos em imagem estática para caber no banco de dados.", "warning", 6000);
                        base64Data = await compressToBase64(pendingMediaFile, 800, 0.7); // Vira JPEG estático
                    } else {
                        base64Data = await compressToBase64(pendingMediaFile, 800, 0.7);
                    }
                } else {
                    // Arquivos pequenos vão direto (mantém o GIF animado se for pequeno)
                    base64Data = await fileToBase64(pendingMediaFile);
                }

                const docRef = doc(db, 'artifacts', systemAppId, 'users', currentUser.uid, 'exercises', activeExerciseSlug);
                const isCustom = customExercisesData[activeExerciseSlug]?.isCustom || false;
                
                await setDoc(docRef, {
                    name: activeExerciseName,
                    category: modalActiveCategoryContext,
                    media: base64Data, // Salva o texto Base64 em vez da URL do Storage
                    isCustom: isCustom
                }, { merge: true });
                
                showToast("Imagem salva com sucesso!", "success");
                setupModalState(true, base64Data);

            } catch (error) {
                console.error("Erro ao Salvar:", error);
                
                // Se der erro de limite excedido mesmo após a compressão
                if (error.code === 'resource-exhausted' || (error.message && error.message.includes('exceeds the maximum'))) {
                    showToast("Erro: Imagem ainda é muito grande para o Banco de Dados. Escolha uma foto mais leve.", "error", 5000);
                } else {
                    showToast("Erro ao guardar. Verifique sua conexão.", "error");
                }
            } finally {
                DOM.mediaLoading.classList.add('hidden');
                DOM.mediaLoading.style.display = 'none';
            }
        };

        // ==========================================
        // ADICIONAR EXERCÍCIO
        // ==========================================
        document.getElementById('btn-open-add-modal').onclick = () => {
            DOM.newExCat.value = activeCategory;
            DOM.newExName.value = '';
            DOM.addModal.classList.remove('hidden');
            DOM.addModal.classList.add('flex');
        };

        document.getElementById('btn-close-add-modal').onclick = () => {
            DOM.addModal.classList.add('hidden');
            DOM.addModal.classList.remove('flex');
        };

        document.getElementById('btn-save-exercise').onclick = async () => {
            if (!currentUser) return;
            const name = DOM.newExName.value.trim();
            const category = DOM.newExCat.value;
            
            if (!name) {
                showToast("Escreva o nome do exercício.", "error");
                return;
            }

            const slug = createSlug(name);
            const docRef = doc(db, 'artifacts', systemAppId, 'users', currentUser.uid, 'exercises', slug);

            try {
                await setDoc(docRef, {
                    name: name,
                    category: category,
                    isCustom: true
                }, { merge: true });
                
                showToast("Exercício criado com sucesso!", "success");
                DOM.addModal.classList.add('hidden');
                DOM.addModal.classList.remove('flex');
            } catch (error) {
                console.error(error);
                showToast("Erro ao criar exercício.", "error");
            }
        };

        document.getElementById('btn-close-exercise-modal').onclick = closeExerciseModal;
        document.getElementById('btn-back-categories').onclick = () => switchView('categories');

        // ==========================================
        // AUTENTICAÇÃO E DADOS
        // ==========================================
        async function initApp() {
            if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                try {
                    await signInWithCustomToken(auth, __initial_auth_token);
                } catch(e) { console.error("Auth Error:", e); }
            }
            
            onAuthStateChanged(auth, (user) => {
                if (user) {
                    currentUser = user;
                    if (user.photoURL) {
                        DOM.userAvatar.src = user.photoURL;
                        DOM.userAvatar.classList.remove('hidden');
                    }
                    
                    DOM.loginScreen.classList.add('hidden');
                    DOM.loginScreen.classList.remove('flex');
                    DOM.mainApp.classList.remove('hidden');
                    DOM.mainApp.classList.add('flex');
                    
                    setupDataListener();
                } else {
                    currentUser = null;
                    if (userDataListener) userDataListener(); 
                    DOM.mainApp.classList.add('hidden');
                    DOM.mainApp.classList.remove('flex');
                    DOM.loginScreen.classList.remove('hidden');
                    DOM.loginScreen.classList.add('flex');
                    DOM.userAvatar.classList.add('hidden');
                }
            });
        }

        document.getElementById('btn-google-login').onclick = async () => {
            document.getElementById('login-loading').classList.remove('hidden');
            const provider = new GoogleAuthProvider();
            try {
                await signInWithPopup(auth, provider);
            } catch (error) {
                console.error("Login Error:", error);
                showToast("Falha no login com Google.", "error");
            } finally {
                document.getElementById('login-loading').classList.add('hidden');
            }
        };

        document.getElementById('btn-logout').onclick = () => signOut(auth);

        function setupDataListener() {
            if (!currentUser) return;
            const userExercisesRef = collection(db, 'artifacts', systemAppId, 'users', currentUser.uid, 'exercises');
            
            userDataListener = onSnapshot(userExercisesRef, (snapshot) => {
                customExercisesData = {};
                snapshot.forEach(doc => {
                    customExercisesData[doc.id] = doc.data();
                });
                
                if (currentView === 'categories') {
                    renderCategories();
                } else if (currentView === 'exercises') {
                    renderExercisesList();
                }
            }, (error) => {
                console.error("Erro ao ler banco de dados:", error);
                if(error.code === 'permission-denied') {
                    showToast("Ative as Regras do Firestore para sincronizar.", "error", 6000);
                } else {
                    showToast("Erro ao sincronizar dados.", "error");
                }
            });
        }

        initApp();
        renderCategories();

    </script>
</body>
</html>
