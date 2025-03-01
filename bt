import logging
import json
import os
import datetime
from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import Application, CommandHandler, CallbackQueryHandler, ContextTypes
from apscheduler.schedulers.background import BackgroundScheduler

TOKEN = "7658744424:AAH3kuLvTLfF2oqpEhjA1xejDx20Ti0-5SE"
BIRTHDAY = datetime.datetime(2006, 1, 8, 0, 0)  # Заменить на свою дату рождения

# Запуск логирования
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# Пусть файл, в который будем записывать данные
data_file = 'user_data.json'

# Функция для загрузки данных из файла
def load_data():
    if os.path.exists(data_file):
        with open(data_file, 'r') as file:
            return json.load(file)
    else:
        return {}

# Функция для сохранения данных в файл
def save_data(data):
    with open(data_file, 'w') as file:
        json.dump(data, file, indent=4)

# Функция для добавления или обновления данных о пользователе
def add_or_update_user_data(user_id, interval, birthday):
    data = load_data()  # Загружаем текущие данные

    # Если пользователь уже есть в данных, обновляем информацию
    if user_id in data:
        data[user_id]['interval'] = interval  # Обновляем интервал
        data[user_id]['birthday'] = birthday.strftime("%Y-%m-%d %H:%M:%S")  # Обновляем дату рождения
        print(f"Данные пользователя {user_id} обновлены.")
    else:
        # Если пользователя нет, добавляем его
        data[user_id] = {
            'interval': interval,
            'birthday': birthday.strftime("%Y-%m-%d %H:%M:%S")
        }
        print(f"Данные пользователя {user_id} добавлены.")

    save_data(data)  # Сохраняем обновлённые данные


# Создаем планировщик (используем синхронную версию)
scheduler = BackgroundScheduler()

async def send_life_time(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    """Функция расчета прожитого времени и отправки сообщения"""
    now = datetime.datetime.now()
    delta = now - BIRTHDAY
    years = delta.days // 365
    days = delta.days % 365
    hours = delta.seconds // 3600
    weeks = delta.days // 7

    text = (f"Вы прожили:\n"
            f"{years} лет\n"
            f"{weeks} недель\n"
            f"{days} дней\n"
            f"{hours} часов")

    await update.message.reply_text(text)

async def send_life_time_callback(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    """Обработчик кнопки"""
    query = update.callback_query
    await query.answer()
    await send_life_time(query, context)

async def set_schedule(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    """Установка периодичности сообщений"""
    if context.args and context.args[0].isdigit():
        interval = int(context.args[0])

        # Очищаем старые задачи
        scheduler.remove_all_jobs()

        # Добавляем новую задачу
        scheduler.add_job(
            send_life_time,
            "interval",
            days=interval,
            args=[update, context],
            id=str(update.effective_user.id),
            replace_existing=True
        )

        await update.message.reply_text(f"Рассылка установлена каждые {interval} дней.")
    else:
        await update.message.reply_text("Используйте: /set <количество_дней> (например, /set 7)")

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    """Команда /start с кнопкой"""
    keyboard = [[InlineKeyboardButton("Запросить возраст", callback_data="life_time")]]
    reply_markup = InlineKeyboardMarkup(keyboard)

    await update.message.reply_text(
        "Привет! Я календарь жизни.\n"
        "📌 Нажми кнопку, чтобы узнать возраст.\n"
        "📌 Установи напоминание командой: /set <дни>.",
        reply_markup=reply_markup
    )

async def set_birthday(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    """Функция для изменения даты рождения"""
    user_id = update.message.from_user.id

    if context.args and len(context.args) == 1:
        try:
            new_birthday = datetime.datetime.strptime(context.args[0], "%Y-%m-%d")
            # Обновляем данные о пользователе
            add_or_update_user_data(user_id, 5, new_birthday)  # Предположим, интервал = 5

            await update.message.reply_text(f"Дата рождения успешно обновлена на {new_birthday.strftime('%Y-%m-%d')}.")
        except ValueError:
            await update.message.reply_text("Неверный формат даты. Пожалуйста, используйте формат YYYY-MM-DD.")
    else:
        await update.message.reply_text("Используйте команду /set_birthday <новая_дата> для изменения даты рождения.")

def main():
    """Основная функция запуска бота"""
    app = Application.builder().token(TOKEN).build()

    # Команды
    app.add_handler(CommandHandler("start", start))
    app.add_handler(CommandHandler("set", set_schedule))
    app.add_handler(CommandHandler("set_birthday", set_birthday))
    app.add_handler(CallbackQueryHandler(send_life_time_callback, pattern="life_time"))

    # Запускаем планировщик
    scheduler.start()

    # Запуск бота
    app.run_polling()

if __name__ == "__main__":
    main()
