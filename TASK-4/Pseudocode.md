BASLA
  // Kimlik Doğrulama
  DÖNGÜ
    YAZ "Öğrenci No giriniz:"
    OKU ogr_no
    YAZ "Şifre giriniz:"
    OKU sifre
    EĞER kimlik_dogrula(ogr_no, sifre) İSE
      KIR // giriş başarılı, döngüden çık
    DEĞİLSE
      YAZ "Hatalı giriş. Tekrar deneyiniz."
    BİTİR
  BİTİR

  // Başlangıç durumları
  secilen_dersler ← ∅
  toplam_kredi ← 0
  GPA ← ogrenci_gpa(ogr_no)

  // Ders Listesi Gösterimi
  dersler ← donem_ders_listesi()
  YAZ "Açılan Dersler:"
  DÖNGÜ ders IN dersler
    YAZ ders.kod, ders.ad, ders.kredi, ders.gün_saat, "Kontenjan:", ders.kontenjan_durumu
  BİTİR

  // Ekle/Çıkar İşlemleri
  DÖNGÜ
    YAZ "[E]kle, [Ç]ıkar, [B]itir?"
    OKU islem
    EĞER islem = "E" İSE
      YAZ "Eklemek istediğiniz ders kodu:"
      OKU kod
      ders ← dersi_bul(dersler, kod)
      EĞER ders = YOK İSE
        YAZ "Ders bulunamadı."
        DEVAM
      BİTİR

      // 1) Kontenjan kontrolü
      EĞER ders.kontenjan_dolu_mu? İSE
        YAZ "Kontenjan dolu. Ders eklenemedi."
        DEVAM
      BİTİR

      // 2) Ön koşul kontrolü
      EĞER önkoşul_var_mı(ders) İSE
        EĞER önkoşul_tamam_mı(ogr_no, ders) DEĞİLSE
          YAZ "Ön koşul sağlanmıyor. Ders eklenemedi."
          DEVAM
        BİTİR
      BİTİR

      // 3) Zaman çakışması kontrolü
      EĞER zaman_çakışır_mı(ders, secilen_dersler) İSE
        YAZ "Zaman çakışması var. Ders eklenemedi."
        DEVAM
      BİTİR

      // 4) Kredi limiti kontrolü
      EĞER (toplam_kredi + ders.kredi) > 35 İSE
        YAZ "Kredi limiti aşılıyor (≤ 35). Ders eklenemedi."
        DEVAM
      BİTİR

      // Tüm kontroller geçti -> ekle
      secilen_dersler ← secilen_dersler ∪ {ders}
      toplam_kredi ← toplam_kredi + ders.kredi
      YAZ "Ders eklendi."
    
    DEĞİLSE EĞER islem = "Ç" İSE
      YAZ "Çıkarmak istediğiniz ders kodu:"
      OKU kod
      ders ← dersi_bul(secilen_dersler, kod)
      EĞER ders = YOK İSE
        YAZ "Seçilenler arasında bu ders yok."
      DEĞİLSE
        secilen_dersler ← secilen_dersler \ {ders}
        toplam_kredi ← toplam_kredi - ders.kredi
        YAZ "Ders çıkarıldı."
      BİTİR

    DEĞİLSE EĞER islem = "B" İSE
      KIR
    DEĞİLSE
      YAZ "Geçersiz seçim."
    BİTİR
  BİTİR

  // Kayıt Özeti
  YAZ "---- Kayıt Özeti ----"
  DÖNGÜ d IN secilen_dersler
    YAZ d.kod, d.ad, d.kredi, d.gün_saat
  BİTİR
  YAZ "Toplam kredi:", toplam_kredi

  // Danışman Onayı Gerekiyor mu?
  EĞER GPA < 2.5 İSE
    YAZ "GPA < 2.5: Danışman onayı gerekli."
    EĞER danışman_onayla(ogr_no, secilen_dersler) DEĞİLSE
      YAZ "Danışman onayı reddedildi. Kayıt tamamlanamadı."
      BİTİR
    BİTİR
  BİTİR

  // Kayıt Onayı (Öğrenci)
  YAZ "Kaydı onaylıyor musunuz? [E/H]"
  OKU onay
  EĞER onay = "E" İSE
    EĞER kaydı_kesinleştir(ogr_no, secilen_dersler) İSE
      YAZ "Kayıt başarıyla tamamlandı."
    DEĞİLSE
      YAZ "Kayıt sırasında hata oluştu."
    BİTİR
  DEĞİLSE
    YAZ "Kayıt iptal edildi."
  BİTİR
BİTİR
