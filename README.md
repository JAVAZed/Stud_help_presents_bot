import logging
import os
from datetime import datetime

try:
    import ssl
    SSL_AVAILABLE = True
except ImportError:
    SSL_AVAILABLE = False

from aiogram import Bot, Dispatcher, executor, types
from aiogram.types import ReplyKeyboardMarkup, KeyboardButton

API_TOKEN = os.getenv("BOT_TOKEN")
ADMIN_ID = int(os.getenv("ADMIN_ID", 8190819662))

logging.basicConfig(level=logging.INFO)

bot = Bot(token=API_TOKEN)
dp = Dispatcher(bot)

# Keyboards
lang_kb = ReplyKeyboardMarkup(resize_keyboard=True)
lang_kb.add(KeyboardButton('Oʻzbek'), KeyboardButton('Русский'), KeyboardButton('English'))

tovar_kb_uz = ReplyKeyboardMarkup(resize_keyboard=True)
tovar_kb_uz.add("Kurs ishlari", "Insho/ilmiy maqola", "Taqdimot")

tovar_kb_ru = ReplyKeyboardMarkup(resize_keyboard=True)
tovar_kb_ru.add("Курсовые работы", "Эссе/научные статьи", "Презентации")

tovar_kb_en = ReplyKeyboardMarkup(resize_keyboard=True)
tovar_kb_en.add("Course papers", "Essays/Scientific articles", "Presentations")

confirm_kb = ReplyKeyboardMarkup(resize_keyboard=True).add(KeyboardButton("Оплатил"))

presentation_kb = ReplyKeyboardMarkup(resize_keyboard=True)
for i in range(10, 50, 5):
    presentation_kb.add(str(i))

@dp.message_handler(commands=['start'])
async def send_welcome(message: types.Message):
    await message.answer("Salom! Здравствуйте! Hello!\nIltimos, tilni tanlang. Пожалуйста, выберите язык. Please choose a language:", reply_markup=lang_kb)

@dp.message_handler(lambda message: message.text in ['Oʻzbek', 'Русский', 'English'])
async def choose_language(message: types.Message):
    if message.text == 'Oʻzbek':
        await message.answer("Quyidagilardan birini tanlang:", reply_markup=tovar_kb_uz)
    elif message.text == 'Русский':
        await message.answer("Пожалуйста, выберите товар:", reply_markup=tovar_kb_ru)
    elif message.text == 'English':
        await message.answer("Please select an item:", reply_markup=tovar_kb_en)

@dp.message_handler(lambda message: message.text in ["Taqdimot", "Презентации", "Presentations"])
async def choose_presentation_size(message: types.Message):
    await message.answer("Nechta sahifali taqdimot kerak? / Сколько слайдов? / How many slides do you need?", reply_markup=presentation_kb)

@dp.message_handler(lambda message: message.text in [str(i) for i in range(10, 50, 5)])
async def handle_presentation_selection(message: types.Message):
    pages = int(message.text)
    amount = pages * 1000
    await message.answer(f"{pages} sahifali taqdimot narxi: {amount} soʻm\nToʻlovni quyidagi karta raqamiga yuboring:")
    await message.answer("Karta: 9860 1601 2762 9600\nIsm: Javoxir Baxshilloyev")
    await message.answer_photo(
        photo='https://i.ibb.co/LhRgRLXf/qr.png',
        caption="Skanerlash orqali tezkor toʻlov (Click QR)"
    )
    await message.answer("Toʻlovingizni amalga oshirgach, 'Оплатил' tugmasini bosing:", reply_markup=confirm_kb)

    admin_message = (
        f"[НОВАЯ ЗАЯВКА: ПРЕЗЕНТАЦИЯ]\n"
        f"Имя: {message.from_user.full_name}\n"
        f"Username: @{message.from_user.username or 'нет'}\n"
        f"ID: {message.from_user.id}\n"
        f"Презентация: {pages} стр. ({amount} сум)\n"
        f"Время: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}"
    )
    await bot.send_message(chat_id=ADMIN_ID, text=admin_message)

@dp.message_handler(lambda message: message.text in [
    "Kurs ishlari", "Insho/ilmiy maqola",
    "Курсовые работы", "Эссе/научные статьи",
    "Course papers", "Essays/Scientific articles"
])
async def show_payment_info(message: types.Message):
    product_prices = {
        "Kurs ishlari": "100 000 soʻm",
        "Курсовые работы": "100 000 сум",
        "Course papers": "100,000 UZS",

        "Insho/ilmiy maqola": "50 000 soʻm",
        "Эссе/научные статьи": "50 000 сум",
        "Essays/Scientific articles": "50,000 UZS",
    }
    price = product_prices.get(message.text, "Narxi belgilanmagan")

    await message.answer(f"Ushbu xizmat narxi: {price}\nToʻlovni quyidagi karta raqamiga yuboring:")
    await message.answer("Karta: 9860 1601 2762 9600\nIsm: Javoxir Baxshilloyev")
    await message.answer_photo(
        photo='https://i.ibb.co/LhRgRLXf/qr.png',
        caption="Skanerlash orqali tezkor toʻlov (Click QR)"
    )
    await message.answer("Toʻlovingizni amalga oshirgach, 'Оплатил' tugmasini bosing:", reply_markup=confirm_kb)

    admin_message = (
        f"[НОВАЯ ЗАЯВКА]\n"
        f"Имя: {message.from_user.full_name}\n"
        f"Username: @{message.from_user.username or 'нет'}\n"
        f"ID: {message.from_user.id}\n"
        f"Выбрал: {message.text} ({price})\n"
        f"Время: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}"
    )
    await bot.send_message(chat_id=ADMIN_ID, text=admin_message)

@dp.message_handler(lambda message: message.text == "Оплатил")
async def confirm_payment(message: types.Message):
    await message.answer("Toʻlovingiz qabul qilindi! Yaqinda siz bilan bogʻланamiz.\nОплата получена! Мы скоро свяжемся с вами.\nPayment received! We will contact you soon.")
    admin_message = (
        f"[ОПЛАТА ПОДТВЕРЖДЕНА]\n"
        f"Имя: {message.from_user.full_name}\n"
        f"Username: @{message.from_user.username or 'нет'}\n"
        f"ID: {message.from_user.id}\n"
        f"Время: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}"
    )
    await bot.send_message(chat_id=ADMIN_ID, text=admin_message)

if __name__ == '__main__':
    executor.start_polling(dp, skip_updates=True)
    
SyntaxError: multiple statements found while compiling a single statement

