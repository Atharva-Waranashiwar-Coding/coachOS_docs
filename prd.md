# Product Requirements

## Product Vision

CoachOS helps coaches manage athlete development with structured profiles, video review, AI-assisted observations, coach-approved feedback, and drill assignments.

## Problem Statement

Coaches often track athletes, practice notes, videos, and training feedback across disconnected tools. This makes review history hard to search, slows down feedback, and limits visibility into athlete progress.

## Target Users

- Coaches managing individual athletes or small teams
- Athletes receiving feedback and assigned drills
- Program admins who may later manage teams, subscriptions, and staff

## MVP Objective

Deliver a coach-facing workflow for login, athlete management, video upload, AI-generated review, coach approval, and drill assignment.

## MVP Features

- Coach signup and login
- JWT-based authentication
- Coach dashboard
- Athlete profile CRUD
- Practice session and timeline tracking
- Video upload and metadata storage
- AI review generation from video metadata or transcript inputs
- Coach edit and approval workflow
- Drill assignment tracking
- Basic athlete dashboard

## Out-Of-Scope Features

- Payments and subscriptions
- Native mobile apps
- Real-time chat
- Custom computer vision model training
- Multi-tenant enterprise administration
- Wearable device integrations

## Success Metrics

- A coach can complete the full review workflow without manual database edits
- Video upload metadata is persisted and linked to athletes
- AI review output is editable before athlete visibility
- Each MVP service has tests for core endpoints
- The stack can run locally with documented setup steps

## 6-Week Sprint Plan

- Week 1: Repository setup, docs, auth service foundation, local Docker Compose
- Week 2: Auth service endpoints, JWT, coach user model, frontend login shell
- Week 3: Athlete service, athlete profiles, coach-athlete relationship, dashboard routes
- Week 4: Media service, signed upload flow, video metadata, practice sessions
- Week 5: AI review service, prompt strategy, review status workflow, coach approval UI
- Week 6: Drill assignment, athlete dashboard, integration testing, deployment preparation

## Long-Term Vision

CoachOS becomes a full athlete development operating system with team management, longitudinal performance insights, richer AI video analysis, drill libraries, athlete communication, and scalable cloud deployment.
