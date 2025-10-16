BAŞLA
  // 1) GİRİŞ
  EĞER kullanıcı_oturumda_mı() İSE
    DEVAM_ET
  DEĞİLSE
    giriş_başarılı ← kullanıcı_giriş_yap()
    EĞER giriş_başarılı İSE DEVAM_ET
    DEĞİLSE
      YAZ "Giriş yapılmadı. İşlem sonlandırıldı."
      BİTİR

  // 2) GEZİNME & ÜRÜN EKLEME
  DÖNGÜ kategoriler_ve_ürünler_arasında_gezin()
    ürün ← kullanıcı_ürün_seç()
    EĞER ürün = YOK İSE ÇIKIŞ_DÖNGÜ
    adet ← kullanıcı_adet_seç()
    EĞER stok_yeterli_mi(ürün, adet) İSE
      sepete_ekle(ürün, adet)
      YAZ "Ürün sepete eklendi."
    DEĞİLSE
      YAZ "Yetersiz stok."
    EĞER kullanıcı_devam_mı() DEĞİL İSE ÇIKIŞ_DÖNGÜ
  SON_DÖNGÜ

  // 3) SEPET DÜZENLEME
  DÖNGÜ sepeti_görüntüle()
    işlem ← kullanıcı_sepet_işlemi_seç() // artır/azalt/kaldır/devam
    EĞER işlem = "artır" VEYA işlem = "azalt" İSE
      yeni_adet ← kullanıcının_belirlediği_adet()
      EĞER stok_yeterli_mi(seçili_ürün, yeni_adet) İSE
        adet_güncelle(seçili_ürün, yeni_adet)
      DEĞİLSE
        YAZ "Yetersiz stok."
    EĞER işlem = "kaldır" İSE
      sepetten_kaldır(seçili_ürün)
    EĞER işlem = "devam" İSE ÇIKIŞ_DÖNGÜ
  SON_DÖNGÜ

  ara_toplam ← sepet_ara_toplam_hesapla()

  // 4) İNDİRİM KODU
  EĞER kullanıcı_kupon_girdi_mi() İSE
    kupon ← kupon_kodu_al()
    EĞER kupon_geçerli_mi(kupon, sepet) İSE
      indirim ← kupon_indirim_hesapla(kupon, sepet)
    DEĞİLSE
      YAZ "Geçersiz kupon."
      indirim ← 0
  DEĞİLSE
    indirim ← 0

  net_sepet ← ara_toplam - indirim

  // 5) MİNİMUM TUTAR (50 TL)
  EĞER net_sepet >= 50 İSE
    DEVAM_ET
  DEĞİLSE
    YAZ "Minimum sipariş tutarı 50 TL."
    GİT "SEPET DÜZENLEME" // kullanıcı ürün ekleyebilir
    // pratikte yukarıdaki bloğa döndürülür

  // 6) KARGO
  EĞER net_sepet >= 200 İSE
    kargo_ücreti ← 0
  DEĞİLSE
    kargo_ücreti ← kargo_hesapla(kullanıcı_adresi)
  sipariş_tutarı ← net_sepet + kargo_ücreti

  // 7) ÖDEME YÖNTEMİ
  ödeme_yöntemi ← kullanıcı_ödeme_yöntemi_seç()
  EĞER ödeme_yöntemi = "KART" İSE
    sonuç ← kart_ödemesi_al(sipariş_tutarı, kart_bilgileri)
    EĞER sonuç = "BAŞARILI" İSE
      SİPARİŞ_OLUŞTUR()
    DEĞİLSE
      YAZ "Ödeme başarısız. Yeniden deneyin veya yöntem değiştirin."
      GİT "ÖDEME YÖNTEMİ"
  EĞER ödeme_yöntemi = "HAVALE" İSE
    IBAN_bilgi_göster()
    sipariş_durumu ← "ÖDEME BEKLENİYOR"
    SİPARİŞ_OLUŞTUR()
  EĞER ödeme_yöntemi = "KAPIDA" İSE
    EĞER kapıda_müsait_mi(kullanıcı_adresi) İSE
      ek_ücret ← kapıda_ödeme_ücreti()
      sipariş_tutarı ← sipariş_tutarı + ek_ücret
      SİPARİŞ_OLUŞTUR()
    DEĞİLSE
      YAZ "Kapıda ödeme mevcut değil."
      GİT "ÖDEME YÖNTEMİ"

  // 8) ONAY & SON
  stokları_düş()
  fatura_oluştur()
  bildirim_gönder(e-posta, SMS)
  kargo_sürecini_başlat()
  YAZ "Siparişiniz oluşturuldu."
BİTİR
