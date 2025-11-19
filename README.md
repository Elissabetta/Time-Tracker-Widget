# Time Tracker Widget: Single-File, Multi-Mode Activity Logger

## Project Overview

This project implements a lightweight, front-end **Activity Time Tracker Widget**. It is designed as a single-file, zero-dependency solution leveraging standard HTML, CSS (via Tailwind CSS), and vanilla JavaScript.

The primary goal is to provide users with a clean, low-latency interface for real-time tracking of various activities, offering flexible data persistence options suitable for both secure cloud environments and local development/desktop use.

---

## Architecture and Persistence Layer

The widget operates on a **Single-Page Application (SPA) principle** where all rendering logic is handled client-side via JavaScript functions (`renderUI`, `renderActivityButtons`, etc.) to minimize DOM manipulation overhead.

### Persistence Modes

The application dynamically switches its persistence layer based on the runtime environment:

1.  **Cloud Mode (Primary):** When the application detects valid Firebase configuration (`__firebase_config`), it initializes **Firebase Firestore**. Data is stored across multiple public documents (`/artifacts/{appId}/public/data/trackerState` for configuration and active state, and `/artifacts/{appId}/public/data/activityLogs` for history). This mode provides **real-time synchronization** and multi-user capability.
2.  **Local Mode (Fallback):** If Firebase configuration is unavailable or empty, the application falls back to **Browser LocalStorage**. All application state (activities, logs, totals, active session) is serialized to a single JSON object and persisted locally, ensuring functionality in isolated environments.

### Data Synchronization

The application relies on the `onSnapshot` listener (in Cloud Mode) or explicit `saveLocalState` calls (in Local Mode) to maintain data integrity.

---

## Core Features and Functionality

### 1. Low-Latency, Reactive UI

A dedicated function, `renderUI()`, is called immediately after every state mutation (e.g., starting or stopping an activity).

* **Solution for Instant Feedback:** Unlike conventional models that wait for database confirmation before updating the UI, this widget employs a **"client-first" rendering model**. Upon user interaction, the internal `state` object is updated, and `renderUI()` is executed *before* the asynchronous database save (`saveState()`). This ensures **zero perceived latency** for the user, regardless of network conditions.

### 2. Real-Time Dynamic Timer

When an activity is active (`state.activeActivity` is not null), the `startTimer()` function initializes a `setInterval` that executes `updateActiveTimerDisplay()` every 1000ms. This provides a continuously updating, granular timer display, reflecting the session's duration in real-time.

### 3. Activity Management

Users can dynamically add, rename, and delete activities via a dedicated management panel. The system is resilient to deletion: if an active activity is deleted, the running session is automatically stopped, logged, and the accumulated duration is added to the running total before the activity is removed from the master list.

### 4. Comprehensive Data Management (Backup/Restore)

* **Export:** Logs are exported to a standard CSV file, preserving all core metrics: `activityId`, `activityName`, `startTime`, `endTime`, `durationMs`.
* **Import (Restore Mechanism):** The CSV import functionality serves as a full-state restore utility. The system reads the CSV, **updates the master list of activities**, **replaces the current log history**, and then **re-calculates all running cumulative totals** based exclusively on the imported log entries. This ensures a clean and complete data recovery from a backup file.

---

## Technology Stack

| Component | Technology | Role |
| :--- | :--- | :--- |
| **Markup & Structure** | HTML5 | Application scaffolding. |
| **Styling** | Tailwind CSS | Utility-first framework for responsive, modern UI design. |
| **Logic & UI Rendering** | Vanilla JavaScript (ES6+) | State management, timer logic, DOM manipulation, and persistence handling. |
| **Persistence (Cloud)** | Firebase Firestore | Real-time NoSQL database for multi-user synchronization. |
| **Persistence (Local)** | LocalStorage API | Local browser-based data persistence fallback. |

This architecture ensures high performance and versatility, making the widget suitable for both collaborative cloud environments and individual desktop use cases.
