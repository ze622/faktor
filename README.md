<!DOCTYPE html>
<html lang="fa" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>فاکتورساز پیشرفته</title>
    <!-- کتابخانه خواندن فایل‌های اکسل -->
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

    <div class="section">
        <h4>۱. لیست قیمت‌ها (اکسل یا متن)</h4>
        <input type="file" id="excelInput" accept=".xlsx, .xls, .csv" onchange="handleExcel(event)" style="margin-bottom: 10px;">
        <p style="font-size: 12px; color: #666;">یا لیست محصولات و قیمت‌ها را اینجا بچسبانید (هر خط: نام محصول و قیمت):</p>
        <textarea id="priceListInput" placeholder="کرم نارگیل 636
صابون گل 500"></textarea>
    </div>

    <div class="section">
        <h4>۲. سفارش مشتری</h4>
        <p style="font-size: 12px; color: #666;">نام محصول و تعداد را وارد کنید (مثلاً: کرم 1):</p>
        <textarea id="orderInput" placeholder="کرم 1
صابون 2"></textarea>
        <button onclick="generateInvoice()">ساخت فاکتور</button>
    </div>

    <div class="section">
        <h4>۳. خروجی فاکتور</h4>
        <div class="output-box" id="output">فاکتور اینجا نمایش داده می‌شود...</div>
        <button onclick="copyInvoice()" style="margin-top:10px; background-color: #28a745;">کپی فاکتور</button>
    </div>

    <script>
        // ۱. بررسی رمز عبور در ابتدای ورود به برنامه
        let password = prompt("لطفاً رمز عبور را وارد کنید:");
        if (password !== "1234") {
            alert("رمز اشتباه است!");
            document.body.innerHTML = "<h2 style='text-align:center; color:red; margin-top:50px;'>دسترسی غیرمجاز. لطفاً صفحه را رفرش کنید و رمز صحیح را وارد نمایید.</h2>";
            throw new Error("Invalid Password");
        }

        let masterDatabase = {}; // حافظه برنامه برای نگهداری قیمت‌ها

        // خواندن فایل اکسل
        function handleExcel(e) {
            const file = e.target.files[0];
            if (!file) return;
            const reader = new FileReader();
            reader.onload = function(e) {
                try {
                    const data = new Uint8Array(e.target.result);
                    const workbook = XLSX.read(data, {type: 'array'});
                    const sheet = workbook.Sheets[workbook.SheetNames[0]];
                    const json = XLSX.utils.sheet_to_json(sheet, {header: 1});
                    
                    json.forEach(row => {
                        if (row[0] && row[1]) {
                            masterDatabase[row[0].toString().trim()] = parseInt(row[1]);
                        }
                    });
                    alert("اکسل با موفقیت بارگذاری شد و قیمت‌ها ذخیره شدند!");
                } catch (err) {
                    alert("خطا در خواندن فایل اکسل!");
                }
            };
            reader.readAsArrayBuffer(file);
        }

        // ساخت فاکتور
        function generateInvoice() {
            // خواندن و ترکیب قیمت‌های متنی (اگر کاربر در کادر متن قیمت وارد کرده باشد)
            const manualText = document.getElementById('priceListInput').value;
            if (manualText.trim() !== "") {
                manualText.split('\n').forEach(line => {
                    const parts = line.trim().split(/\s+/);
                    if (parts.length >= 2) {
                        const price = parseInt(parts.pop());
                        const name = parts.join(' ');
                        masterDatabase[name] = price;
                    }
                });
            }

            // خواندن سفارشات مشتری
            const ordersText = document.getElementById('orderInput').value;
            const orders = ordersText.split('\n');
            let invoice = "";
            let total = 0;

            orders.forEach(line => {
                if (!line.trim()) return;
                const parts = line.trim().split(/\s+/);
                const qty = parseInt(parts.pop() || 1); // آخرین بخش تعداد است (پیش‌فرض ۱)
                const name = parts.join(' '); // بقیه متن نام محصول است

                // جستجوی هوشمند (اگر نام ناقص وارد شده باشد)
                let foundName = Object.keys(masterDatabase).find(k => k.includes(name) || name.includes(k)) || name;
                let price = masterDatabase[foundName] || 0;
                let lineTotal = price * qty;

                invoice += `${foundName} ${qty} عدد ${lineTotal}\n`;
                total += lineTotal;
            });

            invoice += `\nجمع کل: ${total}`;
            document.getElementById('output').innerText = invoice;
        }

        // دکمه کپی به کلیپ‌بورد
        function copyInvoice() {
            const textToCopy = document.getElementById('output').innerText;
            navigator.clipboard.writeText(textToCopy).then(() => {
                alert("فاکتور با موفقیت کپی شد!");
            }).catch(err => {
                alert("خطا در کپی کردن متن!");
            });
        }
    </script>
</body>
</html>
