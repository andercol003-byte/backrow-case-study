# BackRow

A social app for college students, built around courses and campus communities — think class-based group chats, rankings, and campus-specific communities, gated by school verification so every member is a real student at your school.

**[Download on the App Store](https://apps.apple.com/us/app/backrow-social/id6803615023)**

> This is a showcase repo for my resume/portfolio. BackRow's actual source is closed-source (private repo) since it's a live product with real users — this page summarizes the architecture and my role without exposing source code, infra details, or production data.

---

## What it does

- **School-gated signup** — invite/referral-based onboarding, email verification scoped to a student's school
- **Courses & communities** — students join their classes and campus groups, post, comment, and vote
- **Direct messages** — 1:1 and group chat, scoped same-school, with media attachments
- **Rankings** — community-voted rankings with ballots, cooldowns, and anti-abuse safeguards
- **Moderation & admin tooling** — role-based moderation (reports, bans, content removal), full audit logging, school approval workflows
- **Mobile + web** — native iOS app and a web client sharing the same backend and business logic

## My role

Solo founder & full-stack engineer — product, mobile app (iOS/Expo), web app, backend API, database design, AWS infrastructure, load testing/capacity planning, and day-to-day operations end to end.

## Tech stack

**Frontend**
- Next.js (App Router) — web
- Expo / React Native — iOS app
- Shared TypeScript package for cross-platform validation, types, and business logic

**Backend**
- Node.js API (Express) with JWT/session auth
- PostgreSQL (Amazon RDS), including read-replica routing for hot read paths
- `SECURITY DEFINER` Postgres functions as the authorization boundary for sensitive operations (role/ban checks enforced server-side at the database layer, not just the API layer)

**Infrastructure (AWS)**
- ECS (Fargate) for the API
- RDS Postgres (primary + read replica)
- CloudFront + WAF in front of the web/API surface
- Infrastructure as code via AWS CDK
- Load-tested and capacity-planned for a real campus-scale launch (gradual rollout, autoscaling, overload fallback behavior validated under synthetic load)

**Other**
- AWS SES for transactional email
- Expo EAS for native builds and over-the-air updates
- Push notifications (APNs via Expo)

## Notable engineering decisions

- **Database-enforced authorization** — rather than trusting a JWT claim at the API layer alone, every sensitive write (bans, role changes, content removal, course approvals) re-checks the actor's live role/ban status inside a `SECURITY DEFINER` Postgres function, with every such action writing an audit log row in the same transaction.
- **Same-school data boundaries** — direct messages and course communities are scoped to a student's verified school, enforced at both the API and database layer.
- **Read/write path separation** — read-heavy, latency-sensitive endpoints are routed to a Postgres read replica, while anything a user needs to see reflected immediately after writing (e.g. just joining a course) deliberately stays on the primary to avoid replica-lag bugs.
- **Capacity validated under real load** — synthetic load testing against a disposable copy of the infrastructure, with deliberate overload scenarios to confirm graceful degradation (shedding load) rather than cascading failure.

---

*Screenshots and a more detailed write-up coming soon.*
