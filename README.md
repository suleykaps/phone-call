# phone-call

JSON tabanlı bir veritabanı oluşturmak, özellikle küçük çaplı SaaS projeleri için SQLite gibi ilişkisel veritabanlarına hafif ve hızlı bir alternatif sağlar. Bu yapıyı, kullanıcının bilgilerini ve kredi bakiyesini JSON dosyasında tutarak sadeleştirebiliriz.

Aşağıda, JSON tabanlı bir veritabanı mimarisini nasıl oluşturacağınızı ve entegre edeceğinizi gösteren örnek kodlar yer alıyor.

---

## **1. Proje Yapısında Güncellemeler**

Yeni dosya yapısı:

```
/phonecall_saas
│
├── /app.py               --> Ana Streamlit uygulaması
├── /services
│     ├── __init__.py
│     ├── payment.py      --> Iyzico entegrasyonu
│     ├── auth.py         --> Kimlik doğrulama
│     └── credits.py      --> Kredi işlemleri
├── /db
│     ├── __init__.py
│     └── json_db.py      --> JSON veri yönetimi
└── /data
      └── database.json   --> JSON veritabanı dosyası
└── requirements.txt
```

---

## **2. JSON Tabanlı Veritabanı Yapısı**

`data/database.json` adlı bir dosya oluşturun. JSON dosyasının başlangıç yapısı:

```json
{
    "users": []
}
```

---

## **3. `json_db.py` - JSON Veri Yönetimi**  

Aşağıdaki kod, kullanıcı verilerini JSON dosyasından okur, yazar ve günceller.

```python
import json
import os

DB_PATH = "./data/database.json"

def read_db():
    """Veritabanını okur ve Python dict olarak döner."""
    if not os.path.exists(DB_PATH):
        with open(DB_PATH, 'w') as f:
            json.dump({"users": []}, f)
    with open(DB_PATH, 'r') as f:
        return json.load(f)

def write_db(data):
    """Python dict'i JSON veritabanına yazar."""
    with open(DB_PATH, 'w') as f:
        json.dump(data, f, indent=4)

def find_user(email):
    """Belirli bir e-posta ile kullanıcıyı bulur."""
    db = read_db()
    for user in db["users"]:
        if user["email"] == email:
            return user
    return None

def add_user(email, password):
    """Yeni bir kullanıcı ekler."""
    db = read_db()
    if find_user(email) is None:
        new_user = {"email": email, "password": password, "credits": 0}
        db["users"].append(new_user)
        write_db(db)
        return new_user
    return None

def update_credits(email, amount):
    """Kullanıcının kredilerini günceller."""
    db = read_db()
    for user in db["users"]:
        if user["email"] == email:
            user["credits"] += amount
            write_db(db)
            return user["credits"]
    return None
```

---

## **4. `auth.py` - Kimlik Doğrulama Servisi**

Bu servis, kullanıcı girişini ve kaydını JSON veritabanını kullanarak yönetir.

```python
from db.json_db import find_user, add_user

def login(email, password):
    user = find_user(email)
    if user and user["password"] == password:
        return user
    return None

def register_user(email, password):
    return add_user(email, password)
```

---

## **5. `credits.py` - Kredi Yönetimi**

Kullanıcı kredilerini görüntüleme ve kullanma işlevlerini içerir.

```python
from db.json_db import find_user, update_credits

def get_user_credits(email):
    user = find_user(email)
    return user["credits"] if user else 0

def use_credits(email, amount):
    user = find_user(email)
    if user and user["credits"] >= amount:
        update_credits(email, -amount)
        return True
    return False
```

---

## **6. `app.py` - Ana Streamlit Uygulaması**

Ana arayüzde, kimlik doğrulama ve kredi kullanımı işlevlerini JSON veritabanına entegre ettik.

```python
import streamlit as st
from services.auth import login, register_user
from services.credits import get_user_credits, use_credits

if 'user' not in st.session_state:
    option = st.sidebar.selectbox("Login or Register", ["Login", "Register"])

    if option == "Login":
        email = st.sidebar.text_input("Email")
        password = st.sidebar.text_input("Password", type="password")
        if st.sidebar.button("Login"):
            user = login(email, password)
            if user:
                st.session_state.user = user
                st.success("Login successful!")
            else:
                st.error("Login failed.")
    elif option == "Register":
        email = st.sidebar.text_input("Email")
        password = st.sidebar.text_input("Password", type="password")
        if st.sidebar.button("Register"):
            user = register_user(email, password)
            if user:
                st.success("Registration successful!")
            else:
                st.error("User already exists.")

if 'user' in st.session_state:
    user = st.session_state.user
    st.sidebar.write(f"Welcome, {user['email']}")

    credits = get_user_credits(user["email"])
    st.sidebar.write(f"Remaining Credits: {credits}")

    phone_number = st.text_input("Enter Phone Number")
    if st.button("Make Call"):
        if use_credits(user["email"], 1):
            st.success(f"Calling {phone_number}...")
        else:
            st.error("Insufficient credits.")
```

---

## **7. Gereken Paketler (`requirements.txt`)**

```plaintext
streamlit
```

---

## **8. Uygulamanın Çalıştırılması**

Aşağıdaki komut ile uygulamayı çalıştırın:

```bash
streamlit run app.py
```

---

## **Sonuç**

Bu yapı, **JSON tabanlı bir veritabanı** ile kullanıcı oturumu ve kredi yönetimini sağlayan modüler bir SaaS mimarisi sunar. Böylece veriler ilişkisel bir veritabanı yerine hafif JSON dosyasında saklanır, bu da hızlı geliştirme ve kolay bakım sağlar.

Herhangi bir sorunla karşılaşırsanız yardımcı olmaktan memnuniyet duyarım!


