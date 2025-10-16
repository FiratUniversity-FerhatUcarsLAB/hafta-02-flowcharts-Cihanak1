Başla
  // Sistem başlangıcı
  initialize_system()
  load_config()                 // sensör listesi, eşik değerleri, kullanıcı kontaktları
  set_mode(DISARM)              // başlangıç modu (ARM/DISARM)
  last_sensor_time = now()

Fonksiyon initialize_system():
  aç_sensörler()
  test_hoparlör_ve_led()
  bağlan_iletişim_servisleri() // SMS, push, e-posta
  başlat_log() 

Fonksiyon read_all_sensors():
  readings = {}
  her sensör s için sensör_okuma = read_sensor(s)
    eğer sensör_okuma başarısız ise
      log("sensor error", s)
      attempts = 0
      tekrarla while attempts < MAX_RETRY
        sensör_okuma = read_sensor(s)
        attempts += 1
        eğer başarılı ise break
      eğer hala başarısız ise mark_sensor_offline(s)
    readings[s] = sensör_okuma
  döndür readings

Fonksiyon detect_threat(readings, mode):
  threats = []
  eğer mode == DISARM ise
    döndür boş liste  // sistem etkisizse alarm üretme (isteğe bağlı: sadece bildirim)
  her (sensor, value) in readings:
    eğer sensor.type == "hareket" ve value > sensor.threshold then
      eğer recent_event_duplicate(sensor) == false then
        threats.append({type: "HAREKET", sensor: sensor, severity: compute_severity(value)})
    eğer sensor.type == "kapı_pencere" ve value == OPEN then
      threats.append({type: "GIRIS", sensor: sensor, severity: HIGH})
    eğer sensor.type == "duman" ve value >= sensor.threshold then
      threats.append({type: "YANGIN", sensor: sensor, severity: CRITICAL})
    ... // diğer sensör tipleri (gaz, su baskını, cam kırılma)
  önceliklendir threats by severity (CRITICAL > HIGH > MEDIUM > LOW)
  döndür threats

Fonksiyon handle_threats(threats):
  eğer threats boş ise döndür
  primary = threats[0]
  log("threat detected", primary)
  eğer primary.severity == CRITICAL then
    trigger_alarm(mode="siren+strobe")
    unlock_safety_paths_if_needed()
    send_notifications(all_contacts, primary, include_location=true, urgent=true)
    call_emergency_services_if_configured(primary)
  elif primary.severity == HIGH then
    trigger_alarm(mode="siren")
    send_notifications(primary_owner, primary, urgent=true)
    record_video_clip(primary.sensor)
  else
    // MEDIUM / LOW
    record_event(primary)
    send_notifications(primary_owner, primary, urgent=false)
  set_cooldown(primary.sensor, COOLDOWN_PERIOD)

Fonksiyon trigger_alarm(mode):
  if mode includes siren then activate_siren()
  if mode includes strobe then activate_strobe()
  start_alarm_timer(DEFAULT_ALARM_DURATION)
  log("alarm triggered", mode)

Fonksiyon send_notifications(targets, threat, include_location=false, urgent=false):
  message = format_message(threat, include_location)
  for each target in targets:
    if urgent then send_push_notification(target, message, high_priority=true)
    else send_push_notification(target, message)
    fallback: send_sms_or_email_if_no_ack(target, message)

Ana döngü (sonsuz):
  while true:
    mode = get_system_mode()                    // ARM / DISARM / HOME / AWAY
    readings = read_all_sensors()
    threats = detect_threat(readings, mode)
    handle_threats(threats)
    perform_periodic_tasks_if_due()             // sağlık kontrolleri, senkronizasyon, log upload
    sleep(SHORT_INTERVAL)                       // örn. 200-500 ms veya event-driven bekleme

Dur
