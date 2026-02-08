<!DOCTYPE html>
<html>
  <head>
    <title>AR Projesi - Dönen Renkli Çerçeve</title>
    <!-- A-Frame ve AR.js -->
    <script src="https://aframe.io/releases/1.2.0/aframe.min.js"></script>
    <script src="https://raw.githack.com/AR-js-org/AR.js/master/aframe/build/aframe-ar.js"></script>

    <style>
      /* --- DÖNEN ÇERÇEVE KAPSAYICISI --- */
      #border-wrapper {
        position: fixed;
        top: 15px;
        right: 15px;
        z-index: 1000;
        
        /* Çerçevenin kalınlığını buradan ayarlayabilirsin (padding) */
        padding: 4px; 
        
        border-radius: 30px; /* Yuvarlak köşeler (Hap şekli) */
        overflow: hidden;    /* Dışarı taşan renkleri kırp */
        display: flex;
        justify-content: center;
        align-items: center;
      }

      /* Arkada Sürekli Dönen Renkli Katman */
      #border-wrapper::before {
        content: '';
        position: absolute;
        width: 200%;  /* Kutudan daha geniş olmalı ki dönerken boşluk kalmasın */
        height: 200%;
        
        /* Renkler JS ile buraya atanacak (conic-gradient) */
        background: conic-gradient(red, blue, green, yellow, red); 
        
        /* Dönme Animasyonu */
        animation: spin 3s linear infinite; 
      }

      /* Animasyon Tanımı */
      @keyframes spin {
        0% { transform: rotate(0deg); }
        100% { transform: rotate(360deg); }
      }

      /* --- İÇERİK KUTUSU (Siyah Alan) --- */
      #content-box {
        position: relative;
        background-color: rgba(40, 40, 40, 0.9); /* Koyu gri zemin */
        color: white;
        padding: 8px 15px;
        border-radius: 26px; /* Dış çerçeveden biraz daha az kavis */
        display: flex;
        align-items: center;
        gap: 10px;
        font-family: Arial, sans-serif;
        font-weight: bold;
        z-index: 2; /* Renklerin üstünde durması için */
      }

      /* "i" Butonu */
      .info-btn {
        width: 24px;
        height: 24px;
        background-color: white;
        color: #333;
        border-radius: 50%;
        border: none;
        font-weight: 900;
        font-size: 14px;
        cursor: pointer;
        display: flex; align-items: center; justify-content: center;
      }

      /* --- MODAL (Açılır Pencere) --- */
      #modal-overlay {
        display: none;
        position: fixed; top: 0; left: 0; width: 100%; height: 100%;
        background-color: rgba(0,0,0,0.8); z-index: 2000;
        justify-content: center; align-items: center;
      }
      #modal-content {
        background-color: #222; color: #fff; padding: 20px;
        border-radius: 10px; width: 80%; max-width: 400px;
        font-family: sans-serif;
      }
      .close-btn { background: #ff4444; color: white; border: none; padding: 5px 10px; float: right; cursor: pointer; border-radius: 4px;}
    </style>
  </head>
  
  <body style="margin : 0px; overflow: hidden;">

    <!-- 1. DÖNEN ÇERÇEVELİ ARAYÜZ -->
    <div id="border-wrapper">
      <div id="content-box">
        <span id="version-text">v1.6.0</span>
        <button class="info-btn" onclick="openModal()">i</button>
      </div>
    </div>

    <!-- Modal -->
    <div id="modal-overlay">
      <div id="modal-content">
        <h3>Sürüm Geçmişi</h3>
        <ul>
          <li><strong>v1.6.0</strong>: Çerçeve renkleri parçalı ve dönüyor.</li>
          <li><strong>v1.2.0</strong>: Küp fıstık yeşili oldu.</li>
        </ul>
        <button class="close-btn" onclick="closeModal()">Kapat</button>
      </div>
    </div>

    <!-- 2. AR SAHNESİ -->
    <a-scene embedded arjs>
      <a-marker type="pattern" url="qr/pattern-ar_projesi.patt">
        <a-box position="0 0.5 0" material="color: #93C572; opacity: 1;"></a-box>
      </a-marker>
      <a-entity camera></a-entity>
    </a-scene>

    <!-- 3. JAVASCRIPT RENK MANTIĞI -->
    <script>
      // --- AYARLAR ---
      const CURRENT_VERSION = "v1.6.0"; // Burayı değiştirdiğinde renkler de değişir!

      // Renk Paletleri Listesi (Sırasıyla değişir)
      const palettes = [
        // 0. Palet (v1.0.0 vb.): Standart Gökkuşağı
        ['red', 'orange', 'yellow', 'green', 'blue', 'indigo', 'violet'],
        
        // 1. Palet (v1.1.0 vb.): Neon Siber
        ['#00FF00', '#000000', '#00FF00', '#000000'],
        
        // 2. Palet (v1.2.0 vb.): Türk Bayrağı
        ['red', 'white', 'red', 'white'],

        // 3. Palet (v1.3.0 vb.): Google Renkleri
        ['#4285F4', '#DB4437', '#F4B400', '#0F9D58'],
        
        // ...
        
        // 6. Palet (v1.6.0 - SENİN GÖRSELİN): Mor, Yeşil, Kırmızı, Turuncu
        ['#800080', '#90EE90', '#FF0000', '#FFA500'] 
      ];

      function setupDynamicBorder() {
        document.getElementById('version-text').innerText = CURRENT_VERSION;

        // Versiyon numarasını al (v1.6.0 -> 6)
        const parts = CURRENT_VERSION.replace('v', '').split('.');
        const minorVer = parseInt(parts[1]); 

        // Listeden sıradaki paleti seç (Sayı listeden büyükse başa döner)
        const selectedPalette = palettes[minorVer % palettes.length];
        
        // Renkleri CSS için hazırla
        // Conic gradient için renkleri virgülle birleştiriyoruz
        // Önemli: Geçişin pürüzsüz olması için ilk rengi en sona tekrar ekliyoruz
        let gradientColors = [...selectedPalette, selectedPalette[0]].join(', ');

        // CSS değişkenini güncelle (::before elementine etki etmesi için)
        // JavaScript ile pseudo-element'e (::before) doğrudan erişilemez,
        // bu yüzden yeni bir stil kuralı ekliyoruz.
        
        const styleSheet = document.createElement("style");
        styleSheet.innerText = `
          #border-wrapper::before {
            background: conic-gradient(${gradientColors}) !important;
          }
        `;
        document.head.appendChild(styleSheet);
      }

      // Sayfa yüklenince çalıştır
      setupDynamicBorder();

      // Modal Fonksiyonları
      function openModal() { document.getElementById('modal-overlay').style.display = 'flex'; }
      function closeModal() { document.getElementById('modal-overlay').style.display = 'none'; }
    </script>
  </body>
</html>
