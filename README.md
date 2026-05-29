# Yapay-zeka-i-in-yard-m-Jarvis-
Ev yapımı yapay zeka için yardım 
kodun gelistirilmesi gerekiyor bu yuzden yardim istiyoruz. kod:import os
import sys
import time
import requests
import json

current_lang = "tr"
HAFIZA_DOSYASI = "hafiza.json"
konusma_gecmisi = []


def hafiza_yukle():
    global konusma_gecmisi

    konusma_gecmisi = [
        {
            "role": "system",
            "content": "Senin adın Jarvis. Kullanıcının asistanısın."
        }
    ]

    if os.path.exists(HAFIZA_DOSYASI):
        try:
            with open(HAFIZA_DOSYASI, "r", encoding="utf-8") as f:
                veri = json.load(f)

            konusmalar = [
                m for m in veri
                if m["role"] != "system"
            ]

            konusma_gecmisi.extend(konusmalar[-10:])

        except:
            pass


def hafiza_kaydet():
    try:
        with open(HAFIZA_DOSYASI, "w", encoding="utf-8") as f:
            json.dump(
                konusma_gecmisi,
                f,
                ensure_ascii=False,
                indent=4
            )
    except:
        pass


def ekrana_yaz_ve_konus(metin):
    global current_lang

    toast_komut = (
        f'termux-toast -b black -c white -g top "{metin}"'
    )

    os.system(
        toast_komut.encode(
            "utf-8",
            "ignore"
        ).decode("utf-8")
    )

    print("\n[Jarvis]: ", end="", flush=True)

    for harf in metin:
        print(harf, end="", flush=True)
        time.sleep(0.01)

    print()

    konusma_gecmisi.append({
        "role": "assistant",
        "content": metin
    })

    hafiza_kaydet()

    lang_code = (
        "tr-TR"
        if current_lang == "tr"
        else "en-US"
    )

    komut = (
        f'termux-tts-speak -l {lang_code} "{metin}"'
    )

    os.system(
        komut.encode(
            "utf-8",
            "ignore"
        ).decode("utf-8")
    )


def yapay_zeka_beyni(soru):
    global current_lang
    global konusma_gecmisi

    soru_temiz = soru.lower().strip()

    if soru_temiz in [
        "dil değiştir",
        "change language"
    ]:

        if current_lang == "tr":
            current_lang = "en"
            return "Language switched to English."

        else:
            current_lang = "tr"
            return "Dil Türkçe olarak değiştirildi."

    if soru_temiz in [
        "çık",
        "exit",
        "quit"
    ]:
        sys.exit()

    print("[Jarvis düşünüyor...]")

    konusma_gecmisi.append({
        "role": "user",
        "content": soru
    })

    url = "https://router.huggingface.co/v1/chat/completions"

    HF_TOKEN = "TOKENINI_BURAYA_YAZ"

    payload = {
        "model": "meta-llama/Llama-3.1-8B-Instruct",
        "messages": konusma_gecmisi,
        "max_tokens": 200,
        "temperature": 0.8
    }

    headers = {
        "Authorization": f"Bearer {HF_TOKEN}",
        "Content-Type": "application/json"
    }

    try:
        response = requests.post(
            url,
            json=payload,
            headers=headers,
            timeout=20
        )

        if response.status_code == 200:
            data = response.json()

            cevap = data["choices"][0]["message"]["content"]

            return cevap

        else:
            return f"Hata kodu: {response.status_code}"

    except Exception as e:
        return f"Hata oluştu: {str(e)}"


if __name__ == "__main__":

    os.system("clear")

    print("========================")
    print(" JARVIS AI SİSTEMİ ")
    print("========================")

    hafiza_yukle()

    print("\n[Jarvis]: Sistem çevrimiçi.")

    while True:

        try:
            komut = input("\nSorunuzu Yazın: ")

            if komut.strip():

                cevap = yapay_zeka_beyni(komut)

                ekrana_yaz_ve_konus(cevap)

        except KeyboardInterrupt:

            print("\nProgram kapatıldı.")

            sys.exit()
        
