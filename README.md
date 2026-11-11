<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:EAF6EC,100:38A347&height=200&section=header&text=CropSense&fontSize=64&fontColor=14361A&fontAlignY=38&desc=plant%20health%2C%20in%20the%20farmer's%20own%20language&descSize=18&descAlignY=58&descColor=1E4D25" width="100%" alt="CropSense banner"/>

**IoT soil sensors + a leaf-photo disease model + practical advice, in English, हिन्दी, ಕನ್ನಡ, தமிழ் and తెలుగు.**

![JavaScript](https://img.shields.io/badge/Vanilla%20JS-F7DF1E?style=for-the-badge&logo=javascript&logoColor=111)
![Firebase](https://img.shields.io/badge/Firebase-RTDB%20%7C%20Firestore-FFCA28?style=for-the-badge&logo=firebase&logoColor=111)
![Flask](https://img.shields.io/badge/Flask-disease%20model-38A347?style=for-the-badge&logo=flask&logoColor=white)
![i18n](https://img.shields.io/badge/languages-5-14361A?style=for-the-badge)

</div>

---

## 🌱 The problem

A smallholder farmer notices spots on his tomato leaves. Is it early blight? Too much water? Not enough sun? The agronomist is far away, most farm apps are in English, and a wrong guess costs a week of crop and money spent on the wrong spray.

## 🌿 What CropSense does

Meet *Ramesh*, the farmer the dashboard is designed around. He opens CropSense and gets:

| | Feature | How it works |
|---|---|---|
| 📡 | **Live field readings** | Soil moisture, temperature, humidity and light stream in from field sensors through **Firebase Realtime Database**. |
| 🚨 | **Plain-language alerts** | Threshold rules turn numbers into actions: *"Moisture at 28%, below the safe 35%. Water in the morning."* |
| 📷 | **Leaf-photo diagnosis** | He uploads a leaf photo, a **Flask image-classification model** (`/predict`) returns the disease and a confidence score, and the app turns that into care steps. |
| 🛒 | **What to buy & how to use it** | Suggested treatments with where to buy them and how to apply them, for each disease. |
| 💰 | **Cost estimator** | A weekly input budget based on the diagnosis, which he can save or share on **WhatsApp**. |
| 📝 | **Did it work?** | After treatment he logs the outcome and an "after" photo. CropSense shows a **trend chart** and a **before/after comparison**. |
| 🗣️ | **His language** | The whole interface switches between **5 languages** (115 translated UI strings) with proper Indic fonts. |

## 🧭 Architecture

```mermaid
flowchart LR
    subgraph Field
      S["ESP32 sensor node<br/>moisture, temp, humidity, light<br/>+ stress class"]
    end
    S --> R[("Firebase<br/>Realtime DB")]
    R -->|"live readings"| UI["CropSense dashboard<br/>(single-page web app)"]
    UI -->|"leaf photo"| ML["Flask disease model<br/>POST /predict"]
    ML -->|"disease + confidence"| UI
    UI -->|"advice history"| FS[("Cloud Firestore")]
    UI -->|"treatment logs + photos"| IDB[("IndexedDB<br/>on the device")]
```

## 🧠 Engineering decisions

- **It works when the model doesn't.** If the model server can't be reached, CropSense falls back to the **stress class computed on the ESP32 sensor node** and still gives advice instead of breaking. Rural connectivity is unreliable, so this was a must.
- **Moved logs from `localStorage` to IndexedDB.** Photo-heavy logs quickly hit `localStorage`'s ~5 MB limit. I moved storage to **IndexedDB** and wrote a **one-time migration** so existing users kept their history.
- **A bug fix worth telling.** Saving got stuck on *"Saving…"* because Firebase functions imported in an ES module weren't visible to the classic script. I tracked it down to a silent `ReferenceError` and exposed the functions on `window`.
- **i18n without a framework.** Every label carries a `data-i18n` key, and one `T(key)` lookup falls back to English if a string is missing.
- **No build step.** One HTML file and Firebase from the CDN, so it runs anywhere you can open a browser.

## 🚀 Run it

```bash
git clone https://github.com/Stanvi09/CropSense.git
cd CropSense
python -m http.server 8000     # open http://localhost:8000
```

**Bring your own backend:**
1. Create a Firebase project and paste its web config into the `firebaseConfig` block in `index.html`.
2. Start your plant-disease model server and set `window._modelAPIUrl` to its URL (it must accept a multipart `image` at `POST /predict` and return `{ raw_label, confidence }`, where `raw_label` is a PlantVillage-style class such as `Tomato_Early_blight`).

## 📁 Project structure

```
CropSense/
├── index.html      # the whole app: UI, styles, translations, alert rules,
│                   # Firebase wiring, IndexedDB log store, charts
└── README.md
```

## 🛣️ Roadmap

- [ ] Add the model-training notebook and Flask server to this repo
- [ ] Split `index.html` into `css/`, `js/` and `i18n/` files
- [ ] Offline mode (service worker) for patchy rural networks
- [ ] Voice read-out of advice for farmers who prefer listening

---

<div align="center">

**Tanvi Sadashiv**
<br/>
<a href="https://www.linkedin.com/in/tanvi-sadashiv-3713ba381"><img src="https://img.shields.io/badge/LinkedIn-connect-0A66C2?style=flat-square&logo=linkedin" alt="LinkedIn"/></a>
<a href="https://github.com/Stanvi09"><img src="https://img.shields.io/badge/GitHub-Stanvi09-181717?style=flat-square&logo=github" alt="GitHub"/></a>

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:38A347,100:EAF6EC&height=100&section=footer" width="100%"/>

</div>
