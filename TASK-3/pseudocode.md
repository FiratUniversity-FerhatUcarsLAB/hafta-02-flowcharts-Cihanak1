Fonksiyon KimlikDogrula(TCKimlik, Sifre)
    // Veritabanında kimlik ve şifre kontrolü yapılır
    Eğer (TCKimlik ve Sifre sistemde kayıtlı ise)
        Dön TRUE
    Aksi halde
        Dön FALSE
Son

Fonksiyon PoliklinikListesiGoster()
    // Örnek: Dahiliye, Kardiyoloji, Göz, Ortopedi...
Son

Fonksiyon DoktorListesiGoster(Poliklinik)
    // Seçilen polikliniğe göre doktorları listeler
Son

Fonksiyon UygunSaatleriGetir(Doktor)
    // Doktorun uygun olduğu saatleri döndürür
Son

Fonksiyon SmsGonder(TCKimlik, Doktor, Saat)
    // Kullanıcıya SMS ile randevu bilgisi gönderir
Son
