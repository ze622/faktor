<!DOCTYPE html>
<html lang="fa" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>فاکتورساز</title>
    <!-- لود کردن SDK ایتا -->
    <script src="https://eitaa.com/js/telegram-web-app.js"></script>
    <script src="https://cdn.sheetjs.com/xlsx-0.20.2/package/dist/xlsx.full.min.js"></script>

    <style>
        body { font-family: Tahoma, sans-serif; background-color: #f4f4f9; padding: 20px; direction: rtl; margin: 0; }
        .container { max-width: 600px; margin: auto; background: white; padding: 20px; border-radius: 15px; box-shadow: 0 5px 15px rgba(0,0,0,0.1); }
        h2 { text-align: center; color: #333; margin-bottom: 20px; }
        input[type="text"], input[type="number"], textarea { width: 100%; padding: 10px; margin-top: 8px; border: 1px solid #ccc; border-radius: 5px; box-sizing: border-box; font-size: 14px; }
        button { width: 100%; padding: 12px; margin-top: 15px; background-color: #28a745; color: white; border: none; border-radius: 5px; cursor: pointer; font-size: 16px; transition: background-color 0.3s ease; }
        button:hover { background-color: #218838; }
        #copyButton { background-color: #1677b8; margin-top: 10px; }
        #copyButton:hover { background-color: #12669e; }
        #tableContainer { margin-top: 25px; overflow-x: auto; }
        table { width: 100%; border-collapse: collapse; margin-top: 10px; }
        th, td { border: 1px solid #ddd; padding: 10px; text-align: center; font-size: 13px; }
        th { background-color: #eee; font-weight: bold; }
        tr:nth-child(even) { background-color: #f9f9f9; }
        .total-row td { font-weight: bold; background-color: #e0e0e0; }
        .form-group { margin-bottom: 15px; }
        label { display: block; margin-bottom: 5px; font-weight: bold; color: #555; }
        textarea { resize: vertical; min-height: 80px; }

        /* صفحه ورود */
        #login-overlay { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: #f4f4f9; display: flex; justify-content: center; align-items: center; z-index: 9999; }
        .login-card { width: 90%; max-width: 350px; background: white; padding: 25px; border-radius: 15px; box-shadow: 0 5px 15px rgba(0,0,0,0.2); text-align: center; }
        .admin-msg { color: #856404; background: #fff3cd; padding: 10px; border-radius: 5px; font-size: 13px; margin-bottom: 15px; border: 1px solid #ffeeba; }
        #main-app { display: none; }
    </style>
</head>
<body>

    <!-- صفحه ورود -->
    <div id="login-overlay">
        <div class="login-card">
            <h3>ورود به برنامه</h3>
            <div class="admin-msg">⚠️ برای دریافت رمز عبور به ادمین مراجعه کنید.</div>
            <input type="password" id="passInput" placeholder="رمز عبور">
            <button onclick="checkPass()">ورود</button>
        </div>
    </div>

    <!-- برنامه اصلی -->
    <div id="main-app">
        <div class="container">
            <h2>فاکتورساز</h2>

            <div class="form-group">
                <label for="customerName">نام مشتری:</label>
                <input type="text" id="customerName" placeholder="نام و نام خانوادگی مشتری">
            </div>
            <div class="form-group">
                <label for="customerPhone">شماره تماس:</label>
                <input type="text" id="customerPhone" placeholder="شماره همراه یا ثابت">
            </div>
            <div class="form-group">
                <label for="factorDescription">شرح فاکتور (اختیاری):</label>
                <textarea id="factorDescription" placeholder="توضیحات اضافی برای فاکتور"></textarea>
            </div>

            <div class="form-group">
                <label for="productName">نام محصول/خدمت:</label>
                <input type="text" id="productName" placeholder="نام محصول یا خدمت">
            </div>
            <div class="form-group">
                <label for="productQuantity">تعداد:</label>
                <input type="number" id="productQuantity" placeholder="تعداد" value="1">
            </div>
            <div class="form-group">
                <label for="productPrice">قیمت واحد (تومان):</label>
                <input type="number" id="productPrice" placeholder="قیمت واحد به تومان">
            </div>

            <button onclick="addProduct()">افزودن به فاکتور</button>

            <div id="tableContainer">
                <p>لیست محصولات فاکتور:</p>
                <table id="invoiceTable">
                    <thead>
                        <tr>
                            <th>ردیف</th>
                            <th>نام محصول</th>
                            <th>تعداد</th>
                            <th>قیمت واحد (تومان)</th>
                            <th>مبلغ کل</th>
                        </tr>
                    </thead>
                    <tbody>
                        <!-- محصولات اینجا اضافه می‌شوند -->
                    </tbody>
                </table>
                <table id="totalRowTable" style="margin-top: 10px;">
                     <tr class="total-row">
                        <td colspan="4" style="text-align: left;">جمع کل فاکتور:</td>
                        <td id="totalAmount">0 تومان</td>
                    </tr>
                </table>
            </div>

            <button id="copyButton" onclick="copyInvoice()">کپی فاکتور</button>
        </div>
    </div>

    <script>
        // ۱. بررسی امن برای SDK ایتا
        try {
            if (window.Telegram && window.Telegram.WebApp) {
                const tg = window.Telegram.WebApp;
                tg.ready();
                tg.expand();
            }
        } catch (e) {
            console.log("SDK ایتا در این محیط در دسترس نیست.");
        }

        // ۲. منطق ورود امن
        const MY_PASSWORD = "GAPGPTMASKTOKENxgtnsrgp4gX0X"; 
        let isLoggedIn = false;

        function checkPass() {
            const input = document.getElementById('passInput').value;
            if (input === MY_PASSWORD) {
                isLoggedIn = true;
                document.getElementById('login-overlay').style.display = 'none';
                document.getElementById('main-app').style.display = 'block';
            } else {
                alert("رمز عبور اشتباه است!");
            }
        }

        // ۳. منطق اصلی فاکتور ساز
        let invoiceItems = [];
        let rowCounter = 1;

        function addProduct() {
            if (!isLoggedIn) return;

            const productName = document.getElementById('productName').value.trim();
            const productQuantity = parseInt(document.getElementById('productQuantity').value) || 1;
            const productPrice = parseFloat(document.getElementById('productPrice').value) || 0;

            if (!productName || productQuantity <= 0 || productPrice < 0) {
                alert("لطفاً نام محصول، تعداد (حداقل ۱) و قیمت واحد را به درستی وارد کنید.");
                return;
            }

            const lineTotal = productQuantity * productPrice;
            invoiceItems.push({
                id: rowCounter,
                name: productName,
                quantity: productQuantity,
                price: productPrice,
                lineTotal: lineTotal
            });

            renderTable();

            document.getElementById('productName').value = '';
            document.getElementById('productQuantity').value = '1';
            document.getElementById('productPrice').value = '';
            document.getElementById('productName').focus();
        }

        function renderTable() {
            const tableBody = document.querySelector("#invoiceTable tbody");
            tableBody.innerHTML = '';
            let totalAmount = 0;

            invoiceItems.forEach(item => {
                const row = tableBody.insertRow();
                row.innerHTML = `
                    <td>${item.id}</td>
                    <td>${item.name}</td>
                    <td>${item.quantity}</td>
                    <td>${item.price.toLocaleString('fa-IR')}</td>
                    <td>${item.lineTotal.toLocaleString('fa-IR')}</td>
                `;
                totalAmount += item.lineTotal;
            });

            document.getElementById('totalAmount').textContent = totalAmount.toLocaleString('fa-IR') + ' تومان';
            rowCounter++;
        }

        function copyInvoice() {
            if (!isLoggedIn) return; 

            const customerName = document.getElementById('customerName').value.trim() || "مشتری گرامی";
            const customerPhone = document.getElementById('customerPhone').value.trim() || "---";
            const factorDescription = document.getElementById('factorDescription').value.trim();

            let invoiceText = `✨ فاکتور فروش ✨\n\n`;
            invoiceText += `نام مشتری: ${customerName}\n`;
            invoiceText += `شماره تماس: ${customerPhone}\n`;
            if (factorDescription) {
                invoiceText += `توضیحات: ${factorDescription}\n`;
            }
            invoiceText += `\n------------------------------------\n`;
            invoiceText += `ردیف | نام محصول | تعداد | قیمت واحد | مبلغ کل\n`;
            invoiceText += `------------------------------------\n`;

            invoiceItems.forEach(item => {
                invoiceText += `${item.id} | ${item.name} | ${item.quantity} | ${item.price.toLocaleString('fa-IR')} | ${item.lineTotal.toLocaleString('fa-IR')}\n`;
            });

            invoiceText += `------------------------------------\n`;
            invoiceText += `جمع کل: ${document.getElementById('totalAmount').textContent}\n`;
            invoiceText += `\nبا تشکر از حسن انتخاب شما!`;

            navigator.clipboard.writeText(invoiceText).then(() => {
                alert("فاکتور با موفقیت کپی شد! می‌توانید آن را در ایتا پیست کنید.");
                try {
                    if (window.Telegram && window.Telegram.WebApp) {
                        window.Telegram.WebApp.close();
                    }
                } catch (e) {}
            }).catch(err => {
                alert("خطا در کپی کردن فاکتور: " + err);
            });
        }

        document.addEventListener('DOMContentLoaded', () => {
             renderTable();
        });
    </script>
</body>
</html>
