import telebot
import os
import json
import time
from dotenv import load_dotenv

load_dotenv()

TOKEN = "8601340130:AAHjXHycNv66Hc0xvzNc06HJNRr3l56ytC8"

if not TOKEN:
    raise ValueError("TOKEN не задан!")

bot = telebot.TeleBot(TOKEN)

# 👑 ГЛАВНЫЙ АДМИН (OWNER)
OWNER_ID = 7925843350

# ---------- загрузка данных ----------
try:
    with open("data.json", "r") as f:
        data = json.load(f)
except:
    data = {
        "credits": {},
        "requests": {},
        "admins": [OWNER_ID]
    }

def save():
    with open("data.json", "w") as f:
        json.dump(data, f, indent=2)

ADMINS = data.get("admins", [])

# ---------- проверки ----------
def is_admin(user_id):
    return user_id in ADMINS

def is_owner(user_id):
    return user_id == OWNER_ID

# ---------- кредит заявка ----------
@bot.message_handler(commands=['credit'])
def credit(message):
    user = str(message.from_user.id)
    args = message.text.split()

    if len(args) < 3:
        bot.reply_to(message, "Пример: /credit 1000000 7")
        return

    amount = int(args[1])
    days = int(args[2])

    data["requests"][user] = {"amount": amount, "days": days}
    save()

    bot.reply_to(message, "📄 Заявка отправлена")

    for admin in ADMINS:
        bot.send_message(admin,
            f"📩 Заявка\nID: {user}\nСумма: {amount}\nСрок: {days}\n\n/approve {user}\n/deny {user}"
        )

# ---------- approve ----------
@bot.message_handler(commands=['approve'])
def approve(message):
    if not is_admin(message.from_user.id):
        return

    parts = message.text.split()
    if len(parts) < 2:
        return

    user = parts[1]

    req = data["requests"].get(user)
    if not req:
        return

    percent = 0.10
    total = int(req["amount"] * (1 + percent))
    payment = total // req["days"]

    data["credits"][user] = {
        "total": total,
        "payment": payment,
        "last_pay": time.time()
    }

    del data["requests"][user]
    save()

    bot.send_message(user,
        f"🏦 Кредит одобрен\nДолг: {total}\nПлатёж: {payment}/день"
    )

# ---------- deny ----------
@bot.message_handler(commands=['deny'])
def deny(message):
    if not is_admin(message.from_user.id):
        return

    parts = message.text.split()
    if len(parts) < 2:
        return

    user = parts[1]

    if user in data["requests"]:
        del data["requests"][user]
        save()

    bot.send_message(user, "❌ Отказ")

# ---------- мой кредит ----------
@bot.message_handler(commands=['mycredit'])
def mycredit(message):
    user = str(message.from_user.id)

    c = data["credits"].get(user)
    if not c:
        bot.reply_to(message, "Нет кредита")
        return

    bot.reply_to(message,
        f"📊 Долг: {c['total']}\nПлатёж: {c['payment']}"
    )

# ---------- оплата ----------
@bot.message_handler(commands=['pay'])
def pay(message):
    user = str(message.from_user.id)
    args = message.text.split()

    if user not in data["credits"]:
        bot.reply_to(message, "Нет кредита")
        return

    if len(args) < 2:
        bot.reply_to(message, "Пример: /pay 1000")
        return

    amount = int(args[1])

    data["credits"][user]["total"] -= amount
    save()

    bot.reply_to(message,
        f"Осталось: {data['credits'][user]['total']}"
    )

# ---------- добавить админа (ТОЛЬКО OWNER) ----------
@bot.message_handler(commands=['addadmin'])
def add_admin(message):
    if not is_owner(message.from_user.id):
        bot.reply_to(message, "❌ Только владелец может добавлять админов")
        return

    parts = message.text.split()
    if len(parts) < 2:
        bot.reply_to(message, "Пример: /addadmin 123456789")
        return

    new_admin = int(parts[1])

    if new_admin not in ADMINS:
        ADMINS.append(new_admin)
        data["admins"] = ADMINS
        save()

    bot.reply_to(message, f"✅ Админ добавлен: {new_admin}")

# ---------- удалить админа (ТОЛЬКО OWNER) ----------
@bot.message_handler(commands=['deladmin'])
def del_admin(message):
    if not is_owner(message.from_user.id):
        bot.reply_to(message, "❌ Только владелец может удалять админов")
        return

    parts = message.text.split()
    if len(parts) < 2:
        return

    admin_id = int(parts[1])

    if admin_id == OWNER_ID:
        bot.reply_to(message, "❌ Нельзя удалить владельца")
        return

    if admin_id in ADMINS:
        ADMINS.remove(admin_id)
        data["admins"] = ADMINS
        save()

    bot.reply_to(message, "❌ Админ удалён")

# ---------- автоответ ----------
@bot.message_handler(func=lambda m: True)
def auto(m):
    if not m.text:
        return

    text = m.text.lower()

    if "кредит" in text:
        bot.reply_to(m, "📄 /credit сумма дни")
    elif "банк" in text:
        bot.reply_to(m, "🏦 РП банк работает")
    else:
        bot.reply_to(m, "Напиши /credit")

bot.polling()
