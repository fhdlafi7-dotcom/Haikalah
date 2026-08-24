<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>تطبيق تصميم الهياكل</title>
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
            width: 320px;
            background-color: #ffffff;
            border-left: 1px solid #ddd;
            padding: 20px;
            display: flex;
            flex-direction: column;
            gap: 20px;
            box-shadow: -2px 0 5px rgba(0,0,0,0.05);
            overflow-y: auto;
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
            gap: 8px;
        }

        .control-group label {
            font-size: 0.9rem;
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
            gap: 10px;
        }

        .tool-btn {
            padding: 10px;
            background-color: #f8f9fa;
            border: 1px solid #ddd;
            border-radius: 6px;
            cursor: pointer;
            transition: 0.2s;
            font-size: 0.9rem;
        }

        .tool-btn:hover {
            background-color: #007bff;
            color: white;
            border-color: #007bff;
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
            transition: transform 0.1s ease;
        }

        .a4-page {
            width: 210mm;
            height: 297mm;
            background-color: white;
            box-shadow: 0 0 15px rgba(0, 0, 0, 0.15);
            position: relative;
            /* للتأكد من أبعاد A4 الحقيقية على الشاشة */
        }
    </style>
</head>
<body>

    <!-- لوحة التحكم (اليمين) -->
    <aside>
        <h2>لوحة التحكم</h2>

        <div class="control-group">
            <label>الأشكال والأسهم</label>
            <div class="btn-grid">
                <button class="tool-btn">مربع</button>
                <button class="tool-btn">مستطيل</button>
                <button class="tool-btn">سهمي ➔</button>
                <button class="tool-btn">خط مستقيم</button>
            </div>
        </div>

        <div class="control-group">
            <label for="line-width">سمك الخط (بكسل)</label>
            <input type="number" id="line-width" value="2" min="1" max="20">
        </div>

        <div class="control-group">
            <label for="line-style">نمط الخط</label>
            <select id="line-style">
                <option value="solid">متصل (Solid)</option>
                <option value="dashed">متقطع (Dashed)</option>
                <option value="dotted">منقط (Dotted)</option>
            </select>
        </div>

        <div class="control-group">
            <label for="font-family">نوع الخط</label>
            <select id="font-family">
                <option value="Cairo">كاييرو (Cairo)</option>
                <option value="Amiri">أميري (Amiri)</option>
                <option value="Tahoma">تاهوما (Tahoma)</option>
                <option value="Arial">أريال (Arial)</option>
            </select>
        </div>

        <div class="control-group">
            <label for="line-color">لون الخط أو الشكل</label>
            <input type="color" id="line-color" value="#000000" style="height: 40px; cursor: pointer;">
        </div>
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
            <div class="a4-page">
                <!-- هنا سيتم رسم الهيكل أو إدراج العناصر لاحقاً -->
            </div>
        </div>
    </main>

    <script>
        // إعدادات الزووم (التكبير والتصغير)
        let currentZoom = 1;
        const workspace = document.getElementById('workspace');
        const zoomLevelText = document.getElementById('zoom-level');

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
    </script>
</body>
</html>
