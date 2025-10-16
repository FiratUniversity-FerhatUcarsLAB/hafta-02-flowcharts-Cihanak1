BASLA

// Başlangıç değişkenleri (gerçek sistemde bunlar veri tabanından / karttan okunur)
correctPIN ← 1234                // gerçek PIN (örnek)
maxPinAttempts ← 3
pinAttempts ← 0
cardInserted ← DOĞRU             // kart takılı mı?
accountBalance ← 5000.00         // hesap bakiyesi (TL)
dailyLimit ← 2000.00             // günlük çekim limiti (TL)
dailyWithdrawn ← 0.00            // bugün zaten çekilmiş toplam (TL)
atmCashAvailable ← 10000.00      // ATM'de kalan nakit (TL)
continueSession ← DOĞRU

EĞER cardInserted DEĞİLSE
    YAZ "Lütfen kartınızı takınız."
    DUR
SON

// PIN doğrulama (en çok 3 deneme)
DÖNGÜ (pinAttempts < maxPinAttempts) VE (continueSession = DOĞRU)
    YAZ "Lütfen PIN kodunuzu giriniz:"
    OKU enteredPIN

    EĞER enteredPIN = correctPIN İSE
        YAZ "PIN doğrulandı. İşlemlere yönlendiriliyorsunuz."
        BREAK   // döngüden çık, PIN başarılı
    DEĞİLSE
        pinAttempts ← pinAttempts + 1
        kalanHak ← maxPinAttempts - pinAttempts
        EĞER kalanHak > 0 İSE
            YAZ "Hatalı PIN. Kalan hak: " + kalanHak
        DEĞİLSE
            YAZ "PIN hakkınız tükendi. Kart bloke ediliyor."
            // burada kart bloke işlemi yapılır
            continueSession ← YANLIŞ
            // işlem sonlandırılacak
        SON
    SON
SON  // PIN doğrulama döngüsü sonu

EĞER continueSession = YANLIŞ VE (pinAttempts >= maxPinAttempts) İSE
    YAZ "Oturum sonlandırılıyor."
    DUR
SON

// Başarılı PIN sonrası işlem döngüsü — kullanıcı istediği kadar işlem tekrar edebilir
DÖNGÜ (continueSession = DOĞRU)
    YAZ "Mevcut bakiye: " + accountBalance + " TL"
    YAZ "Bugün çekilen toplam: " + dailyWithdrawn + " TL (Günlük limit: " + dailyLimit + " TL)"
    YAZ "Lütfen yapılacak işlemi seçin:"
    YAZ "1 - Para çekme"
    YAZ "2 - Bakiye görüntüle"
    YAZ "3 - Kart iade / Çıkış"
    OKU choice

    EĞER choice = 1 İSE
        // Para çekme akışı
        YAZ "Çekmek istediğiniz tutarı giriniz (20 TL katları). İptal için 0 giriniz:"
        OKU withdrawAmount

        EĞER withdrawAmount = 0 İSE
            YAZ "İşlem iptal edildi. Ana menüye dönülüyor."
            DEVAM ET  // döngünün başına dön
        SON

        // Tutar pozitif mi kontrolü
        EĞER withdrawAmount <= 0 İSE
            YAZ "Geçersiz tutar. Lütfen pozitif bir tutar girin."
            DEVAM ET
        SON

        // 20 TL katı kontrolü
        EĞER (withdrawAmount % 20) <> 0 İSE
            YAZ "İşlem başarısız: Lütfen 20 TL'nin katları şeklinde bir tutar girin."
            DEVAM ET
        SON

        // ATM'de yeterli nakit var mı?
        EĞER withdrawAmount > atmCashAvailable İSE
            YAZ "ATM'de yeterli nakit yok. Lütfen daha küçük bir tutar deneyin."
            DEVAM ET
        SON

        // Hesap bakiyesi kontrolü
        EĞER withdrawAmount > accountBalance İSE
            YAZ "İşlem başarısız: Hesabınızda yeterli bakiyeniz yok."
            DEVAM ET
        SON

        // Günlük limit kontrolü
        remainingDailyLimit ← dailyLimit - dailyWithdrawn
        EĞER withdrawAmount > remainingDailyLimit İSE
            YAZ "İşlem başarısız: Günlük limit aşılıyor. Kalan günlük limit: " + remainingDailyLimit + " TL"
            DEVAM ET
        SON

        // Tüm kontroller geçildiyse parayı ver ve güncelle
        atmCashAvailable ← atmCashAvailable - withdrawAmount
        accountBalance ← accountBalance - withdrawAmount
        dailyWithdrawn ← dailyWithdrawn + withdrawAmount

        YAZ withdrawAmount + " TL veriliyor. Lütfen paranızı ve fişi alınız."
        YAZ "Yeni bakiye: " + accountBalance + " TL"
        YAZ "Bugün çekilen toplam: " + dailyWithdrawn + " TL"

        // İşlem başarılı, istenirse tekrar seçenek
        YAZ "Başka işlem yapmak ister misiniz? (E/H)"
        OKU another
        EĞER another = "E" VEYA another = "e" İSE
            DEVAM ET    // döngüye devam et
        DEĞİLSE
            YAZ "Kart iade ediliyor. Oturum sonlandırılıyor."
            continueSession ← YANLIŞ
            DUR
        SON

    DEĞİLSE EĞER choice = 2 İSE
        // Bakiye görüntüleme
        YAZ "Mevcut bakiye: " + accountBalance + " TL"
        YAZ "Bugün çekilen toplam: " + dailyWithdrawn + " TL"
        YAZ "Ana menüye dönmek için herhangi bir tuşa basın."
        OKU dummy
        DEVAM ET

    DEĞİLSE EĞER choice = 3 İSE
        // Kart iade / çıkış
        YAZ "Kart iade ediliyor. Oturum sonlandırılıyor."
        continueSession ← YANLIŞ
        DUR

    DEĞİLSE
        YAZ "Geçersiz seçenek. Lütfen tekrar deneyin."
        DEVAM ET
    SON

SON  // işlem döngüsü sonu

YAZ "Güle güle."
DUR

