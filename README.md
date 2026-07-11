<div align="center">

# 🏛️ TouRA
### AI-Powered Smart Tourism Platform for Egypt

*Turning fragmented, chaotic trip planning into one intelligent, end-to-end experience.*

[![Report](https://img.shields.io/badge/📄-Full%20Report-blue?style=for-the-badge)](#)
[![Presentation](https://img.shields.io/badge/🎞️-Presentation-orange?style=for-the-badge)](#)
[![Demo](https://img.shields.io/badge/🎬-Demo%20Video-red?style=for-the-badge)](#)

Graduation Project · Faculty of Computers and Artificial Intelligence, Cairo University · June 2026

</div>

---

## ✨ Overview

Planning a trip to Egypt today means juggling three or four disconnected apps — one for ideas, one for maps, one for reviews, and a separate, often unreliable channel just to find a guide. **TouRA** replaces that chaos with a single platform: users pick a **theme** (romantic, family, friends, solo), a location, and a budget, and get back a fully organized, AI-generated plan on an interactive map — with the ability to bid-hire a verified local guide, chat in real time, and rate everything afterward.

Beyond the planning layer, TouRA's signature feature is turning a **silent museum visit into an interactive one**: point a phone at a statue, and an AI agent identifies it and tells its story.

This README focuses on the part of the system I personally designed and built — **the two AI microservices** — while briefly covering the rest of the platform for context.

---

## 🤖 The AI Features — My Core Contribution

TouRA's intelligence is delivered through two independent Python microservices, fully decoupled from the core backend and reached through thin HTTP clients. This isolation let me iterate on the AI stack freely and swap in mocks during development, without the rest of the system ever needing to know what was happening underneath.

### 🗿 Statue Storyteller — AI Monument Recognition & Narration

A tourist photographs a statue or monument; the system identifies it and responds with an accurate, conversational historical narrative.

| Component | Details |
|---|---|
| **Dataset** | Built from scratch — no reliable open dataset of Egyptian artifacts exists. 75 distinct monuments, 15+ verified images each (1,100+ images), sourced from museum collections and archaeological archives, then quality-filtered and augmented for realistic lighting variation. |
| **Visual model** | Evaluated ResNet-50, CLIP-ViT-B/32, and custom Siamese networks before selecting **CLIP-ViT-L-14** — chosen because it embeds images *and* text into a shared semantic space instead of doing closed-set classification. Result: **~87% top-1 / ~96% top-3** retrieval accuracy on a 150-image validation set, versus ~78% (smaller CLIP) and ~62% (ResNet-50). |
| **Fallback recognition** | When the in-house dataset can't confidently match an artifact, the agent calls a **Google Lens reverse-image search** tool instead. |
| **Conversational engine** | A compiled **LangGraph** `StateGraph`: one model node (served by the **Groq SDK**, LLaMA) bound to the Google Lens tool, deciding mid-turn whether to call it or answer directly. The service is fully **stateless** — every turn is reconstructed from PostgreSQL (last 3 messages → typed message objects → replayed through the graph), making it easy to scale horizontally. |
| **Stack** | FastAPI · SQLAlchemy 2.x + psycopg3 · LangGraph · Groq SDK · OpenRouter · DINOv2 · Google Lens API · Azure Container Apps · Azure PostgreSQL |

### 🗺️ AI Travel Planner — Multi-Agent Itinerary Generation

Generates realistic, personalized multi-day itineraries from a theme, destination, dates, and candidate places — instead of the generic, popularity-ranked lists competitors return.

| Component | Details |
|---|---|
| **Multi-agent pipeline** | Built with **CrewAI**. An **Events Discovery crew** (LLaMA-3.3-70B via Groq) uses a Tavily web-search tool to find real, date-relevant concerts, festivals, and exhibitions. Its output feeds an **Itinerary Planner crew** (GPT-4o-mini via OpenRouter), which schedules attractions per day — grouping nearby places together, minimizing backtracking, and keeping each day to a realistic 6–9 hours. |
| **Realistic travel times** | After the draft itinerary is built, real road travel times are pulled from **OSRM** to recompute stop timings. If OSRM is slow or down, the service silently falls back to a **Haversine-distance estimate** (~30 km/h average), so routing failures never block a response. |
| **Reliable structured output** | Results are validated against a Pydantic `TravelItinerary` schema, with a manual validation fallback for the rare cases where CrewAI's automatic parsing fails on an otherwise valid model response. |
| **Stack** | FastAPI · Pydantic v2 · CrewAI · Groq SDK · OpenRouter · Tavily · OSRM · Docker on Hugging Face Spaces |

### 💡 Design Philosophy

Both services avoid hard dependence on any single paid vendor: LLM calls are split across Groq and OpenRouter, routing has a free-tier path with a pure-math fallback, and recognition pairs an in-house dataset with an external-search fallback. This kept the entire AI stack running on a **$0 student budget** while staying architecturally ready to plug in production-grade providers later.

---

## 🧱 System Architecture (Brief)

The AI services sit alongside a modular platform I collaborated on with my team:

- **Backend** — Modular .NET (Clean Architecture): Users, Trips, Conversations, Notifications, Payments — communicating through versioned contracts and a Transactional Outbox/Inbox for reliable messaging.
- **Mobile app** — React Native + Expo, offline-first with local SQLite, real-time chat & notifications via SignalR.
- **Admin dashboard** — React + TypeScript + Vite, Redux Toolkit/RTK Query, Leaflet.

Full diagrams (Use Case, ERD, UML, Sequence, Backend Architecture) are available in the [full report](#).

---

## 🛠️ Tech Stack

<table>
<tr><td><b>AI — Recognition</b></td><td>CLIP-ViT-L-14 · DINOv2 · Google Lens · LangGraph · Groq (LLaMA)</td></tr>
<tr><td><b>AI — Trip Planning</b></td><td>CrewAI · Groq (LLaMA-3.3-70B) · OpenRouter (GPT-4o-mini) · Tavily · OSRM</td></tr>
<tr><td><b>AI Services Backend</b></td><td>FastAPI · SQLAlchemy 2.x · Pydantic v2</td></tr>
<tr><td><b>Core Backend</b></td><td>ASP.NET Core · EF Core · SignalR · Clean Architecture</td></tr>
<tr><td><b>Mobile</b></td><td>React Native · Expo · Expo Router · React Query · Zustand · SQLite</td></tr>
<tr><td><b>Admin Dashboard</b></td><td>React · TypeScript · Vite · Redux Toolkit · Leaflet</td></tr>
<tr><td><b>Infrastructure</b></td><td>Docker · Azure Container Apps · Azure PostgreSQL · Hugging Face Spaces · MinIO</td></tr>
</table>

---

## 👥 Team

Alaa Wael Mohamed · Yussuf Mohammad Taha · Omar Osama Hassan · Anas Abdelnasser Ibrahim · Mohammad Rabea · Hager Hassan

**Supervised by:** Dr. Ibrahim Gomaa · Dr. Asmaa Ahmed · TA. Nehal Akram
Faculty of Computers and Artificial Intelligence, Cairo University

---

<div align="center">

📄 [Full Report](#) &nbsp;·&nbsp; 🎞️ [Presentation](#) &nbsp;·&nbsp; 🎬 [Demo Video](#)

</div>
