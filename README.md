import os
import yt_dlp
from shazamio import Shazam
from telegram import Update
from telegram.ext import ApplicationBuilder, MessageHandler, CommandHandler, filters, ContextTypes

TOKEN = os.getenv("8682219812:AAH_6S25VUs1CMtOJghzkBocCbNL6h9PbRA")

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text("🎬 Link yubor — muzikani topaman!")

async def detect(update: Update, context: ContextTypes.DEFAULT_TYPE):
    url = update.message.text
    await update.message.reply_text("⏳ Yuklanmoqda...")

    try:
        ydl_opts = {'format': 'best', 'outtmpl': 'video.%(ext)s'}
        with yt_dlp.YoutubeDL(ydl_opts) as ydl:
            ydl.download([url])

        video = [f for f in os.listdir() if f.startswith("video.")][0]
        os.system(f"ffmpeg -i {video} audio.mp3")

        await update.message.reply_text("🔍 Aniqlanmoqda...")

        shazam = Shazam()
        result = await shazam.recognize("audio.mp3")

        track = result.get("track", {})
        title = track.get("title", "Topilmadi")
        artist = track.get("subtitle", "")

        await update.message.reply_text(f"🎵 {title} - {artist}")

    except Exception as e:
        await update.message.reply_text(f"❌ {e}")

app = ApplicationBuilder().token(TOKEN).build()
app.add_handler(CommandHandler("start", start))
app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, detect))

app.run_polling()
