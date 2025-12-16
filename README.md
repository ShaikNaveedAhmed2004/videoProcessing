Async Video Processing System
Overview

This project implements an asynchronous video processing system that allows users to upload videos, request multiple output variants, track processing progress over time, and download completed outputs.

The system is designed to separate long-running video processing from HTTP requests, ensuring responsive APIs, persistent task state, and recoverable processing.

This project was built as part of the Video Forge Hackathon and focuses on system design, asynchronous processing, and explainability rather than media quality.

Tech Stack (Mandatory)

Frontend: React.js

Backend: Node.js (Express)

Database: PostgreSQL

High-Level Architecture
[ React Frontend ]
        |
        | HTTP (Upload / Fetch Status / Download)
        |
[ Node.js API Server ]
        |
        | Inserts tasks & state
        |
[ PostgreSQL Database ]
        |
        | Polling
        |
[ Background Worker ]

Key Principle

Uploading a file and processing a file are treated as separate responsibilities.

Functional Flow
1. Video Upload

User uploads a video (≤ 200 MB)

Supported formats: MP4, MOV, WebM

User selects one or more output variants

API returns immediately after task creation

2. Task Creation

Each output variant is stored as an independent task

Tasks are persisted in PostgreSQL with initial state QUEUED

3. Asynchronous Processing

A background worker continuously polls for QUEUED tasks

Tasks move through the lifecycle:

QUEUED → PROCESSING → COMPLETED / FAILED


Processing is simulated to represent long-running work

4. Frontend State Tracking

Frontend polls backend every few seconds

Displays:

Uploaded videos

Associated tasks

Real-time task state updates

State persists across page reloads

5. Output Download

Completed tasks expose a download link

Outputs are served directly by the backend

Task Model

Each processing task includes:

Unique task identifier (UUID)

Reference to uploaded video

Output format and resolution profile

Task state (QUEUED, PROCESSING, COMPLETED, FAILED)

Timestamps for state transitions

Error information (if failed)

Asynchronous Processing Design

Video processing never occurs inside HTTP request handlers

Background worker handles long-running tasks

Multiple tasks can be processed independently

Failure of one task does not affect others

Task state is persisted, enabling recovery after restarts

Persistence & Reliability

PostgreSQL is the single source of truth

Refreshing the frontend does not reset progress

Partial success is handled correctly

System remains stable during worker or task failures

Frontend Features

Upload videos with variant selection

View all uploaded videos

View all tasks per video

Observe task state changes over time

Download completed outputs

UI reflects real backend state only (no simulated data)

Setup Instructions
Backend
cd video-processing-backend
npm install
npm run dev


Start background worker:

node src/worker.js

Frontend
cd video-processing-frontend
npm install
npm start


Frontend runs at:

http://localhost:3000


Backend runs at:

http://localhost:5000

Why a Background Worker?

To prevent HTTP request blocking and ensure responsiveness, all long-running work is handled outside request handlers.

Why Polling?

Polling offers a simple, reliable mechanism for time-based state updates and aligns well with the persistence requirements of the system.

Failure Handling

Each task is isolated. Failures are captured per task and do not impact the system or other tasks.

Scalability Considerations

Multiple workers can be added

Database ensures task durability

Processing logic can be replaced with real transcoding without architectural changes

Limitations & Trade-offs

Video transcoding is simulated (media quality not evaluated)

Polling is used instead of WebSockets for simplicity

Single worker process (can be horizontally scaled)

These trade-offs were chosen to prioritize clarity, correctness, and explainability.

Future Improvements

Real video transcoding using FFmpeg

Distributed task queue (Redis / Kafka)

WebSocket-based real-time updates

Retry and cancellation support

Authentication and user isolation

Conclusion

This system demonstrates a correctly designed asynchronous backend, explicit task lifecycle modeling, persistent state management, and a frontend that accurately reflects backend state over time.

The implementation satisfies all functional, architectural, and reliability requirements of the problem statement.
