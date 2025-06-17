import tkinter as tk
from tkinter import PhotoImage
from PIL import Image, ImageTk
import random
import pyttsx3  # pyttsx3 kutubxonasini import qilamiz (matnni ovozga aylantirish uchun)
from gtts import gTTS
from playsound import playsound
import os
import uuid
import mysql.connector
from mysql.connector import Error
import tkinter as tk
from tkinter import messagebox


conn = None
cursor = None
user_name = ""
user_surname = ""

correct_count = 0
wrong_count = 0
selected_lang = "ru"  # boshlanishda o‘zbekcha bo‘lsin

def connect_to_database():
    global conn, cursor
    try:
        conn = mysql.connector.connect(
            host="localhost",
            user="root",
            password="",  # 🔐 MySQL parolingizni kiriting
            database="til_dasturi"
        )
        cursor = conn.cursor()
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS results (
                id INT AUTO_INCREMENT PRIMARY KEY,
                name VARCHAR(100),
                surname VARCHAR(100),
                category VARCHAR(255),
                question TEXT,
                user_answer TEXT,
                correct_answer TEXT,
                is_correct BOOLEAN,
                language VARCHAR(10),
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
            )
        """)
        conn.commit()
    except Error as e:
        print(f"MySQL ulanishda xato: {e}")

def ask_user_info(root):
    info_window = tk.Toplevel(root)
    info_window.title("Foydalanuvchi ma'lumotlari")
    info_window.geometry("600x300")
    info_window.grab_set()

    tk.Label(info_window, text="Ismingiz! What is your name? Как вас зовут? Aapka naam kya hai?", font=("Arial", 12)).pack(pady=5)
    name_entry = tk.Entry(info_window, font=("Arial", 12))
    name_entry.pack(pady=5)

    tk.Label(info_window, text="Familyangi? Your surname? Ваша фамилия? aapka upnaam?", font=("Arial", 12)).pack(pady=5)
    surname_entry = tk.Entry(info_window, font=("Arial", 12))
    surname_entry.pack(pady=5)

    def save_info_and_continue():
        global user_name, user_surname
        user_name = name_entry.get().strip()
        user_surname = surname_entry.get().strip()
        if user_name and user_surname:
            info_window.destroy()
        else:
            messagebox.showwarning("Ogohlantirish", "Iltimos, ism va familya kiriting!")

    tk.Button(info_window, text="Boshlash", font=("Arial", 12), bg="green", fg="white",
              command=save_info_and_continue).pack(pady=20)



def save_result(category, question, user_answer, correct_answer, is_correct, lang):
    if conn:
        try:
            cursor.execute("""
                INSERT INTO results (name, surname, category, question, user_answer, correct_answer, is_correct, language)
                VALUES (%s, %s, %s, %s, %s, %s, %s, %s)
            """, (
                user_name,
                user_surname,
                category,
                question,
                user_answer,
                correct_answer,
                is_correct,
                lang
            ))
            conn.commit()
        except Error as e:
            print(f"Ma'lumotni yozishda xato: {e}")



def speak_gtts(text, lang='uz'):
    try:
        if lang == 'uz':
            lang = 'ru'  # ❗ uz o‘rniga ru ishlatiladi
            lang = 'hi'
            lang = 'en'

        filename = f"voice_{uuid.uuid4()}.mp3"
        tts = gTTS(text=text, lang=lang)
        tts.save(filename)
        playsound(filename)
        os.remove(filename)
    except Exception as e:
        print(f"Ovoz chiqarishda xato: {e}")


def set_language(lang_code):
    global selected_lang
    selected_lang = lang_code
    print(f"🔄 Tanlangan til! Выбранный язык!: {selected_lang}")

    til_nomi = {
        "uz": "O‘zbek",
        "hi": "Hindi",
        "ru": "Русский",
        "en": "English"
    }

    message_label.config(text=f"Tanlangan til! Выбранный язык!: {til_nomi.get(lang_code, 'Tanlanmagan')}")

    if selected_lang == "uz":
        speak_gtts("Til o‘zgartirildi: O‘zbek", lang='uz')
    elif selected_lang == "hi":
        speak_gtts("भाषा बदल गई: हिंदी", lang='hi')
    elif selected_lang == "ru":
        speak_gtts("Язык изменён: Русский", lang='ru')
    elif selected_lang == "en":
        speak_gtts("Language changed: English", lang='en')





# pyttsx3 dvigatelini ishga tushirish va Alisa ovozini tanlash
engine = pyttsx3.init()
voices = engine.getProperty('voices')
for voice in voices:
    if "alisa" in voice.name.lower():
        engine.setProperty('voice', voice.id)
        break
engine.setProperty('rate', 150)  # Nutq tezligini sozlash (default 200)

# Asosiy oynani yaratish
root = tk.Tk()
root.state('zoomed') 
root.title("Интерактивное изучение хинди, русского и английского языков для детей/बच्चों के लिए इंटरएक्टिव हिंदी, रूसी और अंग्रेज़ी सीखना")
root.geometry("1366x768")


root.config(bg="#f0f8ff")  # Fon rangini o‘rnatish

label1 = tk.Label(root, text=" Интерактивное изучение хинди, русского и английского языков для детей/बच्चों के लिए इंटरएक्टिव हिंदी, रूसी और अंग्रेज़ी सीखना", font=("Arial", 20, "bold"), fg="darkblue", width=200, bg="#f0f8ff")
label1.pack(pady=20)



lang_frame = tk.Frame(root, bg="#f0f8ff")
lang_frame.pack(pady=10)

tk.Button(lang_frame, text="🇺🇿 O‘zbek", font=("Arial", 12), width=12, command=lambda: set_language("uz")).pack(side=tk.LEFT, padx=10)
tk.Button(lang_frame, text="🇮🇳 Hindi", font=("Arial", 12), width=12, command=lambda: set_language("hi")).pack(side=tk.LEFT, padx=10)
tk.Button(lang_frame, text="🇷🇺 Русский", font=("Arial", 12), width=12, command=lambda: set_language("ru")).pack(side=tk.LEFT, padx=10)
tk.Button(lang_frame, text="🇬🇧 English", font=("Arial", 12), width=12, command=lambda: set_language("en")).pack(side=tk.LEFT, padx=10)





# Kategoriyalar uchun rasmlar yuklash
try:
    cat_image = Image.open("cat.png").resize((250, 180))
    cat_image = ImageTk.PhotoImage(cat_image)

    it_image = Image.open("it.jpg").resize((250, 180))
    it_image = ImageTk.PhotoImage(it_image)

    yol_image = Image.open("yol.jpg").resize((250, 180))
    yol_image = ImageTk.PhotoImage(yol_image)

    sher_image = Image.open("sher.jpg").resize((250, 180))
    sher_image = ImageTk.PhotoImage(sher_image)

    fil_image = Image.open("fil.jpg").resize((250, 180))
    fil_image = ImageTk.PhotoImage(fil_image)

    ot_image = Image.open("ot.jpg").resize((250, 180))
    ot_image = ImageTk.PhotoImage(ot_image)

    maymun_image = Image.open("maymun.jpg").resize((250, 180))
    maymun_image = ImageTk.PhotoImage(maymun_image)

    jirafa_image = Image.open("jirafa.jpg").resize((250, 180))
    jirafa_image = ImageTk.PhotoImage(jirafa_image)

    zebra_image = Image.open("zebra.jpg").resize((250, 180))
    zebra_image = ImageTk.PhotoImage(zebra_image)

    ayiq_image = Image.open("ayiq.jpg").resize((250, 180))
    ayiq_image = ImageTk.PhotoImage(ayiq_image)


############################## hayvonlar tugadi ##################################################
    

    apple_image = Image.open("apple.jpg").resize((250, 180))
    apple_image = ImageTk.PhotoImage(apple_image)

    banana_image = Image.open("banana.jpg").resize((250, 180))
    banana_image = ImageTk.PhotoImage(banana_image)

    uzum_image = Image.open("uzum.jpg").resize((250, 180))
    uzum_image = ImageTk.PhotoImage(uzum_image)

    apelsin_image = Image.open("apilsen.jpg").resize((250, 180))
    apelsin_image = ImageTk.PhotoImage(apelsin_image)

    mango_image = Image.open("mango.jpg").resize((250, 180))
    mango_image = ImageTk.PhotoImage(mango_image)

    ananas_image = Image.open("anonas.jpg").resize((250, 180))
    ananas_image = ImageTk.PhotoImage(ananas_image)

    qulupnay_image = Image.open("qulupnay.jpg").resize((250, 180))
    qulupnay_image = ImageTk.PhotoImage(qulupnay_image)

    shorva_image = Image.open("shurva.jpg").resize((250, 180))
    shorva_image = ImageTk.PhotoImage(shorva_image)

    tarvuz_image = Image.open("torvuz.jpg").resize((250, 180))
    tarvuz_image = ImageTk.PhotoImage(tarvuz_image)

    papaya_image = Image.open("papaya.jpg").resize((250, 180))
    papaya_image = ImageTk.PhotoImage(papaya_image)


############################## mevalar tugadi ##################################################
    papugay_image = Image.open("papugay.jpg").resize((250, 180))
    papugay_image = ImageTk.PhotoImage(papugay_image)

    mynah_image = Image.open("mayna.jpg").resize((250, 180))
    mynah_image = ImageTk.PhotoImage(mynah_image)

    Pigeon_image = Image.open("kabutar1.jpg").resize((250, 180))
    Pigeon_image = ImageTk.PhotoImage(Pigeon_image)

    boyqush_image = Image.open("boyqush.jpg").resize((250, 180))
    boyqush_image = ImageTk.PhotoImage(boyqush_image)

    pingvin_image = Image.open("pinvin.png").resize((250, 180))
    pingvin_image = ImageTk.PhotoImage(pingvin_image)

    tavus_qushi_image = Image.open("tustoviq.png").resize((250, 180))
    tavus_qushi_image = ImageTk.PhotoImage(tavus_qushi_image)

    oqqush_image = Image.open("oqqush.jpg").resize((250, 180))
    oqqush_image = ImageTk.PhotoImage(oqqush_image)

    karga_image = Image.open("qarga.jpg").resize((250, 180))
    karga_image = ImageTk.PhotoImage(karga_image)

    kumush_qush_image = Image.open("kumush_qush.jpg").resize((250, 180))
    kumush_qush_image = ImageTk.PhotoImage(kumush_qush_image)

    yelkan_qushi_image = Image.open("yelkan.jpeg").resize((250, 180))
    yelkan_qushi_image = ImageTk.PhotoImage(yelkan_qushi_image)

############################## mevalar tugadi ##################################################

    # Ranglar
    red_image = Image.open("red.jpg").resize((250, 180))
    red_image = ImageTk.PhotoImage(red_image)

    blue_image = Image.open("blue.jpg").resize((250, 180))
    blue_image = ImageTk.PhotoImage(blue_image)

    green_image = Image.open("green.png").resize((250, 180))
    green_image = ImageTk.PhotoImage(green_image)

    yellow_image = Image.open("yellow.jpg").resize((250, 180))
    yellow_image = ImageTk.PhotoImage(yellow_image)

    black_image = Image.open("black.jpg").resize((250, 180))
    black_image = ImageTk.PhotoImage(black_image)

    white_image = Image.open("white.jpg").resize((250, 180))
    white_image = ImageTk.PhotoImage(white_image)

    # O‘quv qurollari
    book_image = Image.open("book.png").resize((250, 180))
    book_image = ImageTk.PhotoImage(book_image)

    pen_image = Image.open("pen.png").resize((250, 180))
    pen_image = ImageTk.PhotoImage(pen_image)

    pencil_image = Image.open("pencil.png").resize((250, 180))
    pencil_image = ImageTk.PhotoImage(pencil_image)

    bag_image = Image.open("bag.png").resize((250, 180))
    bag_image = ImageTk.PhotoImage(bag_image)

    ruler_image = Image.open("ruler.jpg").resize((250, 180))
    ruler_image = ImageTk.PhotoImage(ruler_image)

    # Ovqatlar
    bread_image = Image.open("bread.jpg").resize((250, 180))
    bread_image = ImageTk.PhotoImage(bread_image)

    rice_image = Image.open("rice.jpeg").resize((250, 180))
    rice_image = ImageTk.PhotoImage(rice_image)

    meat_image = Image.open("meat.png").resize((250, 180))
    meat_image = ImageTk.PhotoImage(meat_image)

    egg_image = Image.open("egg.jpg").resize((250, 180))
    egg_image = ImageTk.PhotoImage(egg_image)

    milk_image = Image.open("milk.jpg").resize((250, 180))
    milk_image = ImageTk.PhotoImage(milk_image)


############################## ranglar tugadi ##################################################

except Exception as e:
    print(f"Rasmni yuklashda xato: {e}")

# Hayvonlar, mevalar va qushlar uchun ro‘yxatlar
animals = [
    ("Mushuk! बिल्ली(billi)! Кошка! Cat!", cat_image),
    ("It! कुत्ता(kutta)! Собака! dog!", it_image),
    ("Yo‘lbars! बाघ (baagh)! Тигр! Tiger!", yol_image),
    ("Sher! शेर (sher)! Лев! Lion!", sher_image),
    ("Fil! हाथी (haathi)! Слон! Elephant!", fil_image),
    ("Ot! घोड़ा (ghoda)! Лошадь! Horse!", ot_image),
    ("Maymun! बंदर (bandar)! Обезьяна! Monkey!", maymun_image),
    ("Jirafa! जिराफ़ (jiraaf)! Жираф! Giraffe!", jirafa_image),
    ("Zebra! ज़ेबरा (zebra)! Зебра! Zebra!", zebra_image),
    ("Ayiq! भालू (bhalu)! Медведь! Bear", ayiq_image)
]

fruits = [
    ("Olma! सेब (seb)! Яблоко! Apple!", apple_image),
    ("Banan! केला (kela)! Банан! Banana!", banana_image),
    ("Uzum! अंगूर (angoor)! Виноград! Grapes!", uzum_image),
    ("Apelsin! संतरा (santra)! Апельсин! Orange!", apelsin_image),
    ("Mango! आम (aam)! Манго! Mango!", mango_image),
    ("Ananas! अनानास (ananas)! Ананас! Pineapple!", ananas_image),
    ("Qulupnay! स्ट्रॉबेरी (strawberry)! Клубника! Strawberry!", qulupnay_image),
    ("Sho‘rva! सूप (soop)! Суп! Soup!", shorva_image),
    ("Tarvuz! तरबूज (tarbooj)! Арбуз! Watermelon!", tarvuz_image),
    ("Papayya! पपीता (papita)! Папайя! Papaya!", papaya_image)
]



birds = [
    ("Popugay! तोता (tota)! Попугай! Parrot!", papugay_image),
    ("Mayna! मैना (maina)! Майна! Mynah!", mynah_image),
    ("Kabutar! कबूतर (kabootar)! Голубь! Pigeon!", Pigeon_image),
    ("Boyqush! उल्लू (ullu)! Сова! Owl!", boyqush_image),
    ("Pingvin! पेंगुइन (penguin)! Пингвин! Penguin!", pingvin_image),
    ("Tovus qushi! मोर (mor)! Павлин! Peacock!", tavus_qushi_image),
    ("Oqqush! हंस (hans)! Лебедь! Swan!", oqqush_image),
    ("Qarg‘a! कौआ (kauaa)! Ворона! Crow!", karga_image),
    ("Kumush qush! (silver bird)! Серебристая птица! Silver bird!", kumush_qush_image),
    ("Albatros! अल्बाट्रॉस (albatross)! Альбатрос! Albatross!", yelkan_qushi_image)
]





##########################################################################

colors = [
    ("Qizil! लाल (laal)! Красный! Red!", red_image),
    ("Ko‘k! नीला (neela)! Синий! Blue!", blue_image),
    ("Yashil! हरा (hara)! Зелёный! Green!", green_image),
    ("Sariq! पीला (peela)! Жёлтый! Yellow!", yellow_image),
    ("Qora! काला (kaala)! Чёрный! Black!", black_image),
    ("Oq! सफेद (saphed)! Белый! White!", white_image)
]


school_items = [
    ("Kitob! किताब (kitaab)! Книга! Book!", book_image),
    ("Ruchka! पेन (pen)! Ручка! Pen!", pen_image),
    ("Qalam! पेंसिल (pencil)! Карандаш! Pencil!", pencil_image),
    ("Sumka! बैग (bag)! Рюкзак! Сумка! Bag!", bag_image),
    ("Chizg‘ich! पैमाना (paimaana)! Линейка! Ruler!", ruler_image)
]



foods = [
    ("Non! रोटी (roti)! Хлеб! Bread!", bread_image),
    ("Guruch! चावल (chaawal)! Рис! Rice!", rice_image),
    ("Go‘sht! मांस (maans)! Мясо! Meat!", meat_image),
    ("Tuxum! अंडा (anda)! Яйцо! Egg!", egg_image),
    ("Sut! दूध (doodh)! Молоко! Milk!", milk_image)
]










###########################################################################
# Hozirgi kategoriya va javobni kuzatib borish uchun global o‘zgaruvchilar

current_category = None
current_answer = None


# So‘zni ko‘rsatish uchun yorliq (label)
word_label = tk.Label(root, text="", font=("Arial", 30, "bold"), fg="darkblue", width=50, bg="#f0f8ff")
word_label.pack(pady=20)

# Rasmni ko‘rsatish uchun yorliq
image_label = tk.Label(root, bg="#f0f8ff")
image_label.pack(pady=20)

# Xabarlarni ko‘rsatish uchun yorliq
message_label = tk.Label(root, text="", font=("Arial", 20, "italic"), fg="green", bg="#f0f8ff")
message_label.pack(pady=20)

# So‘z va rasmni ko‘rsatish, shuningdek matnni ovozga aylantirish
def show_word_and_image(category):
    global current_category, current_answer
    if category == "Hayvonlar / जानवर (Jaanvar)/ Животные / Animals":
        words_images = animals
    elif category == "Mevalar / फल (Phal) / Фрукты / Fruits":
        words_images = fruits
    elif category == "Qushlar / पक्षी (Pakshi) / Птицы / Birds":
        words_images = birds
    elif category == "Ranglar / रंग (Rang) / Цвета / Colors":
        words_images = colors
    elif category == "O‘quv qurollari / अध्ययन सामग्री / Школьные принадлежности / School supplies":
        words_images = school_items
    elif category == "Ovqatlar / खाना (khaana) / Питание / Meals":
        words_images = foods
    else:
        words_images = []

    current_answer, image = random.choice(words_images)
    current_answer = current_answer.strip()  # ⚠️ MUHIM QATOR!
    image_label.config(image=image)
    word_label.config(text=" Bu nima? / यह क्या है?/ (Yeh kya hai?) / Что это? / What is this?")
    current_category = category
    message_label.config(text="")
    create_answer_buttons()



# Foydalanuvchining javobini tekshirish
def check_answer(user_answer):
    global current_answer, message_label, correct_count, wrong_count

    is_correct = 1 if user_answer.lower().strip() == current_answer.lower().strip() else 0

    if is_correct:
        correct_count += 1
        message_label.config(text=f"✅ To'g'ri! верно!  right!  | ✅ {correct_count} | ❌ {wrong_count}", fg="green")
        speak_gtts("Barakalla! Молодец! Bravo! Shabaash!", lang=selected_lang)
        root.after(1500, lambda: show_word_and_image(current_category))
    else:
        wrong_count += 1
        message_label.config(text=f"❌ Xato! Ошибка! Error! Galti! | ✅ {correct_count} | ❌ {wrong_count}", fg="red")
        speak_gtts("Yana urinib ko‘r! Попробуй снова! Try again! Phir se koshish karo!", lang=selected_lang)

    # ✅ Faqat shu yerda natijani bazaga yozing
    save_result(current_category, word_label.cget("text"), user_answer, current_answer, is_correct, selected_lang)


    # ✅ Javobni MySQL bazaga saqlash
def save_result(category, question, user_answer, correct_answer, is_correct, lang):
    if conn:
        try:
            cursor.execute("""
                INSERT INTO results (name, surname, category, question, user_answer, correct_answer, is_correct, language)
                VALUES (%s, %s, %s, %s, %s, %s, %s, %s)
            """, (
                user_name,
                user_surname,
                category,
                question,
                user_answer,
                correct_answer,
                is_correct,
                lang
            ))
            conn.commit()
        except Error as e:
            print(f"Ma'lumotni yozishda xato: {e}")


# Tugmalarni o‘chirish
def clear_previous_buttons():
    for widget in frame_buttons.winfo_children():
        widget.destroy()

# Javob tugmalarini yaratish
def create_answer_buttons():
    global current_answer
    clear_previous_buttons()

    # Barcha kategoriyalardan tanlovlar olish
    all_items = animals + fruits + birds + colors + school_items + foods
    answer_options = [item[0] for item in all_items if item[0] != current_answer]

    # Tasodifiy 9 ta noto‘g‘ri + 1 to‘g‘ri javob
    answer_options = random.sample(answer_options, 9)
    answer_options.append(current_answer)
    random.shuffle(answer_options)

    # Tugmalarni 2 qatorda 5 tadan joylashtirish
    for i, option in enumerate(answer_options):
        row = i // 5
        col = i % 5
        answer_button = tk.Button(
            frame_buttons,
            text=option,
            font=("Arial", 12),
            bg="#90EE90",
            width=38,
            height=2,
            command=lambda answer=option: check_answer(answer)  # ✅ shunday bo‘lishi shart!
        )
        answer_button.grid(row=row, column=col, padx=13, pady=5)




# Kategoriya tugmalarini yaratish
def create_category_buttons():
    categories = [
        ("Hayvonlar / जानवर (Jaanvar)/ Животные / Animals", "Hayvonlar"),
        ("Mevalar / फल (Phal) / Фрукты / Fruits", "Mevalar"),
        ("Qushlar / पक्षी (Pakshi) / Птицы / Birds", "Qushlar"),
        ("Ranglar / रंग (Rang) / Цвета / Colors", "Ranglar"),
        ("O‘quv qurollari / अध्ययन सामग्री / Школьные принадлежности / School supplies", "Maktab buyumlari"),
        ("Ovqatlar / खाना (khaana) / Питание / Meals", "Ovqatlar")
    ]
    
    for category_label, _ in categories:
        btn = tk.Button(
            root, 
            text=category_label, 
            font=("Arial", 14), 
            bg="#ADD8E6", 
            width=80, 
            height=1,
            command=lambda cat=category_label: speak_and_show_category(cat)  # Faqat category_label o'tkazilmoqda
        )
        btn.pack(pady=5)





# Dasturdan chiqish tugmasi
#exit_button = tk.Button(root, text="Выход / Exit", font=("Arial", 14), bg="#FF6347", width=20, height=2,
#                        command=root.quit)
#exit_button.pack(pady=20)

# Javob tugmalarini o‘rnatish uchun ramka
frame_buttons = tk.Frame(root, bg="#f0f8ff")
frame_buttons.pack(pady=20)



# Dastur boshlanganda ovozli ko‘rsatma
def speak_choose_category():
    if selected_lang == "ru":
        speak_gtts("Пожалуйста, выберите категорию.", lang='ru')
    elif selected_lang == "hi":
        speak_gtts("कृपया श्रेणी चुनें।", lang='hi')
    elif selected_lang == "uz":
        speak_gtts("Iltimos, kategoriya tanlang.", lang='uz')
    elif selected_lang == "en":
        speak_gtts("Please choose a category.", lang='en')




root.after(500, speak_choose_category)  # 1 sekunddan keyin ovozda aytsin



def speak_and_show_category(category_label):
    """Tanlangan kategoriyani ovozda aytib, so'zlarni ko'rsatish"""
    # Ovozda kategoriya nomini aytish
    if selected_lang == "uz":
        speak_gtts("Siz " + category_label.split('/')[0].strip() + " kategoriyasini tanladingiz", lang='uz')
    elif selected_lang == "hi":
        speak_gtts("आपने " + category_label.split('/')[1].split('(')[0].strip() + " श्रेणी चुनी है", lang='hi')
    elif selected_lang == "ru":
        speak_gtts("Вы выбрали категорию " + category_label.split('/')[2].strip(), lang='ru')
    elif selected_lang == "en":
        speak_gtts("You selected " + category_label.split('/')[3].strip() + " category", lang='en')
    
    # So'zlarni ko'rsatish
    show_word_and_image(category_label)



def speak_gtts_safe(text, lang_code):
    """gTTS bilan tilda gapirish: uz bo‘lsa ru orqali"""
    real_lang = lang_code
    if lang_code == "uz":
        real_lang = "ru"  # uz ovoz yo‘q, shuning uchun ruscha bilan gapiramiz
        real_lang = "hi"
        real_lang = "en"



    try:
        filename = f"voice_{uuid.uuid4()}.mp3"
        tts = gTTS(text=text, lang=real_lang)
        tts.save(filename)
        playsound(filename)
        os.remove(filename)
    except Exception as e:
        print(f"Ovoz chiqarishda xato: {e}")


# ✅ BAZA ULANADI
connect_to_database()

# ✅ TKINTER OYNA YARATILADI
ask_user_info(root)

# ✅ KATEGORIYA TUGMALARI
create_category_buttons()

# ✅ OVOZLI KO‘RSATMA
root.after(500, speak_choose_category)

# ✅ DASTUR ISHGA TUSHADI
root.mainloop()

# ✅ CHIQISHDA BAZANI YOPISH
if conn:
    cursor.close()
    conn.close()


root = tk.Tk()
connect_to_database()
ask_user_info(root)



# Dastur oynasini boshlash
create_category_buttons()
root.mainloop()

