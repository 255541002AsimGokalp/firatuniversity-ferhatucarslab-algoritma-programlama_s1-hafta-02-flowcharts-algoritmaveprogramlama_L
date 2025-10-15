BAŞLA
    kalan_hak ← 3
    günlük_limit ← 5000
    bakiye ← 10000

    DÖNGÜ kalan_hak > 0 İKEN
        YAZ "Lütfen PIN'inizi giriniz: "
        OKU girilen_pin

        EĞER girilen_pin == dogru_pin İSE
            YAZ "PIN doğrulandı."
            
            DÖNGÜ TRUE İKEN
                YAZ "Çekmek istediğiniz tutarı giriniz (20 TL katı olmalı): "
                OKU tutar

                EĞER tutar % 20 ≠ 0 İSE
                    YAZ "HATA: Tutar 20 TL'nin katı olmalıdır!"
                DEĞİLSE
                    EĞER tutar > bakiye İSE
                        YAZ "HATA: Yetersiz bakiye!"
                    DEĞİLSE
                        EĞER tutar > günlük_limit İSE
                            YAZ "HATA: Günlük limit aşıldı!"
                        DEĞİLSE
                            bakiye ← bakiye - tutar
                            günlük_limit ← günlük_limit - tutar
                            YAZ "İşlem başarılı. Kalan bakiye: ", bakiye
                        SON
                    SON
                SON

                YAZ "Başka işlem yapmak istiyor musunuz? (E/H)"
                OKU cevap
                EĞER cevap == "H" VEYA cevap == "h" İSE
                    ÇIK
                SON
            SON DÖNGÜ

            ÇIK  // PIN doğrulandıktan sonra işlemler bittiğinde sistemden çık
        DEĞİLSE
            kalan_hak ← kalan_hak - 1
            YAZ "Hatalı PIN! Kalan deneme hakkınız: ", kalan_hak
        SON
    SON DÖNGÜ

    EĞER kalan_hak == 0 İSE
        YAZ "Kartınız bloke edilmiştir. Lütfen bankanızla iletişime geçiniz."
    SON
BİTİR
