import telebot
from telebot import types
from datetime import datetime, timedelta

# إعداد البوت مع Token الخاص بك
API_TOKEN = '7472356917:AAEg8DpGspdp67mKmqg6XaNjDIhufNV6NRI'
bot = telebot.TeleBot(API_TOKEN)

# تخزين معرّف مدير البوت (حسابك) الذي سيتلقى لقطات الشاشة
ADMIN_CHAT_ID = '@AboabdoTalbesa'

# معرف قناة التليجرام للتحقق من الاشتراك
TELEGRAM_CHANNEL_USERNAME = '@alsafwaislam'

# تخزين النقاط (النجوم) للمستخدمين
user_stars = {}
# تخزين تاريخ الهدية اليومية
user_gift_date = {}

# إرسال رسالة الترحيب عند بدء البوت مع إجبار الاشتراك في قناة التليجرام والواتساب
@bot.message_handler(commands=['start'])
def send_welcome(message):
    user_id = message.from_user.id
    
    # التحقق من اشتراك المستخدم في قناة Telegram
    user_status = bot.get_chat_member(TELEGRAM_CHANNEL_USERNAME, user_id).status
    
    if user_status == 'left':
        bot.send_message(
            message.chat.id,
            f"يرجى الاشتراك في قناة التليجرام أولاً: {TELEGRAM_CHANNEL_USERNAME} ثم أعد المحاولة."
        )
    else:
        bot.send_message(
            message.chat.id,
            "مرحباً بكم في بوت متجر الصفوة للألبسة الشرعية - يمكنكم الحصول على جلابية مجانا وذلك بجمع نجوم حتى تصل ل550 نجمة ثم تستبدلها بجلابية. للمزيد من الاستفسار تواصل معنا عبر الواتساب: https://wa.me/message/YG2S4AEYBGIWP1"
        )
        
        # عرض قائمة المهام
        markup = types.InlineKeyboardMarkup(row_width=1)
        gift_btn = types.InlineKeyboardButton("🎁 الحصول على الهدية اليومية (5 ⭐)", callback_data="gift")
        like_page_btn = types.InlineKeyboardButton("👍 أعجب بصفحتنا على الفيسبوك (20 ⭐)", url="https://www.facebook.com/alsafwaislam/")
        instagram_btn = types.InlineKeyboardButton("📸 انشر منشور على الإنستجرام (20 ⭐)", url="https://www.instagram.com/alsafwaislam/")
        review_product_btn = types.InlineKeyboardButton("🛒 قيّم منتج في موقع الصفوة (20 ⭐)", url="https://alsafwaislam.com/?post_type=product")
        share_link_btn = types.InlineKeyboardButton("🔗 شارك الرابط الخاص بك (70 ⭐)", callback_data="share_link")
        google_maps_btn = types.InlineKeyboardButton("🌍 قيّمنا على خرائط جوجل (20 ⭐)", url="https://maps.app.goo.gl/fDxP6Y1niQkvRwrg7")
        whatsapp_share_btn = types.InlineKeyboardButton("📱 اشترك في قناة الواتساب (20 ⭐)", url="https://whatsapp.com/channel/0029Val7vgLATRSivsNvBJ3f/110")
        status_share_btn = types.InlineKeyboardButton("📲 انشر في حالتك على الواتساب (20 ⭐)", callback_data="https://whatsapp.com/channel/0029Val7vgLATRSivsNvBJ3f/110")
        check_stars_btn = types.InlineKeyboardButton("⭐ معرفة عدد النجوم", callback_data="check_stars")
        exchange_stars_btn = types.InlineKeyboardButton("🔄 استبدال النجوم (550 ⭐)", callback_data="exchange_stars")
        submit_screenshot_btn = types.InlineKeyboardButton("📸 إرسال لقطة شاشة", callback_data="send_screenshot")
        markup.add(gift_btn, like_page_btn, instagram_btn, review_product_btn, google_maps_btn, share_link_btn, whatsapp_share_btn, status_share_btn, check_stars_btn, exchange_stars_btn, submit_screenshot_btn)
        
        bot.send_message(message.chat.id, "اختر المهمة التي ترغب في تنفيذها لجمع النجوم:", reply_markup=markup)

# إضافة النجوم للمستخدم
def add_stars(user_id, stars):
    if user_id in user_stars:
        user_stars[user_id] += stars
    else:
        user_stars[user_id] = stars

    bot.send_message(user_id, f"تم إضافة {stars} نجوم لحسابك. لديك الآن {user_stars[user_id]} نجوم.")

# عرض عدد النجوم للمستخدم
def show_stars(user_id):
    stars = user_stars.get(user_id, 0)
    bot.send_message(user_id, f"لديك {stars} ⭐.")

# استبدال النجوم
def exchange_stars(user_id):
    if user_id in user_stars and user_stars[user_id] >= 550:
        user_stars[user_id] -= 550
        bot.send_message(user_id, "تم استبدال 550 نجمة بجلابية مجانية! يمكنك الآن الحصول عليها.")
    else:
        bot.send_message(user_id, "ليس لديك ما يكفي من النجوم لاستبدالها.")

# التعامل مع الضغط على الأزرار (غير الأزرار التي تحتوي على روابط)
@bot.callback_query_handler(func=lambda call: True)
def handle_tasks(call):
    user_id = call.from_user.id

    # التحقق من الاشتراك في قناة Telegram أولاً
    user_status = bot.get_chat_member(TELEGRAM_CHANNEL_USERNAME, user_id).status
    if user_status == 'left':
        bot.send_message(call.message.chat.id, f"يرجى الاشتراك في قناة التليجرام أولاً: {TELEGRAM_CHANNEL_USERNAME}.")
        return

    if call.data == "gift":
        now = datetime.now()
        last_gift_date = user_gift_date.get(user_id)

        # تحقق مما إذا كان المستخدم قد حصل على الهدية اليوم
        if last_gift_date and last_gift_date.date() == now.date():
            bot.send_message(user_id, "لقد حصلت على الهدية اليومية بالفعل اليوم. يمكنك الحصول عليها غدًا.")
        else:
            add_stars(user_id, 5)
            user_gift_date[user_id] = now  # تحديث تاريخ آخر هدية
            bot.answer_callback_query(call.id, "تم إضافة 5 نجوم كهدية يومية.")
    
    elif call.data == "send_screenshot":
        bot.send_message(call.message.chat.id, "يرجى إرسال لقطة الشاشة هنا.")
    
    elif call.data == "share_link":
        bot.send_message(call.message.chat.id, "قم بمشاركة الرابط الخاص بك ثم أرسل لنا لقطة شاشة.")

    elif call.data == "share_status":
        bot.send_message(call.message.chat.id, "يرجى نشر الحالة على الواتساب ثم إرسال لقطة شاشة.")

    elif call.data == "check_stars":
        show_stars(user_id)

    elif call.data == "exchange_stars":
        exchange_stars(user_id)

# التعامل مع الصور (لقطات الشاشة) التي يرسلها المستخدم
@bot.message_handler(content_types=['photo'])
def handle_screenshot(message):
    user_id = message.from_user.id
    username = message.from_user.username
    
    # إرسال الصورة إلى مدير البوت (حسابك)
    bot.send_message(ADMIN_CHAT_ID, f"📸 المستخدم @{username} (ID: {user_id}) أرسل لقطة شاشة.")
    bot.forward_message(ADMIN_CHAT_ID, message.chat.id, message.message_id)

    # تأكيد الاستلام للمستخدم
    bot.reply_to(message, "تم استلام لقطة الشاشة. سيتم التحقق منها وإضافة النجوم لاحقًا.")

# بدء البوت
bot.polling()
