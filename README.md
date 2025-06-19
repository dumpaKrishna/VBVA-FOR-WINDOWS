**#VOICE - BASED VIRTUAL ASSISTANCE FOR WINDOWS**

import speech_recognition as sr
import pyttsx3
import webbrowser
import datetime
import subprocess
import json
import base64
from cryptography.fernet import Fernet
import os

# ========== Security Configuration ==========

# Use a securely generated encryption key (store it in a secure file in production)
key_file = "secure.key"
if not os.path.exists(key_file):
    with open(key_file, "wb") as f:
        f.write(Fernet.generate_key())

with open(key_file, "rb") as f:
    ENCRYPTION_KEY = f.read()

fernet = Fernet(ENCRYPTION_KEY)

# ========== Voice Engine ==========
engine = pyttsx3.init()

def speak(text):
    print(f"Assistant: {text}")
    engine.say(text)
    engine.runAndWait()

# ========== Access Control ==========
def authenticate_user():
    speak("Please say the access passphrase.")
    command = listen()
    if "open sesame" in command.lower():
        speak("Access granted.")
        return True
    else:
        speak("Access denied.")
        return False

# ========== Speech Recognition ==========
def listen():
    recognizer = sr.Recognizer()
    with sr.Microphone() as source:
        print("Listening...")
        recognizer.adjust_for_ambient_noise(source)
        try:
            audio = recognizer.listen(source, timeout=5)
            command = recognizer.recognize_google(audio).lower()
            print(f"You said: {command}")
            return command
        except:
            speak("Sorry, I didn't catch that.")
            return ""

# ========== Command Processing ==========
def process_command(command):
    if "time" in command:
        now = datetime.datetime.now().strftime("%H:%M:%S")
        speak(f"The time is {now}")
    elif "date" in command:
        today = datetime.date.today().strftime("%B %d, %Y")
        speak(f"Today's date is {today}")
    elif "open google" in command:
        speak("Opening Google")
        webbrowser.open("https://www.google.com")
    elif "open notepad" in command:
        speak("Opening Notepad")
        subprocess.Popen(["notepad.exe"])
    elif "exit" in command or "quit" in command:
        speak("Goodbye!")
        return False
    else:
        speak("I didn't understand that command.")
    return True

# ========== Secure Logging ==========
def log_command(command):
    log = {
        "command": command,
        "timestamp": str(datetime.datetime.now())
    }
    encrypted = fernet.encrypt(json.dumps(log).encode())
    with open("commands.securelog", "ab") as f:
        f.write(encrypted + b"\n")

# ========== Main ==========
def main():
    speak("Secure Voice Assistant initialized.")
    if not authenticate_user():
        return

    active = True
    while active:
        command = listen()
        if not command:
            continue
        log_command(command)
        active = process_command(command)

if __name__ == "__main__":
    main()
