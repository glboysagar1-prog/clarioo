# Data Flow Diagrams (DFD) - Nano Banana Pro Edition 🍌

This document outlines the flow of data through the **Clarioo** system, featuring high-fidelity process maps for advanced system components.

## 0. Level 0: Context Diagram
The highest-level view of the system ecosystem.

```mermaid
graph TD
    %% External Entities
    Student[User / Student]
    Mentor[Mentor]
    Google[Google Auth / APIs]
    LLM[Gemini AI Model]
    Bank[Payment Gateway]
    Supabase[Supabase Platform]

    %% System
    System((Clarioo Platform))

    %% Data Flows
    Student -->|Credentials / Profile Data| System
    System -->|Dashboard / Content| Student
    
    Mentor -->|Availability / Session Notes| System
    System -->|Booking Requests / Schedule| Mentor
    
    System -->|Auth Tokens / Calendar Sync| Google
    Google -->|User Data / Events| System
    
    System -->|Prompts (Roadmap/Quiz)| LLM
    LLM -->|Generated Content (JSON)| System
    
    Student -->|Payment Details| System
    System -->|Transaction Request| Bank
    Bank -->|Confirmation| System
    
    System <-->|Realtime Subscriptions| Supabase
```

## 1. Level 1: System Breakdown
Core functional processes interacting with data stores.

```mermaid
graph LR
    %% Entities
    User[User]
    DB[(Supabase DB)]
    Cache[(Redis Cache)]
    AI_Svc[AI Service]

    %% Processes
    P1((1.0 Auth & \nOnboarding))
    P2((2.0 Mentorship \nManagement))
    P3((3.0 AI Tools \nEngine))
    P4((4.0 Video \nCommunication))
    P5((5.0 Job \nTracking))

    %% Flows
    User -->|Sign In| P1
    P1 -->|Validate| DB
    P1 -->|Session Token| User

    User -->|Book Session| P2
    P2 -->|Check Availability| DB
    P2 -->|Notify| User

    User -->|Request Roadmap| P3
    P3 -->|Check Cache| Cache
    P3 -->|Generate| AI_Svc
    AI_Svc -->|Result| P3
    P3 -->|Save| DB
    P3 -->|Content| User

    User -->|Join Room| P4
    P4 -->|Signaling| DB
    P4 -->|Media Stream| User
    
    User -->|Update Job Status| P5
    P5 -->|Update Record| DB
```

## 2. Detailed Data Flows (Level 2)

### 2.1 Authentication Process
OAuth + Onboarding logic.

```mermaid
sequenceDiagram
    participant User
    participant Client as Frontend Client
    participant Auth as Supabase Auth
    participant DB as Users Table

    User->>Client: Click "Continue with Google"
    Client->>Auth: OAuth Request (Scope: profile, email)
    Auth-->>Client: Redirect to Google
    User->>Auth: Authenticate & Authorize
    Auth-->>Client: Session Token (JWT)
    
    Client->>DB: Query User (by Email)
    alt User Exists
        DB-->>Client: Return User Profile
    else New User
        Client->>User: Show Onboarding Form
        User->>Client: Submit Focus/College
        Client->>DB: INSERT New User Record
        DB-->>Client: Confirmation
    end
    
    Client-->>User: Redirect to /home
```

### 2.2 AI Roadmap Generation Flow
Complex AI generation with Redis caching strategy.

```mermaid
flowchart TD
    Start([User Requests Roadmap]) --> Valid{Has Credits?}
    Valid -- No --> Error([Show "Upgrade" Prompt])
    Valid -- Yes --> CheckCache{Check Redis Cache}
    
    CheckCache -- Hit --> FetchCache[Fetch JSON from Redis]
    FetchCache --> Render
    
    CheckCache -- Miss --> Construct[Construct Prompt with \nUser Context]
    Construct --> CallAI[Call Google Gemini API]
    CallAI --> Parse[Parse JSON Response]
    Parse --> SaveDB[(Save to 'Roadmaps' Table)]
    SaveDB --> CacheStore[Store in Redis \n(TTL: 24h)]
    CacheStore --> Deduct[Deduct User Credits]
    Deduct --> Render([Render Interactive Roadmap])
```

### 2.3 Mentor Booking Lifecycle
State machine for session management.

```mermaid
stateDiagram-v2
    [*] --> Browsing
    Browsing --> Requested: Student Books Slot
    Requested --> Accepted: Mentor Approves
    Requested --> Rejected: Mentor Declines
    
    Accepted --> Scheduled: Calendar Event Created
    Scheduled --> InProgress: Session Starts
    
    InProgress --> Completed: Call Ends
    Completed --> Reviewed: Student leaves Rating
    
    Rejected --> [*]
    Reviewed --> [*]
```

### 2.4 Video Call Signaling (WebRTC)
Real-time peer connection establishment.

```mermaid
sequenceDiagram
    participant PeerA as User A (Caller)
    participant Signal as Supabase Realtime
    participant PeerB as User B (Callee)

    PeerA->>Signal: SUBSCRIBE to Room Channel
    PeerB->>Signal: SUBSCRIBE to Room Channel
    
    PeerA->>PeerA: Create Offer (SDP)
    PeerA->>Signal: PUBLISH 'offer' payload
    Signal->>PeerB: BROADCAST 'offer'
    
    PeerB->>PeerB: Set Remote Desc (Offer)
    PeerB->>PeerB: Create Answer (SDP)
    PeerB->>Signal: PUBLISH 'answer' payload
    Signal->>PeerA: BROADCAST 'answer'
    
    PeerA->>PeerA: Set Remote Desc (Answer)
    
    Note right of PeerA: ICE Candidate Exchange
    PeerA->>Signal: PUBLISH 'ice-candidate'
    Signal->>PeerB: BROADCAST 'ice-candidate'
    PeerB->>Signal: PUBLISH 'ice-candidate'
    Signal->>PeerA: BROADCAST 'ice-candidate'
    
    PeerA<-->>PeerB: P2P Media Stream Established 🎥
```

### 2.5 Job Application Tracker
Kanban board logic flow.

```mermaid
graph TD
    UserInput[User Drags Job Card] --> Detect[Detect Drop Column]
    Detect --> Validate[Validate New Stage]
    Validate --> Optimistic[Optimistic UI Update]
    
    Optimistic --> API[Call Update API]
    API --> DB[(Update 'job_tracker' Table)]
    
    DB -- Success --> Sync[Sync Store State]
    DB -- Error --> Revert[Revert UI Change]
    Revert --> Toast[Show Error Notification]
```
