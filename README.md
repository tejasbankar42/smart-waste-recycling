# Smart-Waste-Collection-Recycling-Platform
The application should allow users to select a waste category, provide pickup information, submit a request, and track its status. An administrative dashboard should allow collection requests to be viewed and managed.


# ♻️ Smart Waste Collection & Recycling Platform

A modern web application designed to streamline waste disposal and```markdown
# ♻️ Smart Waste Collection & Recycling Platform

A full-stack web application designed to streamline municipal and private waste management. The platform enables households and businesses to schedule classified waste pickups, track request statuses in real-time, and provides waste operators with an administrative dashboard to manage routes and monitor recycling analytics.

---

## 🌟 Key Features

### For Residents & Businesses (User Portal)
* **Waste Categorization:** Choose from distinct categories such as Organic, Recyclable (Plastics, Paper, Metal), E-Waste, and Hazardous materials with built-in disposal guidelines.
* **Pickup Scheduling:** Select a preferred pickup date, time window, and enter accurate location details.
* **Request Tracking:** Real-time status indicators (`Pending`, `Scheduled`, `In Progress`, `Completed`, `Cancelled`).
* **Pickup History:** Log of past disposal requests to help users monitor their recycling habits.

### For Waste Operators (Admin Dashboard)
* **Request Management:** View, search, filter, and update the status of incoming collection requests.
* **Visual Analytics:** Interactive dashboards powered by Recharts showing waste collection breakdown by category, daily pickup volumes, and completion rates.
* **Search & Filters:** Filter pickups by status, category, date, or address for route planning.

---

## 🛠️ Tech Stack

* **Frontend:** React, Tailwind CSS, Lucide Icons, Axios
* **Data Visualization:** Recharts
* **Backend:** Python 3.10+, FastAPI, Pydantic, Uvicorn
* **Database:** SQLite with SQLAlchemy ORM

---

## 📂 Project Structure

```text
smart-waste-platform/
├── backend/
│   ├── app/
│   │   ├── main.py              # FastAPI entry point & routes
│   │   ├── models.py            # SQLAlchemy database models
│   │   ├── schemas.py           # Pydantic data schemas
│   │   └── database.py          # SQLite engine & session setup
│   ├── requirements.txt         # Backend Python dependencies
│   └── waste_management.db      # SQLite database file (generated)
├── frontend/
│   ├── src/
│   │   ├── components/          # Reusable UI components (Navbar, Cards, Modals)
│   │   ├── pages/               # UserForm, UserHistory, AdminDashboard
│   │   ├── App.jsx              # Main React router & layout
│   │   └── index.css            # Tailwind CSS directives
│   ├── package.json
│   └── tailwind.config.js
└── README.md
