# super-duper-spoon
Jarvis is an intelligent voice assistant desigimport pyttsx3
import speech_recognition as sr
import openai
import os
import webbrowser
import datetime

import wikipedia
import cv2
import requests
from PyDictionary import PyDictionary

# Initialization
engine = pyttsx3.init()
recognizer = sr.Recognizer()
dictionary = PyDictionary()
openai.api_key = "YOUR_API_KEY"  # Replace with your real API key

# Speak out text
def speak(text):
    print("Assistant:", text)
    engine.say(text)
    engine.runAndWait()

# Wake Word Detection
def wait_for_wake_word():
    speak("Say 'Hey Jarvis' to activate me.")
    while True:
        with sr.Microphone() as source:
            audio = recognizer.listen(source)
            try:
                wake = recognizer.recognize_google(audio)
                if "jarvis" in wake.lower():
                    speak("Yes sir?")
                    return
            except:
                continue

# Listen to voice command
def take_command():
    with sr.Microphone() as source:
        speak("Listening...")
        recognizer.adjust_for_ambient_noise(source)
        audio = recognizer.listen(source)
    try:
        command = recognizer.recognize_google(audio)
        print("You said:", command)
        return command.lower()
    except sr.UnknownValueError:
        return "i didn't catch that"
    except sr.RequestError:
        return "Sorry, the service is unavailable"

# Ask ChatGPT
def ask_gpt(prompt):
    try:
        response = openai.ChatCompletion.create(
            model="gpt-3.5-turbo",
            messages=[{"role": "user", "content": prompt}]
        )
        return response['choices'][0]['message']['content']
    except:
        return "Sorry, I couldn't connect to GPT."

# Define a word using dictionary
def define_word(word):
    try:
        meaning = dictionary.meaning(word)
        if meaning:
            for key, value in meaning.items():
                speak(f"{key}: {value[0]}")
                print(f"{key}: {value[0]}")
        else:
            speak("Sorry, I couldn't find the meaning.")
    except:
        speak("Something went wrong while fetching the meaning.")

# Chat Mode
def chat_mode():
    speak("Chat mode activated. Type 'exit' to quit.")
    while True:
        user_input = input("You: ")
        if user_input.lower() in ["exit", "quit"]:
            speak("Exiting chat mode. Back to voice commands.")
            break
        elif user_input.startswith("define "):
            word = user_input.replace("define", "").strip()
            define_word(word)
        else:
            response = ask_gpt(user_input)
            print("Assistant:", response)
            speak(response)

# Note Taking
def take_note():
    speak("What should I write?")
    note = take_command()
    with open("notes.txt", "a") as file:
        file.write(f"{note}\n")
    speak("Note saved.")

def read_notes():
    try:
        with open("notes.txt", "r") as file:
            notes = file.read()
            speak("Here are your notes.")
            speak(notes)
    except:
        speak("You don't have any notes yet.")

# Weather

def get_weather(city="Delhi"):
    api_key = "YOUR_OPENWEATHER_API_KEY"
    url = f"http://api.openweathermap.org/data/2.5/weather?q={city}&appid={api_key}&units=metric"
    try:
        data = requests.get(url).json()
        if data.get("main"):
            temp = data["main"]["temp"]
            desc = data["weather"][0]["description"]
            speak(f"The temperature in {city} is {temp} degrees with {desc}")
        else:
            speak("Couldn't fetch weather.")
    except:
        speak("Error fetching weather.")

# Music

def play_music():
    music_dir = "C:\\Users\\YourName\\Music"
    try:
        songs = os.listdir(music_dir)
        os.startfile(os.path.join(music_dir, songs[0]))
        speak("Playing music.")
    except:
        speak("Music folder not found or empty.")

# Face Unlock (Dummy)
def face_unlock():
    speak("Launching camera for face scan.")
    cap = cv2.VideoCapture(0)
    if not cap.isOpened():
        speak("Cannot access camera.")
        return
    speak("Press Q to close camera.")
    while True:
        ret, frame = cap.read()
        if not ret:
            break
        cv2.imshow("Face Unlock", frame)
        if cv2.waitKey(1) & 0xFF == ord('q'):
            break
    cap.release()
    cv2.destroyAllWindows()
    speak("Camera closed.")
    

# System Control
def perform_task(command):
    if "chat mode" in command:
        chat_mode()
    elif "take note" in command:
        take_note()
    elif "read notes" in command:
        read_notes()
    elif "weather" in command:
        get_weather()
    elif "play music" in command:
        play_music()
    elif "face unlock" in command:
        face_unlock()
    elif "shutdown" in command:
        speak("Shutting down the system.")
        os.system("shutdown /s /t 1")
    elif "restart" in command:
        speak("Restarting the system.")
        os.system("shutdown /r /t 1")
    elif "open youtube" in command:
        speak("Opening YouTube")
        webbrowser.open("https://www.youtube.com")
    elif "open google" in command:
        speak("Opening Google")
        webbrowser.open("https://www.google.com")
    elif "open code" in command:
        speak("Opening VS Code")
        os.system("code")
    elif "open notepad" in command:
        speak("Opening Notepad")
        os.system("notepad")
    elif "time" in command:
        current_time = datetime.datetime.now().strftime("%I:%M %p")
        speak(f"The time is {current_time}")
    elif "date" in command:
        current_date = datetime.datetime.now().strftime("%d %B %Y")
        speak(f"Today is {current_date}")
    elif "who is" in command or "what is" in command:
        try:
            topic = command.replace("who is", "").replace("what is", "").strip()
            info = wikipedia.summary(topic, sentences=2)
            speak(info)
        except:
            speak("Sorry, I couldn't find that on Wikipedia.")
    elif "define" in command:
        word = command.replace("define", "").strip()
        define_word(word)
    elif "exit" in command or "stop" in command:
        speak("Goodbye Maggi!")
        exit()
    else:
        response = ask_gpt(command)
        speak(response)

# Start assistant
speak("Initializing Jarvis...")
wait_for_wake_word()
speak("Hello Maggi, I am your assistant. How can I help you?")

# Main Loop
while True:
    cmd = take_command()
    if cmd != "i didn't catch that":
        perform_task(cmd)
    else:
        speak(cmd)ned to simplify your day, respond instantly, and feel like a personal AI companion. It understands natural speech, answers questions, controls apps and devices, manages tasks, and automates routine actions. Just say “Hey Jarvis” to get help with anything from information to productivity.
