BAŞLA

YAZ "TC Kimlik Numaranızı Giriniz:"
OKU tc_no

EĞER tc_no geçerli değilse
    YAZ "Geçersiz kimlik numarası!"
    BİTİR
DEĞİLSE
    YAZ "Poliklinik seçiniz:"
    OKU poliklinik

    YAZ "Uygun doktorlar listeleniyor..."
    DOKTOR_LISTESİ = doktor_listesi_getir(poliklinik)

    HER doktor in DOKTOR_LISTESİ İÇİN
        YAZ doktor
    SON

    YAZ "Doktor seçiniz:"
    OKU doktor

    YAZ "Uygun saatler listeleniyor..."
    SAAT_LISTESİ = uygun_saatler_getir(doktor)

    HER saat in SAAT_LISTESİ İÇİN
        YAZ saat
    SON

    YAZ "Randevu saati seçiniz:"
    OKU saat

    YAZ "Randevunuz onaylanıyor..."
    YAZ "Randevu Onaylandı! SMS ile bilgilendirme gönderildi."

BİTİR
