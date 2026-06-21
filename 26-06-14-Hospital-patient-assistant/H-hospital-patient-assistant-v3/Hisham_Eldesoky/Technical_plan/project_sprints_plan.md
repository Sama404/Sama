# Hospital Patient Assistant - Modules and Sprints Plan

This document breaks down the Hospital Patient Assistant v3 project into independent modules. Each module is divided into actionable sprints with clear development steps, including example inputs and outputs.

---

## Module 1: Core Infrastructure & Orchestration (MVP Phase 1)
**Goal:** Set up the foundation, patient database, and the Orchestrator Agent that routes incoming traffic.

### Sprint 1.1: Project Setup & Database Initialization
*   **What to develop:** Initialize the FastAPI backend, set up PostgreSQL for the Patient EMR (Electronic Medical Record), and establish basic API routes.
*   **How to implement:** Use Python with FastAPI and SQLAlchemy for database connections. Define the `Patient` and `Session` database tables.
*   **Input:** API request to create a new patient profile.
*   **Output:** `{"patient_id": "123", "status": "created"}`

### Sprint 1.2: Orchestrator Agent (Routing & Intent)
*   **What to develop:** An agent that classifies user input to decide if the patient wants to schedule an appointment or report a medical complaint.
*   **How to implement:** Use a fast LLM (e.g., Claude Haiku or GPT-4o-mini) strictly for intent classification.
*   **Input:** `{"raw_message": "I need to book an appointment with a cardiologist."}`
*   **Output:** `{"request_type": "scheduling", "routed_to": "Scheduling_Agent"}`

---

## Module 2: Scheduling System (MVP Phase 2)
**Goal:** Allow patients to book, reschedule, or find alternative doctors easily.

### Sprint 2.1: Doctor Availability Database
*   **What to develop:** A database schema and operations for tracking doctor schedules and departments.
*   **How to implement:** Create tables in PostgreSQL for `Doctors`, `Departments`, and `TimeSlots`.
*   **Input:** Query for available Cardiology slots on Oct 12th.
*   **Output:** `[{"slot_id": "A1", "time": "10:00 AM", "doctor": "Dr. Smith"}]`

### Sprint 2.2: Scheduling Agent
*   **What to develop:** A conversational agent that fetches available slots and confirms bookings with the user.
*   **How to implement:** Give the LLM access to database tool-calling (functions to search and reserve slots).
*   **Input:** `{"request_text": "Book Dr. Smith at 10 AM", "patient_id": "123"}`
*   **Output:** `{"booking_confirmation": {"appointment_id": "899", "status": "confirmed"}}`

---

## Module 3: Triage & Symptom Intake (MVP Phase 3)
**Goal:** Safely collect medical symptoms and calculate the urgency of the patient's condition.

### Sprint 3.1: Symptom Intake Agent
*   **What to develop:** An agent that extracts structured data from the patient's natural language descriptions (and potentially images).
*   **How to implement:** Use a vision-capable LLM instructed to output strict JSON containing the body part, symptom, and duration.
*   **Input:** `{"complaint_text": "My chest has been hurting terribly for 30 minutes."}`
*   **Output:** `{"structured_symptoms": [{"symptom": "chest pain", "duration": "30 mins", "severity": "high"}]}`

### Sprint 3.2: Triage Agent (Deterministic Rules)
*   **What to develop:** A rule-based engine (NOT an AI LLM) that calculates the Emergency Severity Index (ESI).
*   **How to implement:** Write strict Python logic that checks for red-flag keywords (like "chest pain" or "stroke") to immediately assign an urgency level.
*   **Input:** `{"structured_symptoms": [{"symptom": "chest pain"}]}`
*   **Output:** `{"urgency_level": 1, "status": "Life-threat"}`

---

## Module 4: Clinical Reasoning & Safety (MVP Phase 4)
**Goal:** Generate grounded medical advice and pass it through strict safety checks before showing it to the user.

### Sprint 4.1: Diagnosis Reasoning Agent
*   **What to develop:** An agent that combines patient history, symptoms, and medical guidelines (RAG) to suggest the right medical department.
*   **How to implement:** Use a strong reasoning model (e.g., Claude 3.5 Sonnet) and connect it to a Vector DB (Qdrant) containing medical guidelines.
*   **Input:** `{"structured_symptoms": [{"symptom": "fever"}], "emr_history": "History of asthma"}`
*   **Output:** `{"candidate_department": "Pulmonology", "confidence": 0.85}`

### Sprint 4.2: Clinical Safety Check Gate
*   **What to develop:** A strict validation step to ensure the agent's diagnosis doesn't hallucinate or prescribe dangerous contraindications.
*   **How to implement:** Use a deterministic Python script that cross-references patient allergies and medications with the agent's response.
*   **Input:** `{"draft_diagnosis": "Take Ibuprofen", "patient_allergies": ["NSAIDs"]}`
*   **Output:** `{"pass_fail": "Fail", "contraindication_flags": ["NSAID allergy detected"]}`

---

## Module 5: Emergency & Automation (MVP Phases 5, 6, 7)
**Goal:** Handle severe emergencies securely and run background hospital processes like feedback collection.

### Sprint 5.1: Emergency Agent
*   **What to develop:** A system that triggers ambulance dispatch and clinician alerts in parallel if Triage scores it a Level 1.
*   **How to implement:** Use Python async tasks. Start a countdown timer; if a human clinician does not override it, trigger the dispatch API.
*   **Input:** `{"urgency_level": "Life-threat", "patient_id": "123"}`
*   **Output:** `{"dispatch_decision": "timed", "timer_seconds": 60, "clinician_alerted": true}`

### Sprint 5.2: Background Agents (Loyalty & Feedback)
*   **What to develop:** Agents that act independently after appointments are finished to award points and check on patients.
*   **How to implement:** Use background task queues (Redis Queue/Celery). Loyalty points run immediately after booking; Feedback runs via a cron job 24h later.
*   **Input:** `{"event": "booking_completed", "patient_id": "123"}`
*   **Output:** `{"updated_loyalty_balance": 150}`
