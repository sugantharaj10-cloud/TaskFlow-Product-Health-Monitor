# TaskFlow Product Health Monitor

A conversational, multi-agent voice AI system built with ElevenLabs Workflow Builder. Designed as an interactive analytics and health monitoring tool for Product Managers (PMs) and Technical Program Managers (TPMs), this system analyzes real-time SaaS health data across three dedicated analytical tracks: User Feedback & NPS, Support Tickets & Churn Risks, and Product Engagement & Conversion Metrics.   

---

## Live Demo
* **Interactive Voice Agent:** [Talk to TaskFlow Product Health Monitor](https://elevenlabs.io/app/agents/agents/agent_9501m4f7rapeferv390jsf8xnxtz?branchId=agtbrch_0601m4f7rcs6ep5b7hkaj6r83vv1)

---


## Overview & Architecture Pattern
Monolithic voice agents frequently fail when handling multi-faceted product analytics: large combined prompts lead to cross-domain hallucinations (e.g., citing support ticket churn figures when discussing customer NPS scores) and unfocused responses.   

The TaskFlow Product Health Monitor enforces strict knowledge base scoping and dynamic conversation orchestration:   
* **Strict Single-KB Scoping:** Unlike shared-context architectures, each specialist subagent receives only one specific data file. This creates strict information boundaries and eliminates cross-domain hallucination.   
* **Data-First Persona:** Responses adopt an analytical, data-first tone—leading directly with hard numbers, sample sizes, benchmarks, and dates rather than generic conversation filler.   
* **Bidirectional Cross-Routing:** Six backward/cross-routing edges allow users to seamlessly pivot between analytical tracks mid-call without getting stuck or restarting the conversation.   

---

## Architecture Diagram
The system consists of 5 nodes connected by 12 LLM condition edges (3 forward, 3 session termination, and 6 bidirectional cross-routing). 
```
                                  [ Start Node ]
                                         │
                                         ▼
                                [ 1. Greeter Node ]
                            (Intake Router — Zero KB)
                         /               │               \
           LLM Condition           LLM Condition           LLM Condition
                       /                 │                 \
                      ▼                  ▼                  ▼
     [ 2. User Feedback Analyst ] ◄──────────────► [ 4. Product Metrics Analyst ]
        (KB: User Feedback ONLY)   Cross-Routing     (KB: Product Metrics ONLY)
                     │        ▲                  ▲         │
                     │         \                /          │
                     │          \              /           │
       Cross-Routing │           ▼            ▼            │ Cross-Routing
                     │    [ 3. Support Ticket Analyst ]    │
                     │       (KB: Support Tickets ONLY)    │
                     │                   │                 │
                     └───────────────────┼─────────────────┘
                                         │ LLM Condition (Call Complete)
                                         ▼
                                [ 5. End Call Node ]
```
---
                                
## Analytical Domains & Scoped Knowledge Bases
All data models cover TaskFlow SaaS operations for January–February 2026.   

|Subagent Node | Knowledge Base File | Domain Scope & Sample Insights |
| --- | --- | --- |
| **Greeter Node** | None (Routing Only) | Directs callers to the right domain; never answers queries directly. |
| **User Feedback AnalystK** | KB_User_Feedback.txt  |  NPS trends (score: 38, down from 45), app store reviews (iOS 3.8★, Play Store 3.6★), 12 user interview findings, and top feature requests (Dark mode leads with 289 requests). |
| **Support Ticket Analyst** | KB_Support_Tickets.txt | Total tickets (4,218; +25% MoM), category breakdowns (Performance 24%, Mobile 19%), CSAT decline (78%), and high-risk enterprise accounts (28 accounts, $892K ARR at risk).|
| **Product Metrics Analyst** | KB_Product_Metrics.txt |  Revenue ($1.24M MRR, 94% NRR), engagement (28,400 DAU), conversion funnels (8.4% free-to-paid), cohort retention, and system latency (Dashboard p50 at 3.2s — Critical).  |
| **End** | None |  Terminates the call when user concludes the session. |
 

---

## Routing Conditions (12 LLM Edges)

### 1. Forward Routing from Greeter (3 Edges)


* **Greeter → User Feedback Analyst:** `"The user is asking about NPS, net promoter score, user feedback, app reviews, app store ratings, user interviews, user sentiment, feature requests, in-app feedback, or what users are saying."`
  
* **Greeter → Support Ticket Analyst:** `"The user is asking about support tickets, ticket volume, ticket categories, resolution times, CSAT, customer satisfaction scores, escalations, churn risk, at-risk accounts, or support team capacity."`
  
* **Greeter → Product Metrics Analyst:** `The user is asking about product metrics, DAU, WAU, engagement, feature usage, conversion rates, free-to-paid conversion, retention, cohort analysis, MRR, revenue, page load times, performance, error rates, uptime, or competitive win/loss data."`

## Bidirectional Cross-Routing (6 Edges)

* **User Feedback → Support Ticket Analyst:** `The user is now asking about support tickets, ticket volume, ticket categories, resolution times, CSAT, churn risk, at-risk accounts, or support team capacity instead of user feedback topics."`
  
* **User Feedback → Product Metrics Analyst:** `The user is now asking about product metrics, DAU, WAU, engagement, conversion rates, retention, MRR, revenue, page load times, performance, error rates, or competitive data instead of user feedback topics."`
  
* **Support Ticket → User Feedback Analyst:** `The user is now asking about NPS, user feedback, app reviews, user interviews, user sentiment, feature requests, or in-app feedback instead of support ticket topics."`
     
* **Support Ticket → Product Metrics Analyst:** `The user is now asking about product metrics, DAU, WAU, engagement, conversion rates, retention, MRR, revenue, page load times, performance, error rates, or competitive data instead of support ticket topics."`  

* **Product Metrics → User Feedback Analyst:** `The user is now asking about NPS, user feedback, app reviews, user interviews, user sentiment, feature requests, or in-app feedback instead of product metrics topics. "`
  
* **Product Metrics → Support Ticket Analyst:** `The user is now asking about support tickets, ticket volume, ticket categories, resolution times, CSAT, churn risk, at-risk accounts, or support team capacity instead of product metrics topics."`  

## Session Termination (3 Edges)

* **All Analysts → End Call:** `The user has indicated they are done, said goodbye, thanked the agent, or has no more questions."`  

## Verification & Testing Suite

|Test Scenario | User Utterance | Expected Agent Response |
| --- | --- | --- |
|1. User Feedback Routing | "What's our NPS looking like? | "Routes to User Feedback; reports NPS of 38 (down from 45 in Jan) across 1,842 responses, citing slow dashboard loading (34%) as the top detractor reason. |
|2. Support Ticket Routing | "Which enterprise accounts are we at risk of losing?" | Routes to Support Tickets; names top 5 accounts: Acme Corp ($120K), GlobalTech ($95K), Pinnacle Systems ($88K), DataDrive ($78K), Brightpath ($67K) with specific pain points. | 
|3. Product Metrics Routing | "How's our conversion funnel doing?" | Routes to Product Metrics; cites 62% activation, 34% aha moment, and 8.4% overall free-to-paid conversion (down from 11.2%), flagging trial performance drop-offs.|
|4. Dynamic Cross-Routing | Starts with "What are users saying about search?" then asks "How many support tickets are about search? | "Answers user sentiment first, then cross-routes seamlessly to Support Tickets to cite 534 tickets (13% of total) and Elasticsearch backlog status. |  
|5. Strict Boundary Enforcement |In Support Ticket track, asks "What's our MRR?" | Agent declines financial reporting and redirects caller to the Product Metrics track.  | 
