import discord
from discord.ext import commands

intents = discord.Intents.default()
intents.message_content = True

bot = commands.Bot(command_prefix='!', intents=intents)

user_data = {}

# (Soru, anahtar, yıllık çarpan, etki katsayısı)
questions = [
    ("Günde kaç plastik şişe kullanıyorsun?", "plastik", 365, 0.02),
    ("Haftada kaç poşet kullanıyorsun?", "poset", 52, 0.005),
    ("Haftada kaç gün et yersin?", "et", 52, 5),
    ("Haftada kaç kez araba kullanırsın (10+ km)?", "araba", 52, 4),
    ("Günde kaç saat elektrikli cihaz kullanırsın?", "elektrik", 365, 0.4),
    ("Günde kaç bardak kahve içersin?", "kahve", 365, 130),
    ("Haftada kaç kez çamaşır makinesi çalıştırırsın?", "camasir", 52, 70),
    ("Yılda kaç yeni kıyafet alırsın?", "kiyafet", 1, 7500),
    ("Haftada kaç öğün dışarıdan sipariş verirsin?", "siparis", 52, 0.1),
    ("Geri dönüşüm yapıyor musun? (evet/hayır)", "geri_donusum", 1, 1),
    ("Günde kaç kağıt havlu kullanırsın?", "havlu", 365, 0.01),
    ("Cihazlarını standby modda bırakıyor musun? (evet/hayır)", "standby", 1, 100)
]

@bot.event
async def on_ready():
    print(f"✅ Bot çalışıyor: {bot.user}")

@bot.command()
async def zararhesapla(ctx):
    await ctx.send(f"{ctx.author.mention}, sorular DM'den gelecek. Hazır mısın?")
    user_data[ctx.author.id] = {}
    await ask_question(ctx.author, 0)

async def ask_question(user, index):
    if index >= len(questions):
        await calculate_damage(user)
        return

    soru, key, *_ = questions[index]
    await user.send(f"{index+1}. {soru}")

    def check(m):
        return m.author == user and isinstance(m.channel, discord.DMChannel)

    msg = await bot.wait_for('message', check=check)
    user_data[user.id][key] = msg.content.strip().lower()
    await ask_question(user, index + 1)

async def calculate_damage(user):
    cevaplar = user_data.get(user.id, {})
    toplam_co2 = toplam_su = toplam_plastik = toplam_agac = 0

    for _, key, katsayi, carpim in questions:
        giris = cevaplar.get(key, "0")
        if giris in ["evet", "hayır"]:
            giris = "1" if giris == "evet" else "0"

        try:
            deger = float(giris)
        except:
            deger = 0

        etki = deger * katsayi * carpim

        if key in ["plastik", "poset", "siparis"]:
            toplam_plastik += etki
        elif key in ["et", "araba", "elektrik", "standby"]:
            toplam_co2 += etki
        elif key in ["kahve", "camasir", "kiyafet"]:
            toplam_su += etki
        elif key == "havlu":
            toplam_agac += etki

    await user.send(
        f"🌍 **Yıllık Zarar Raporun:**\n"
        f"🟤 Karbon Salımı: {toplam_co2:.1f} kg CO₂\n"
        f"💧 Su Tüketimi: {toplam_su:.0f} litre\n"
        f"🧴 Plastik Atık: {toplam_plastik:.1f} kg\n"
        f"🌳 Ağaç Kaybı: {toplam_agac:.2f} ağaç\n\n"
        f"`!dönüşümebaşla` yazarak öneriler alabilirsin."
    )

@bot.command()
async def dönüşümebaşla(ctx):
    await ctx.author.send(
        "🌱 Çevreyi korumak için:\n"
        "- Cam şişe kullan\n"
        "- Et tüketimini azalt\n"
        "- Elektronikleri tamamen kapat\n"
        "- Geri dönüşüm kutularını kullan\n"
        "- Giysi alımını azalt\n"
        "- Plastik ambalajdan kaçın"
    )

@bot.command()
async def zararsıfırla(ctx):
    user_data.pop(ctx.author.id, None)
    await ctx.send(f"{ctx.author.mention}, verilerin sıfırlandı. Yeni hesap için `!zararhesapla` yaz.")

bot.run("TOKEN")
