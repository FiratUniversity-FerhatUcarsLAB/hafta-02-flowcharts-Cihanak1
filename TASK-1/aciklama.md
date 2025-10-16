İsim - Soy isim Cihan Akalın 
Öğrenci No:250541107

DUR programı/oturumu sonlandırır.
DEVAM ET ile döngünün başına dönülür (bir sonraki işlem için).
BREAK ile iç döngüden çıkılır (örn. PIN doğrulandıktan sonra).
% operatörü modül almak için kullanıldı; (withdrawAmount % 20) <> 0 ifadesi 20'nin katı olmadığını kontrol eder.
Günlük limit dailyLimit - dailyWithdrawn ile hesaplanır; işlem başarılıysa dailyWithdrawn güncellenir.
PIN hakkı tükenirse kart bloke edilir ve oturum sonlandırılır.
atmCashAvailable değişkeni ATM'nin fiziksel nakit seviyesini temsil eder; gerçek ATM'lerde banknot kombinasyonu kontrolü de gerekir (ör. 20/50/100 TL vs). Burada sadece toplam miktar ve 20 TL katı kontrolü yapılmıştır.
Geliştirme için ek özellikler: işlem fişi basma, parola gizleme (***), zaman damgası, çoklu para birimi, farklı işlem türleri (para yatırma, para transferi), banknot kombinasyonu kontrolü.
