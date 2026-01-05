import asyncio
import random
from telegram import Update
from telegram.ext import Application, CommandHandler, ContextTypes

TOKEN = "DÁN_TOKEN_MỚI_CỦA_BẠN_VÀO_ĐÂY"

BOT_NAME = "BOT7977"
SIGN = "🤖 Bot của id tiktok: ductai1227"

running = False
round_id = 1
win = 0
lose = 0

def roll_dice():
    return random.randint(1,6), random.randint(1,6), random.randint(1,6)

def tai_xiu(total):
    return "XỈU" if total <= 10 else "TÀI"

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    global running
    if running:
        await update.message.reply_text("⚠️ Bot đang chạy rồi!")
        return

    running = True
    await update.message.reply_text(f"✅ {BOT_NAME} ĐANG DỰ ĐOÁN")
    asyncio.create_task(run_bot(update, context))

async def stop(update: Update, context: ContextTypes.DEFAULT_TYPE):
    global running
    running = False
    await update.message.reply_text("⛔ Bot đã dừng")

async def run_bot(update: Update, context: ContextTypes.DEFAULT_TYPE):
    global running, round_id, win, lose

    chat_id = update.message.chat_id

    while running:
        # Dự đoán
        predict = random.choice(["TÀI", "XỈU"])

        msg = await context.bot.send_message(
            chat_id=chat_id,
            text=(
                f"🎯 {BOT_NAME} ĐANG DỰ ĐOÁN\n"
                f"📌 Phiên số: {round_id}\n"
                f"📊 Dự đoán: {predict}\n\n"
                f"{SIGN}"
            )
        )

        # Đếm ngược 10s
        for t in range(10, 0, -1):
            await context.bot.edit_message_text(
                chat_id=chat_id,
                message_id=msg.message_id,
                text=(
                    f"🎯 {BOT_NAME} ĐANG DỰ ĐOÁN\n"
                    f"📌 Phiên số: {round_id}\n"
                    f"📊 Dự đoán: {predict}\n"
                    f"⏳ Còn {t}s\n\n"
                    f"{SIGN}"
                )
            )
            await asyncio.sleep(1)

        # Kết quả
        d1, d2, d3 = roll_dice()
        total = d1 + d2 + d3
        result = tai_xiu(total)

        if result == predict:
            win += 1
            status = "ĐÚNG 🟢"
        else:
            lose += 1
            status = "SAI 🔴"

        await context.bot.edit_message_text(
            chat_id=chat_id,
            message_id=msg.message_id,
            text=(
                f"🎯 KẾT QUẢ PHIÊN {round_id}\n"
                f"🎲 Xúc xắc: {d1} 🎲 {d2} 🎲 {d3}\n"
                f"📊 Tổng: {total} → {result}\n\n"
                f"🤖 Bot dự đoán: {status}\n"
                f"✅ Đúng: {win} | ❌ Sai: {lose}\n\n"
                f"{SIGN}"
            )
        )

        round_id += 1
        await asyncio.sleep(40)  # đủ 50s / phiên

async def main():
    app = Application.builder().token(TOKEN).build()
    app.add_handler(CommandHandler("start", start))
    app.add_handler(CommandHandler("stop", stop))
    print("Bot đang chạy...")
    await app.run_polling()

if __name__ == "__main__":
    asyncio.run(main())
