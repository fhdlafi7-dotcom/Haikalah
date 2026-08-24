<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>تطبيق هيكلة A4 التفاعلي</title>
    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }

        body {
            display: flex;
            height: 100vh;
            background-color: #f0f2f5;
            overflow: hidden;
        }

        /* لوحة التحكم على اليمين */
        aside {
            width: 340px;
            background-color: #ffffff;
            border-left: 1px solid #ddd;
            padding: 20px;
            display: flex;
            flex-direction: column;
            gap: 15px;
            box-shadow: -2px 0 5px rgba(0,0,0,0.05);
            overflow-y: auto;
            z-index: 100;
        }

        aside h2 {
            font-size: 1.2rem;
            color: #333;
            border-bottom: 2px solid #007bff;
            padding-bottom: 8px;
        }

        .control-group {
            display: flex;
            flex-direction: column;
            gap: 6px;
        }

        .control-group label {
            font-size: 0.85rem;
            color: #555;
            font-weight: bold;
        }

        .control-group input, 
        .control-group select {
            padding: 8px 12px;
            border: 1px solid #ccc;
            border-radius: 6px;
            font-size: 0.9rem;
            outline: none;
        }

        .control-group input:focus, 
        .control-group select:focus {
            border-color: #007bff;
        }

        .btn-grid {
            display: grid;
            grid-template-columns: repeat(2, 1fr);
            gap: 8px;
        }

        .tool-btn {
            padding: 10px;
            background-color: #f8f9fa;
            border: 1px solid #ddd;
            border-radius: 6px;
            cursor: pointer;
            transition: 0.2s;
            font-size: 0.85rem;
            font-weight: bold;
            color: #333;
        }

        .tool-btn:hover {
            background-color: #007bff;
            color: white;
            border-color: #007bff;
        }

        .action-btn {
            padding: 10px;
            background-color: #dc3545;
            color: white;
            border: none;
            border-radius: 6px;
            cursor: pointer;
            font-weight: bold;
            transition: 0.2s;
        }

        .action-btn:hover {
            background-color: #c82333;
        }

        /* مساحة العمل على اليسار */
        main {
            flex: 1;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            position: relative;
            overflow: auto;
            background-color: #e4e6eb;
        }

        /* شريط التحكم بالزووم */
        .zoom-controls {
            position: absolute;
            top: 20px;
            left: 20px;
            background: white;
            padding: 8px 15px;
            border-radius: 8px;
            box-shadow: 0 2px 5px rgba(0,0,0,0.1);
            display: flex;
            gap: 10px;
            align-items: center;
            z-index: 10;
        }

        .zoom-controls button {
            padding: 5px 10px;
            cursor: pointer;
            background: #f1f1f1;
            border: 1px solid #ccc;
            border-radius: 4px;
        }

        .zoom-controls button:hover {
            background: #e2e2e2;
        }

        /* صفحة A4 */
        .workspace-container {
            transform-origin: center center;
            transition: transform 0.05s ease;
            margin: 50px;
        }

        .a4-page {
            width: 210mm;
            height: 297mm;
            background-color: white;
            box-shadow: 0 0 15px rgba(0, 0, 0, 0.15);
            position: relative;
            overflow: hidden;
            cursor: default;
        }

        /* عناصر التصميم داخل الصفحة */
        .canvas-element {
            position: absolute;
            cursor: move;
            user-select: none;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        .canvas-element.selected {
            outline: 2px dashed #007bff;
        }
    </style>
</head>
<body>

    <!-- لوحة التحكم (اليمين) -->
    <aside>
        <h2>لوحة التحكم</h2>

        <div class="control-group">
            <label>انقر لإضافة الشكل فوراً</label>
            <div class="btn-grid">
                <button class="tool-btn" onclick="addShape('rect')">مربع ▢</button>
                <button class="tool-btn" onclick="addShape('arrow')">سهم ➔</button>
                <button class="tool-btn" onclick="addShape('line')">خط ⎯</button>
                <button class="tool-btn" onclick="addShape('text')">نص 🔤</button>
            </div>
        </div>

        <div class="control-group" id="text-input-group">
            <label for="custom-text">محتوى النص</label>
            <input type="text" id="custom-text" value="أدخل النص هنا">
        </div>

        <div class="control-group">
            <label for="line-width">سمك الخط / الإطار (بكسل)</label>
            <input type="number" id="line-width" value="2" min="1" max="20">
        </div>

        <div class="control-group">
            <label for="line-style">نمط الحدود</label>
            <select id="line-style">
                <option value="solid">متصل (Solid)</option>
                <option value="dashed">متقطع (Dashed)</option>
                <option value="dotted">منقط (Dotted)</option>
            </select>
        </div>

        <div class="control-group">
            <label for="font-family">نوع الخط (للنصوص)</label>
            <select id="font-family">
                <option value="Cairo">كاييرو (Cairo)</option>
                <option value="Amiri">أميري (Amiri)</option>
                <option value="Tahoma">تاهوما (Tahoma)</option>
                <option value="Arial">أريال (Arial)</option>
            </select>
        </div>

        <div class="control-group">
            <label for="element-color">اللون</label>
            <input type="color" id="element-color" value="#000000" style="height: 35px; cursor: pointer; width: 100%;">
        </div>

        <hr style="border: 0; border-top: 1px solid #ddd;">
        
        <button class="action-btn" onclick="deleteSelectedElement()">حذف العنصر المحدد</button>
        <button class="action-btn" style="background-color: #6c757d;" onclick="clearPage()">مسح الصفحة بالكامل</button>
    </aside>

    <!-- مساحة العمل وصفحة A4 (اليسار) -->
    <main>
        <!-- أداة الزووم -->
        <div class="zoom-controls">
            <button onclick="zoomOut()">-</button>
            <span id="zoom-level">100%</span>
            <button onclick="zoomIn()">+</button>
        </div>

        <!-- الحاوية القابلة للزووم -->
        <div class="workspace-container" id="workspace">
            <div class="a4-page" id="a4-page">
                <!-- العناصر المرسومة ستظهر هنا ديناميكياً -->
            </div>
        </div>
    </main>

    <script>
        // المتغيرات العامة
        let currentZoom = 1;
        let selectedElement = null;

        const workspace = document.getElementById('workspace');
        const zoomLevelText = document.getElementById('zoom-level');
        const a4Page = document.getElementById('a4-page');

        // التحكم بالزووم
        function updateZoom() {
            workspace.style.transform = `scale(${currentZoom})`;
            zoomLevelText.innerText = Math.round(currentZoom * 100) + '%';
        }

        function zoomIn() {
            if (currentZoom < 2.5) {
                currentZoom += 0.1;
                updateZoom();
            }
        }

        function zoomOut() {
            if (currentZoom > 0.4) {
                currentZoom -= 0.1;
                updateZoom();
            }
        }

        // إضافة الشكل مباشرة عند النقر على الزر
        function addShape(type) {
            const el = document.createElement('div');
            el.className = 'canvas-element';
            
            const borderWidth = document.getElementById('line-width').value;
            const borderStyle = document.getElementById('line-style').value;
            const borderColor = document.getElementById('element-color').value;
            const fontFamily = document.getElementById('font-family').value;

            // مكان ظهور العنصر الافتراضي داخل الصفحة
            el.style.left = '50px';
            el.style.top = '50px';

            if (type === 'rect') {
                el.style.width = '150px';
                el.style.height = '100px';
                el.style.border = `${borderWidth}px ${borderStyle} ${borderColor}`;
                el.style.backgroundColor = 'transparent';
            } else if (type === 'line') {
                el.style.width = '200px';
                el.style.height = `${borderWidth}px`;
                el.style.backgroundColor = borderColor;
            } else if (type === 'arrow') {
                el.style.width = '120px';
                el.style.height = '40px';
                el.innerHTML = `<svg width="120" height="40" viewBox="0 0 120 40"><line x1="0" y1="20" x2="110" y2="20" stroke="${borderColor}" stroke-width="${borderWidth}" stroke-dasharray="${borderStyle === 'dashed' ? '5,5' : borderStyle === 'dotted' ? '2,2' : 'none'}" /><polygon points="110,15 120,20 110,25" fill="${borderColor}"/></svg>`;
            } else if (type === 'text') {
                const textContent = document.getElementById('custom-text').value;
                el.style.padding = '5px';
                el.style.fontFamily = fontFamily;
                el.style.fontSize = '16px';
                el.style.color = borderColor;
                el.innerText = textContent;
            }

            // تفعيل السحب والإفلات للتحريك
            makeDraggable(el);

            a4Page.appendChild(el);
            selectElement(el);
        }

        // وظيفة تحريك العناصر بالماوس داخل الصفحة
        function makeDraggable(element) {
            let isDragging = false;
            let startX, startY;

            element.addEventListener('mousedown', function(e) {
                e.stopPropagation();
                selectElement(element);
                isDragging = true;
                startX = e.clientX - element.offsetLeft;
                startY = e.clientY - element.offsetTop;
            });

            document.addEventListener('mousemove', function(e) {
                if (!isDragging) return;
                let newX = e.clientX - startX;
                let newY = e.clientY - startY;
                
                element.style.left = newX + 'px';
                element.style.top = newY + 'px';
            });

            document.addEventListener('mouseup', function() {
                isDragging = false;
            });
        }

        function selectElement(element) {
            document.querySelectorAll('.canvas-element').forEach(el => el.classList.remove('selected'));
            selectedElement = element;
            selectedElement.classList.add('selected');
        }

        // إلغاء التحديد عند النقر على خلفية الصفحة البيضاء
        a4Page.addEventListener('click', function(e) {
            if (e.target === a4Page) {
                if (selectedElement) {
                    selectedElement.classList.remove('selected');
                    selectedElement = null;
                }
            }
        });

        // زر الحذف
        function deleteSelectedElement() {
            if (selectedElement) {
                selectedElement.remove();
                selectedElement = null;
            } else {
                alert('الرجاء اختيار عنصر أولاً لحذفه.');
            }
        }

        // مسح الصفحة بالكامل
        function clearPage() {
            if (confirm('هل أنت متأكد من مسح كافة العناصر في الصفحة؟')) {
                a4Page.innerHTML = '';
                selectedElement = null;
            }
        }
    </script>
</body>
</html>
