from aiogram import Bot, Dispatcher, types
from aiogram.types import InlineKeyboardMarkup, InlineKeyboardButton, ReplyKeyboardMarkup, KeyboardButton
from aiogram.contrib.fsm_storage.memory import MemoryStorage
from aiogram.dispatcher import FSMContext
from aiogram.dispatcher.filters.state import State, StatesGroup
from aiogram.utils import executor

API_TOKEN = '6905373703:AAEBIj-CPDjTHwWbsidl2zE6W0qA_eeTzX0'
ADMIN_ID = 868660293

bot = Bot(token=API_TOKEN)
dp = Dispatcher(bot, storage=MemoryStorage())

bookmakers = ['1xBet', 'Linebet', '1WIN', 'Mostbet']

class TopUp(StatesGroup):
    bookmaker = State()
    user_id = State()
    amount = State()
    proof = State()

class Withdraw(StatesGroup):
    bookmaker = State()
    user_id = State()
    secret_code = State()
    card_number = State()

main_menu = ReplyKeyboardMarkup(resize_keyboard=True)
main_menu.add(KeyboardButton("Hisobni toʻldirish"))
main_menu.add(KeyboardButton("Hisobdan chiqarish"))
main_menu.add(KeyboardButton("Admin"))

def bookmaker_keyboard():
    kb = InlineKeyboardMarkup(row_width=2)
    buttons = [InlineKeyboardButton(text=name, callback_data=name) for name in bookmakers]
    kb.add(*buttons)
    return kb

@dp.message_handler(commands=['start'])
async def start(msg: types.Message):
    await msg.answer("Xush kelibsiz! Kerakli bo‘limni tanlang:", reply_markup=main_menu)

@dp.message_handler(lambda m: m.text == "Admin")
async def admin_info(msg: types.Message):
    await msg.answer("Admin: @Sunnatillo2001")

@dp.message_handler(lambda m: m.text == "Hisobni toʻldirish")
async def start_topup(msg: types.Message):
    await msg.answer("Qaysi bukmeker?", reply_markup=bookmaker_keyboard())
    await TopUp.bookmaker.set()

@dp.callback_query_handler(state=TopUp.bookmaker)
async def get_bookmaker(call: types.CallbackQuery, state: FSMContext):
    await state.update_data(bookmaker=call.data)
    await call.message.answer("ID raqamingizni kiriting:")
    await TopUp.next()

@dp.message_handler(state=TopUp.user_id)
async def get_user_id(msg: types.Message, state: FSMContext):
    await state.update_data(user_id=msg.text)
    await msg.answer("Minimal: 10,000 so'm\nMaksimal: 3,000,000 so'm\nMiqdorni kiriting:")
    await TopUp.next()

@dp.message_handler(state=TopUp.amount)
async def get_amount(msg: types.Message, state: FSMContext):
    await state.update_data(amount=msg.text)
    await msg.answer("Toʻlovni ushbu karta raqamiga yuboring:\n\n8600 1204 8301 6252\nSiddiqova Oyjamol\n\nToʻlov chek rasmini yuboring.")
    await TopUp.next()

@dp.message_handler(content_types=types.ContentType.PHOTO, state=TopUp.proof)
async def get_proof(msg: types.Message, state: FSMContext):
    data = await state.get_data()
    caption = f"Yangi TO‘LOV SO‘ROVI:\n\nBookmaker: {data['bookmaker']}\nID: {data['user_id']}\nMiqdor: {data['amount']} so'm\n\nTasdiqlang!"
    await bot.send_photo(chat_id=ADMIN_ID, photo=msg.photo[-1].file_id, caption=caption)
    await msg.answer("To‘lov so‘rovingiz yuborildi. Tez orada admin tekshiradi.")
    await state.finish()

@dp.message_handler(lambda m: m.text == "Hisobdan chiqarish")
async def start_withdraw(msg: types.Message):
    await msg.answer("Qaysi bukmeker?", reply_markup=bookmaker_keyboard())
    await Withdraw.bookmaker.set()

@dp.callback_query_handler(state=Withdraw.bookmaker)
async def get_bookmaker_withdraw(call: types.CallbackQuery, state: FSMContext):
    await state.update_data(bookmaker=call.data)
    await call.message.answer("ID raqamingizni kiriting:")
    await Withdraw.next()

@dp.message_handler(state=Withdraw.user_id)
async def get_id_withdraw(msg: types.Message, state: FSMContext):
    await state.update_data(user_id=msg.text)
    await msg.answer("Maxsus kodni kiriting:")
    await Withdraw.next()

@dp.message_handler(state=Withdraw.secret_code)
async def get_code(msg: types.Message, state: FSMContext):
    await state.update_data(secret_code=msg.text)
    await msg.answer("Kartangiz raqamini yuboring:")
    await Withdraw.next()

@dp.message_handler(state=Withdraw.card_number)
async def get_card(msg: types.Message, state: FSMContext):
    data = await state.get_data()
    text = f"Yangi CHIQARISH SO‘ROVI:\n\nBookmaker: {data['bookmaker']}\nID: {data['user_id']}\nKod: {data['secret_code']}\nKarta: {msg.text}"
    await bot.send_message(chat_id=ADMIN_ID, text=text)
    await msg.answer("So‘rovingiz yuborildi. Tez orada admin bog‘lanadi.")
    await state.finish()

if __name__ == '__main__':
    executor.start_polling(dp, skip_updates=True)
