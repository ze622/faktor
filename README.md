<!DOCTYPE html>
<html lang="fa" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>فاکتورساز پیشرفته ایتا</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <style>
        body { font-family: Tahoma, sans-serif; padding: 15px; background: #f9f9f9; }
        textarea { width: 100%; height: 100px; margin-bottom: 10px; border-radius: 8px; border: 1px solid #ccc; padding: 8px; box-sizing: border-box; }
        .section { background: white; padding: 15px; margin-bottom: 15px; border-radius: 10px; box-shadow: 0 2px 5px rgba(0,0,0,0.1); }
        h4 { margin-top: 0; color: #444; }
        button { width: 100%; padding: 12px; background: #007bff; color: white; border: none; border-radius: 5px; font-weight: bold; cursor: pointer; }
        button:hover { background: #0056b3; }
        .output-box { background: #e8f5e9; padding: 15px; border: 2px dashed #4caf50; min-height: 100px; white-space: pre-wrap; margin-top: 10px; border-radius: 8px; }
    </style>
</head>
<body>

    <script>
        // تغییر رمز به 1122
        let password = GAPGPTMASKTOKENp8hm2srfy7X0X"رمز عبور را وارد کنید:");
        if (password !== "1122") { 
            alert("رمز اشتباه است!");
            document.body.innerHTML = "<h2 style='text-align:center; color:red; margin-top:50px;'>دسترسی غیرمجاز.</h2>";
            throw new Error("Invalid Password");
        }
    </script>

    <h2 style="text-align: center;">فاکتورساز پیشرفته</h2>

    <div class="section">
        <h4>۱. لیست قیمت‌ها (اکسل یا متن)</h4>
        <input type="file" id="excelInput" accept=".xlsx, .xls, .csv" onchange="handleExcel(event)" style="margin-bottom: 5px;">
        <div id="uploadStatus" style="font-size: 13px; font-weight: bold; margin-bottom: 10px;"></div>
        <textarea id="priceListInput" placeholder="مداد مشکی 1000
پاک کن 500"></textarea>
    </div>

    <div class="section">
        <h4>۲. سفارش مشتری</h4>
        <textarea id="orderInput" placeholder="مداد 2
پاک کن"></textarea>
        <button onclick="generateInvoice()">ساخت فاکتور</button>
    </div>

    <div class="section">
        <h4>۳. خروجی فاکتور</h4>
        <div class="output-box" id="output">فاکتور آماده نمایش...</div>
        <button onclick="copyInvoice()" style="margin-top:10px; background-color: #28a745;">کپی فاکتور</button>
    </div>

    <script>
        let masterDatabase = {};

        function handleExcel(e) {
            const file = e.target.files[0];
            const statusEl = document.getElementById('uploadStatus');
            if (!file) return;
            
            const reader = new FileReader();
            reader.onload = function(e) {
                try {
                    const data = new Uint8Array(e.target.result);
                    const workbook = XLSX.read(data, {type: 'array'});
                    const sheet = workbook.Sheets[workbook.SheetNames[0]];
                    const json = XLSX.utils.sheet_to_json(sheet, {header: 1});
                    json.forEach(row => { if (row[0] && row[1]) masterDatabase[row[0].toString().trim()] = parseInt(row[1]); });
                    
                    // نمایش تایید در زیر کادر
                    statusEl.innerText = "فایل اکسل با موفقیت بارگذاری شد!";
                    statusEl.style.color = "green";
                } catch (err) { 
                    statusEl.innerText = "خطا در خواندن فایل اکسل!";
                    statusEl.style.color = "red";
                }
            };
            reader.readAsArrayBuffer(file);
        }

        function generateInvoice() {
            const manualText = document.getElementById('priceListInput').value;
            manualText.split('\n').forEach(line => {
                const parts = line.trim().split(/\s+/);
                if (parts.length >= 2) {
                    const price = parseInt(parts.pop());
                    const name = parts.join(' ');
                    masterDatabase[name] = price;
                }
            });

            const orders = document.getElementById('orderInput').value.split('\n');
            let invoice = "";
            let total = 0;

            orders.forEach(line => {
                if (!line.trim()) return;
                
                let parts = line.trim().split(/\s+/);
                let qty = 1;
                let isExplicit = false;

                // حذف کلمه "عدد" اگر انتهای خط باشد
                if (parts[parts.length - 1] === "عدد") {
                    parts.pop(); 
                }

                // بررسی وجود عدد در انتها
                let lastWord = parts[parts.length - 1];
                if (!isNaN(parseInt(lastWord))) {
                    qty = parseInt(parts.pop());
                    isExplicit = true;
                }
                
                let name = parts.join(' ');
                let foundKey = Object.keys(masterDatabase).find(k => k.includes(name) || name.includes(k));
                let foundName = foundKey || name;
                let price = masterDatabase[foundKey] || 0;
                let lineTotal = price * qty;

                if (isExplicit) {
                    invoice += `${foundName} ${qty} عدد ${lineTotal}\n`;
                } else {
                    invoice += `${foundName} ${lineTotal}\n`;
                }
                
                total += lineTotal;
            });

            invoice += `\nجمع کل: ${total}`;
            document.getElementById('output').innerText = invoice;
        }

        function copyInvoice() {
            const textToCopy = document.getElementById('output').innerText;
            navigator.clipboard.writeText(textToCopy).then(() => alert("فاکتور کپی شد!"));
        }
    </script>
</body>
</html>
