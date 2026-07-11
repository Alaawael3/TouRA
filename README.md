# TouRA — AI-Powered Smart Tourism Platform 🇪🇬

TouRA is an AI-powered smart tourism platform built to solve a real problem: tourists visiting Egypt are forced to juggle 3–4 disconnected apps to plan a single outing — one for ideas, one for maps, one for reviews, and separate, often untrustworthy channels for guides and transportation. TouRA unifies theme-based trip planning, a guide bidding marketplace, real-time map visualization, ratings/reviews, and two custom-built AI microservices into a single mobile experience.

This was my Graduation Project at the Faculty of Computers and Artificial Intelligence, Cairo University (Information Systems Department, June 2026).

> 📄 **Full Report:** [link to documentation]
> 🎞️ **Presentation Slides:** [link to presentation]
> 🎬 **Demo Video:** [link to demo video]

---

## 📌 Project Overview

TouRA lets a user pick a theme (romantic, family, friends, solo), a location, and a budget, and generates a complete, organized outing plan — instead of relying on generic popularity-based recommendations. On top of planning, TouRA layers:

- **Theme-based AI trip generation** with a live interactive map
- **A guide marketplace** where verified local guides bid on trips (InDrive-style)
- **Group trips**, live guide location sharing, and real-time chat via SignalR
- **Ratings & reviews** for guides and places
- **Two AI microservices** that are the technical core of the project (detailed below)

The backend is a modular .NET system (Users, Trips, Conversations, Notifications, Payments) built with Clean Architecture, a Transactional Outbox/Inbox for reliable inter-module messaging, and a database-per-module design. The mobile app is React Native + Expo, and the admin dashboard is React + TypeScript. These are covered briefly here — the focus of this README, and of my personal contribution to the project, is the **AI layer**.

---

## 🤖 The AI Features (My Focus)

TouRA's AI capabilities are built as two independent Python microservices, decoupled from the core .NET backend and accessed through thin HTTP clients. This let us iterate on the AI stack freely without touching the rest of the system, and swap in mock services during local development and testing.

### 1. Statue Storyteller — AI Monument Recognition & Narration

The Statue Storyteller lets a tourist snap a photo of a statue or monument and get an accurate, conversational historical narrative back — turning a silent museum visit into an interactive one.

**How it works:**
- **Custom image dataset** — since no reliable open dataset of Egyptian artifacts exists, I built one from scratch: 75 distinct monuments/statues, each with a minimum of 15 verified images (1,100+ images total), sourced from museum digital collections, archaeological databases, and tourism photography, then quality-filtered and augmented (brightness/contrast/color adjustments) to simulate real-world museum lighting conditions.
- **Visual understanding** — I evaluated CNN embeddings (ResNet-50), smaller CLIP variants, and custom Siamese networks before selecting **CLIP-ViT-L-14**, which projects both images and text into a shared semantic space rather than doing closed-set classification. On a 150-image validation set it hit **~87% top-1** and **~96% top-3** retrieval accuracy, clearly outperforming the alternatives (ResNet-50 ~62%, CLIP-ViT-B/32 ~78%).
- **Fallback recognition** — when an artifact isn't confidently matched in our own dataset, the agent falls back to a **Google Lens reverse-image search** tool.
- **Conversational agent** — built with **LangGraph** as a compiled `StateGraph`: a single model node (served via the **Groq SDK**, LLaMA) is bound to the Google Lens tool and decides mid-turn whether to call it or answer directly. Rather than keeping state in memory, every turn is rebuilt from PostgreSQL: the last 3 messages are loaded, converted into typed `HumanMessage`/`AIMessage` objects, and replayed through the graph — which keeps the service stateless and horizontally scalable.
- **Stack:** FastAPI, SQLAlchemy 2.x + psycopg3, LangGraph, Groq SDK, OpenRouter, DINOv2, Google Lens API, deployed on Azure Container Apps with an Azure-managed PostgreSQL instance.

### 2. AI Travel Planner — Multi-Agent Itinerary Generation

The AI Travel Planner generates realistic, personalized multi-day itineraries from a theme, destination, dates, and a candidate list of places — instead of the generic, popularity-ranked lists most competitors return.

**How it works:**
- **Multi-agent pipeline with CrewAI** — an **Events Discovery crew** (LLaMA-3.3-70B via Groq) searches the web through a Tavily tool for real, date-relevant concerts, festivals, and exhibitions, and produces a structured `EventsReport`. Its output feeds an **Itinerary Planner crew** (GPT-4o-mini via OpenRouter), which schedules attractions across days — grouping geographically close places together, minimizing backtracking, and keeping each day's schedule to a realistic 6–9 hours.
- **Realistic travel-time correction** — after the draft itinerary is produced, actual road travel times are pulled from **OSRM** to recalculate start/end times between stops; if OSRM is slow or unavailable, the service transparently falls back to a **Haversine-distance estimate** (assuming ~30 km/h average driving speed) so the pipeline never blocks on a routing failure.
- **Robust structured output** — results are validated against a Pydantic `TravelItinerary` schema, with a manual validation pass as a fallback for the cases where CrewAI's automatic structured parsing fails on an otherwise valid LLM response.
- **Stack:** FastAPI, Pydantic v2, CrewAI, Groq SDK, OpenRouter, Tavily, OSRM, deployed as a Docker container on Hugging Face Spaces.

### Why this design

Both services deliberately avoid hard dependence on any single paid vendor: LLM calls are split across Groq and OpenRouter, routing has a free-tier OSRM path with a pure-math fallback, and recognition has a dataset-first approach with an external-search fallback. This kept the AI stack fully functional on a $0 student budget while staying architecturally ready to swap in production-grade providers later (see the report's Future Work section).

---

## 🧱 System Architecture (Brief)

- **Backend:** Modular .NET (Clean Architecture) — Users, Trips, Conversations, Notifications, Payments modules, communicating through versioned Contracts and a Transactional Outbox/Inbox for reliable messaging.
- **Mobile app:** React Native + Expo, offline-first with local SQLite, real-time chat/notifications via SignalR.
- **Admin dashboard:** React + TypeScript + Vite, Redux Toolkit/RTK Query, Leaflet for maps.
- **AI microservices:** Statue Storyteller & AI Travel Planner (see above) — the two components I designed, built, and evaluated.

Full diagrams (Use Case, ERD, UML, Sequence, Backend Architecture) are in the report linked above.

---

## 🛠️ Tech Stack Summary

| Layer | Technologies |
|---|---|
| AI — Recognition | CLIP-ViT-L-14, DINOv2, Google Lens, LangGraph, Groq (LLaMA) |
| AI — Trip Planning | CrewAI, Groq (LLaMA-3.3-70B), OpenRouter (GPT-4o-mini), Tavily, OSRM |
| AI Services Backend | FastAPI, SQLAlchemy 2.x, Pydantic v2 |
| Core Backend | ASP.NET Core, EF Core, SignalR, Clean Architecture |
| Mobile | React Native, Expo, Expo Router, React Query, Zustand, SQLite |
| Admin Dashboard | React, TypeScript, Vite, Redux Toolkit, Leaflet |
| Infra | Docker, Azure Container Apps, Azure PostgreSQL, Hugging Face Spaces, MinIO |

---

## 👥 Team

Alaa Wael Mohamed · Yussuf Mohammad Taha · Omar Osama Hassan · Anas Abdelnasser Ibrahim · Mohammad Rabea · Hager Hassan

Under the supervision of Dr. Ibrahim Gomaa, Dr. Asmaa Ahmed, and TA. Nehal Akram — Faculty of Computers and Artificial Intelligence, Cairo University.

---

## 📎 Links

- 📄 Full Report: [add link]
- 🎞️ Presentation: [add link]
- 🎬 Demo Video: [add link]
