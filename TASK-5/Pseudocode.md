BAŞLA
    // Başlatma
    YAZ "Sistem başlatılıyor..."
    alarmDurumu ← KAPALI
    tehditSeviyesi ← "YOK"
    günlükLimitBildirim ← 0
    sonBildirimZamanı ← 0
    sessizMod ← YANLIŞ

    DÖNGÜ DOĞRU // 7/24 sürekli çalış
        // 1) Sensörleri Oku
        OKU kapıPencere, hareketPIR, duman, CO, camKırılma, titreşim, kameraAI, sabotaj, internetDurumu, pilSeviyesi

        // 2) Hızlı Sağlık Kontrolleri
        EĞER-İSE pilSeviyesi < %15 İSE
            YAZ "Uyarı: Pil seviyesi düşük"
            EĞER-İSE (Şimdi - sonBildirimZamanı) > 10dk İSE BİLDİR "Pil düşük", KRİTİK; sonBildirimZamanı ← Şimdi
        BİTİR
        EĞER-İSE internetDurumu = KAPALI İSE
            YAZ "Uyarı: İnternet bağlantısı yok"
            // SMS/GSM yedek kanala geç
            BİLDİR "İnternet yok, GSM'e geçildi", BİLGİ
        BİTİR

        // 3) Tehdit Değerlendirme (kurallı birleştirme & ağırlıklandırma)
        tehditSkoru ← 0
        EĞER-İSE kapıPencere=AÇILDI VE hareketPIR=VAR İSE tehditSkoru += 60
        EĞER-İSE camKırılma=VAR VE titreşim=YÜKSEK İSE tehditSkoru += 50
        EĞER-İSE kameraAI=İNSAN_TESPİT İSE tehditSkoru += 40
        EĞER-İSE duman=VAR VEYA CO=YÜKSEK İSE tehditSkoru += 100   // Can güvenliği öncelikli
        EĞER-İSE sabotaj=VAR İSE tehditSkoru += 80

        // 4) Tehdit Seviyesi Eşikleri
        EĞER-İSE tehditSkoru ≥ 100 İSE tehditSeviyesi ← "KRİTİK"
        DEĞİLSE EĞER-İSE tehditSkoru ≥ 60 İSE tehditSeviyesi ← "YÜKSEK"
        DEĞİLSE EĞER-İSE tehditSkoru ≥ 30 İSE tehditSeviyesi ← "ORTA"
        DEĞİLSE tehditSeviyesi ← "DÜŞÜK"
        BİTİR

        // 5) Aksiyonlar
        EĞER-İSE tehditSeviyesi = "DÜŞÜK" İSE
            YAZ "Düşük seviye alarm: sadece olay kaydı"
            KAYDET "Düşük tehdit", ZAMAN=Şimdi
        BİTİR

        EĞER-İSE tehditSeviyesi = "ORTA" İSE
            YAZ "Orta seviye: yerel uyarı ve bildirim"
            EĞER-İSE sessizMod = YANLIŞ İSE YEREL_BUZZER_AÇ(5 sn) BİTİR
            BİLDİR "Orta seviye tehdit algılandı", UYARI
            alarmDurumu ← AÇIK
        BİTİR

        EĞER-İSE tehditSeviyesi = "YÜKSEK" İSE
            YAZ "Yüksek seviye: siren + anlık bildirimler"
            SİREN_AÇ
            BİLDİR "Yüksek seviye ihlal!", KRİTİK
            CANLI_GÖRÜNTÜ_PAYLAŞ(kameraAI)
            alarmDurumu ← AÇIK
        BİTİR

        EĞER-İSE tehditSeviyesi = "KRİTİK" İSE
            YAZ "Kritik seviye: siren + acil çağrı + tüm kanallar"
            SİREN_AÇ
            BİLDİR "KRİTİK: DUMAN/GAZ!", ACİL
            ARAMA_YAP("İtfaiye/Acil")
            alarmDurumu ← AÇIK
        BİTİR

        // 6) Bildirim Hızı Sınırla (rate limit)
        EĞER-İSE (Şimdi - sonBildirimZamanı) > 60 sn İSE
            // Yeni olaylar için tekrar bildirim göndermeye izin ver
            sonBildirimZamanı ← Şimdi
        BİTİR

        // 7) Alarm Sıfırlama/Susturma
        EĞER-İSE KOMUT_GELDİ("ALARM_SIFIRLA") İSE
            SİREN_KAPAT
            alarmDurumu ← KAPALI
            YAZ "Alarm sıfırlandı."
        BİTİR
        EĞER-İSE KOMUT_GELDİ("SESSİZ_MOD_AÇ") İSE sessizMod ← DOĞRU BİTİR
        EĞER-İSE KOMUT_GELDİ("SESSİZ_MOD_KAPAT") İSE sessizMod ← YANLIŞ BİTİR

        // 8) Döngü Sonu Gecikmesi (debounce + CPU rahatlatma)
        UYKU(200 ms)
    BİTİR DÖNGÜ
BİTİR
