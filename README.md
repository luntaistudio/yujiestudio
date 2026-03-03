# yujiestudio
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>嶼界文創工作室 - 勞務報酬單系統</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2pdf.js/0.10.1/html2pdf.bundle.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/signature_pad@4.0.0/dist/signature_pad.umd.min.js"></script>
    
    <style>
        :root { --primary: #2c3e50; --accent: #8e44ad; --bg: #f9f9fb; }
        body { font-family: "PingFang TC", "Microsoft JhengHei", sans-serif; background: var(--bg); margin: 0; padding: 20px; color: #333; }
        .wrapper { max-width: 850px; margin: 0 auto; }
        
        /* 編輯面板 */
        .editor-card { background: white; padding: 25px; border-radius: 12px; box-shadow: 0 4px 15px rgba(0,0,0,0.1); margin-bottom: 30px; border-top: 6px solid var(--accent); }
        .editor-card h2 { margin-top: 0; font-size: 1.2rem; color: var(--accent); border-bottom: 1px solid #eee; padding-bottom: 10px; }
        
        .input-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(250px, 1fr)); gap: 15px; margin: 15px 0; }
        .form-group { display: flex; flex-direction: column; }
        .form-group label { font-weight: bold; margin-bottom: 5px; font-size: 0.9rem; }
        .form-group input, .form-group select { padding: 10px; border: 1px solid #ddd; border-radius: 6px; }

        /* 簽名與上傳區 */
        .upload-section { display: flex; gap: 20px; flex-wrap: wrap; margin-top: 20px; }
        .sig-box { border: 1px dashed #ccc; background: #fff; border-radius: 8px; width: 320px; }
        canvas { width: 100%; height: 150px; cursor: crosshair; }
        
        /* PDF 預覽區域 */
        #pdf-area { background: white; padding: 50px; min-height: 1000px; box-sizing: border-box; }
        .receipt-header { text-align: center; margin-bottom: 30px; }
        .receipt-header h1 { font-size: 26px; letter-spacing: 5px; margin-bottom: 5px; }
        .receipt-header p { margin: 0; color: #666; font-size: 0.9rem; }
        
        .info-table { width: 100%; border-collapse: collapse; margin-top: 20px; }
        .info-table th, .info-table td { border: 1px solid #000; padding: 12px; text-align: left; }
        .info-table th { background-color: #f2f2f2; width: 20%; }
        
        .bank-passbook-preview { margin-top: 20px; border: 1px solid #eee; max-width: 100%; height: auto; display: none; }

        .btn-main { background: var(--accent); color: white; padding: 15px; border: none; border-radius: 8px; font-size: 1.1rem; cursor: pointer; font-weight: bold; width: 100%; margin-top: 25px; transition: 0.3s; }
        .btn-main:hover { background: #732d91; box-shadow: 0 4px 12px rgba(142, 68, 173, 0.3); }
        
        @media print { .no-print { display: none; } }
    </style>
</head>
<body>

<div class="wrapper">
    <div class="editor-card no-print">
        <h2>Step 1. 填寫領款資訊</h2>
        <div class="input-grid">
            <div class="form-group">
                <label>領款人姓名</label>
                <input type="text" id="in-name" placeholder="請填寫本名" oninput="updatePreview()">
            </div>
            <div class="form-group">
                <label>身分證/居留證字號</label>
                <input type="text" id="in-id" placeholder="A123456789" oninput="updatePreview()">
            </div>
            <div class="form-group">
                <label>給付總額 (未扣稅前)</label>
                <input type="number" id="in-amount" value="0" oninput="updatePreview()">
            </div>
            <div class="form-group">
                <label>所得類別</label>
                <select id="in-type" onchange="updatePreview()">
                    <option value="9B 執行業務所得 (稿費/演講)">9B 執行業務所得</option>
                    <option value="50 薪資所得">50 薪資所得</option>
                    <option value="9A 專業執行業務">9A 專業執行業務</option>
                </select>
            </div>
        </div>

        <h2>Step 2. 憑證與簽名</h2>
        <div class="upload-section">
            <div class="form-group">
                <label>上傳存摺影本 (照片/截圖)</label>
                <input type="file" accept="image/*" onchange="handlePassbook(this)">
            </div>
            <div class="sig-box">
                <canvas id="sig-pad"></canvas>
                <div style="padding:5px; background:#eee; display:flex; justify-content:space-between;">
                    <span style="font-size:12px;">領款人手寫簽名區</span>
                    <button onclick="clearSig()" style="font-size:12px;">清除重簽</button>
                </div>
            </div>
        </div>

        <button class="btn-main" onclick="generatePDF()">產生正式 PDF 並下載</button>
    </div>

    <div id="pdf-area">
        <div class="receipt-header">
            <h1>勞 務 報 酬 收 據</h1>
            <p>嶼界文創工作室 | Islands Cultural & Creative Studio</p>
        </div>
        
        <p>茲向 <strong>嶼界文創工作室</strong> 領取勞務報酬，內容如下：</p>
        
        <table class="info-table">
            <tr>
                <th>所得人姓名</th>
                <td id="out-name">______</td>
                <th>身分證字號</th>
                <td id="out-id">______</td>
            </tr>
            <tr>
                <th>所得類別</th>
                <td id="out-type" colspan="3">9B 執行業務所得</td>
            </tr>
            <tr>
                <th>給付總額 (A)</th>
                <td id="out-total" style="font-weight:bold;">$0</td>
                <th>扣繳稅額 (10%)</th>
                <td id="out-tax">$0</td>
            </tr>
            <tr>
                <th>二代健保 (2.11%)</th>
                <td id="out-hi">$0</td>
                <th>實付金額 (A-B-C)</th>
                <td id="out-final" style="font-weight:bold; font-size:1.2rem; color:#d35400;">$0</td>
            </tr>
        </table>

        <div style="margin-top: 30px;">
            <p><strong>領受人簽章：</strong></p>
            <img id="pdf-sig" style="display:none; max-height: 80px; border-bottom: 1px solid #000;">
        </div>

        <div style="margin-top: 30px;">
            <p><strong>附件：存摺影本影本</strong></p>
            <img id="pdf-passbook" class="bank-passbook-preview">
        </div>

        <div style="margin-top: 50px; font-size: 0.8rem; color: #555;">
            註：本收據依稅法規定辦理。單筆給付達 NT$20,011 扣繳 10% 稅金；單筆達 NT$20,000 扣繳 2.11% 補充保費。<br>
            <p style="text-align:right;">填表日期：2026 年 3 月 3 日</p>
        </div>
    </div>
</div>

<script>
    const canvas = document.getElementById('sig-pad');
    const signaturePad = new SignaturePad(canvas);

    function resizeCanvas() {
        const ratio = Math.max(window.devicePixelRatio || 1, 1);
        canvas.width = canvas.offsetWidth * ratio;
        canvas.height = canvas.offsetHeight * ratio;
        canvas.getContext("2d").scale(ratio, ratio);
        signaturePad.clear();
    }
    window.onload = resizeCanvas;

    function clearSig() { signaturePad.clear(); document.getElementById('pdf-sig').style.display = 'none'; }

    function updatePreview() {
        const name = document.getElementById('in-name').value;
        const id = document.getElementById('in-id').value;
        const amount = parseFloat(document.getElementById('in-amount').value) || 0;
        const type = document.getElementById('in-type').value;

        const tax = amount > 20010 ? Math.floor(amount * 0.1) : 0;
        const hi = amount >= 20000 ? Math.round(amount * 0.0211) : 0;
        const final = amount - tax - hi;

        document.getElementById('out-name').innerText = name || "______";
        document.getElementById('out-id').innerText = id || "______";
        document.getElementById('out-type').innerText = type;
        document.getElementById('out-total').innerText = '$' + amount.toLocaleString();
        document.getElementById('out-tax').innerText = '$' + tax.toLocaleString();
        document.getElementById('out-hi').innerText = '$' + hi.toLocaleString();
        document.getElementById('out-final').innerText = '$' + final.toLocaleString();
    }

    function handlePassbook(input) {
        if (input.files && input.files[0]) {
            const reader = new FileReader();
            reader.onload = function(e) {
                const img = document.getElementById('pdf-passbook');
                img.src = e.target.result;
                img.style.display = 'block';
            }
            reader.readAsDataURL(input.files[0]);
        }
    }

    async function generatePDF() {
        if (signaturePad.isEmpty()) return alert("請先完成簽名");
        
        document.getElementById('pdf-sig').src = signaturePad.toDataURL();
        document.getElementById('pdf-sig').style.display = 'block';

        const element = document.getElementById('pdf-area');
        const name = document.getElementById('in-name').value || '嶼界文創領款人';
        
        const opt = {
            margin: 10,
            filename: `嶼界文創_報酬單_${name}.pdf`,
            image: { type: 'jpeg', quality: 0.95 },
            html2canvas: { scale: 2 },
            jsPDF: { unit: 'mm', format: 'a4', orientation: 'portrait' }
        };
        html2pdf().set(opt).from(element).save();
    }
</script>

</body>
</html>
