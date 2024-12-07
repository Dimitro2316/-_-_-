import telebot
import requests
import logging

logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

TOKEN = '7607335670:AAGbDeeU1wklZE8Hx_vTKOTP4bXNkUz3S48'

# API ключ для OpenWeatherMap
API_KEY = '086a57e3faeadc16ac898ea09b611848'

bot = telebot.TeleBot(TOKEN)


@bot.message_handler(commands=['start'])
def start(message):
    bot.send_message(message.chat.id, 'Привет! Напишите название города, чтобы получить погоду.')


@bot.message_handler(func=lambda message: True)
def get_weather(message):
    city = message.text
    try:
        response = requests.get(
            f"http://api.openweathermap.org/data/2.5/weather?q={city}&appid={API_KEY}&units=metric"
        )
        response.raise_for_status()

        data = response.json()

        if data['cod'] == 200:
            temp = data['main']['temp']
            feels_like = data['main']['feels_like']
            description = data['weather'][0]['description']
            humidity = data['main']['humidity']
            wind_speed = data['wind']['speed']


            weather_info = f"Погода в {city}:\n" \
                           f"Температура: {temp}°C\n" \
                           f"Ощущается как: {feels_like}°C\n" \
                           f"Описание: {description}\n" \
                           f"Влажность: {humidity}%\n" \
                           f"Скорость ветра: {wind_speed} м/с"


            bot.send_message(message.chat.id, weather_info)

        else:
            bot.send_message(message.chat.id, f"Город {city} не найден.")


    except requests.exceptions.RequestException as e:
        logging.error(f"Ошибка при запросе к API: {e}")
        bot.send_message(message.chat.id, "Ошибка при получении данных о погоде. Проверьте подключение к сети.")
    except KeyError as e:
        logging.error(f"Ошибка в формате ответа API: {e}")
        bot.send_message(message.chat.id, "Ошибка в формате ответа API. Попробуйте снова позже.")
    except Exception as e:
        logging.error(f"Неизвестная ошибка: {e}")
        bot.send_message(message.chat.id, "Произошла непредвиденная ошибка. Попробуйте снова позже.")


if __name__ == "__main__":
    bot.infinity_polling()
