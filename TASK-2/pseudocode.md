BAŞLA
    // 1. KULLANICI GİRİŞİ
    GİRİŞ_EKRANI_GÖSTER()
    KULLANICI_BİLGİLERİ_AL(email, sifre)
    
    EĞER KULLANICI_BİLGİLERİ_DOĞRU(email, sifre) İSE
        OTURUM_BAŞLAT(kullanıcıID)
        SEPET = ÖNCEKİ_SEPETİ_YÜKLE(kullanıcıID)
    DEĞİLSE
        HATA_MESAJI("Geçersiz giriş bilgisi")
        SİSTEMDEN_ÇIK()
    SON
    
    // 2. ÜRÜN EKLEME
    DÖNGÜ ürün_seçimi_süresi_boyunca
        ÜRÜN = ÜRÜN_SEÇ()
        EĞER ÜRÜN_SEÇİLDİ(ÜRÜN) İSE
            STOK = STOK_KONTROL(ÜRÜN)
            
            EĞER STOK > 0 İSE
                ADET = KULLANICIDAN_ADET_AL()
                EĞER ADET <= STOK İSE
                    SEPETE_EKLE(ÜRÜN, ADET)
                    MESAJ("Ürün sepete eklendi.")
                DEĞİLSE
                    MESAJ("Yetersiz stok. Maksimum " + STOK + " adet eklenebilir.")
                SON
            DEĞİLSE
                MESAJ("Ürün stokta yok.")
            SON
        DEĞİLSE
            ÇIKIŞ_SEÇİLDİ_Mİ? = KONTROL_ET()
            EĞER ÇIKIŞ_SEÇİLDİ_Mİ? İSE DÖNGÜDEN_ÇIK()
        SON
    SON_DÖNGÜ
    
    // 3. SEPET GÖRÜNTÜLEME VE DÜZENLEME
    SEPETİ_GÖSTER(SEPET)
    DÖNGÜ kullanıcı_sepet_düzenleme_yapıyor_mu?
        İŞLEM = SEPET_İŞLEM_SEÇ()
        EĞER İŞLEM == "Adet Artır" İSE
            ÜRÜN = ÜRÜN_SEÇ()
            STOK = STOK_KONTROL(ÜRÜN)
            EĞER SEPET[ÜRÜN].adet + 1 <= STOK İSE
                SEPET[ÜRÜN].adet += 1
            DEĞİLSE
                MESAJ("Yetersiz stok.")
            SON
        EĞER İŞLEM == "Ürün Sil" İSE
            ÜRÜN = ÜRÜN_SEÇ()
            SEPETTEN_SİL(ÜRÜN)
        EĞER İŞLEM == "Devam" İSE
            DÖNGÜDEN_ÇIK()
        SON
        SEPET_TOPLAMINI_GÜNCELLE()
    SON_DÖNGÜ
    
    // 4. İNDİRİM KODU
    EĞER KULLANICI_KOD_GİRMEK_İSTİYOR_MU? İSE
        KOD = İNDİRİM_KODU_AL()
        EĞER KOD_GEÇERLİ_Mİ(KOD) İSE
            İNDİRİM_TUTARI = KODU_UYGULA(SEPET, KOD)
            MESAJ("İndirim uygulandı: " + İNDİRİM_TUTARI + " TL")
        DEĞİLSE
            MESAJ("Geçersiz veya süresi dolmuş kod.")
        SON
    SON
    
    // 5. KARGO HESAPLAMA
    ADRES = ADRES_SEÇ_VEYA_GİR()
    KARGO_SEÇENEKLERİ = KARGO_HESAPLA(ADRES, SEPET)
    KARGO = KULLANICIDAN_KARGO_SEÇ(KARGO_SEÇENEKLERİ)
    SEPET_TOPLAMI = HESAPLA_TOPLAM(SEPET, KARGO)
    
    // 6. ÖDEME AŞAMASI
    ÖDEME_YÖNTEMİ = ÖDEME_YÖNTEMİ_SEÇ()
    EĞER ÖDEME_YÖNTEMİ == "Kredi Kartı" İSE
        KART_BİLGİLERİ_AL()
        SONUÇ = ÖDEME_SAĞLAYICISINA_GÖNDER(KART_BİLGİLERİ, SEPET_TOPLAMI)
        EĞER SONUÇ == "ONAY" İSE
            SİPARİŞ_OLUŞTUR(kullanıcıID, SEPET, ADRES, KARGO)
            STOK_GÜNCELLE(SEPET)
            FATURA_OLUŞTUR()
            MESAJ("Ödeme başarılı! Siparişiniz oluşturuldu.")
        DEĞİLSE
            MESAJ("Ödeme reddedildi. Lütfen tekrar deneyin.")
        SON
    DEĞİLSE_EĞER ÖDEME_YÖNTEMİ == "Kapıda Ödeme" İSE
        SİPARİŞ_OLUŞTUR(kullanıcıID, SEPET, ADRES, KARGO)
        MESAJ("Siparişiniz alındı, ödemeyi teslimatta yapabilirsiniz.")
    DEĞİLSE_EĞER ÖDEME_YÖNTEMİ == "Havale" İSE
        BANKA_BİLGİLERİNİ_GÖSTER()
        SİPARİŞ_OLUŞTUR(kullanıcıID, SEPET, ADRES, KARGO)
    SON
    
    // 7. SİPARİŞ TAKİBİ
    MESAJ("Sipariş numaranız: " + SİPARİŞ_NO)
    KARGO_TAKİP_BİLGİSİ_GÖSTER()
    
    // 8. OTURUM SONLANDIRMA
    OTURUMU_KAPAT()
BİTİR
