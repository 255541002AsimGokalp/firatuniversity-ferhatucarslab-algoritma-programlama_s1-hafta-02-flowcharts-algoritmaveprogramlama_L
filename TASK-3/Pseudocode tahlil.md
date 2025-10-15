BAŞLA

YAZ "TC Kimlik Numaranızı Giriniz:"
OKU tc_no

EĞER tc_no geçerli değilse
    YAZ "Geçersiz kimlik numarası!"
    BİTİR
DEĞİLSE
    EĞER tahlil_var_mı(tc_no) == HAYIR İSE
        YAZ "Tahlil kaydı bulunamadı."
    DEĞİLSE
        EĞER sonuc_hazir_mı(tc_no) == EVET İSE
            YAZ "Tahlil Sonucu:"
            YAZ tahlil_sonuc(tc_no)
            YAZ "PDF olarak indirmek ister misiniz? (E/H)"
            OKU secim
            EĞER secim == 'E' İSE
                YAZ "PDF indiriliyor..."
            SON
        DEĞİLSE
            YAZ "Tahlil sonucu henüz hazır değil, lütfen daha sonra tekrar deneyiniz."
        SON
    SON

BİTİR
