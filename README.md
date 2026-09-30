<!DOCTYPE html>
<html lang="fa" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>فاکتورساز پیشرفته</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <style>
        body { font-family: Tahoma, sans-serif; padding: 15px; background: #f9f9f9; }
        textarea { width: 100%; height: 100px; margin-bottom: 10px; border-radius: 8px; }
        .section { background: white; padding: 10px; margin-bottom: 15px; border-radius: 10px; box-shadow: 0 2px 5px rgba(0,0,0,0.1); }
        h4 { margin-top: 0; color: #444; }
        button { width: 100%; padding: 12px; background: #007bff; color: white; border: none; border-radius: 5px; font-weight: bold; }
        .output-box { background: #e8f5e9; padding: 15px; border: 2px dashed #4caf50; min-height: 100px; white-space: pre-wrap; margin-top: 10px; }
    </style>
</head>
<body>

    <div class="section">
        <h4>۱. لیست قیمت‌ها (اکسل یا متن)</h4>
        <input type="file" id="excelInput" accept=".xlsx, .xls, .csv" onchange="handleExcel(event)">
        <p style="font-size: 12px;">یا لیست را اینجا بچسبانید (نام محصول - قیمت):</p>
        <textarea id="priceListInput" placeholder="کرم نارگیل 636
صابون گل 500"></textarea>
    </div>

    <div class="section">
        <h4>۲. سفارش مشتری</h4>
        <textarea id="orderInput" placeholder="مثال:
کرم 1
صابون 2"></textarea>
        <button onclick="generateInvoice()">ساخت فاکتور</button>
    </div>

    <div class="output-box" id="output">فاکتور اینجا نمایش داده می‌شود...</div>
    <button onclick="copyInvoice()" style="margin-top:10px; background-color: #28a745;">کپی فاکتور</button>

    <script>
        let masterDatabase = {}; // حافظه برنامه

        // ۱. خواندن اکسل
        function handleExcel(e) {
            const file = e.target.files[0];
            const reader = new FileReader();
            reader.onload = function(e) {
                const data = new Uint8Array(e.target.result);
                const workbook = XLSX.read(data, {type: 'array'});
                const sheet = workbook.Sheets[workbook.SheetNames[0]];
                const json = XLSX.utils.sheet_to_json(sheet, {header: 1});
                
                json.forEach(row => {
                    if (row[0] && row[1]) masterDatabase[row[0].trim()] = parseInt(row[1]);
                });
                alert("اکسل با موفقیت بارگذاری شد!");
            };
            reader.readAsArrayBuffer(file);
        }

        // ۲. ساخت فاکتور
        function generateInvoice() {
            // ترکیب حافظه از متن (اگر دستی چسبانده شده)
            const manualText = document.getElementById('priceListInput').value;
            manualText.split('\n').forEach(line => {
                const parts = line.split(/\s+/);
                if (parts.length >= 2) masterDatabase[parts[0].trim()] = parseInt(parts[1]);
            });

            // خواندن سفارش
            const orders = document.getElementById('orderInput').value.split('\n');
            let invoice = "";
            let total = 0;

            orders.forEach(line => {
                if (!line.trim()) return;
                const parts = line.split(/\s+/);
                const name = parts[0];
                const qty = parseInt(parts[1] || 1);

                // جستجوی هوشمند (اگر اسم دقیق نبود)
                let foundName = Object.keys(masterDatabase).find(k => k.includes(name) || name.includes(k)) || name;
                let price = masterDatabase[foundName] || 0;

                invoice += `${foundName} ${qty} عدد ${price * qty}\n`;
                total += (price * qty);
            });

            invoice += `\nجمع کل: ${total}`;
            document.getElementById('output').innerText = invoice;
        }

        function copyInvoice() {
            navigator.clipboard.writeText(document.getElementById('output').innerText);
            alert("کپی شد!");
        }
    </script>
</body>
</html>
