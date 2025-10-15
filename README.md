# Aselmuzicea
API_ID = 20289977
API_HASH="f036f7de8eee7df5bad4b99ac0892708"
BOT_TOKEN = "8313669907:AAGyBDT97AKoiFipKftfcxHPSSjHeSp3FbU"
SESSION_STRING = "aselmuzic"
import asyncio
from pyrogram import Client, filters
from pytgcalls import PyTgCalls
from pytgcalls.types import InputStream, AudioPiped
import yt_dlp
from config import *

app = Client("music_bot", api_id=API_ID, api_hash=API_HASH, bot_token=BOT_TOKEN)
pytgcalls = PyTgCalls(app)

queue = {}

def download_audio(url):
    ydl_opts = {
        "format": "bestaudio/best",
        "outtmpl": "downloads/%(title)s.%(ext)s",
        "quiet": True,
    }
    with yt_dlp.YoutubeDL(ydl_opts) as ydl:
        info = ydl.extract_info(url, download=True)
        return ydl.prepare_filename(info), info["title"]

@app.on_message(filters.command("play"))
async def play(_, message):
    if len(message.command) < 2:
        return await message.reply("🎵 Kullanım: `/play <YouTube link veya isim>`")

    url = message.text.split(" ", 1)[1]
    await message.reply("🔎 Müzik indiriliyor...")

    file, title = download_audio(url)

    if message.chat.id not in queue:
        queue[message.chat.id] = []

    queue[message.chat.id].append(file)

    if len(queue[message.chat.id]) == 1:
        await pytgcalls.join_group_call(
            message.chat.id,
            InputStream(AudioPiped(file))
        )
        await message.reply(f"🎶 Şimdi çalıyor: **{title}**")
    else:
        await message.reply(f"🎵 Kuyruğa eklendi: **{title}**")

@app.on_message(filters.command("skip"))
async def skip(_, message):
    if message.chat.id in queue and len(queue[message.chat.id]) > 1:
        queue[message.chat.id].pop(0)
        file = queue[message.chat.id][0]
        await pytgcalls.change_stream(message.chat.id, InputStream(AudioPiped(file)))
        await message.reply("⏭ Sonraki şarkıya geçildi.")
    else:
        await pytgcalls.leave_group_call(message.chat.id)
        queue.pop(message.chat.id, None)
        await message.reply("🎵 Kuyruk bitti, çıkış yapıldı.")

@app.on_message(filters.command("pause"))
async def pause(_, message):
    await pytgcalls.pause_stream(message.chat.id)
    await message.reply("⏸ Müzik duraklatıldı.")

@app.on_message(filters.command("resume"))
async def resume(_, message):
    await pytgcalls.resume_stream(message.chat.id)
    await message.reply("▶️ Müzik devam ediyor.")

@app.on_message(filters.command("stop"))
async def stop(_, message):
    await pytgcalls.leave_group_call(message.chat.id)
    queue.pop(message.chat.id, None)
    await message.reply("🛑 Müzik durduruldu ve çıkış yapıldı.")

@app.on_message(filters.command("queue"))
async def show_queue(_, message):
    if message.chat.id not in queue or not queue[message.chat.id]:
        return await message.reply("📭 Kuyruk boş.")
    text = "**🎶 Şarkı Kuyruğu:**\\n"
    for i, song in enumerate(queue[message.chat.id], start=1):
        text += f"{i}. {song.split('/')[-1]}\\n"
    await message.reply(text)

async def main():
    await app.start()
    await pytgcalls.start()
    print("🎵 Bot çalışıyor...")
    await asyncio.get_event_loop().create_future()

asyncio.run(main())
pyrogram
tgcrypto
pytgcalls
yt-dlp
ffmpeg-python
pyrogram
tgcrypto
pytgcalls
yt-dlp
ffmpeg-python
