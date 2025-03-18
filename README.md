# VBVA-FOR-WINDOWS

import speech_recognition as sr
import pyttsx3
import os
import subprocess
import webbrowser
import datetime
import time
import requests
import threading
import keyboard
import smtplib
from email.message import EmailMessage
import json

def speak(text):
    engine = pyttsx3.init()
    engine.say(text)
    engine.runAndWait()

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
        except sr.UnknownValueError:
            speak("Sorry, I didn't catch that. Please repeat.")
        except sr.RequestError:
            speak("Sorry, my speech service is down.")
        except sr.WaitTimeoutError:
            speak("No command detected. Please try again.")
        return ""

def open_application(command):
    apps = {
        "notepad": "notepad.exe",
        "calculator": "calc.exe",
        "command prompt": "cmd.exe",
        "cmd": "cmd.exe",
        "browser": "start chrome",
        "explorer": "explorer",
        "task manager": "taskmgr",
        "paint": "mspaint",
        "settings": "start ms-settings:"
    }
    
    for key in apps:
        if key in command:
            speak(f"Opening {key}")
            subprocess.run(apps[key], shell=True)
            return
    
    speak("Sorry, I don't know how to open that.")

def get_time():
    now = datetime.datetime.now().strftime("%H:%M:%S")
    speak(f"The time is {now}")

def get_date():
    today = datetime.date.today().strftime("%B %d, %Y")
    speak(f"Today's date is {today}")

def google_search(query):
    speak(f"Searching Google for {query}")
    webbrowser.open(f"https://www.google.com/search?q={query}")

def get_weather():
    API_KEY = "your_api_key_here"
    CITY = "New York"
    url = f"http://api.openweathermap.org/data/2.5/weather?q={CITY}&appid={API_KEY}&units=metric"
    response = requests.get(url).json()
    if response.get("main"):
        temp = response["main"]["temp"]
        condition = response["weather"][0]["description"]
        speak(f"The current temperature in {CITY} is {temp} degrees Celsius with {condition}.")
    else:
        speak("Sorry, I couldn't fetch the weather details.")

def fetch_news():
    API_KEY = "your_news_api_key_here"
    url = f"https://newsapi.org/v2/top-headlines?country=us&apiKey={API_KEY}"
    response = requests.get(url).json()
    articles = response.get("articles", [])[:5]
    if articles:
        speak("Here are the top news headlines.")
        for article in articles:
            speak(article['title'])
    else:
        speak("Sorry, I couldn't fetch the news.")

def control_system(command):
    if "shutdown" in command:
        speak("Shutting down the system")
        os.system("shutdown /s /t 5")
    elif "restart" in command:
        speak("Restarting the system")
        os.system("shutdown /r /t 5")
    elif "lock" in command:
        speak("Locking the system")
        os.system("rundll32.exe user32.dll,LockWorkStation")
    elif "sleep" in command:
        speak("Putting the system to sleep")
        os.system("rundll32.exe powrprof.dll,SetSuspendState 0,1,0")

def set_alarm(time_str):
    target_time = datetime.datetime.strptime(time_str, "%H:%M").time()
    while True:
        now = datetime.datetime.now().time()
        if now.hour == target_time.hour and now.minute == target_time.minute:
            speak("Time's up! Your alarm is ringing!")
            break
        time.sleep(30)

def send_email(to_email, subject, body):
    sender_email = "your_email@gmail.com"
    sender_password = "your_password"
    
    msg = EmailMessage()
    msg["From"] = sender_email
    msg["To"] = to_email
    msg["Subject"] = subject
    msg.set_content(body)
    
    with smtplib.SMTP_SSL("smtp.gmail.com", 465) as server:
        server.login(sender_email, sender_password)
        server.send_message(msg)
    
    speak("Email has been sent successfully.")

def main():
    speak("Voice assistant activated.")
    while True:
        command = listen()
        if not command:
            continue
        
        if 'open' in command:
            open_application(command)
        elif 'time' in command:
            get_time()
        elif 'date' in command:
            get_date()
        elif 'search' in command:
            google_search(command.replace("search", "").strip())
        elif 'weather' in command:
            get_weather()
        elif 'news' in command:
            fetch_news()
        elif 'set alarm' in command:
            alarm_time = command.replace("set alarm for", "").strip()
            speak(f"Setting an alarm for {alarm_time}")
            threading.Thread(target=set_alarm, args=(alarm_time,)).start()
        elif "send email" in command:
            speak("Please provide the recipient's email address.")
            to_email = listen()
            speak("What is the subject?")
            subject = listen()
            speak("What is the message?")
            body = listen()
            send_email(to_email, subject, body)
        elif any(word in command for word in ["shutdown", "restart", "lock", "sleep"]):
            control_system(command)
        elif 'exit' in command or 'quit' in command:
            speak("Are you sure you want to exit?")
            confirm = listen()
            if 'yes' in confirm or 'confirm' in confirm:
                speak("Goodbye!")
                break
            else:
                speak("Resuming operations.")

if __name__ == "__main__":
    main()
