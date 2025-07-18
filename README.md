from tkinter import *
from tkinter import ttk
from deep_translator import GoogleTranslator
import re


def to_prereform_russian(text):
    # Сначала переводим в современный русский (если текст на другом языке)
    if any(char.isalpha() and ord(char) > 128 for char in text):
        text = GoogleTranslator(source='auto', target='ru').translate(text)
    
    # Основные правила замены
    replacements = {
        'е': 'ѣ',    # ять
        'ф': 'ѳ',     # фита
        'и': 'i',     # десятиричное и
    }
    
    # Особые случаи (словарные исключения)
    exceptions = {
        'мир': 'мiръ',
        'есть': 'ѣсть',
        'зеленый': 'зеленый',
    }
    
    # Сначала заменяем исключения
    for modern, old in exceptions.items():
        text = re.sub(r'\b' + modern + r'\b', old, text)
    
    # Затем общие правила
    for modern, old in replacements.items():
        text = text.replace(modern, old)
    
    # Добавляем твердые знаки после согласных на конце слов
    text = re.sub(r'([бвгджзклмнпрстфхцчшщ])([ ,.!?;\n])', r'\1ъ\2', text)
    
    return text

LANGUAGES = {
    'Английский': 'en',
    'Русский': 'ru',
    'Французский': 'fr',
    'Немецкий': 'de',
    'Испанский': 'es',
    'Китайский': 'zh',
    'Японский': 'ja',
    'Сербский': 'sr',
    'Словацкий': 'sk',
    'Словенский': 'sl',
    'Польский': 'pl',
    'Украинский': 'uk', 
    'Белоруский': 'be'
}


def trans_text():
    text1 = tex.get("1.0", END).strip()
    if not text1:
        return
        
    target = lang_var.get()
    
    if target == 'Русский (дореф.)':
        # Если выбран дореформенный русский
        if any(char.isalpha() and ord(char) > 128 for char in text1):
            # Если текст не на русском - сначала переводим на русский
            modern_russian = GoogleTranslator(source='auto', target='ru').translate(text1)
            translated = to_prereform_russian(modern_russian)
        else:
            # Если текст уже на русском - просто конвертируем
            translated = to_prereform_russian(text1)
    else:
        # Для всех других языков - обычный перевод
        translated = GoogleTranslator(source='auto', target=LANGUAGES[target]).translate(text1)
    
    output.delete("1.0", END)
    output.insert("1.0", translated)
 
root = Tk()
root.title("Переводчик")
root.geometry("500x450")
all_languages = list(LANGUAGES.keys()) + ['Русский (дореф.)']
lang_var = StringVar(root)
lang_var.set('Английский')  # Язык по умолчанию

label = Label(root, fg="black", text="Введите текст")
label.place(relx=0.5, y=30, anchor=CENTER) #Узнать про анкор 
tex = Text(root, width=45, height=5 )
tex.place(relx=0.5, y=160, anchor=CENTER)


lang_label = Label(root, text="Перевести на:")
lang_label.place(relx=0.3, y=78, anchor=CENTER)

lang_combobox = ttk.Combobox(root, textvariable=lang_var, values=all_languages)
lang_combobox.place(relx=0.6, y=80, anchor=CENTER)

btn = Button(root, text="Перевести", command=trans_text )
btn.place(relx=0.5, y=215, anchor=CENTER)

output = Text(root, width=45, height=5)
output.place(relx=0.5, y=270, anchor=CENTER)


root.mainloop()
