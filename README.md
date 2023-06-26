import telebot
import datetime

TOKEN = '6281972584:AAGtbR3b40rDin9Q7kJf6mFVQZwi8ufUfGo'
CHANNEL_ID = '368252510'

bot = telebot.TeleBot(TOKEN)

@bot.message_handler(commands=['start'])
def start(message):
    chat_id = message.chat.id
    registration_date = datetime.datetime.now()
    
    # Проверка подписки пользователя на канал
    user_subscribed = check_subscription(chat_id)
    
    if user_subscribed:
        # Отправляем напоминание через 4 дня
        reminder_date = registration_date + datetime.timedelta(days=4)
        reminder_text = "Через 10 дней доступ к марафону будет закрыт"
        schedule_reminder(chat_id, reminder_date, reminder_text)
        
        # Отправляем напоминание через 10 дней
        reminder_date = registration_date + datetime.timedelta(days=10)
        reminder_text = "Через 4 дня доступ к марафону будет закрыт"
        schedule_reminder(chat_id, reminder_date, reminder_text)
    else:
        bot.send_message(chat_id, "Вы не подписаны на канал.")
        # Действия при отсутствии подписки
        
def check_subscription(chat_id):
    # Используем метод `get_chat_member` для проверки подписки пользователя на канал
    response = bot.get_chat_member(CHANNEL_ID, chat_id)
    if response.status == 'left' or response.status == 'kicked':
        return False
    else:
        return True

def schedule_reminder(chat_id, reminder_date, reminder_text):
    now = datetime.datetime.now()
    time_delta = (reminder_date - now).total_seconds()
    if time_delta > 0:
        # Используем метод `send_message` с параметром `parse_mode='Markdown'`, если нужно отправить форматированный текст.
        bot.send_message(chat_id, reminder_text)
        bot.send_message(chat_id, f"Напоминание запланировано на {reminder_date}")
        bot.send_message(chat_id, "Для отмены напоминания отправьте /cancel_reminder")
        
        # Запланируем выполнение функции `send_scheduled_message` через заданное время
        bot.scheduler_instance.schedule_once(send_scheduled_message, time_delta, chat_id, reminder_text)
    else:
        bot.send_message(chat_id, "Напоминание уже просрочено.")

def send_scheduled_message(chat_id, reminder_text):
    bot.send_message(chat_id, reminder_text)

@bot.message_handler(commands=['cancel_reminder'])
def cancel_reminder(message):
    # Отменяем запланированное напоминание
    bot.scheduler_instance.cancel_all()
    bot.send_message(message.chat.id, "Напоминания отменены.")

bot.polling()# qqq
