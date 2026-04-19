<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Drone Mapping Tool - Digdo Edition</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* Menggunakan Font Inter agar lebih DKV & Modern */
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700;900&display=swap');
        
        body { font-family: 'Inter', sans-serif; }

        @media print {
            body * { visibility: hidden; }
            #invoiceArea, #invoiceArea * { visibility: visible; }
            #invoiceArea { position: absolute; left: 0; top: 0; width: 100%; margin: 0; padding: 0; }
            .no-print { display: none !important; }
            input { border: none !important; outline: none !important; background: transparent !important; }
        }
        
        /* Aksen tipografi DKV */
        .tracking-tight-heavy { letter-spacing: -0.05em; }
        .tracking-widest-heavy { letter-spacing: 0.2em; }
    </style>
</head>
<body class="bg-white text-slate-950 p-4 md:p-8">

    <div class="max-w-3xl mx-auto space-y-10">
        
        <div class="text-center no-print py-6 border-b-2 border-slate-100">
            <h1 class="text-4xl font-black tracking-tight-heavy text-slate-950">DRONE MISSION <span class="text-blue-600">PLANNER</span></h1>
            <p class="text-slate-500 text-sm mt-1 uppercase tracking-widest">Digital Geospatial Solutions</p>
        </div>

        <div class="bg-white p-8 rounded-2xl shadow-2xl border border-slate-100 no-print relative overflow-hidden">
            <div class="absolute top-0 left-0 right-0 h-2 bg-amber-500"></div>

            <div class="flex justify-between items-center mb-8">
                <h2 class="font-extrabold text-xl text-slate-900 tracking-tight">Parameter Misi</h2>
                <div class="text-right monospace text-xs font-bold text-slate-400 space-x-3">
                    <span>Ha: Rp 250/m</span>
                    <span>Jalan: Rp 150/m</span>
                    <span class="text-blue-600">Min: Rp 2jt</span>
                </div>
            </div>
            
            <form id="calcForm" class="space-y-8">
                <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                    <div>
                        <label class="block font-bold text-xs text-slate-600 uppercase mb-2 tracking-wider">Tinggi Terbang (m)</label>
                        <input type="number" id="tinggi" value="150" class="w-full p-3 bg-slate-50 border border-slate-200 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 outline-none transition">
                    </div>
                    <div>
                        <label class="block font-bold text-xs text-slate-600 uppercase mb-2 tracking-wider">Overlap Samping (%)</label>
                        <input type="number" id="sidelap" value="80" class="w-full p-3 bg-slate-50 border border-slate-200 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 outline-none transition">
                    </div>
                </div>

                <div class="p-6 bg-slate-50 rounded-xl border border-slate-100">
                    <div class="flex gap-6 mb-5 border-b border-slate-200 pb-4">
                        <label class="flex items-center gap-3 cursor-pointer font-bold text-slate-700">
                            <input type="radio" name="mapType" value="area" checked onclick="toggleInput('area')" class="w-5 h-5 text-blue-600 border-slate-300 focus:ring-blue-500"> 
                            Area (Hektar)
                        </label>
                        <label class="flex items-center gap-3 cursor-pointer font-bold text-slate-700">
                            <input type="radio" name="mapType" value="jalan" onclick="toggleInput('jalan')" class="w-5 h-5 text-blue-600 border-slate-300 focus:ring-blue-500"> 
                            Jalan (Meter)
                        </label>
                    </div>

                    <div id="inputArea">
                        <label class="block font-semibold mb-2 text-sm text-slate-600">Luas Total Area (Ha):</label>
                        <input type="number" id="nilaiArea" placeholder="Contoh: 15" class="w-full p-3 border border-slate-200 rounded-lg outline-none focus:ring-2 focus:ring-blue-500">
                    </div>

                    <div id="inputJalan" class="hidden space-y-4">
                        <input type="number" id="nilaiPanjangJalan" placeholder="Panjang Total Jalan (m)" class="w-full p-3 border border-slate-200 rounded-lg outline-none focus:ring-2 focus:ring-blue-500">
                        <input type="number" id="nilaiLebarJalan" value="50" placeholder="Lebar Koridor (m)" class="w-full p-3 border border-slate-200 rounded-lg outline-none focus:ring-2 focus:ring-blue-500">
                    </div>
                </div>

                <button type="button" onclick="kalkulasiTotal()" class="w-full bg-slate-950 hover:bg-black text-white font-black py-5 rounded-lg shadow-lg uppercase tracking-widest-heavy transition-all transform hover:-translate-y-1">
                    Hitung Analisis Pekerjaan
                </button>
            </form>

            <div id="hasil" class="mt-12 hidden animate-in fade-in slide-in-from-bottom-5 duration-500">
                <div id="resultCard" class="bg-slate-950 rounded-2xl p-8 text-white shadow-2xl relative overflow-hidden">
                    <div class="absolute -bottom-10 -right-10 text-[120px] font-black text-slate-900/50 rotate-[-15deg] select-none no-print">DATA</div>

                    <h3 class="text-amber-400 font-black uppercase text-xs mb-6 tracking-widest-heavy border-b border-slate-800 pb-3">Rincian Operasional & Estimasi Biaya</h3>
                    
                    <div class="grid grid-cols-2 md:grid-cols-3 gap-4 mb-8 text-center relative z-10">
                        <div class="bg-slate-900/80 p-4 rounded-lg border border-slate-800 relative">
                            <p class="text-slate-500 text-[9px] uppercase font-bold tracking-wider">GSD</p>
                            <p id="resGsd" class="text-xl font-black text-white">-</p>
                        </div>
                        <div class="bg-slate-900/80 p-4 rounded-lg border border-slate-800">
                            <p class="text-slate-500 text-[9px] uppercase font-bold tracking-wider">Jarak Antar Jalur</p>
                            <p id="resSpacing" class="text-xl font-black text-white">-</p>
                        </div>
                        <div class="bg-slate-800 p-4 rounded-lg col-span-2 md:col-span-1 border border-amber-950">
                            <p class="text-amber-500 text-[9px] uppercase font-bold tracking-wider">Total Panjang Jalur</p>
                            <p id="resTotalLintasan" class="text-2xl font-black text-amber-400">-</p>
                        </div>
                        <div class="bg-slate-900 p-4 rounded-lg border border-slate-800">
                            <p class="text-slate-500 text-[9px] uppercase font-bold tracking-wider">Kebutuhan Baterai</p>
                            <p id="resBaterai" class="text-lg font-bold text-blue-300">-</p>
                        </div>
                        <div class="bg-slate-900 p-4 rounded-lg border border-slate-800 col-span-1">
                            <p class="text-slate-500 text-[9px] uppercase font-bold tracking-wider">Durasi Lapangan</p>
                            <p id="resHari" class="text-lg font-bold text-blue-300">-</p>
                        </div>
                    </div>

                    <div class="space-y-3 mb-10 text-sm border-t border-slate-800 pt-6 relative z-10">
                        <div class="flex justify-between items-center border-b border-slate-800 pb-2">
                            <span class="text-slate-400 font-medium">Harga Dasar (Jalur x Rate)</span>
                            <span id="resPokok" class="font-mono text-white">-</span>
                        </div>
                        <div id="rowSurcharge" class="flex justify-between items-center text-orange-400 hidden border-b border-slate-800 pb-2">
                            <span class="font-medium">Surcharge Overlap >89% (15%)</span>
                            <span id="resSurcharge" class="font-mono font-bold">-</span>
                        </div>
                        <div id="rowMinOrder" class="flex justify-between items-center text-blue-400 hidden border-b border-slate-800 pb-2">
                            <span class="font-medium font-italic">Penyesuaian Minimum Order</span>
                            <span id="resAdjMin" class="font-mono font-bold">-</span>
                        </div>
                    </div>

                    <div class="flex flex-col md:flex-row justify-between items-start md:items-end gap-4 relative z-10">
                        <div>
                            <p class="text-[10px] uppercase text-slate-500 tracking-widest">Total Estimasi Tagihan</p>
                            <p id="resHarga" class="text-5xl font-black text-green-400 tracking-tight-heavy">-</p>
                        </div>
                        <button onclick="showInvoiceForm()" class="w-full md:w-auto bg-blue-600 hover:bg-blue-700 text-white text-xs font-black py-4 px-8 rounded-lg transition uppercase tracking-wider shadow-lg">
                            Lanjut ke Administrasi
                        </button>
                    </div>
                </div>
            </div>
        </div>

        <div id="invoiceInputSection" class="bg-white p-8 rounded-xl shadow-lg hidden no-print text-left border border-slate-100">
            <h3 class="font-black text-lg mb-6 text-slate-900 uppercase tracking-tight">Detail Administrasi Invoice</h3>
            <div class="grid grid-cols-1 md:grid-cols-2 gap-5 mb-6">
                <div>
                    <label class="block text-[10px] font-bold text-slate-500 uppercase mb-1.5 ml-1 tracking-wider">Nama Klien / Perusahaan</label>
                    <input type="text" id="invClient" placeholder="Contoh: PT. Maju Bersama Nusantara" class="w-full p-3 bg-slate-50 border border-slate-200 rounded-lg focus:ring-2 focus:ring-blue-500 outline-none transition">
                </div>
                <div>
                    <label class="block text-[10px] font-bold text-slate-500 uppercase mb-1.5 ml-1 tracking-wider">Nama Project</label>
                    <input type="text" id="invProject" placeholder="Contoh: Pemetaan Perkebunan Sawit Blok A" class="w-full p-3 bg-slate-50 border border-slate-200 rounded-lg focus:ring-2 focus:ring-blue-500 outline-none transition">
                </div>
            </div>
            <button onclick="generateInvoice()" class="w-full bg-green-600 text-white font-black py-4 rounded-lg hover:bg-green-700 shadow-md transition uppercase tracking-widest">Terbitkan Invoice Resmi</button>
        </div>

        <div id="invoiceArea" class="bg-white p-12 hidden mx-auto shadow-2xl text-left border border-slate-100" style="width: 210mm; min-height: 297mm;">
            
            <div class="flex justify-between items-start border-b-4 border-slate-950 pb-10 mb-10">
                <div class="space-y-1">
                    <h1 class="text-5xl font-black uppercase tracking-tight-heavy text-slate-950">INVOICE</h1>
                    <p class="text-sm text-slate-600 font-medium" id="invDate"></p>
                </div>
                <div class="text-right space-y-1">
                    <h2 class="font-black text-xl uppercase tracking-tight text-slate-900">Freelance Surveyor</h2>
                    <p class="text-sm text-slate-500">Aerial Mapping & Data Services</p>
                    <div class="mt-3 flex justify-end items-center gap-2 border border-slate-200 p-2 rounded bg-slate-50 no-print">
                        <span class="text-[10px] font-bold text-slate-400 uppercase tracking-wider">No:</span>
                        <input type="text" id="invNumberInput" placeholder="Ketik No. Inv Manual..." class="font-mono text-xs text-right w-32 focus:ring-1 focus:ring-blue-500 outline-none bg-transparent">
                    </div>
                    <p class="text-xs font-mono text-slate-700 mt-2 print-only hidden" id="displayInvNumber"></p>
                </div>
            </div>

            <div class="mb-12 grid grid-cols-3 gap-6">
                <div class="col-span-2 space-y-1">
                    <p class="text-[10px] uppercase font-bold text-slate-400 tracking-widest mb-1.5">Ditujukan Kepada:</p>
                    <p id="displayClient" class="font-black text-3xl text-slate-950 leading-tight">-</p>
                    <p id="displayProject" class="text-xl text-slate-600 font-medium">-</p>
                </div>
                <div class="text-right no-print">
                    <div class="w-20 h-20 bg-slate-950 rounded-full ml-auto flex items-center justify-center text-amber-400 font-black text-3xl rotate-[-15deg]">DB</div>
                </div>
            </div>

            <table class="w-full mb-12 text-left border-collapse">
                <thead>
                    <tr class="border-b-2 border-slate-950 text-[10px] uppercase text-slate-950 font-black tracking-widest">
                        <th class="py-4 px-2 w-1/2">Deskripsi Layanan</th>
                        <th class="py-4 px-2">Spesifikasi & Durasi Misi</th>
                        <th class="py-4 px-2 text-right">Jumlah (IDR)</th>
                    </tr>
                </thead>
                <tbody class="text-sm">
                    <tr class="border-b border-slate-100">
                        <td class="py-8 px-2 align-top">
                            <p class="font-black text-slate-900 text-base">Aerial Mapping & Data Processing</p>
                            <p class="text-xs text-slate-500 mt-1 leading-relaxed">Akuisisi data foto udara presisi tinggi dan pengolahan menjadi produk fotogrametri (Orthomosaic & DSM/DSM).</p>
                        </td>
                        <td class="py-8 px-2 align-top text-xs text-slate-700 leading-loose font-medium" id="invTech">-</td>
                        <td class="py-8 px-2 align-top text-right font-black text-lg text-slate-900" id="invSubtotal">-</td>
                    </tr>
                    <tr id="invSurchargeRow" class="hidden text-orange-700 italic font-medium bg-orange-50/50">
                        <td class="py-3 px-2">Surcharge Overlap (>89%)</td>
                        <td class="text-xs px-2">Kompensasi kepadatan data & waktu processing</td>
                        <td class="py-3 px-2 text-right font-bold" id="invSurchargeVal">-</td>
                    </tr>
                    <tr id="invMinOrderRow" class="hidden text-blue-700 italic font-medium bg-blue-50/50">
                        <td class="py-3 px-2 border-l-4 border-blue-500">Penyesuaian Minimum Order</td>
                        <td class="text-xs px-2">Sesuai kebijakan minimum flight operations fee</td>
                        <td class="py-3 px-2 text-right font-bold" id="invMinOrderVal">-</td>
                    </tr>
                </tbody>
                <tfoot>
                    <tr class="border-t-4 border-slate-950">
                        <td colspan="2" class="py-8 text-right uppercase text-[10px] text-slate-400 tracking-widest font-black align-middle">Total Nilai Tagihan</td>
                        <td class="py-8 text-right text-5xl font-black text-amber-600 tracking-tight-heavy align-middle" id="invTotal">-</td>
                    </tr>
                </tfoot>
            </table>

            <div class="grid grid-cols-2 gap-12 text-[10px] border-t-2 border-dashed border-slate-200 pt-10 mt-auto">
                <div class="space-y-3 relative">
                    <div class="absolute -top-5 -left-5 text-6xl font-black text-slate-100 select-none">PAY</div>
                    <p class="font-black uppercase text-slate-900 tracking-wider relative z-10">Informasi Pembayaran:</p>
                    <div class="bg-slate-50 p-5 border-l-4 border-blue-600 rounded-r relative z-10 space-y-1 text-slate-800">
                        <p class="font-medium">Pembayaran penuh via transfer bank:</p>
                        <p class="mt-2">Bank: <span class="font-bold text-slate-950">Bank Rakyat Indonesia (BRI)</span></p>
                        <p>No. Rekening: <span class="font-black text-blue-800 text-sm tracking-tight">5794 0101 3826 500</span></p>
                        <p>Atas Nama: <span class="font-bold uppercase text-slate-950">Digdo</span></p>
                    </div>
                </div>
                <div class="text-right flex flex-col justify-between items-end relative">
                     <div class="absolute -top-5 -right-5 text-6xl font-black text-slate-100 select-none">SIGN</div>
                    <p class="font-black uppercase text-slate-900 tracking-wider relative z-10">Freelance Surveyor,</p>
                    <div class="relative z-10 mt-16">
                        <div class="border-b-2 border-slate-950 w-60 ml-auto mb-2.5"></div>
                        <p class="font-black text-base text-slate-950">Digdo Bayu Riky Subagja</p>
                        <p class="text-slate-500 font-medium">Licensed Drone Pilot & Geodetic Surveyor</p>
                    </div>
                </div>
            </div>
            
            <div class="no-print mt-16 flex gap-4 justify-center border-t border-slate-100 pt-8">
                <button onclick="handlePrint()" class="bg-slate-950 text-white px-10 py-4 rounded-lg font-black shadow-xl hover:bg-black transition uppercase tracking-widest text-xs">Cetak / Save PDF</button>
                <button onclick="document.getElementById('invoiceArea').classList.add('hidden')" class="bg-slate-100 text-slate-600 px-6 py-4 rounded-lg font-bold hover:bg-slate-200 transition uppercase text-[10px]">Tutup Preview</button>
            </div>
        </div>
    </div>

    <script>
        // --- KONFIGURASI KAKU (JANGAN DIUBAH USER UI) ---
        const CONFIG = {
            minOrder: 2000000,
            rateArea: 250, // per meter lintasan
            rateJalan: 150, // per meter lintasan
            metersPerBattery: 4500,
            maxBatteriesPerDay: 4,
            sensorWidth: 13.2, // Autel Evo II Pro
            focalLength: 8.8,
            imageWidthPx: 5472
        };

        let globalData = {};

        function toggleInput(type) {
            document.getElementById('inputArea').classList.toggle('hidden', type !== 'area');
            document.getElementById('inputJalan').classList.toggle('hidden', type !== 'jalan');
        }

        function formatIDR(n) { return new Intl.NumberFormat('id-ID', { style: 'currency', currency: 'IDR', minimumFractionDigits: 0 }).format(n); }

        function kalkulasiTotal() {
            const H = parseFloat(document.getElementById('tinggi').value);
            const rawLap = parseFloat(document.getElementById('sidelap').value);
            const type = document.querySelector('input[name="mapType"]:checked').value;
            
            if (!H || isNaN(H)) return alert("Masukkan tinggi terbang valid");
            if (rawLap >= 100 || rawLap < 0) return alert("Overlap tidak valid (0-99%)");

            // 1. Hitung GSD & Spacing
            const gsd = (H * CONFIG.sensorWidth * 100) / (CONFIG.focalLength * CONFIG.imageWidthPx);
            const footprintWidth = (CONFIG.imageWidthPx * gsd) / 100;
            const spacing = footprintWidth * (1 - (rawLap/100));
            
            let lintasan = 0; 
            let rate = (type === 'area' ? CONFIG.rateArea : CONFIG.rateJalan);

            // 2. Hitung Panjang Lintasan
            if(type === 'area') {
                const valArea = parseFloat(document.getElementById('nilaiArea').value);
                if(!valArea || isNaN(valArea)) return alert("Masukkan luas area (Ha) valid");
                lintasan = (valArea * 10000 / spacing) * 1.1; // Safety factor 10% untuk belokan
            } else {
                const valPanjang = parseFloat(document.getElementById('nilaiPanjangJalan').value);
                const valLebar = parseFloat(document.getElementById('nilaiLebarJalan').value) || 50;
                if(!valPanjang || isNaN(valPanjang)) return alert("Masukkan panjang jalan valid");
                
                let jlr = Math.ceil(valLebar / spacing);
                if (jlr < 2) jlr = 2; // Minimal 2 jalur untuk koridor
                lintasan = valPanjang * jlr;
            }

            // 3. Hitung Operasional (Baterai & Hari) -> Baru Dihitung di Sini
            const totalBat = Math.ceil(lintasan / CONFIG.metersPerBattery);
            const totalHari = Math.ceil(totalBat / CONFIG.maxBatteriesPerDay);

            // 4. Hitung Biaya
            const pokok = lintasan * rate;
            const surcharge = rawLap > 89 ? pokok * 0.15 : 0;
            const totalBeforeMin = pokok + surcharge;
            const adjMin = totalBeforeMin < CONFIG.minOrder ? CONFIG.minOrder - totalBeforeMin : 0;
            const final = totalBeforeMin + adjMin;

            // Simpan data kalkulasi akhir
            globalData = { gsd, spacing, lintasan, rate, pokok, surcharge, adjMin, final, type, H, rawLap, totalBat, totalHari };

            // --- UPDATE UI HASIL (NO PRINT) ---
            document.getElementById('hasil').classList.remove('hidden');
            
            // Data Teknis Utama (Stand Out)
            document.getElementById('resGsd').innerText = gsd.toFixed(2) + " cm/px";
            document.getElementById('resSpacing').innerText = spacing.toFixed(1) + " m";
            document.getElementById('resTotalLintasan').innerText = Math.round(lintasan).toLocaleString('id-ID') + " m";
            
            // Data Operasional (Baru)
            document.getElementById('resBaterai').innerText = totalBat + " Pcs";
            document.getElementById('resHari').innerText = totalHari + " Hari";

            // Data Biaya
            document.getElementById('resPokok').innerText = formatIDR(pokok);
            document.getElementById('resHarga').innerText = formatIDR(final);

            // Tampilkan baris surcharge jika ada
            const rowS = document.getElementById('rowSurcharge');
            if(surcharge > 0) {
                rowS.classList.remove('hidden');
                document.getElementById('resSurcharge').innerText = "+ " + formatIDR(surcharge);
            } else rowS.classList.add('hidden');

            // Tampilkan baris penyesuaian min order jika ada
            const rowM = document.getElementById('rowMinOrder');
            if(adjMin > 0) {
                rowM.classList.remove('hidden');
                document.getElementById('resAdjMin').innerText = "+ " + formatIDR(adjMin);
            } else rowM.classList.add('hidden');

            document.getElementById('hasil').scrollIntoView({behavior: 'smooth'});
        }

        function showInvoiceForm() {
            document.getElementById('invoiceInputSection').classList.remove('hidden');
            document.getElementById('invoiceInputSection').scrollIntoView({behavior: 'smooth'});
        }

        function generateInvoice() {
            const client = document.getElementById('invClient').value || "Klien Umum";
            const project = document.getElementById('invProject').value || "Mapping Project";
            
            document.getElementById('displayClient').innerText = client;
            document.getElementById('displayProject').innerText = project;
            document.getElementById('invDate').innerText = new Date().toLocaleDateString('id-ID', { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' });

            // Isi Detail Spesifikasi & Durasi di Tabel Invoice
            document.getElementById('invTech').innerHTML = `
                <div class="space-y-1 monospace text-[11px]">
                    <p>• Tinggi Terbang: <span class="font-bold text-slate-900">${globalData.H} m</span></p>
                    <p>• Resolusi (GSD): <span class="font-bold text-slate-900">${globalData.gsd.toFixed(2)} cm/px</span></p>
                    <p>• Overlap/Sidelap: <span class="font-bold text-slate-900">${globalData.rawLap}%</span> (Jarak Jalur: ${globalData.spacing.toFixed(1)}m)</p>
                    <p>• Vol. Lintasan: <span class="font-bold text-slate-900">${Math.round(globalData.lintasan).toLocaleString('id-ID')} m</span></p>
                    <p>• Estimasi Energi: <span class="font-bold text-slate-900">${globalData.totalBat} Baterai</span></p>
                    <p class="mt-2 text-blue-700 font-bold border-t border-slate-100 pt-1">• Estimasi Pengerjaan Lapangan: <span class="text-sm">${globalData.totalHari} Hari Kerja</span></p>
                </div>
            `;
            
            // Isi Biaya
            document.getElementById('invSubtotal').innerText = formatIDR(globalData.pokok);
            
            const isr = document.getElementById('invSurchargeRow');
            if(globalData.surcharge > 0) {
                isr.classList.remove('hidden');
                document.getElementById('invSurchargeVal').innerText = "+ " + formatIDR(globalData.surcharge);
            } else isr.classList.add('hidden');

            const imr = document.getElementById('invMinOrderRow');
            if(globalData.adjMin > 0) {
                imr.classList.remove('hidden');
                document.getElementById('invMinOrderVal').innerText = "+ " + formatIDR(globalData.adjMin);
            } else imr.classList.add('hidden');

            // Total Akhir (Emas)
            document.getElementById('invTotal').innerText = formatIDR(globalData.final);

            // Tampilkan Area Invoice
            document.getElementById('invoiceArea').classList.remove('hidden');
            document.getElementById('invoiceArea').scrollIntoView({behavior: 'smooth'});
        }

        // Handle sinkronisasi input manual no invoice untuk print
        function handlePrint() {
            const printOnlyNo = document.getElementById('displayInvNumber');
            const inputNo = document.getElementById('invNumberInput').value;
            
            if(inputNo) {
                printOnlyNo.innerText = "No: " + inputNo;
                printOnlyNo.classList.remove('hidden');
            } else {
                printOnlyNo.classList.add('hidden');
            }
            
            window.print();
        }
    </script>
</body>
</html>
