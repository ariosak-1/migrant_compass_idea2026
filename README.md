# migrant_compass_idea2026# migrant compass

- professors:
   marica moreno-baez
   professor berges (?) academic lead
    
    https://acleddata.com/
    
    https://data.humdata.org/
    
    https://open-meteo.com/
    
    - **Database:** **PostgreSQL with the PostGIS extension**. It is built specifically to calculate distances between latitude/longitude points and your risk zones.
    - **Routing Logic:** **NetworkX (Python library)** to quickly build your paths and apply your risk scores as weights.
    - **AI Framework:** **LangChain or LlamaIndex**. These Python tools let you easily pass data from your PostgreSQL database directly into an OpenAI or Anthropic API model.
- 

**How the Data Flows: API to Code to Figma**

To make this work, **your app architecture requires a Backend Server** (built in Python with a framework like FastAPI or Flask). The code lives on this server, fetches data from the APIs, calculates the risk scores, and feeds them to your user interface (UX).

Here is exactly how the data moves from the open-source APIs all the way to the screen you designed in Figma.

---

**1. Connecting the APIs to Your Code**

Your Python server acts as a centralized scheduler. It uses an HTTP library called `requests` or `httpx` to ping ACLED, Open-Meteo, and UNOCHA. It pulls down their data, calculates the risk metrics, and serves them up in a clean format called **JSON**.

```
[ACLED / Weather APIs]
       │ (Raw JSON Data)
       ▼
[Your Python Backend Server] ──► Calculates Risk Scores & Weighted Paths
       │ (Your Custom API)
       ▼
[Your Live Mobile App] ──► Renders the Map & Chat Interface
```

For example, when a user opens the app, your code instantly triggers a web request to Open-Meteo like this behind the scenes:

**python**

```
importrequests# 1. Ask the weather API for data at specific migrant route coordinatesurl ="https://open-meteo.com"response = requests.get(url)weather_data = response.json()# 2. Your code evaluates the hazard leveltemperature = weather_data['hourly']['temperature_2m'][0]risk_points =0if temperature >40:risk_points =50# Add heavy penalty for extreme heat

print(f"Calculated Weather Risk:{risk_points}")
```

Use code with caution.

---

**2. Bridging the Figma UX with Your Live Code**

Figma is a static visual blueprint; it doesn't execute code or connect directly to live databases. To turn your Figma design into a real app, you or a developer will write **frontend code** (using React Native or Flutter) that mimics your Figma layers exactly, but swaps the placeholder text for your server's live data.

**How UI components translate from Figma to Code:**

- **The Route Map:** In Figma, you probably drew a solid colored line. In your real code, you will use a package like **Mapbox** or **Google Maps SDK**. Your Python server will send a list of low-risk latitude/longitude coordinates to the app, and Mapbox will draw that path live on the screen.
- **Risk Badges (Green / Yellow / Red):** In Figma, you designed static colored buttons. In code, you will write a conditional statement: `if risk_score > 70 { show RedBadge } else { show GreenBadge }`.
- **The AI Chat Interface:** In Figma, you designed text bubbles. In your real app frontend, you will link a typing input field to your backend server, which forwards the user's text to an AI model (like OpenAI) mixed with your calculated risk metrics, and returns the response dynamically.

---

**The Blueprint for Your App Stack**

To turn your Figma layout into an active app, you will plug these pieces together:

1. **Design (Frontend UX):** React Native or Flutter (built to look identical to your **Figma design**).
2. **App Logic (Backend Server):** **Python (FastAPI)** running on a cloud host (like Heroku, AWS, or Render).
3. **Data Crushing:** **PostgreSQL + PostGIS database** to store safe shelters, coordinates, and layer the incoming API hazards.
4. **AI Layer:** **OpenAI API** connected to your backend via **LangChain** to generate the multilingual guidance text.

Now that the data pipeline makes sense, how should we proceed? I can provide a **Python backend script that connects to an API and outputs data for your UI**, or we can review the specific **UI panels you have in Figma** to map out what data fields they r
the migrant compass application
