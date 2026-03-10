# 🕌 Alexa Adhan Automation for Home Assistant

A reliable and smart automation to play the **Adhan** (or any audio file) on Amazon Alexa devices using Home Assistant. This project bypasses common Alexa limitations (like SSL requirements and audio format restrictions) and includes a feature to restore the volume level after the Adhan finishes.

---

## 🌍 Features | المميزات
* 🕋 **Prayer Times Sync:** Integrated with `Islamic Prayer Times`.
* 🔊 **Volume Management:** Automatically saves current volume and restores it after Adhan.
* 🛠️ **SSML Integration:** Bypasses Alexa's strict audio playback rules.
* 🌐 **Remote Access:** Works seamlessly with DuckDNS/HTTPS.

---

## 🛠️ Requirements | المتطلبات

1. **Islamic Prayer Times Integration:** To get accurate prayer timings.
2. **Alexa Media Player (HACS):** To control your Echo devices.
3. **DuckDNS (or Nabu Casa):** To provide a secure **HTTPS** link (Required by Alexa).
4. **Converted Audio File:** Alexa is very picky about audio formats.

---

## ⚠️ Audio Specifications | مواصفات الملف الصوتي
To play the file via the `TTS` notify service, your MP3 file **MUST** follow these specs exactly:
- **Codec:** MP3
- **Bitrate:** 48 kbps
- **Sample Rate:** 16000 Hz 👈 (Very Important)
- **Channels:** Mono
- **Duration:** Preferably under 90 seconds.

---

💡 Quick Tips | نصائح سريعة
File Location: Place your audio in the /config/www/ folder. Access it in the code using /local/.

HTTPS: Make sure your DuckDNS URL is accessible from outside your network.

Troubleshooting: If you hear a "chime" but no audio, re-check your file's 16000Hz sample rate.

---

🤝 Contributing
Feel free to fork this project and submit pull requests if you have any improvements!


## 📋 Automation Code | كود الأتمتة

```yaml
alias: Prayer Time Adhan Alexa
description: Play Adhan on all Alexa devices and restore state afterwards
triggers:
  - trigger: time
    at:
      - sensor.islamic_prayer_times_fajr_prayer
      - sensor.islamic_prayer_times_dhuhr_prayer
      - sensor.islamic_prayer_times_asr_prayer
      - sensor.islamic_prayer_times_maghrib_prayer
      - sensor.islamic_prayer_times_isha_prayer
actions:
  # 1. Save current state (Snapshot)
  - action: scene.create
    data:
      scene_id: before_adhan_state
      snapshot_entities:
        - media_player.your_alexa_device # Change to your device name

  # 2. Set Adhan volume
  - action: media_player.volume_set
    target:
      entity_id:
        - media_player.your_alexa_device
    data:
      volume_level: 0.4

  # 3. Play Adhan via SSML
  - action: notify.alexa_media
    data:
      target:
        - media_player.your_alexa_device
      data:
        type: tts
      message: |
        <speak>
          <audio src='https://YOUR_DUCK_DNS_URL/local/YOUR_ADHAN_FILE.mp3'/>
        </speak>

  # 4. Wait for audio to finish (adjust to your file length)
  - delay: "00:01:40"

  # 5. Restore previous state (Volume)
  - action: scene.turn_on
    target:
      entity_id: scene.before_adhan_state

mode: restart
