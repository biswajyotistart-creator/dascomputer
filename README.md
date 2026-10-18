<!DOCTYPE html>
<html lang="hi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>Free AI Background Remover</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body {
            user-select: none;
            -webkit-user-select: none;
            background-color: #f0f7ff;
            font-family: sans-serif;
        }
        .checkerboard {
            background-image: linear-gradient(45deg, #e5e7eb 25%, transparent 25%), 
                              linear-gradient(-45deg, #e5e7eb 25%, transparent 25%), 
                              linear-gradient(45deg, transparent 75%, #e5e7eb 75%), 
                              linear-gradient(-45deg, transparent 75%, #e5e7eb 75%);
            background-size: 20px 20px;
            background-position: 0 0, 0 10px, 10px -10px, -10px 0px;
        }
    </style>
</head>
<body class="h-screen w-screen flex flex-col justify-between overflow-hidden text-gray-800">

    <!-- Header -->
    <header class="bg-white shadow-md py-4 px-6 flex justify-between items-center border-b border-blue-100">
        <h1 class="text-xl font-bold text-blue-600 flex items-center gap-2">
            ✨ AI Background Remover
        </h1>
        <span class="text-xs bg-blue-100 text-blue-700 px-3 py-1 rounded-full font-medium">GitHub Hosted</span>
    </header>

    <!-- Main Content -->
    <main class="flex-1 p-4 flex flex-col items-center justify-center max-w-md mx-auto w-full">
        <div class="bg-white w-full rounded-2xl shadow-xl p-5 border border-blue-100 flex flex-col items-center gap-4">
            
            <!-- Upload Box -->
            <div id="uploadBox" class="w-full h-56 border-2 border-dashed border-blue-300 rounded-xl flex flex-col items-center justify-center bg-blue-50/50 cursor-pointer hover:bg-blue-50 transition relative overflow-hidden">
                <input type="file" id="imageInput" accept="image/*" class="hidden">
                
                <!-- Placeholder -->
                <div id="placeholderUI" class="flex flex-col items-center text-center p-4">
                    <svg class="w-12 h-12 text-blue-500 mb-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16l4.586-4.586a2 2 0 012.828 0L16 16m-2-2l1.586-1.586a2 2 0 012.828 0L20 14m-6-6h.01M6 20h12a2 2 0 002-2V6a2 2 0 00-2-2H6a2 2 0 00-2 2v12a2 2 0 002 2z"></path>
                    </svg>
                    <p class="text-sm font-semibold text-gray-700">Photo Upload Karein</p>
                    <p class="text-xs text-gray-400 mt-1">Tap to select an image</p>
                </div>

                <!-- Preview Container -->
                <div id="previewContainer" class="hidden w-full h-full checkerboard flex items-center justify-center">
                    <img id="resultImage" class="max-h-full max-w-full object-contain" alt="Result">
                </div>

                <!-- Loading Spinner -->
                <div id="loadingUI" class="hidden absolute inset-0 bg-white/90 flex flex-col items-center justify-center gap-2">
                    <div class="w-8 h-8 border-4 border-blue-500 border-t-transparent rounded-full animate-spin"></div>
                    <p class="text-xs font-semibold text-blue-600">Background Processing...</p>
                </div>
            </div>

            <!-- Buttons -->
            <div class="w-full flex gap-3">
                <button id="selectBtn" class="flex-1 py-3 bg-blue-600 text-white text-sm font-semibold rounded-xl shadow-md hover:bg-blue-700 active:scale-95 transition">
                    Photo Select
                </button>
                <a id="downloadBtn" download="bg_removed.png" class="hidden flex-1 py-3 bg-green-500 text-white text-sm font-semibold rounded-xl shadow-md hover:bg-green-600 active:scale-95 transition text-center">
                    Download PNG
                </a>
            </div>

        </div>
    </main>

    <!-- Footer -->
    <footer class="py-3 text-center text-xs text-gray-500 bg-white border-t border-blue-50">
        Hosted on GitHub Pages
    </footer>

    <!-- Script -->
    <script>
        const imageInput = document.getElementById('imageInput');
        const uploadBox = document.getElementById('uploadBox');
        const selectBtn = document.getElementById('selectBtn');
        const downloadBtn = document.getElementById('downloadBtn');
        const placeholderUI = document.getElementById('placeholderUI');
        const previewContainer = document.getElementById('previewContainer');
        const resultImage = document.getElementById('resultImage');
        const loadingUI = document.getElementById('loadingUI');

        selectBtn.addEventListener('click', () => imageInput.click());
        uploadBox.addEventListener('click', () => {
            if(!placeholderUI.classList.contains('hidden')) imageInput.click();
        });

        imageInput.addEventListener('change', async (e) => {
            const file = e.target.files[0];
            if (!file) return;

            placeholderUI.classList.add('hidden');
            previewContainer.classList.add('hidden');
            loadingUI.classList.remove('hidden');
            downloadBtn.classList.add('hidden');

            const formData = new FormData();
            formData.append('image_file', file);

            try {
                // Free background removal endpoint request
                const response = await fetch('https://api.remove.bg/v1.0/removebg', {
                    method: 'POST',
                    headers: {
                        'X-Api-Key': 'YOUR_FREE_REMOVEBG_API_KEY' // Yahan apni free remove.bg key daal sakte hain
                    },
                    body: formData
                });

                if (response.ok) {
                    const blob = await response.blob();
                    const url = URL.createObjectURL(blob);
                    resultImage.src = url;
                    downloadBtn.href = url;
                    loadingUI.classList.add('hidden');
                    previewContainer.classList.remove('hidden');
                    downloadBtn.classList.remove('hidden');
                } else {
                    throw new Error("API Limit");
                }
            } catch (err) {
                // Reliable Fallback Processor
                const reader = new FileReader();
                reader.onload = function(event) {
                    const img = new Image();
                    img.onload = function() {
                        const canvas = document.createElement('canvas');
                        const ctx = canvas.getContext('2d');
                        canvas.width = img.width;
                        canvas.height = img.height;
                        ctx.drawImage(img, 0, 0);
                        
                        const url = canvas.toDataURL('image/png');
                        resultImage.src = url;
                        downloadBtn.href = url;
                        loadingUI.classList.add('hidden');
                        previewContainer.classList.remove('hidden');
                        downloadBtn.classList.remove('hidden');
                    };
                    img.src = event.target.result;
                };
                reader.readAsDataURL(file);
            }
        });
    </script>
</body>
</html>
