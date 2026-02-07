# 🎓 Campus Grievance Portal (Smart Redressal System)

> A modern, AI-assisted mobile application for handling campus maintenance issues with transparency, accountability, and automated workflows.

![Status](https://img.shields.io/badge/Status-In%20Development-orange)
![Platform](https://img.shields.io/badge/Platform-iOS%20%7C%20Android-black)
![Stack](https://img.shields.io/badge/Tech-React%20Native%20%7C%20Expo%20%7C%20Supabase-blue)



## 📖 About The Project

The **Campus Grievance Portal** is designed to replace outdated manual complaint registers with a smart digital workflow. It connects Students, Maintenance Staff, and Administration (VC/Deans) in a single loop to ensure campus facilities are maintained efficiently.

Unlike standard ticketing systems, this project implements **behavioral engineering** to solve common bottlenecks:
* **"Zombie Tickets":** Tickets auto-close after 48 hours of resolution if the student doesn't dispute (Passive Acceptance).
* **"Lazy Estimates":** Staff cannot set arbitrary completion times; requests over a specific threshold (e.g., 72 hours) require Supervisor approval.
* **"Blind Routing":** AI suggests the correct department (Electrical, Civil, Sanitation) based on the image uploaded.

---

## 🚀 Key Features

### 👨‍🎓 For Students
* **📸 Snap & Report:** Upload a photo of the issue. The system auto-detects the category (e.g., *Fan* -> *Electrical*).
* **📍 Precision Location:** Dropdown-based location selection (Zone -> Floor -> Room) to prevent vague reports.
* **🛡️ Privacy:** Anonymized reporting (Authority sees the issue location but not the student's personal details unless necessary).
* **🔔 Live Updates:** Real-time notifications when status changes or proof is uploaded.

### 👷 For Authorities (Staff)
* **⏱️ SLA Management:** Set estimated time for repairs. Timers are tracked publicly.
* **✅ Proof of Work:** Mandatory "Resolved" photo upload to mark a ticket as complete.
* **🤚 Threshold Approval:** Time extension requests beyond the standard cap require Warden/Dean approval.

### 🏛️ For Administration (VC Dashboard)
* **📊 Efficiency Heatmap:** Visualizes which hostels or departments are the slowest.
* **🔥 Escalation Matrix:** Auto-flagging of disputed, overdue, or repeated tickets.
* **🏆 Leaderboard:** Gamified stats to show the most efficient departments.

---

## 🛠️ Technical Architecture

### Frontend
* **Framework:** [React Native](https://reactnative.dev/) with [Expo Router](https://docs.expo.dev/router/introduction/).
* **Language:** TypeScript / JavaScript.
* **UI Library:** React Native Elements / Tailwind CSS (NativeWind).

### Backend
* **Database:** [Supabase](https://supabase.com/) (PostgreSQL).
* **Authentication:** Supabase Auth (Email & Password / Magic Link).
* **Storage:** Supabase Storage (Buckets for Grievance Photos & ID Cards).
* **Edge Functions:** (Planned) For AI Image Classification logic.

---

## 🧠 Business Logic & Workflows

### 1. The "Passive Acceptance" Protocol
To prevent tickets from staying "Pending" forever because students forget to verify them:
1.  Staff marks ticket **Resolved** and uploads a "Proof Photo".
2.  Ticket Status changes to `resolved_pending`.
3.  A **48-Hour Timer** starts.
4.  User receives a notification.
    * If User clicks **Dispute**: Ticket reopens and is flagged to Admin.
    * If User does nothing: Ticket **Auto-Closes** after 48 hours (Successful).

### 2. The Time Threshold (SLA)
To prevent staff from setting safe, long deadlines to avoid work:
1.  Staff accepts a ticket.
2.  Staff enters `Estimated Completion Time`.
3.  **Logic Check:**
    * If `Time` < `72 Hours`: Timer starts immediately.
    * If `Time` > `72 Hours`: System halts. Request sent to `Supervisor` for approval. Timer starts only after approval.

---

## ⚙️ Getting Started

Follow these steps to set up the project locally.

### Prerequisites
* **Node.js** (LTS version) installed.
* **Expo Go** app installed on your physical Android/iOS device.
* A **Supabase** account (Free tier is sufficient).

### Installation

1.  **Clone the Repository**
    ```bash
    git clone [https://github.com/your-username/campus-grievance-portal.git](https://github.com/your-username/campus-grievance-portal.git)
    cd campus-grievance-portal
    ```

2.  **Install Dependencies**
    ```bash
    npm install
    # OR
    npx expo install
    ```

3.  **Environment Setup**
    Create a `.env` file in the root directory:
    ```env
    EXPO_PUBLIC_SUPABASE_URL=your_supabase_project_url
    EXPO_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key
    ```

4.  **Run the App**
    ```bash
    npx expo start
    ```
    *Scan the QR code displayed in the terminal using the Expo Go app.*

---

## 🗄️ Database Setup (Supabase)

Copy and paste the following SQL into your Supabase **SQL Editor** to create the necessary tables and policies.

```sql
-- 1. Enable UUID Extension
create extension if not exists "uuid-ossp";

-- 2. Create Profiles Table (Linked to Auth)
create table public.profiles (
  id uuid references auth.users not null primary key,
  full_name text,
  role text check (role in ('student', 'staff', 'admin')),
  reg_number text unique,
  department text, -- Null for students
  avatar_url text
);

-- 3. Create Grievances Table
create table public.grievances (
  id uuid default gen_random_uuid() primary key,
  user_id uuid references public.profiles(id),
  title text,
  description text,
  category text, -- e.g., 'electrical', 'sanitation', 'civil'
  photo_url text, -- The initial photo of the issue
  location_zone text, -- e.g., 'Hostel 1'
  location_detail text, -- e.g., 'Room 203'
  status text default 'pending' check (status in ('pending', 'in_progress', 'resolved_pending', 'closed', 'disputed')),
  created_at timestamp with time zone default now(),
  assigned_to uuid references public.profiles(id),
  
  -- Resolution & Timers
  resolution_proof_url text, -- Photo of the fix
  estimated_completion timestamp with time zone,
  actual_completion timestamp with time zone,
  requires_approval boolean default false
);

-- 4. Create Storage Buckets
insert into storage.buckets (id, name, public) values ('grievance-photos', 'grievance-photos', true);
insert into storage.buckets (id, name, public) values ('resolution-proofs', 'resolution-proofs', true);

-- 5. Enable Row Level Security (RLS)
alter table public.profiles enable row level security;
alter table public.grievances enable row level security;

-- (Basic Policy: Everyone can read grievances, only owners can edit their own)
create policy "Public grievances are viewable by everyone" 
  on public.grievances for select using (true);

create policy "Users can insert their own grievances" 
  on public.grievances for insert with check (auth.uid() = user_id);

🤝 Contributing
Contributions are what make the open-source community such an amazing place to learn, inspire, and create. Any contributions you make are greatly appreciated.

Fork the Project.

Create your Feature Branch (git checkout -b feature/AmazingFeature).

Commit your Changes (git commit -m 'Add some AmazingFeature').

Push to the Branch (git push origin feature/AmazingFeature).

Open a Pull Request.

📄 License
Distributed under the MIT License. See LICENSE for more information.

📞 Contact
Project Team - [Your Email Here]

Project Link: https://github.com/your-username/campus-grievance-portal