function ogrenciGiris(ogrNo, sifre):
    if not auth(ogrNo, sifre):
        return "Giriş başarısız"
    session.user = loadStudent(ogrNo)
    return "Giriş başarılı"

function dersEkle(session, dersId):
    ders = getDers(dersId)
    if ders == null:
        return fail("Ders bulunamadı")

    errors = []

    # 1 Kontenjan
    if ders.doluluk >= ders.kontenjan:
        errors.append("Kontenjan dolu")

    # 2 Önkoşullar
    foreach ok in ders.onkosullar:
        if not session.user.gecmisDersler.contains(ok.kod) or (ok.minNot and getNot(ok.kod) < ok.minNot):
            errors.append("Önkoşul eksik: " + ok.kod)

    # 3 Zaman çakışması
    foreach secili in session.user.secilenDersler:
        if zamanCakisma(ders.saatleri, secili.saatleri):
            errors.append("Zaman çakışması: " + secili.kod)

    # 4 Kredi limiti
    yeniToplam = session.user.toplamKredi + ders.kredi
    if yeniToplam > 35:
        errors.append("Kredi limiti aşılıyor (şu an " + session.user.toplamKredi + ")")

    # 5 Danışman onayı gereği
    if session.user.gpa < 2.5:
        danışmanGerekli = true
    elif yeniToplam > 30:  # örnek ek koşul
        danışmanGerekli = true
    else:
        danışmanGerekli = false

    # Eğer hata yoksa ekle/pending olarak işaretle
    if errors.isEmpty():
        beginTransaction()
        try:
            if ders.doluluk + 1 > ders.kontenjan:
                rollback()
                return fail("Kontenjan yarışmasında kaybettiniz")
            ders.doluluk += 1
            session.user.secilenDersler.add(ders)
            session.user.toplamKredi = yeniToplam
            if danışmanGerekli:
                ders.status = "PENDING_ADVISOR"
                notifyAdvisor(session.user.advisorId, ders)
            commit()
            return success("Ders eklendi" + (danışmanGerekli ? " (Danışman onayı bekleniyor)" : ""))
        except Exception e:
            rollback()
            return fail("Sistem hatası: " + e.message)
    else:
        return failList(errors)

function dersCikar(session, dersId):
    ders = session.user.secilenDersler.find(dersId)
    if ders == null:
        return fail("Ders seçili değil")
    beginTransaction()
    try:
        session.user.secilenDersler.remove(ders)
        ders.doluluk -= 1
        session.user.toplamKredi -= ders.kredi
        commit()
        return success("Ders çıkarıldı")
    except:
        rollback()
        return fail("Sistem hatası")

function kayitOnayla(session):
    # Tüm seçili dersleri tekrar toplu kontrol et (aynı logic)
    aggregateErrors = []
    foreach ders in session.user.secilenDersler:
        result = runAllChecks(session.user, ders)
        if result.errors not empty:
            aggregateErrors.addAll(result.errors)
    if aggregateErrors not empty:
        return failList(aggregateErrors)
    # Eğer danışman onayı gerektiren pending varsa süreci başlat
    if any pending:
        submitForAdvisorApproval()
        return success("Kaydınız danışman onayına gönderildi")
    finalizeRegistration()
    return success("Kayıt tamamlandı")
