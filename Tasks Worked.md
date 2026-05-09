# UnQuery

## May 2025

1. **Worked on API Spec for Import Dataset and implemented below APIs:**
    
    1. **test_connection**
        
    2. **add-dataset**
        
    3. **update-dataset**
        
    4. **list_datasources**
        
    5. **list_tables from source**
        
    6. **dataset list**
        
    7. **dataset delete**  
        (1 May 2025 – 4 May 2025)
        
2. **Integrated below APIs in UI:**
    
    1. test_connection
        
    2. add-dataset
        
    3. update-dataset
        
    4. list_datasources
        
    5. list_tables from source
        
    6. dataset list  
        (6 May 2025 – 10 May 2025)
        
3. **Enabled bulk dataset metadata creation** (11–12 May 2025)
    
4. Dataset metadata edit and reload bug fix (12 May 2025)
    
5. Imported datasets render without requiring a page reload in home (13 May 2025)
    
6. **Pagination for table data in query responses** (14 May 2025)
    
7. Displayed error message when adding a dataset fails (14–15 May 2025)
    
8. Users and User_session schema (15 May 2025)
    
9. Most recently accessed sessions not showing at top of sidebar (Reload issue pending) (16 May 2025)
    
10. Persistent issue in feedback and tested all APIs end-to-end from UI (19–20 May 2025)
    
11. return_state queries not updating in query_history (22 May 2025)
    
12. **Implemented Chat Page** & Chat route for chat page (26–27 May 2025)
    
13. Loader issue (Analyzing loader) showing in all pages (28 May 2025)
    
14. Thread issues in conversations (29 May 2025)
    

---

## June 2025

1. UI: Fixed thread grouping issue for follow-up and standalone questions in conversations (home and chat windows)
    
2. UI: Home Page and Publishing page label changes:
    
    1. Added Clear Dataset Selection instead of Sidebar
        
    2. Used Summary instead of Explanation during Publishing
        
    3. Removed 4 hardcoded selections in Query Interface
        
3. Fixed message loader issue for follow-up questions
    
4. UI: Unified logic for Bar, Line, and Pie charts by creating a reusable chart component
    
5. Dashboard: Single chart occupies full width instead of one column
    
6. Dataset Import: User lands on Edit page instead of View page
    
7. UI: Chat box supports more text
    
8. Conversation fix: Showing latest message instead of first message
    
9. Loader fix: Display error message and stop loader on failure
    
10. Enforced mandatory context at dataset level in metadata page
    
11. Auto-generated pre-canned queries for datasets in UI
    
12. Option to choose/remove columns from dataset
    
13. **API to import/export dataset metadata from different environments**
    
14. Intent Fix:
    

- Used intent message instead of validate message
    
- Persistent Edit Page after tab switching
    
- Select all button for bulk export
    
- Render error messages before page refresh
    
- Show proper error when data source connection is lost
    
- Fixed dashboard filters
    
- Pie chart rendering for large data
    

15. Get more input from user if intent requires additional information
16. **Improved and redesigned the visual presentation of Pie Charts, Bar Charts, and Line Charts for a more modern and intuitive user experience**
17. **Implemented animated chat responses and restructured the response flow to enhance clarity, engagement, and overall user experience.**
    

---

## July 2025

1. UI Enhancements:
    
    - Chart resizable fix in dashboard
        
    - Dark mode UI issue (text visibility)
        
    - Tab switching issue in add-dataset
        
    - Published queries rendering issue
        
    - Dashboard filters fix
        
    - SQL query rendering for error messages
        
2. Worked on review comments for refactoring Postgres integration
    
3. **CSV upload feature with backend API integration**
    
4. **UI integration for CSV upload**
    
5. CSV dataset query testing
    
6. Mark dataset as private by default with option to make public
    
7. UI Fix: Blank page when selecting new history item without refresh
    
8. **Query limits:**
    
    - **Max 10 queries/day and 50 total**
        
    - **Store usage data, block after limit, reset daily**
        
9. Auto cleanup script:
    
    - Drop tables older than 7 days
        
    - Schedule job to run daily
        
10. Integration testing for end-to-end validation
    

---

## August 2025

1. UI Bug Fixes and Enhancements:
    
    1. Proper dataset visualization page
        
    2. "Select All" button for multiple columns
        
    3. Dataset dropdown in playground
        
    4. Fix file errors persisting after deselection
        
    5. Playground query rendering fix
        
    6. Timestamp inconsistency fix
        
    7. Dataset description overflow fix
        
    8. Share button updates in playground
        
    9. Configure Filter & Visibility button fixes with permissions
        
    10. Handle large data in charts (arrow keys for line/bar, dropdown for pie)
        
    11. Notify unused buttons:
        
        - Dynamic chart radio button
            
        - Mic button beside search
            
        - Export button in conversation
            
        - Share/export/publish buttons in dashboard
            
        - Publish button in playground
            
2. **'Guest' user role for unQuery login access**
    
3. **UI: Home page enhancements and display of query/import limits**
    
4. Improved query results UI (animation + metadata)
    
5. Setup datasets on Azure server
    
6. Sample queries issue fix
    
7. **Configurable UI themes for Agriculture and Transportation projects**
    

---

# Opsight

## September 2025

1. **Prod-Monitoring-Agent: UI and backend implementation** 

# Finternet

## September 2025

1. **Mock APIs for GFF**
    
2. **Workflow API layer implementation**
    
3. Integration of workflow and engine services for token create and transact APIs
    

---

## October 2025

1. Explore and setup Cerbos authorization with Go
    
2. Design access policies and token management using Cerbos
    
3. API spec design for Identity & Account APIs with schemas
    
4. Dockerization for Actors and Credentials APIs
    
5. API implementation for credentials
    

---

## November 2025

1. Explore DIG integration and usage
    
2. POC on OTEL generation and ingestion to ClickStack
    
3. **Emit OTEL from UNITS services**
    

---

## December 2025

1. **Finternet App – Initial version**
    
2. Units service bug fixes
    
3. Update Login & Create Account APIs to OTP-based flow
    
4. **Finternet app backend with onboarding APIs**
    

---

## January 2026

1. Update Postman collection and API specs (refactored login & create account flow)
    
2. **Implement initial version application-wide React routing**
    
3. Refactor finternet app module folder structure
    
4. Bug fixes
    
5. **Transaction lifecycle APIs**
    
6. Refactor token, transaction, and token_transaction search APIs
    
7. **Audit log generation and push events to Kafka**
    
8. **Token lifecycle APIs**
    

---

## February 2026

1. Address & name hashing in Token Engine
    
2. Address & name hashing in frontend
    
3. Address & name hashing in API layer
    
4. **Integrate KYC with Signzy**
    
5. **OTEL integration for app backend**
    
6. Error handling fixes (replace panics with proper errors)
    
7. **Register API with JWT from OTP service**
    
8. **Issue JWT upon verification (asymmetric keys)**
9. **Otel logs, traces and metrics implementation for units and app-backend**
    

---

## March 2026

1. Composite logic support
    
2. Bug fixes
    
3. Login & dashboard behavior fixes
    
4. OTEL metrics for app backend
    
5. OTEL metrics for API layer

---








1. Finternet — Built Token, TokenClass, Transaction & Token-Transaction APIs from scratch in Units, developed the initial version of Finternet PWA with React-based routing and integrated account APIs covering account creation to login flow

2. Finternet — Implemented OTEL logs, traces and metrics across Units, Notification Service and App Backend for production readiness, enabled secure JWT-based authentication with asymmetric keys, and integrated Signzy for KYC-driven user onboarding with address & name hashing across all layers

3. Finternet & Opsight — Implemented the Workflow API layer, integrated workflow and engine services for token create and transact APIs, and contributed to the initial Opsight Production Monitoring Agent (UI and backend APIs).

4. unQuery — Implemented end-to-end dataset management including connection testing, CRUD operations, bulk metadata creation, column selection, pre-cannexd query generation and integrated all APIs into UI with improved query handling, pagination and response rendering

5. unQuery — Added PostgreSQL support, implemented CSV data ingestion with backend and UI integration, introduced query access control with daily limits and usage tracking, and delivered animated chat conversations with enhanced UI/UX across dashboard and dataset flows



Project / Initiative: Finternet — Units Service & App Development
Role Played: Full-stack development (Backend APIs & Frontend Application)

Key Deliverables:

* Implemented Token, TokenClass, Transaction, and Token-Transaction lifecycle APIs from scratch within the Units service
* Built event publishing for token and transaction operations to Kafka, enabling audit trails
* Enhanced Identity and Account APIs with well-defined schemas
* Refactored token, transaction, and search APIs to improve consistency and maintainability
* Developed the initial version of the Finternet PWA with application-wide React-based routing
* Implemented key UI screens and components aligned with core user flows
* Integrated account APIs to support the complete user journey from account creation to login

Business Impact:

* Delivered essential API components required for core financial operations on the Finternet platform

Metrics / Evidence:

* PWA live and accessible with a complete onboarding flow



Project / Initiative: Finternet — Observability, Security & Onboarding
Role Played: Backend Development & Cross-Service Integration

Key Deliverables:

* Implemented OpenTelemetry (OTEL) logs, traces, and metrics across Units, Notification Service, and App Backend
* Built JWT issuance using asymmetric keys upon OTP verification for secure authentication
* Integrated KYC workflow with Signzy for regulatory-grade identity verification during onboarding
* Applied consistent address and name hashing across frontend, API layer, and Token Engine

Business Impact:

* Enhanced system observability, enabling faster debugging and proactive issue detection in production
* Strengthened authentication and onboarding flows with secure and compliant mechanisms

Metrics / Evidence:

* OTEL instrumentation live and actively collecting logs, traces, and metrics across multiple services
* End-to-end KYC and JWT-based authentication flow fully functional
* Hashing consistently implemented across frontend, API, and Token Engine layers


Project / Initiative: Finternet GFF (Global Fintech Fest) & Opsight — Production Monitoring
Role Played: Backend Development & Full-stack Contribution

Key Deliverables:

* Implemented the Workflow API layer for the Finternet GFF use case
* Integrated workflow and engine services for token create and transact APIs
* Worked for the initial version of the Opsight Production Monitoring Agent (UI and backend)
* Implemented backend APIs to support the monitoring agent

Business Impact:

* Enabled structured workflow execution for token operations during the GFF demo

Metrics / Evidence:

* Workflow and engine service integration fully functional for token create and transact flows


Project / Initiative: unQuery — Dataset Management & Query Interface
Role Played: Full-stack Development (Backend APIs & UI Integration)

Key Deliverables:

* Implemented dataset APIs covering connection testing, add, update, list, and delete operations
* Integrated all dataset APIs end-to-end into the UI with robust error handling and user feedback
* Enabled bulk dataset metadata creation and cross-environment metadata export/import
* Implemented user convenient content generation api
* Auto-generated pre-canned queries for datasets within the UI
* Improved query handling with pagination and enhanced response rendering for tabular data

Business Impact:

* Simplified dataset onboarding and reduced manual effort in configuration
* Enhanced query experience and overall usability for end users

Metrics / Evidence:

* All dataset APIs are fully functional and successfully integrated into the UI


Project / Initiative: unQuery — Data Ingestion, Access Control & UI Enhancements
Role Played: Full-stack Development (Backend & UI)

Key Deliverables:

* Implemented CSV upload functionality with backend processing and seamless UI integration
* Enabled querying capabilities on uploaded CSV datasets
* Introduced query limits (10/day, 50 total) with usage tracking, enforcement, and automated daily reset
* Implemented animated chat responses with a restructured conversation flow for improved clarity
* Delivered UI/UX enhancements across chat, dashboard, and dataset workflows
* Introduced a Guest user role to enable broader platform access without full registration
* Built configurable UI themes for Agriculture and Transportation verticals

Business Impact:

* Improved user engagement through intuitive UI and animated, structured chat interactions

Metrics / Evidence:

* CSV ingestion and querying fully functional end-to-end
* Query limit system operational with automated daily reset


---

Over the course of the year, I consistently delivered assigned features end-to-end across Finternet and unQuery projects — spanning backend API implementation, service integrations and frontend development. From building core financial APIs in Finternet to integrating dataset management workflows in unQuery, I made sure to deliver functional and tested outputs while continuously learning and improving along the way.


I took ownership of the modules assigned to me and made sure to follow through on every task with commitment. Whenever I encountered challenges or had doubts, I proactively reached out to my leads for guidance, discussed the approach and ensured I was aligned before proceeding. I also took responsibility for flagging issues early and working collaboratively to resolve them rather than leaving them unaddressed.



I actively collaborated with my team and leads throughout the year — whether it was discussing implementation approaches, seeking clarification on requirements or aligning on API contracts between services. 

I made sure to communicate progress and blockers in a timely manner and worked well within the team to ensure smooth integration across backend and frontend layers. I regularly discussed technical approaches with my team, shared updates when needed and ensured my work was aligned with the overall direction of the project.



Throughout the year, I approached challenges by breaking them down and finding practical solutions with the support of my team and leads. In Finternet, working with Golang was itself a significant learning curve, and over time I progressively improved my debugging skills by identifying, tracing and resolving issues across services. I conducted POCs to evaluate the feasibility of new tools before full integration, which helped in making more informed implementation decisions.

I also explored and integrated new tools and frameworks such as OTEL for observability instrumentation, Signzy for KYC integration and ClickStack for centralized log monitoring. Additionally, I worked with tools like Vault for secrets management, Keycloak for identity and access management and MinIO for object storage — each of which required me to go beyond my existing knowledge and ramp up quickly. These experiences pushed me to think critically, adapt to unfamiliar technical landscapes and apply my learnings effectively within the project context.



I kept the end user experience in mind while working on both Finternet and unQuery. In unQuery, I contributed to improvements such as animated chat conversations, configurable UI themes for specific industry verticals, guest user access and query limit controls — all of which directly improved the usability and accessibility of the product for end users. In Finternet, I worked on user-facing flows such as account creation, login and KYC-driven onboarding as guided by my leads, ensuring the implemented flows were functional and aligned with the expected user experience.



Over the review period, I contributed across Finternet and unQuery — working on both backend and frontend aspects of the projects. I took ownership of my assigned modules, sought guidance from my leads when needed and ensured functional deliverables. I also explored and worked with several new tools and frameworks along the way, which helped me grow technically throughout the year.


---

1. Grow Technically
2. DevOps & Infrastructure Exposure

Goal: Continuously learn and adapt to new technologies and improve problem solving skills. Achievement: Gained hands-on experience with Golang, observability frameworks and several new tools while working across multiple projects.


Goal: Learn and get hands-on with DevOps practices and tools.
Achievement: Gained understanding in Docker and basics Kubernetes, worked on Dockerization of services in unQuery and Finternet. Advanced Kubernetes concepts are still in progress

---


Ownership & Follow-through — Took full ownership of assigned modules, actively collaborated with the team to discuss and resolve challenges, and ensured deliverables were functional and aligned with expectations

Quick Learning & Adaptability — Rapidly ramped up on new technologies and tools, effectively applying them within the project context

Communication — Proactively communicated progress and blockers, sought timely clarifications, and ensured alignment with leads before proceeding

Problem Solving — Tackled challenges by breaking them down systematically, debugging effectively, and seeking guidance when required

Consistency Across Multiple Projects — Maintained consistent contributions across Finternet, unQuery, and Opsight throughout the review period

Full Stack Contribution — Contributed across both backend and frontend layers, adapting to project needs without being limited to a single domain


---

Advanced DevOps & Infrastructure — Looking to go deeper into Kubernetes and explore more advanced infrastructure practices.

System Design Thinking — Develop a stronger understanding of system design principles, particularly around Low Level Design (LLD) and High Level Design (HLD).

AI Tools & Integration — AI tools are already part of day-to-day workflow, but there is still a lot to explore and learn in this space.


---

Deepen Technical Expertise in Finternet — Continue contributing to the Finternet platform by taking on more complex modules and strengthening understanding of the overall system

Advance DevOps & Infrastructure Knowledge — Build beyond foundational knowledge by gaining hands-on experience with advanced Kubernetes

Strengthen System Design Skills — Actively learn and apply Low-Level Design (LLD) and High-Level Design (HLD) concepts to better understand and contribute to architectural decisions

Leverage AI Tools More Effectively — Deepen understanding and practical usage of AI tools within the development workflow to improve productivity and efficiency

---


I explored and experimented with AI models and tools to improve my development workflow. I tried multiple models such as Claude Sonnet, Gemini and Opus to understand their capabilities and identify which worked best for different use cases. I also explored AI-powered development tools like Cursor, Antigravity and Claude Code, taking guidance from my leads on how to effectively adopt them.

One of the challenges I faced was setting up Claude Code, which required troubleshooting and guidance from my leads to get it working correctly. Through these experiences, I learned how to better leverage AI tools for code generation, debugging and speeding up day-to-day development tasks. Overall, these tools have become a useful part of my workflow and I continue to explore ways to use them more effectively.

---
