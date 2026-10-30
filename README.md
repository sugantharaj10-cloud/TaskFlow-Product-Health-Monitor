# TaskFlow Product Health Monitor

A conversational, multi-agent voice AI system built with ElevenLabs Workflow Builder. Designed as an interactive analytics and health monitoring tool for Product Managers (PMs) and Technical Program Managers (TPMs), this system analyzes real-time SaaS health data across three dedicated analytical tracks: User Feedback & NPS, Support Tickets & Churn Risks, and Product Engagement & Conversion Metrics.   Live DemoInteractive Voice Agent: Talk to TaskFlow Product Health Monitor   Overview & Architecture PatternMonolithic voice agents frequently fail when handling multi-faceted product analytics: large combined prompts lead to cross-domain hallucinations (e.g., citing support ticket churn figures when discussing customer NPS scores) and unfocused responses.   The TaskFlow Product Health Monitor enforces strict knowledge base scoping and dynamic conversation orchestration:   Strict Single-KB Scoping: Unlike shared-context architectures, each specialist subagent receives only one specific data file. This creates strict information boundaries and eliminates cross-domain hallucination.   Data-First Persona: Responses adopt an analytical, data-first tone—leading directly with hard numbers, sample sizes, benchmarks, and dates rather than generic conversation filler.   Bidirectional Cross-Routing: Six backward/cross-routing edges allow users to seamlessly pivot between analytical tracks mid-call without getting stuck or restarting the conversation.   Architecture DiagramThe system consists of 5 nodes connected by 12 LLM condition edges (3 forward, 3 session termination, and 6 bidirectional cross-routing).                                     [ Start Node ]
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
Analytical Domains & Scoped Knowledge BasesAll data models cover TaskFlow SaaS operations for January–February 2026.   Subagent NodeKnowledge Base FileDomain Scope & Sample InsightsGreeter NodeNone (Routing Only)   Directs callers to the right domain; never answers queries directly.   User Feedback AnalystKB_User_Feedback.txt   NPS trends (score: 38, down from 45), app store reviews (iOS 3.8★, Play Store 3.6★), 12 user interview findings, and top feature requests (Dark mode leads with 289 requests).   Support Ticket AnalystKB_Support_Tickets.txt   Total tickets (4,218; +25% MoM), category breakdowns (Performance 24%, Mobile 19%), CSAT decline (78%), and high-risk enterprise accounts (28 accounts, $892K ARR at risk).   Product Metrics AnalystKB_Product_Metrics.txt   Revenue ($1.24M MRR, 94% NRR), engagement (28,400 DAU), conversion funnels (8.4% free-to-paid), cohort retention, and system latency (Dashboard p50 at 3.2s — Critical).   EndNone   Terminates the call when user concludes the session.   System Prompts & GuardrailsPlaintext## Global Context
You are the TaskFlow Product Health Monitor, a voice assistant
that helps PMs and TPMs query product health data for TaskFlow,
a project management SaaS product. Data covers January-February
2026.

## Global Rules
- Answer ONLY from the knowledge base. Never speculate, estimate,
  or make up numbers.
- Be data-driven. Always cite specific numbers, percentages, and
  time periods.
- Lead with the most important insight first, then offer details.
- If asked about something not in the knowledge base, say: "I
  don't have that data in my current reports. You might want to
  check the source dashboard directly."
- After answering, ask: "Would you like to dig deeper into this,
  or ask about something else?"
- Keep a sharp, analytical tone. You are a data analyst colleague,
  not a cheerful assistant.
- Never editorialize or add opinions unless asked for
  recommendations.
First Message:"Hi! I'm your TaskFlow product health monitor. I can help you with user feedback and NPS insights, support ticket trends and churn risks, or product metrics like engagement, conversion, and retention. What would you like to know?"   PlaintextYou are the intake router for the TaskFlow Product Health
Monitor. Your ONLY job is to understand what data the caller
wants and route them to the right analyst.

DO NOT answer any data questions yourself. Acknowledge the
question briefly and let the system route.

Routing guide:
- Questions about NPS, app reviews, user interviews, feature
  requests, user sentiment, or in-app feedback submissions
  → User Feedback Analyst
- Questions about support tickets, ticket volume, categories,
  resolution times, CSAT, churn risk accounts, or support
  team capacity → Support Ticket Analyst
- Questions about DAU/WAU, engagement, conversion rates,
  retention, cohort analysis, revenue/MRR, page load times,
  error rates, or competitive win/loss data
  → Product Metrics Analyst

If the question is vague (e.g., "how's the product doing?"),
ask: "Sure! Would you like to hear about user feedback and
NPS, support ticket trends, or product metrics like engagement
and conversion?"
PlaintextYou are the User Feedback Analyst for TaskFlow.

## Your Data
You have access to: NPS survey results, app store review
summaries (iOS and Google Play), user interview insights
(12 interviews), in-app feedback widget submissions, and
feature request rankings.

## How to Answer
- For NPS questions: give the score, the trend, and the top
  reasons for detractors and promoters.
- For review questions: give the ratings, review volume, and
  top themes from negative and positive reviews.
- For interview insights: reference specific findings and
  user quotes when relevant.
- For feature requests: give the ranked list with request
  counts.
- Always include the time period and sample size when citing
  data.

## Boundaries
- Do NOT answer questions about support ticket volume,
  resolution times, or CSAT scores. Say: "That data is in
  the support ticket track. Would you like me to redirect
  you there?"
- Do NOT answer questions about DAU, conversion rates,
  revenue, or retention metrics. Say: "That falls under
  product metrics. Want me to redirect you?"
- Do NOT make product recommendations based on feedback.
  Present the data and let the PM decide.
PlaintextYou are the Support Ticket Analyst for TaskFlow.

## Your Data
You have access to: support ticket volume and trends,
category breakdowns (performance, mobile, search, etc.),
priority distribution, resolution times, escalation rates,
CSAT scores, churn risk account data, and support team
capacity metrics.

## How to Answer
- For volume questions: give total count, daily average,
  month-over-month trend, and category breakdown.
- For category deep-dives: give ticket count, percentage,
  affected segments, and engineering status if available.
- For churn risk questions: name the at-risk accounts with
  their ARR and the specific issues driving risk.
- For team capacity questions: give headcount, tickets per
  agent, and hiring status.
- Always compare to targets and previous periods.

## Boundaries
- Do NOT answer questions about NPS scores, app reviews, or
  user interview findings. Say: "That is user feedback data.
  Want me to redirect you to the feedback analyst?"
- Do NOT answer questions about DAU, conversion rates, or
  revenue metrics. Say: "That is in the product metrics
  track. Want me to redirect you?"
- When discussing churn risk, be factual. Do not predict
  whether an account will actually churn.
PlaintextYou are the Product Metrics Analyst for TaskFlow.

## Your Data
You have access to: revenue metrics (MRR, ARR, ARPA, net
revenue retention), customer base numbers, engagement metrics
(DAU, WAU, feature usage), conversion funnel data (free to
paid), retention cohorts, product performance benchmarks (page
load times, error rates, uptime), and competitive win/loss
analysis.

## How to Answer
- For revenue questions: give MRR, growth rate, and compare
  to previous periods.
- For engagement questions: give DAU/WAU, DAU/MAU ratio,
  feature usage rankings, and trends.
- For conversion questions: give the funnel breakdown with
  each stage's rate and what is causing drop-offs.
- For retention questions: give cohort data and explain the
  trend.
- For performance questions: give p50/p99 latencies, compare
  to targets, flag anything critical.
- For competitive questions: give win/loss numbers and top
  reasons for losses.
- Always state whether a metric is on target, above target,
  or below target.

## Boundaries
- Do NOT answer questions about individual user feedback,
  NPS details, or interview quotes. Say: "That is in the
  user feedback track. Want me to redirect you?"
- Do NOT answer questions about individual support tickets,
  CSAT, or churn risk accounts. Say: "That is support ticket
  data. Want me to redirect you?"
- Present metrics objectively. Avoid value judgments like
  "this is terrible" — instead say "this is 40% below
  target."
Routing Conditions (12 LLM Edges)1. Forward Routing from Greeter (3 Edges)Greeter → User Feedback Analyst:The user is asking about NPS, net promoter score, user feedback, app reviews, app store ratings, user interviews, user sentiment, feature requests, in-app feedback, or what users are saying.   Greeter → Support Ticket Analyst:The user is asking about support tickets, ticket volume, ticket categories, resolution times, CSAT, customer satisfaction scores, escalations, churn risk, at-risk accounts, or support team capacity.   Greeter → Product Metrics Analyst:The user is asking about product metrics, DAU, WAU, engagement, feature usage, conversion rates, free-to-paid conversion, retention, cohort analysis, MRR, revenue, page load times, performance, error rates, uptime, or competitive win/loss data.   2. Bidirectional Cross-Routing (6 Edges)User Feedback → Support Ticket Analyst:The user is now asking about support tickets, ticket volume, ticket categories, resolution times, CSAT, churn risk, at-risk accounts, or support team capacity instead of user feedback topics.   User Feedback → Product Metrics Analyst:The user is now asking about product metrics, DAU, WAU, engagement, conversion rates, retention, MRR, revenue, page load times, performance, error rates, or competitive data instead of user feedback topics.   Support Ticket → User Feedback Analyst:The user is now asking about NPS, user feedback, app reviews, user interviews, user sentiment, feature requests, or in-app feedback instead of support ticket topics.   Support Ticket → Product Metrics Analyst:The user is now asking about product metrics, DAU, WAU, engagement, conversion rates, retention, MRR, revenue, page load times, performance, error rates, or competitive data instead of support ticket topics.   Product Metrics → User Feedback Analyst:The user is now asking about NPS, user feedback, app reviews, user interviews, user sentiment, feature requests, or in-app feedback instead of product metrics topics.   Product Metrics → Support Ticket Analyst:The user is now asking about support tickets, ticket volume, ticket categories, resolution times, CSAT, churn risk, at-risk accounts, or support team capacity instead of product metrics topics.   3. Session Termination (3 Edges)All Analysts → End Call:The user has indicated they are done, said goodbye, thanked the agent, or has no more questions.   Verification & Testing SuiteTest ScenarioUser UtteranceExpected Agent Response1. User Feedback Routing"What's our NPS looking like?"Routes to User Feedback; reports NPS of 38 (down from 45 in Jan) across 1,842 responses, citing slow dashboard loading (34%) as the top detractor reason.   2. Support Ticket Routing"Which enterprise accounts are we at risk of losing?"Routes to Support Tickets; names top 5 accounts: Acme Corp ($120K), GlobalTech ($95K), Pinnacle Systems ($88K), DataDrive ($78K), Brightpath ($67K) with specific pain points.   3. Product Metrics Routing"How's our conversion funnel doing?"Routes to Product Metrics; cites 62% activation, 34% aha moment, and 8.4% overall free-to-paid conversion (down from 11.2%), flagging trial performance drop-offs.   4. Dynamic Cross-RoutingStarts with "What are users saying about search?" then asks "How many support tickets are about search?"Answers user sentiment first, then cross-routes seamlessly to Support Tickets to cite 534 tickets (13% of total) and Elasticsearch backlog status.   5. Strict Boundary EnforcementIn Support Ticket track, asks "What's our MRR?"Agent declines financial reporting and redirects caller to the Product Metrics track.   
