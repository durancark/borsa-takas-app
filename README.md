# Gerekli kütüphaneleri yükleyin: pip install pyTelegramBotAPI yfinance requests

import telebot
import yfinance as yf
import requests
from datetime import datetime

# BOT TOKEN'ınızı buraya yazın (BotFather'dan aldığınız)
BOT_TOKEN = "BURAYA_BOT_TOKEN_YAZIN"

bot = telebot.TeleBot(BOT_TOKEN)

# /start komutu
@bot.message_handler(commands=['start'])
def start_mesaji(message):
    hosgeldin = """
🔥 BORSA BOT'A HOŞGELDİNİZ! 🔥

📊 Kullanılabilir Komutlar:
/derinlik HISSE - Hisse derinlik bilgisi
/fiyat HISSE - Güncel fiyat bilgisi
/endeks - BIST100 durumu
/takas - Detaylı takas analizi (Web App)
/haber - Son borsa haberleri
/help - Yardım

Örnek kullanım: /fiyat AKBNK
    """
    
    # Web App butonu ekle
    markup = telebot.types.InlineKeyboardMarkup()
    web_app = telebot.types.WebAppInfo("https://durancark.github.io/borsa-takas-app/")
    markup.add(telebot.types.InlineKeyboardButton("📊 Takas Analizini Aç", web_app=web_app))
    
    bot.reply_to(message, hosgeldin, reply_markup=markup)

# /help komutu
@bot.message_handler(commands=['help'])
def yardim(message):
    yardim_metni = """
📚 NASIL KULLANILIR:

/fiyat AKBNK - Akbank fiyatı
/fiyat THYAO - THY fiyatı
/derinlik GARAN - Garanti derinlik
/endeks - BIST100 bilgisi

🏢 Popüler Hisseler:
AKBNK, GARAN, ISCTR, THYAO, ASELS, KCHOL
    """
    bot.reply_to(message, yardim_metni)

# /fiyat komutu
@bot.message_handler(commands=['fiyat'])
def fiyat_sorgula(message):
    try:
        # Kullanıcının yazdığı hisse kodunu al
        komut_parcalari = message.text.split()
        if len(komut_parcalari) < 2:
            bot.reply_to(message, "❌ Hisse kodu yazmalısınız!\nÖrnek: /fiyat AKBNK")
            return
        
        hisse_kodu = komut_parcalari[1].upper()
        ticker = f"{hisse_kodu}.IS"  # Türk hisseleri için .IS eki
        
        # Yahoo Finance'den veri çek
        hisse = yf.Ticker(ticker)
        info = hisse.info
        hist = hisse.history(period="1d")
        
        if hist.empty:
            bot.reply_to(message, f"❌ {hisse_kodu} bulunamadı!")
            return
        
        # Güncel fiyat
        guncel_fiyat = hist['Close'][-1]
        acilis_fiyat = hist['Open'][-1]
        yuksek = hist['High'][-1]
        dusuk = hist['Low'][-1]
        
        # Değişim hesapla
        degisim = guncel_fiyat - acilis_fiyat
        degisim_yuzde = (degisim / acilis_fiyat) * 100
        
        # Sonucu formatla
        sonuc = f"""
📊 {hisse_kodu} FİYAT BİLGİSİ

💰 Güncel: {guncel_fiyat:.2f} TL
📈 Açılış: {acilis_fiyat:.2f} TL
⬆️ Yüksek: {yuksek:.2f} TL  
⬇️ Düşük: {dusuk:.2f} TL
📦 Hacim: {hacim:,.0f}

{"🟢" if degisim >= 0 else "🔴"} Değişim: {degisim:.2f} TL ({degisim_yuzde:.2f}%)

⏰ Son Güncelleme: {datetime.now().strftime('%d.%m.%Y %H:%M')}

💡 Diğer hisseler: /fiyat GARAN, /fiyat THYAO
        """
        
        bot.reply_to(message, sonuc)
        
    except Exception as e:
        bot.reply_to(message, f"❌ Hata oluştu: {str(e)}")

# /derinlik komutu (basit versiyon)
@bot.message_handler(commands=['derinlik'])
def derinlik_sorgula(message):
    try:
        komut_parcalari = message.text.split()
        if len(komut_parcalari) < 2:
            bot.reply_to(message, "❌ Hisse kodu yazmalısınız!\nÖrnek: /derinlik AKBNK")
            return
        
        hisse_kodu = komut_parcalari[1].upper()
        ticker = f"{hisse_kodu}.IS"
        
        hisse = yf.Ticker(ticker)
        hist = hisse.history(period="5d")
        
        if hist.empty:
            bot.reply_to(message, f"❌ {hisse_kodu} bulunamadı!")
            return
        
        # Basit derinlik bilgisi (son 5 günün ortalaması)
        ortalama = hist['Close'].mean()
        son_fiyat = hist['Close'][-1]
        
        sonuc = f"""
🎯 {hisse_kodu} DERİNLİK BİLGİSİ

📊 Son Fiyat: {son_fiyat:.2f} TL
📊 5 Gün Ortalaması: {ortalama:.2f} TL
📊 Destek: {hist['Low'].min():.2f} TL
📊 Direnç: {hist['High'].max():.2f} TL

ℹ️ Bu basit bir analiz örneğidir.
Gerçek derinlik için profesyonel veri kaynağı gerekir.
        """
        
        bot.reply_to(message, sonuc)
        
    except Exception as e:
        bot.reply_to(message, f"❌ Hata oluştu: {str(e)}")

# /endeks komutu
@bot.message_handler(commands=['endeks'])
def endeks_bilgi(message):
    try:
        # BIST100 verisi
        bist = yf.Ticker("XU100.IS")
        hist = bist.history(period="1d")
        
        if hist.empty:
            bot.reply_to(message, "❌ BIST100 verisi alınamadı!")
            return
        
        guncel = hist['Close'][-1]
        acilis = hist['Open'][-1]
        degisim = guncel - acilis
        degisim_yuzde = (degisim / acilis) * 100
        
        sonuc = f"""
🇹🇷 BIST100 ENDEKSİ

📊 Güncel: {guncel:.2f}
📈 Açılış: {acilis:.2f}
{"🟢" if degisim >= 0 else "🔴"} Değişim: {degisim:.2f} ({degisim_yuzde:.2f}%)

⏰ {datetime.now().strftime('%H:%M:%S')}
        """
        
        bot.reply_to(message, sonuc)
        
    except Exception as e:
        bot.reply_to(message, f"❌ Hata oluştu: {str(e)}")

# Diğer mesajlar için
@bot.message_handler(func=lambda message: True)
def diger_mesajlar(message):
    bot.reply_to(message, "❓ Anlamadım. /help yazarak komutları görebilirsiniz.")

print("🤖 Bot başlatılıyor...")
print("✅ Bot çalışıyor! Durdurmak için Ctrl+C")

# Bot'u çalıştır
bot.polling(none_stop=True)
