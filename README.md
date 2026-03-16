# ai-presentasi<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI Presentation Generator Pro</title>
    
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    
    <!-- GSAP untuk Animasi -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.2/gsap.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.2/Draggable.min.js"></script>
    
    <!-- PDF.js untuk parsing PDF -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.11.174/pdf.min.js"></script>
    <script>
        pdfjsLib.GlobalWorkerOptions.workerSrc = 'https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.11.174/pdf.worker.min.js';
    </script>
    
    <!-- Mammoth.js untuk parsing DOCX -->
    <script src="https://cdn.jsdelivr.net/npm/mammoth@1.6.0/mammoth.browser.min.js"></script>
    
    <!-- Turndown.js untuk HTML ke Markdown -->
    <script src="https://unpkg.com/turndown/dist/turndown.js"></script>
    
    <!-- Font Awesome -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    
    <!-- Google Fonts -->
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800&family=Playfair+Display:wght@600;700&display=swap" rel="stylesheet">

    <script>
        tailwind.config = {
            theme: {
                extend: {
                    fontFamily: {
                        sans: ['Inter', 'sans-serif'],
                        serif: ['Playfair Display', 'serif'],
                    },
                    colors: {
                        glass: 'rgba(255, 255, 255, 0.1)',
                        glassBorder: 'rgba(255, 255, 255, 0.2)',
                    }
                }
            }
        }
    </script>

    <style>
        body {
            overflow: hidden;
            background: #0f172a;
        }
        
        .glass-panel {
            background: rgba(255, 255, 255, 0.05);
            backdrop-filter: blur(16px);
            -webkit-backdrop-filter: blur(16px);
            border: 1px solid rgba(255, 255, 255, 0.1);
        }
        
        .slide-preview {
            aspect-ratio: 16/9;
            transition: all 0.3s ease;
        }
        
        .slide-preview.active {
            border-color: #6366f1;
            box-shadow: 0 0 0 2px #6366f1;
        }
        
        .gradient-bg {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
        }
        
        .gradient-dark {
            background: linear-gradient(135deg, #1e293b 0%, #0f172a 100%);
        }
        
        .gradient-light {
            background: linear-gradient(135deg, #f8fafc 0%, #e2e8f0 100%);
        }
        
        .gradient-nature {
            background: linear-gradient(135deg, #11998e 0%, #38ef7d 100%);
        }
        
        .gradient-corporate {
            background: linear-gradient(135deg, #373B44 0%, #4286f4 100%);
        }
        
        .gradient-minimal {
            background: linear-gradient(135deg, #232526 0%, #414345 100%);
        }
        
        .shape {
            cursor: move;
            position: absolute;
            user-select: none;
        }
        
        .editable:focus {
            outline: 2px solid #6366f1;
            outline-offset: 2px;
        }
        
        .presentation-mode {
            position: fixed;
            top: 0;
            left: 0;
            width: 100vw;
            height: 100vh;
            z-index: 9999;
            background: black;
        }
        
        .drop-zone {
            border: 2px dashed rgba(255, 255, 255, 0.3);
            transition: all 0.3s ease;
        }
        
        .drop-zone.dragover {
            border-color: #6366f1;
            background: rgba(99, 102, 241, 0.1);
        }
        
        .loading-spinner {
            border: 3px solid rgba(255, 255, 255, 0.1);
            border-left-color: #6366f1;
            border-radius: 50%;
            width: 24px;
            height: 24px;
            animation: spin 1s linear infinite;
        }
        
        @keyframes spin {
            to { transform: rotate(360deg); }
        }
        
        .scrollbar-hide::-webkit-scrollbar {
            display: none;
        }
        .scrollbar-hide {
            -ms-overflow-style: none;
            scrollbar-width: none;
        }
        
        .font-bold { font-weight: 700; }
        .font-italic { font-style: italic; }
    </style>
</head>
<body class="text-white h-screen flex flex-col">

    <!-- Header -->
    <header class="glass-panel border-b border-white/10 px-6 py-4 flex items-center justify-between z-50">
        <div class="flex items-center gap-3">
            <div class="w-10 h-10 bg-gradient-to-br from-indigo-500 to-purple-600 rounded-xl flex items-center justify-center shadow-lg">
                <i class="fas fa-presentation text-white text-lg"></i>
            </div>
            <div>
                <h1 class="font-bold text-xl tracking-tight">AI Presentation Generator</h1>
                <p class="text-xs text-gray-400">Pro Version with Document Parser</p>
            </div>
        </div>

        <div class="flex items-center gap-4">
            <!-- Input Topik -->
            <div class="relative group">
                <input type="text" id="topicInput" placeholder="Masukkan topik presentasi..." 
                    class="bg-white/5 border border-white/10 rounded-lg px-4 py-2 w-64 focus:outline-none focus:border-indigo-500 transition-all text-sm">
                <button onclick="generateFromTopic()" 
                    class="absolute right-1 top-1 bottom-1 bg-indigo-600 hover:bg-indigo-700 px-3 rounded-md text-xs font-medium transition-colors">
                    Generate
                </button>
            </div>

            <div class="h-6 w-px bg-white/10"></div>

            <!-- Upload & URL -->
            <button onclick="toggleImportModal()" class="flex items-center gap-2 px-4 py-2 bg-white/5 hover:bg-white/10 rounded-lg text-sm transition-colors border border-white/10">
                <i class="fas fa-file-import"></i>
                <span>Import</span>
            </button>

            <button onclick="togglePresentationMode()" class="flex items-center gap-2 px-4 py-2 bg-indigo-600 hover:bg-indigo-700 rounded-lg text-sm font-medium transition-colors shadow-lg shadow-indigo-500/30">
                <i class="fas fa-play"></i>
                <span>Presentasi</span>
            </button>

            <button onclick="exportHTML()" class="flex items-center gap-2 px-4 py-2 bg-emerald-600 hover:bg-emerald-700 rounded-lg text-sm font-medium transition-colors">
                <i class="fas fa-download"></i>
                <span>Export</span>
            </button>
        </div>
    </header>

    <!-- Main Content -->
    <div class="flex-1 flex overflow-hidden">
        
        <!-- Sidebar Kiri: Thumbnails -->
        <aside class="w-64 glass-panel border-r border-white/10 flex flex-col">
            <div class="p-4 border-b border-white/10 flex justify-between items-center">
                <span class="text-sm font-medium text-gray-300">Slides</span>
                <div class="flex gap-2">
                    <button onclick="addSlide()" class="w-8 h-8 rounded-lg bg-white/5 hover:bg-white/10 flex items-center justify-center transition-colors" title="Tambah Slide">
                        <i class="fas fa-plus text-xs"></i>
                    </button>
                    <button onclick="deleteSlide()" class="w-8 h-8 rounded-lg bg-white/5 hover:bg-red-500/20 hover:text-red-400 flex items-center justify-center transition-colors" title="Hapus Slide">
                        <i class="fas fa-trash text-xs"></i>
                    </button>
                </div>
            </div>
            
            <div id="thumbnailContainer" class="flex-1 overflow-y-auto p-4 space-y-3 scrollbar-hide">
                <!-- Thumbnails akan di-generate di sini -->
            </div>
            
            <div class="p-4 border-t border-white/10 text-center text-xs text-gray-500">
                <span id="slideCounter">1 / 5</span>
            </div>
        </aside>

        <!-- Area Utama: Editor -->
        <main class="flex-1 flex flex-col bg-black/20 relative">
            
            <!-- Toolbar -->
            <div class="glass-panel border-b border-white/10 px-6 py-3 flex items-center gap-6 overflow-x-auto">
                
                <!-- Tema -->
                <div class="flex items-center gap-2">
                    <span class="text-xs text-gray-400 uppercase tracking-wider font-medium">Tema</span>
                    <div class="flex gap-1">
                        <button onclick="setTheme('gradient')" class="w-8 h-8 rounded-lg gradient-bg border-2 border-transparent hover:border-white transition-all" title="Gradient"></button>
                        <button onclick="setTheme('dark')" class="w-8 h-8 rounded-lg gradient-dark border-2 border-transparent hover:border-white transition-all" title="Dark"></button>
                        <button onclick="setTheme('light')" class="w-8 h-8 rounded-lg gradient-light border-2 border-transparent hover:border-white transition-all" title="Light"></button>
                        <button onclick="setTheme('corporate')" class="w-8 h-8 rounded-lg gradient-corporate border-2 border-transparent hover:border-white transition-all" title="Corporate"></button>
                        <button onclick="setTheme('minimal')" class="w-8 h-8 rounded-lg gradient-minimal border-2 border-transparent hover:border-white transition-all" title="Minimal"></button>
                        <button onclick="setTheme('nature')" class="w-8 h-8 rounded-lg gradient-nature border-2 border-transparent hover:border-white transition-all" title="Nature"></button>
                    </div>
                </div>

                <div class="w-px h-6 bg-white/10"></div>

                <!-- Layout -->
                <div class="flex items-center gap-2">
                    <span class="text-xs text-gray-400 uppercase tracking-wider font-medium">Layout</span>
                    <select id="layoutSelect" onchange="changeLayout(this.value)" class="bg-white/5 border border-white/10 rounded px-3 py-1.5 text-sm focus:outline-none focus:border-indigo-500">
                        <option value="title">Title</option>
                        <option value="content">Content</option>
                        <option value="split">Split</option>
                        <option value="image">Image</option>
                    </select>
                </div>

                <div class="w-px h-6 bg-white/10"></div>

                <!-- Warna Accent -->
                <div class="flex items-center gap-2">
                    <span class="text-xs text-gray-400 uppercase tracking-wider font-medium">Accent</span>
                    <div class="flex gap-1">
                        <button onclick="setAccent('indigo')" class="w-6 h-6 rounded-full bg-indigo-500 border-2 border-transparent hover:border-white transition-all"></button>
                        <button onclick="setAccent('pink')" class="w-6 h-6 rounded-full bg-pink-500 border-2 border-transparent hover:border-white transition-all"></button>
                        <button onclick="setAccent('violet')" class="w-6 h-6 rounded-full bg-violet-500 border-2 border-transparent hover:border-white transition-all"></button>
                        <button onclick="setAccent('emerald')" class="w-6 h-6 rounded-full bg-emerald-500 border-2 border-transparent hover:border-white transition-all"></button>
                        <button onclick="setAccent('amber')" class="w-6 h-6 rounded-full bg-amber-500 border-2 border-transparent hover:border-white transition-all"></button>
                    </div>
                </div>

                <div class="w-px h-6 bg-white/10"></div>

                <!-- Animasi -->
                <div class="flex items-center gap-2">
                    <span class="text-xs text-gray-400 uppercase tracking-wider font-medium">Animasi</span>
                    <select id="animationSelect" onchange="setAnimation(this.value)" class="bg-white/5 border border-white/10 rounded px-3 py-1.5 text-sm focus:outline-none focus:border-indigo-500">
                        <option value="fade">Fade</option>
                        <option value="slideUp">Slide Up</option>
                        <option value="scale">Scale</option>
                        <option value="rotate">Rotate</option>
                    </select>
                </div>

                <div class="w-px h-6 bg-white/10"></div>

                <!-- Shapes -->
                <div class="flex items-center gap-2">
                    <span class="text-xs text-gray-400 uppercase tracking-wider font-medium">Shapes</span>
                    <button onclick="addShape('circle')" class="w-8 h-8 rounded-lg bg-white/5 hover:bg-white/10 flex items-center justify-center transition-colors" title="Lingkaran">
                        <i class="far fa-circle text-xs"></i>
                    </button>
                    <button onclick="addShape('square')" class="w-8 h-8 rounded-lg bg-white/5 hover:bg-white/10 flex items-center justify-center transition-colors" title="Persegi">
                        <i class="far fa-square text-xs"></i>
                    </button>
                    <button onclick="addShape('triangle')" class="w-8 h-8 rounded-lg bg-white/5 hover:bg-white/10 flex items-center justify-center transition-colors" title="Segitiga">
                        <i class="fas fa-play text-xs -rotate-90"></i>
                    </button>
                </div>

                <div class="w-px h-6 bg-white/10"></div>

                <!-- Format Teks -->
                <div class="flex items-center gap-1">
                    <button onclick="toggleFormat('bold')" class="w-8 h-8 rounded-lg bg-white/5 hover:bg-white/10 flex items-center justify-center transition-colors font-bold" title="Bold">
                        B
                    </button>
                    <button onclick="toggleFormat('italic')" class="w-8 h-8 rounded-lg bg-white/5 hover:bg-white/10 flex items-center justify-center transition-colors italic" title="Italic">
                        I
                    </button>
                </div>
            </div>

            <!-- Canvas Area -->
            <div class="flex-1 flex items-center justify-center p-8 overflow-auto bg-black/40 relative" id="canvasContainer">
                <div id="currentSlide" class="relative w-full max-w-5xl aspect-video rounded-xl shadow-2xl overflow-hidden transition-all duration-500 gradient-bg">
                    <!-- Konten slide akan di-render di sini -->
                </div>
            </div>

            <!-- Properties Panel (Floating) -->
            <div class="absolute right-6 top-24 w-72 glass-panel rounded-xl border border-white/10 p-4 space-y-4 max-h-[calc(100vh-8rem)] overflow-y-auto">
                <h3 class="font-semibold text-sm border-b border-white/10 pb-2">Properties</h3>
                
                <div class="space-y-3">
                    <div>
                        <label class="text-xs text-gray-400 block mb-1">Judul Slide</label>
                        <input type="text" id="propTitle" oninput="updateSlideTitle(this.value)" class="w-full bg-white/5 border border-white/10 rounded px-3 py-2 text-sm focus:outline-none focus:border-indigo-500">
                    </div>
                    
                    <div>
                        <label class="text-xs text-gray-400 block mb-1">Subjudul</label>
                        <input type="text" id="propSubtitle" oninput="updateSlideSubtitle(this.value)" class="w-full bg-white/5 border border-white/10 rounded px-3 py-2 text-sm focus:outline-none focus:border-indigo-500">
                    </div>

                    <div>
                        <label class="text-xs text-gray-400 block mb-1">Konten</label>
                        <textarea id="propContent" oninput="updateSlideContent(this.value)" rows="4" class="w-full bg-white/5 border border-white/10 rounded px-3 py-2 text-sm focus:outline-none focus:border-indigo-500 resize-none"></textarea>
                    </div>

                    <div>
                        <label class="text-xs text-gray-400 block mb-1">Notes</label>
                        <textarea id="propNotes" placeholder="Catatan untuk presenter..." rows="3" class="w-full bg-white/5 border border-white/10 rounded px-3 py-2 text-sm focus:outline-none focus:border-indigo-500 resize-none text-gray-400"></textarea>
                    </div>
                </div>

                <div class="pt-2 border-t border-white/10">
                    <button onclick="applyAIToCurrentSlide()" class="w-full py-2 bg-gradient-to-r from-indigo-600 to-purple-600 rounded-lg text-sm font-medium hover:opacity-90 transition-opacity flex items-center justify-center gap-2">
                        <i class="fas fa-magic"></i>
                        <span>Enhance with AI</span>
                    </button>
                </div>
            </div>
        </main>
    </div>

    <!-- Import Modal -->
    <div id="importModal" class="fixed inset-0 bg-black/80 backdrop-blur-sm z-[100] hidden flex items-center justify-center">
        <div class="glass-panel rounded-2xl border border-white/20 w-full max-w-2xl p-6 m-4">
            <div class="flex justify-between items-center mb-6">
                <h2 class="text-xl font-bold">Import Konten</h2>
                <button onclick="toggleImportModal()" class="w-8 h-8 rounded-lg bg-white/5 hover:bg-white/10 flex items-center justify-center">
                    <i class="fas fa-times"></i>
                </button>
            </div>

            <div class="grid grid-cols-2 gap-6">
                <!-- Upload File -->
                <div class="space-y-4">
                    <h3 class="font-medium text-sm text-gray-300 flex items-center gap-2">
                        <i class="fas fa-file-upload"></i>
                        Upload Dokumen
                    </h3>
                    
                    <div id="dropZone" class="drop-zone rounded-xl p-8 text-center cursor-pointer" ondrop="handleDrop(event)" ondragover="handleDragOver(event)" ondragleave="handleDragLeave(event)" onclick="document.getElementById('fileInput').click()">
                        <input type="file" id="fileInput" class="hidden" accept=".pdf,.docx,.txt" onchange="handleFileSelect(event)">
                        <i class="fas fa-cloud-upload-alt text-4xl text-gray-400 mb-3"></i>
                        <p class="text-sm text-gray-300 mb-1">Drag & drop file di sini</p>
                        <p class="text-xs text-gray-500">atau klik untuk browse</p>
                        <p class="text-xs text-gray-600 mt-2">Mendukung PDF, DOCX, TXT</p>
                    </div>

                    <div id="uploadProgress" class="hidden">
                        <div class="flex items-center gap-2 text-sm text-gray-300">
                            <div class="loading-spinner"></div>
                            <span id="uploadStatus">Memproses dokumen...</span>
                        </div>
                    </div>
                </div>

                <!-- URL Input -->
                <div class="space-y-4">
                    <h3 class="font-medium text-sm text-gray-300 flex items-center gap-2">
                        <i class="fas fa-link"></i>
                        Scraping URL
                    </h3>
                    
                    <div class="space-y-3">
                        <input type="url" id="urlInput" placeholder="https://example.com/article" class="w-full bg-white/5 border border-white/10 rounded-lg px-4 py-3 text-sm focus:outline-none focus:border-indigo-500">
                        
                        <div class="flex gap-2">
                            <button onclick="scrapeURL()" class="flex-1 bg-indigo-600 hover:bg-indigo-700 py-2 rounded-lg text-sm font-medium transition-colors">
                                Scraping
                            </button>
                            <button onclick="fetchAndParse()" class="flex-1 bg-white/5 hover:bg-white/10 border border-white/10 py-2 rounded-lg text-sm transition-colors">
                                Fetch & Parse
                            </button>
                        </div>

                        <div class="bg-yellow-500/10 border border-yellow-500/20 rounded-lg p-3 text-xs text-yellow-200">
                            <i class="fas fa-info-circle mr-1"></i>
                            Note: URL scraping menggunakan CORS proxy. Beberapa website mungkin diblokir.
                        </div>

                        <div id="urlProgress" class="hidden">
                            <div class="flex items-center gap-2 text-sm text-gray-300">
                                <div class="loading-spinner"></div>
                                <span>Mengambil konten...</span>
                            </div>
                        </div>
                    </div>
                </div>
            </div>

            <!-- Preview Hasil Parsing -->
            <div id="parsedContent" class="mt-6 hidden">
                <h3 class="font-medium text-sm text-gray-300 mb-2">Preview Konten:</h3>
                <div class="bg-black/30 rounded-lg p-4 max-h-48 overflow-y-auto text-xs text-gray-400 font-mono whitespace-pre-wrap" id="parsedText"></div>
                
                <div class="mt-4 flex justify-end gap-2">
                    <button onclick="generateSlidesFromContent()" class="bg-emerald-600 hover:bg-emerald-700 px-6 py-2 rounded-lg text-sm font-medium transition-colors">
                        Generate Slide dari Konten
                    </button>
                </div>
            </div>
        </div>
    </div>

    <!-- Presentation Mode Overlay -->
    <div id="presentationMode" class="presentation-mode hidden flex items-center justify-center bg-black">
        <div id="presentationSlide" class="w-full h-full flex items-center justify-center p-12">
            <!-- Slide akan di-clone ke sini -->
        </div>
        
        <div class="absolute bottom-8 left-1/2 -translate-x-1/2 flex items-center gap-4 glass-panel px-6 py-3 rounded-full">
            <button onclick="prevSlide()" class="w-10 h-10 rounded-full bg-white/10 hover:bg-white/20 flex items-center justify-center transition-colors">
                <i class="fas fa-chevron-left"></i>
            </button>
            <span id="presentationCounter" class="text-sm font-medium min-w-[60px] text-center">1 / 5</span>
            <button onclick="nextSlide()" class="w-10 h-10 rounded-full bg-white/10 hover:bg-white/20 flex items-center justify-center transition-colors">
                <i class="fas fa-chevron-right"></i>
            </button>
            <div class="w-px h-6 bg-white/20 mx-2"></div>
            <button onclick="togglePresentationMode()" class="text-sm text-gray-400 hover:text-white transition-colors">
                ESC untuk keluar
            </button>
        </div>
    </div>

    <script>
        // State Management
        let slides = [];
        let currentSlideIndex = 0;
        let currentTheme = 'gradient';
        let currentAccent = 'indigo';
        let currentAnimation = 'fade';
        let parsedDocumentContent = '';
        
        // Inisialisasi
        document.addEventListener('DOMContentLoaded', () => {
            // Generate 5 slide default
            generateDefaultSlides();
            renderThumbnails();
            loadSlide(0);
            
            // Keyboard navigation
            document.addEventListener('keydown', (e) => {
                if (document.getElementById('presentationMode').classList.contains('hidden')) {
                    if (e.key === 'ArrowLeft') prevSlide();
                    if (e.key === 'ArrowRight') nextSlide();
                    if (e.key === 'F5' || (e.ctrlKey && e.key === 'f')) {
                        e.preventDefault();
                        togglePresentationMode();
                    }
                } else {
                    if (e.key === 'Escape') togglePresentationMode();
                    if (e.key === 'ArrowLeft') prevSlide();
                    if (e.key === 'ArrowRight') nextSlide();
                }
            });
        });

        // Default Slides
        function generateDefaultSlides() {
            slides = [
                {
                    id: 1,
                    layout: 'title',
                    theme: 'gradient',
                    accent: 'indigo',
                    animation: 'fade',
                    title: 'Selamat Datang',
                    subtitle: 'AI Presentation Generator Pro',
                    content: 'Buat presentasi menakjubkan dengan AI',
                    shapes: [],
                    notes: ''
                },
                {
                    id: 2,
                    layout: 'content',
                    theme: 'gradient',
                    accent: 'indigo',
                    animation: 'slideUp',
                    title: 'Pendahuluan',
                    subtitle: '',
                    content: '• Fitur lengkap dengan AI\n• Upload dokumen PDF, DOCX, TXT\n• Scraping URL otomatis\n• Drag & drop shapes',
                    shapes: [],
                    notes: ''
                },
                {
                    id: 3,
                    layout: 'split',
                    theme: 'gradient',
                    accent: 'indigo',
                    animation: 'scale',
                    title: 'Poin Utama',
                    subtitle: 'Teknologi Modern',
                    content: 'Left: Visualisasi data real-time\nRight: Integrasi AI canggih',
                    shapes: [],
                    notes: ''
                },
                {
                    id: 4,
                    layout: 'image',
                    theme: 'gradient',
                    accent: 'indigo',
                    animation: 'rotate',
                    title: 'Studi Kasus',
                    subtitle: 'Implementasi',
                    content: 'Contoh penggunaan dalam industri',
                    shapes: [],
                    notes: ''
                },
                {
                    id: 5,
                    layout: 'content',
                    theme: 'gradient',
                    accent: 'indigo',
                    animation: 'fade',
                    title: 'Kesimpulan',
                    subtitle: '',
                    content: '• Efisiensi waktu\n• Kualitas profesional\n• Mudah digunakan',
                    shapes: [],
                    notes: ''
                }
            ];
        }

        // Render Thumbnails
        function renderThumbnails() {
            const container = document.getElementById('thumbnailContainer');
            container.innerHTML = '';
            
            slides.forEach((slide, index) => {
                const thumb = document.createElement('div');
                thumb.className = `slide-preview rounded-lg border-2 border-transparent cursor-pointer overflow-hidden relative ${index === currentSlideIndex ? 'active' : 'border-white/10'}`;
                thumb.onclick = () => loadSlide(index);
                
                const miniSlide = document.createElement('div');
                miniSlide.className = `w-full h-full ${getThemeClass(slide.theme)} p-2 flex flex-col justify-center items-center text-center`;
                miniSlide.style.transform = 'scale(0.2)';
                miniSlide.style.transformOrigin = 'center';
                miniSlide.style.width = '500%';
                miniSlide.style.height = '500%';
                miniSlide.style.margin = '-200%';
                
                miniSlide.innerHTML = `
                    <div class="text-white font-bold text-4xl mb-2 truncate w-full px-4">${slide.title}</div>
                    <div class="text-white/70 text-2xl truncate w-full px-4">${slide.subtitle}</div>
                `;
                
                thumb.appendChild(miniSlide);
                container.appendChild(thumb);
            });
            
            document.getElementById('slideCounter').textContent = `${currentSlideIndex + 1} / ${slides.length}`;
        }

        // Load Slide ke Editor
        function loadSlide(index) {
            currentSlideIndex = index;
            const slide = slides[index];
            
            // Update form properties
            document.getElementById('propTitle').value = slide.title;
            document.getElementById('propSubtitle').value = slide.subtitle;
            document.getElementById('propContent').value = slide.content;
            document.getElementById('propNotes').value = slide.notes || '';
            document.getElementById('layoutSelect').value = slide.layout;
            document.getElementById('animationSelect').value = slide.animation;
            
            renderCurrentSlide();
            renderThumbnails();
        }

        // Render Current Slide
        function renderCurrentSlide() {
            const slide = slides[currentSlideIndex];
            const container = document.getElementById('currentSlide');
            
            container.className = `relative w-full max-w-5xl aspect-video rounded-xl shadow-2xl overflow-hidden transition-all duration-500 ${getThemeClass(slide.theme)}`;
            
            let contentHTML = '';
            
            switch(slide.layout) {
                case 'title':
                    contentHTML = `
                        <div class="absolute inset-0 flex flex-col items-center justify-center p-16 text-center animate-${slide.animation}">
                            <h1 class="editable text-5xl md:text-6xl font-bold text-white mb-6 leading-tight" onclick="makeEditable(this, 'title')">${slide.title}</h1>
                            <h2 class="editable text-2xl md:text-3xl text-white/80 font-light" onclick="makeEditable(this, 'subtitle')">${slide.subtitle}</h2>
                            <p class="editable mt-8 text-lg text-white/60 max-w-2xl" onclick="makeEditable(this, 'content')">${slide.content}</p>
                        </div>
                    `;
                    break;
                case 'content':
                    contentHTML = `
                        <div class="absolute inset-0 flex flex-col p-16 animate-${slide.animation}">
                            <h1 class="editable text-4xl md:text-5xl font-bold text-white mb-8" onclick="makeEditable(this, 'title')">${slide.title}</h1>
                            <div class="editable text-xl text-white/90 whitespace-pre-line leading-relaxed" onclick="makeEditable(this, 'content')">${slide.content}</div>
                        </div>
                    `;
                    break;
                case 'split':
                    contentHTML = `
                        <div class="absolute inset-0 flex animate-${slide.animation}">
                            <div class="w-1/2 p-12 flex flex-col justify-center border-r border-white/10">
                                <h1 class="editable text-3xl md:text-4xl font-bold text-white mb-4" onclick="makeEditable(this, 'title')">${slide.title}</h1>
                                <p class="editable text-lg text-white/70" onclick="makeEditable(this, 'subtitle')">${slide.subtitle}</p>
                            </div>
                            <div class="w-1/2 p-12 flex items-center">
                                <div class="editable text-xl text-white/90 whitespace-pre-line" onclick="makeEditable(this, 'content')">${slide.content}</div>
                            </div>
                        </div>
                    `;
                    break;
                case 'image':
                    contentHTML = `
                        <div class="absolute inset-0 flex animate-${slide.animation}">
                            <div class="w-2/3 p-16 flex flex-col justify-center">
                                <h1 class="editable text-4xl md:text-5xl font-bold text-white mb-6" onclick="makeEditable(this, 'title')">${slide.title}</h1>
                                <p class="editable text-xl text-white/80" onclick="makeEditable(this, 'content')">${slide.content}</p>
                            </div>
                            <div class="w-1/3 bg-black/20 flex items-center justify-center">
                                <i class="fas fa-image text-6xl text-white/20"></i>
                            </div>
                        </div>
                    `;
                    break;
            }
            
            // Render shapes
            slide.shapes.forEach((shape, idx) => {
                const shapeEl = document.createElement('div');
                shapeEl.className = 'shape';
                shapeEl.style.left = shape.x + 'px';
                shapeEl.style.top = shape.y + 'px';
                shapeEl.style.width = shape.size + 'px';
                shapeEl.style.height = shape.size + 'px';
                
                if (shape.type === 'circle') {
                    shapeEl.style.borderRadius = '50%';
                    shapeEl.style.backgroundColor = getAccentColor(slide.accent);
                } else if (shape.type === 'square') {
                    shapeEl.style.backgroundColor = getAccentColor(slide.accent);
                } else if (shape.type === 'triangle') {
                    shapeEl.style.width = '0';
                    shapeEl.style.height = '0';
                    shapeEl.style.borderLeft = `${shape.size/2}px solid transparent`;
                    shapeEl.style.borderRight = `${shape.size/2}px solid transparent`;
                    shapeEl.style.borderBottom = `${shape.size}px solid ${getAccentColor(slide.accent)}`;
                    shapeEl.style.backgroundColor = 'transparent';
                }
                
                contentHTML += shapeEl.outerHTML;
            });
            
            container.innerHTML = contentHTML;
            
            // Apply GSAP animation
            gsap.fromTo(container.children, 
                { opacity: 0, y: 20, scale: 0.95 },
                { opacity: 1, y: 0, scale: 1, duration: 0.5, stagger: 0.1, ease: "power2.out" }
            );
            
            // Init draggable shapes
            initDraggableShapes();
        }

        // Theme Classes
        function getThemeClass(theme) {
            const classes = {
                'gradient': 'gradient-bg',
                'dark': 'gradient-dark',
                'light': 'gradient-light text-gray-900',
                'corporate': 'gradient-corporate',
                'minimal': 'gradient-minimal',
                'nature': 'gradient-nature'
            };
            return classes[theme] || classes['gradient'];
        }

        function getAccentColor(accent) {
            const colors = {
                'indigo': '#6366f1',
                'pink': '#ec4899',
                'violet': '#8b5cf6',
                'emerald': '#10b981',
                'amber': '#f59e0b'
            };
            return colors[accent] || colors['indigo'];
        }

        // Editing Functions
        function makeEditable(element, field) {
            element.contentEditable = true;
            element.focus();
            
            element.addEventListener('blur', function() {
                element.contentEditable = false;
                if (field === 'title') slides[currentSlideIndex].title = element.textContent;
                if (field === 'subtitle') slides[currentSlideIndex].subtitle = element.textContent;
                if (field === 'content') slides[currentSlideIndex].content = element.textContent;
                
                // Update properties panel
                if (field === 'title') document.getElementById('propTitle').value = element.textContent;
                if (field === 'subtitle') document.getElementById('propSubtitle').value = element.textContent;
                if (field === 'content') document.getElementById('propContent').value = element.textContent;
                
                renderThumbnails();
            });
        }

        function updateSlideTitle(val) {
            slides[currentSlideIndex].title = val;
            renderCurrentSlide();
            renderThumbnails();
        }

        function updateSlideSubtitle(val) {
            slides[currentSlideIndex].subtitle = val;
            renderCurrentSlide();
        }

        function updateSlideContent(val) {
            slides[currentSlideIndex].content = val;
            renderCurrentSlide();
        }

        // Theme & Layout
        function setTheme(theme) {
            currentTheme = theme;
            slides[currentSlideIndex].theme = theme;
            renderCurrentSlide();
            renderThumbnails();
        }

        function setAccent(accent) {
            currentAccent = accent;
            slides[currentSlideIndex].accent = accent;
            renderCurrentSlide();
        }

        function setAnimation(anim) {
            currentAnimation = anim;
            slides[currentSlideIndex].animation = anim;
        }

        function changeLayout(layout) {
            slides[currentSlideIndex].layout = layout;
            renderCurrentSlide();
        }

        // Shapes
        function addShape(type) {
            const shape = {
                type: type,
                x: 100,
                y: 100,
                size: type === 'triangle' ? 80 : 80
            };
            slides[currentSlideIndex].shapes.push(shape);
            renderCurrentSlide();
        }

        function initDraggableShapes() {
            const shapes = document.querySelectorAll('.shape');
            shapes.forEach((shape, index) => {
                Draggable.create(shape, {
                    bounds: "#currentSlide",
                    onDragEnd: function() {
                        slides[currentSlideIndex].shapes[index].x = this.x;
                        slides[currentSlideIndex].shapes[index].y = this.y;
                    }
                });
            });
        }

        // Format Teks
        function toggleFormat(format) {
            const selection = window.getSelection();
            if (!selection.rangeCount) return;
            
            const range = selection.getRangeAt(0);
            const selectedText = range.toString();
            
            if (selectedText) {
                const span = document.createElement('span');
                if (format === 'bold') span.className = 'font-bold';
                if (format === 'italic') span.className = 'italic';
                span.textContent = selectedText;
                
                range.deleteContents();
                range.insertNode(span);
            }
        }

        // Slide Management
        function addSlide() {
            const newSlide = {
                id: slides.length + 1,
                layout: 'content',
                theme: currentTheme,
                accent: currentAccent,
                animation: 'fade',
                title: 'Slide Baru',
                subtitle: 'Subjudul',
                content: 'Konten slide...',
                shapes: [],
                notes: ''
            };
            slides.push(newSlide);
            renderThumbnails();
            loadSlide(slides.length - 1);
        }

        function deleteSlide() {
            if (slides.length > 1) {
                slides.splice(currentSlideIndex, 1);
                if (currentSlideIndex >= slides.length) currentSlideIndex = slides.length - 1;
                renderThumbnails();
                loadSlide(currentSlideIndex);
            }
        }

        function nextSlide() {
            if (currentSlideIndex < slides.length - 1) {
                loadSlide(currentSlideIndex + 1);
            }
        }

        function prevSlide() {
            if (currentSlideIndex > 0) {
                loadSlide(currentSlideIndex - 1);
            }
        }

        // AI Generation
        function generateFromTopic() {
            const topic = document.getElementById('topicInput').value;
            if (!topic) return;
            
            // Simulasi AI Generation
            const titles = [
                `Pengenalan ${topic}`,
                `Manfaat ${topic}`,
                `Implementasi ${topic}`,
                `Studi Kasus: ${topic}`,
                `Kesimpulan ${topic}`
            ];
            
            slides = titles.map((title, idx) => ({
                id: idx + 1,
                layout: idx === 0 ? 'title' : 'content',
                theme: currentTheme,
                accent: currentAccent,
                animation: ['fade', 'slideUp', 'scale', 'rotate', 'fade'][idx],
                title: title,
                subtitle: idx === 0 ? 'Generated by AI' : '',
                content: idx === 0 ? `Presentasi tentang ${topic}` : `• Poin penting tentang ${topic}\n• Analisis mendalam\n• Rekomendasi strategis`,
                shapes: [],
                notes: ''
            }));
            
            renderThumbnails();
            loadSlide(0);
            
            // Animasi notifikasi
            gsap.fromTo("#currentSlide", {scale: 0.9}, {scale: 1, duration: 0.5, ease: "back.out(1.7)"});
        }

        function applyAIToCurrentSlide() {
            const slide = slides[currentSlideIndex];
            // Simulasi enhance
            slide.content = slide.content + '\n\n• [AI Enhanced] Data pendukung\n• [AI Enhanced] Insight tambahan';
            renderCurrentSlide();
            document.getElementById('propContent').value = slide.content;
        }

        // Import Modal
        function toggleImportModal() {
            const modal = document.getElementById('importModal');
            modal.classList.toggle('hidden');
            if (!modal.classList.contains('hidden')) {
                document.getElementById('parsedContent').classList.add('hidden');
                document.getElementById('uploadProgress').classList.add('hidden');
                document.getElementById('urlProgress').classList.add('hidden');
            }
        }

        // File Upload Handling
        function handleDragOver(e) {
            e.preventDefault();
            e.currentTarget.classList.add('dragover');
        }

        function handleDragLeave(e) {
            e.currentTarget.classList.remove('dragover');
        }

        function handleDrop(e) {
            e.preventDefault();
            e.currentTarget.classList.remove('dragover');
            const files = e.dataTransfer.files;
            if (files.length) processFile(files[0]);
        }

        function handleFileSelect(e) {
            const file = e.target.files[0];
            if (file) processFile(file);
        }

        async function processFile(file) {
            const progress = document.getElementById('uploadProgress');
            const status = document.getElementById('uploadStatus');
            progress.classList.remove('hidden');
            
            const extension = file.name.split('.').pop().toLowerCase();
            
            try {
                if (extension === 'pdf') {
                    status.textContent = 'Memproses PDF...';
                    await parsePDF(file);
                } else if (extension === 'docx') {
                    status.textContent = 'Memproses DOCX...';
                    await parseDOCX(file);
                } else if (extension === 'txt') {
                    status.textContent = 'Memproses TXT...';
                    await parseTXT(file);
                } else {
                    alert('Format file tidak didukung');
                    progress.classList.add('hidden');
                    return;
                }
                
                showParsedContent();
            } catch (error) {
                console.error(error);
                alert('Error memproses file: ' + error.message);
            } finally {
                progress.classList.add('hidden');
            }
        }

        // PDF Parser menggunakan PDF.js
        async function parsePDF(file) {
            const arrayBuffer = await file.arrayBuffer();
            const pdf = await pdfjsLib.getDocument({data: arrayBuffer}).promise;
            
            let fullText = '';
            const maxPages = Math.min(pdf.numPages, 5); // Ambil max 5 halaman
            
            for (let i = 1; i <= maxPages; i++) {
                const page = await pdf.getPage(i);
                const textContent = await page.getTextContent();
                const pageText = textContent.items.map(item => item.str).join(' ');
                fullText += pageText + '\n\n';
            }
            
            parsedDocumentContent = fullText;
        }

        // DOCX Parser menggunakan Mammoth.js
        async function parseDOCX(file) {
            const arrayBuffer = await file.arrayBuffer();
            const result = await mammoth.extractRawText({arrayBuffer});
            parsedDocumentContent = result.value;
        }

        // TXT Parser
        async function parseTXT(file) {
            const text = await file.text();
            parsedDocumentContent = text;
        }

        // URL Scraping
        async function scrapeURL() {
            const url = document.getElementById('urlInput').value;
            if (!url) return;
            
            const progress = document.getElementById('urlProgress');
            progress.classList.remove('hidden');
            
            try {
                // Menggunakan allorigins.win sebagai CORS proxy
                const proxyUrl = `https://api.allorigins.win/get?url=${encodeURIComponent(url)}`;
                const response = await fetch(proxyUrl);
                const data = await response.json();
                
                if (data.contents) {
                    // Parse HTML
                    const parser = new DOMParser();
                    const doc = parser.parseFromString(data.contents, 'text/html');
                    
                    // Hapus script dan style
                    doc.querySelectorAll('script, style, nav, footer, header').forEach(el => el.remove());
                    
                    // Ambil konten utama
                    const article = doc.querySelector('article') || doc.querySelector('main') || doc.querySelector('.content') || doc.body;
                    
                    // Convert ke markdown menggunakan Turndown
                    const turndownService = new TurndownService();
                    const markdown = turndownService.turndown(article.innerHTML);
                    
                    parsedDocumentContent = markdown;
                    showParsedContent();
                }
            } catch (error) {
                console.error(error);
                alert('Error mengambil URL. Coba gunakan metode Fetch & Parse.');
            } finally {
                progress.classList.add('hidden');
            }
        }

        // Alternative Fetch (untuk site yang tidak support CORS)
        async function fetchAndParse() {
            const url = document.getElementById('urlInput').value;
            if (!url) return;
            
            alert('Fitur ini memerlukan backend server. Untuk demo, gunakan file upload atau paste konten manual ke properties panel.');
        }

        function showParsedContent() {
            const container = document.getElementById('parsedContent');
            const textEl = document.getElementById('parsedText');
            
            textEl.textContent = parsedDocumentContent.substring(0, 2000) + (parsedDocumentContent.length > 2000 ? '...' : '');
            container.classList.remove('hidden');
        }

        function generateSlidesFromContent() {
            if (!parsedDocumentContent) return;
            
            // Split konten menjadi beberapa slide berdasarkan paragraf
            const paragraphs = parsedDocumentContent.split(/\n\n+/).filter(p => p.trim().length > 0);
            const chunks = [];
            
            // Gabungkan paragraf menjadi 5 bagian
            const chunkSize = Math.ceil(paragraphs.length / 5);
            for (let i = 0; i < 5; i++) {
                const chunk = paragraphs.slice(i * chunkSize, (i + 1) * chunkSize).join('\n\n');
                if (chunk) chunks.push(chunk);
            }
            
            // Generate slides
            const titles = ['Judul Utama', 'Pendahuluan', 'Poin Utama', 'Analisis', 'Kesimpulan'];
            
            slides = chunks.map((content, idx) => {
                // Bersihkan konten
                const cleanContent = content.substring(0, 500).replace(/[#*_]/g, '');
                
                return {
                    id: idx + 1,
                    layout: idx === 0 ? 'title' : 'content',
                    theme: currentTheme,
                    accent: currentAccent,
                    animation: ['fade', 'slideUp', 'scale', 'rotate', 'fade'][idx],
                    title: idx === 0 ? (cleanContent.split('.')[0] || titles[idx]) : titles[idx],
                    subtitle: idx === 0 ? 'Diimpor dari Dokumen' : '',
                    content: cleanContent,
                    shapes: [],
                    notes: ''
                };
            });
            
            toggleImportModal();
            renderThumbnails();
            loadSlide(0);
            
            // Notifikasi
            gsap.fromTo("#currentSlide", {scale: 0.9, opacity: 0}, {scale: 1, opacity: 1, duration: 0.6});
        }

        // Presentation Mode
        function togglePresentationMode() {
            const mode = document.getElementById('presentationMode');
            const slideContainer = document.getElementById('presentationSlide');
            
            if (mode.classList.contains('hidden')) {
                // Masuk mode presentasi
                mode.classList.remove('hidden');
                updatePresentationSlide();
                document.body.style.overflow = 'hidden';
            } else {
                // Keluar mode presentasi
                mode.classList.add('hidden');
                document.body.style.overflow = '';
            }
        }

        function updatePresentationSlide() {
            const slide = slides[currentSlideIndex];
            const container = document.getElementById('presentationSlide');
            
            container.innerHTML = '';
            const slideEl = document.createElement('div');
            slideEl.className = `w-full h-full ${getThemeClass(slide.theme)} relative overflow-hidden`;
            
            // Clone konten dari current slide
            let content = '';
            switch(slide.layout) {
                case 'title':
                    content = `
                        <div class="absolute inset-0 flex flex-col items-center justify-center p-16 text-center">
                            <h1 class="text-6xl md:text-7xl font-bold text-white mb-8 leading-tight">${slide.title}</h1>
                            <h2 class="text-3xl md:text-4xl text-white/80 font-light">${slide.subtitle}</h2>
                            <p class="mt-12 text-2xl text-white/60 max-w-3xl">${slide.content}</p>
                        </div>
                    `;
                    break;
                case 'content':
                    content = `
                        <div class="absolute inset-0 flex flex-col p-20">
                            <h1 class="text-5xl md:text-6xl font-bold text-white mb-12">${slide.title}</h1>
                            <div class="text-2xl md:text-3xl text-white/90 whitespace-pre-line leading-relaxed">${slide.content}</div>
                        </div>
                    `;
                    break;
                case 'split':
                    content = `
                        <div class="absolute inset-0 flex">
                            <div class="w-1/2 p-16 flex flex-col justify-center border-r border-white/10">
                                <h1 class="text-4xl md:text-5xl font-bold text-white mb-6">${slide.title}</h1>
                                <p class="text-2xl text-white/70">${slide.subtitle}</p>
                            </div>
                            <div class="w-1/2 p-16 flex items-center">
                                <div class="text-2xl text-white/90 whitespace-pre-line leading-relaxed">${slide.content}</div>
                            </div>
                        </div>
                    `;
                    break;
                case 'image':
                    content = `
                        <div class="absolute inset-0 flex">
                            <div class="w-2/3 p-20 flex flex-col justify-center">
                                <h1 class="text-5xl md:text-6xl font-bold text-white mb-8">${slide.title}</h1>
                                <p class="text-2xl text-white/80 leading-relaxed">${slide.content}</p>
                            </div>
                            <div class="w-1/3 bg-black/20 flex items-center justify-center">
                                <i class="fas fa-image text-8xl text-white/20"></i>
                            </div>
                        </div>
                    `;
                    break;
            }
            
            slideEl.innerHTML = content;
            container.appendChild(slideEl);
            
            document.getElementById('presentationCounter').textContent = `${currentSlideIndex + 1} / ${slides.length}`;
            
            // Animasi masuk
            gsap.fromTo(slideEl.children, 
                { opacity: 0, y: 30 },
                { opacity: 1, y: 0, duration: 0.5, stagger: 0.1 }
            );
        }

        // Export HTML
        function exportHTML() {
            let htmlContent = `
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>${slides[0].title}</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; overflow: hidden; }
        .slide {
            width: 100vw;
            height: 100vh;
            display: none;
            position: relative;
        }
        .slide.active { display: block; }
        .gradient-bg { background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); }
        .gradient-dark { background: linear-gradient(135deg, #1e293b 0%, #0f172a 100%); }
        .gradient-light { background: linear-gradient(135deg, #f8fafc 0%, #e2e8f0 100%); color: #1e293b; }
        .gradient-nature { background: linear-gradient(135deg, #11998e 0%, #38ef7d 100%); }
        .gradient-corporate { background: linear-gradient(135deg, #373B44 0%, #4286f4 100%); }
        .gradient-minimal { background: linear-gradient(135deg, #232526 0%, #414345 100%); }
        .content { padding: 4rem; height: 100%; display: flex; flex-direction: column; justify-content: center; }
        h1 { font-size: 3.5rem; margin-bottom: 1rem; color: white; }
        h2 { font-size: 2rem; color: rgba(255,255,255,0.8); font-weight: 300; }
        p, li { font-size: 1.5rem; color: rgba(255,255,255,0.9); line-height: 1.6; }
        .split { display: flex; height: 100%; }
        .split > div { flex: 1; padding: 4rem; display: flex; flex-direction: column; justify-content: center; }
        .split > div:first-child { border-right: 1px solid rgba(255,255,255,0.1); }
        .navigation {
            position: fixed;
            bottom: 2rem;
            left: 50%;
            transform: translateX(-50%);
            background: rgba(0,0,0,0.5);
            padding: 1rem 2rem;
            border-radius: 2rem;
            display: flex;
            gap: 1rem;
            align-items: center;
            z-index: 1000;
        }
        button {
            background: rgba(255,255,255,0.2);
            border: none;
            color: white;
            padding: 0.5rem 1rem;
            border-radius: 0.5rem;
            cursor: pointer;
            font-size: 1rem;
        }
        button:hover { background: rgba(255,255,255,0.3); }
        @media print {
            .slide { display: block !important; page-break-after: always; }
            .navigation { display: none; }
        }
    </style>
</head>
<body>
`;

            slides.forEach((slide, idx) => {
                const isLight = slide.theme === 'light';
                const textColor = isLight ? 'color: #1e293b;' : 'color: white;';
                
                htmlContent += `<div class="slide ${getThemeClass(slide.theme)} ${idx === 0 ? 'active' : ''}" id="slide${idx}">\n`;
                
                if (slide.layout === 'title') {
                    htmlContent += `
    <div class="content" style="text-align: center;">
        <h1 style="${textColor}">${slide.title}</h1>
        <h2 style="${isLight ? 'color: #475569;' : ''}">${slide.subtitle}</h2>
        <p style="margin-top: 2rem; ${isLight ? 'color: #64748b;' : ''}">${slide.content}</p>
    </div>`;
                } else if (slide.layout === 'content') {
                    htmlContent += `
    <div class="content">
        <h1 style="${textColor}">${slide.title}</h1>
        <div style="white-space: pre-line; ${textColor}">${slide.content}</div>
    </div>`;
                } else if (slide.layout === 'split') {
                    htmlContent += `
    <div class="split">
        <div>
            <h1 style="${textColor}">${slide.title}</h1>
            <p style="${isLight ? 'color: #475569;' : ''}">${slide.subtitle}</p>
        </div>
        <div style="align-items: center;">
            <div style="white-space: pre-line; ${textColor}">${slide.content}</div>
        </div>
    </div>`;
                }
                
                htmlContent += '</div>\n';
            });

            htmlContent += `
    <div class="navigation">
        <button onclick="prevSlide()">← Prev</button>
        <span id="counter" style="color: white; min-width: 60px; text-align: center;">1 / ${slides.length}</span>
        <button onclick="nextSlide()">Next →</button>
    </div>

    <script>
        let currentSlide = 0;
        const totalSlides = ${slides.length};
        
        function showSlide(n) {
            document.querySelectorAll('.slide').forEach((slide, idx) => {
                slide.classList.toggle('active', idx === n);
            });
            document.getElementById('counter').textContent = (n + 1) + ' / ' + totalSlides;
        }
        
        function nextSlide() {
            if (currentSlide < totalSlides - 1) {
                currentSlide++;
                showSlide(currentSlide);
            }
        }
        
        function prevSlide() {
            if (currentSlide > 0) {
                currentSlide--;
                showSlide(currentSlide);
            }
        }
        
        document.addEventListener('keydown', (e) => {
            if (e.key === 'ArrowRight') nextSlide();
            if (e.key === 'ArrowLeft') prevSlide();
        });
    </script>
</body>
</html>`;

            // Download file
            const blob = new Blob([htmlContent], {type: 'text/html'});
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = 'presentation.html';
            a.click();
            URL.revokeObjectURL(url);
        }
    </script>
</body>
</html>
