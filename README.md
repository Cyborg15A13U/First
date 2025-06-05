#!/usr/bin/env python
# coding: utf-8

import os
import sys
import requests
import hashlib
import subprocess
import tempfile

# === ✅ 自動アップデート設定 ===
REMOTE_URL = "https://example.com/your_script.py"  # ← あなたがアップロードする場所に変更！
LOCAL_PATH = os.path.abspath(__file__)

def get_sha256(content):
    return hashlib.sha256(content).hexdigest()

def download_latest():
    try:
        resp = requests.get(REMOTE_URL)
        resp.raise_for_status()
        return resp.content
    except Exception as e:
        print(f"[!] ダウンロード失敗: {e}")
        return None

def compare_and_replace():
    remote = download_latest()
    if not remote:
        return False

    with open(LOCAL_PATH, "rb") as f:
        local = f.read()

    if get_sha256(remote) != get_sha256(local):
        print("[↑] 新しいバージョンが見つかりました。更新します。")

        # 一時ファイルに保存してから上書き
        with tempfile.NamedTemporaryFile(delete=False, mode='wb') as tmp:
            tmp.write(remote)
            tmp_path = tmp.name

        os.replace(tmp_path, LOCAL_PATH)

        # 再実行
        print("[▶] 更新済みのプログラムを再実行します。")
        subprocess.run([sys.executable, LOCAL_PATH])
        sys.exit(0)
    else:
        print("[✓] 最新版です。更新不要。")
        return False

# === ✅ アップデート処理開始 ===
compare_and_replace()

# === ✅ ↓↓↓ 以下、本体処理 ↓↓↓ ===

from selenium import webdriver
from selenium.webdriver.common.by import By
import time
import ctypes
import re
import random

SAVE_PATH_FILE = "save_acountpath.txt"

def save_initial_path():
    if not os.path.exists(SAVE_PATH_FILE):
        ID = input("アカウントのユーザー名を入力してください　： 　")
        Pass = input("アカウントのパスワードを入力してください　：　")
        with open(SAVE_PATH_FILE, 'w', encoding='utf-8') as file:
            file.write(ID + '\n')
            file.write(Pass + '\n')
        print(f"Mailアドレスとパスワードが {SAVE_PATH_FILE} に保存されました。")
    else:
        with open(SAVE_PATH_FILE, 'r', encoding='utf-8') as file:
            lines = file.readlines()
            ID = lines[0].strip()
            Pass = lines[1].strip()
        print(f"保存されている　Mailアドレスを使用します: {ID}")
        print(f"保存されている　パスワード　を使用します: {Pass}")
    return ID, Pass

save_path, password = save_initial_path()

attributes = ctypes.windll.kernel32.GetFileAttributesW(SAVE_PATH_FILE)
ctypes.windll.kernel32.SetFileAttributesW(SAVE_PATH_FILE, attributes | 0x02)
print(f"{SAVE_PATH_FILE} を隠しファイルに設定しました。")

# Chromeを起動
driver = webdriver.Chrome()
driver.get("https://leap-me.com/ja/app/text-generator")
time.sleep(3)

driver.find_element(By.XPATH, '//*[@id="input-100"]').send_keys("今日の感想　（できたこと / 感じたこと / もっとこうしたら良かったこと）　ＩＴ関連のＳｃｒａｔｃｈやＩＴパスポートの勉強をしました。 #### できたこと")
driver.find_element(By.XPATH, '//*[@id="input-102"]').send_keys("日誌")
driver.find_element(By.XPATH, '//*[@id="toolForm"]/div[5]/div[2]/label[3]/div').click()
driver.find_element(By.XPATH, '//*[@id="convertButton"]').click()
time.sleep(10)

markdown_text = driver.find_element(By.XPATH, '//*[@id="responseContainer"]').text
match = re.search(r"####\s*(出来たこと|できたこと)\s*(.+?)(?=####|\Z)", markdown_text, re.DOTALL)
if match:
    extracted_text = match.group(2).strip()
    print("#### 出来たこと\n" + extracted_text)
else:
    print("#### 出来たこと セクションが見つかりませんでした。")

# ログイン処理
driver.get("https://yumeske.sanko.ac.jp/")
time.sleep(5)
driver.find_element(By.XPATH, '/html/body/div/button').click()
time.sleep(3)
driver.find_element(By.XPATH, '//*[@id="identifierId"]').send_keys(save_path)
time.sleep(1)
driver.find_element(By.XPATH, '//*[@id="identifierNext"]/div/button/span').click()
time.sleep(3)
driver.find_element(By.XPATH, '//*[@id="password"]/div[1]/div/div[1]/input').send_keys(password)
time.sleep(1)
driver.find_element(By.XPATH, '//*[@id="passwordNext"]/div/button/span').click()
time.sleep(5)
driver.find_element(By.XPATH, '/html/body/div/div[1]/div/div[3]/button[2]').click()
time.sleep(1)
driver.get("https://yumeske.sanko.ac.jp/diary/")
time.sleep(2)
driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")
time.sleep(2)

xpaths = [
    '/html/body/div/div[3]/div[last()]/div/form/div[1]/div/div/div[1]/span/span',
    '/html/body/div/div[3]/div[last()]/div/form/div[1]/div/div/div[2]/span/span',
    '/html/body/div/div[3]/div[last()]/div/form/div[1]/div/div/div[3]/span/span'
]
selected_xpath = random.choice(xpaths)
driver.find_element(By.XPATH, selected_xpath).click()
driver.find_element(By.XPATH, '/html/body/div/div[3]/div[last()]/div/form/div[2]/div/textarea').send_keys(extracted_text)
driver.find_element(By.XPATH, '/html/body/div/div[3]/div[last()]/div/form/div[4]/div[2]').click()
time.sleep(3)
driver.quit()
