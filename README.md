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
        #tableContainer { margin-top: 25px; overflow-x: auto; }
        table { width: 100%; border-collapse: collapse; margin-top: 10px; }
        th, td { border: 1px solid #ddd; padding: 10px; text-align: center; font-size: 13px; }
        th { background-color: #eee; }
        #login-overlay { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: #f4f4f9; display: flex; justify-content: center; align-items: center; z-index: 9999; }
        .login-card { width: 90%; max-width: 350px; background: white; padding: 25px; border-radius: 15px; box-shadow: 0 5px 15px rgba(0,0,0,0.2); text-align: center; }
        #main-app { display: none; }
    </style>
</head>
<body>

    <div id="login-overlay">
        <div class="login-card">
            <h3>ورود به فاکتورساز</h3>
            <input type="password" id="passInput" placeholder="رمز عبور">
            <button onclick="checkPass()">ورود</button>
        </div>
    </div>

    <div id="main-app">
        <div class="container">
            <h2>فاکتورساز</h2>
            <div class="form-group">
                <input type="text" id="customerName" placeholder="نام مشتری">
                <input type="text" id="productName" placeholder="نام محصول">
                <input type="number" id="productQuantity" placeholder="تعداد" value="1">
                <input type="number" id="productPrice" placeholder="قیمت واحد">
            </div>
            <button onclick="addProduct()">افزودن</button>
            <div id="tableContainer">
                <table id="invoiceTable">
                    <thead><tr><th>نام</th><th>تعداد</th><th>قیمت</th></tr></thead>
                    <tbody></tbody>
                </table>
            </div>
            <button id="copyButton" onclick="copyInvoice()">کپی فاکتور</button>
        </div>
    </div>

    <script>
        // رمز را اینجا تغییر دهید
        const MY_PASSWORD = "1234"; 

        function checkPass() {
            const input = document.getElementById('passInput').value;
            if (input === MY_PASSWORD) {
                document.getElementById('login-overlay').style.display = 'none';
                document.getElementById('main-app').style.display = 'block';
            } else {
                alert("رمز اشتباه است");
            }
        }

        let items = [];
        function addProduct() {
            const name = document.getElementById('productName').value;
            const qty = document.getElementById('productQuantity').value;
            const price = document.getElementById('productPrice').value;
            items.push({name, qty, price});
            
            const tbody = document.querySelector("#invoiceTable tbody");
            tbody.innerHTML += `<tr><td>${name}</td><td>${qty}</td><td>${price}</td></tr>`;
        }

        function copyInvoice() {
            let text = "فاکتور خرید:\n" + items.map(i => `${i.name}: ${i.qty} عدد - ${i.price} تومان`).join("\n");
            navigator.clipboard.writeText(text).then(() => alert("کپی شد"));
        }
    </script>
</body>
</html>
