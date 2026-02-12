<div align="center">

# LearnQuest Backend API

**A production-grade REST API powering a gamified, crowdsourced e-learning platform.**

[![Python](https://img.shields.io/badge/Python-3.11+-3776AB?style=flat-square&logo=python&logoColor=white)](https://python.org)
[![Flask](https://img.shields.io/badge/Flask-3.x-000000?style=flat-square&logo=flask&logoColor=white)](https://flask.palletsprojects.com)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-4169E1?style=flat-square&logo=postgresql&logoColor=white)](https://postgresql.org)
[![JWT](https://img.shields.io/badge/Auth-JWT-orange?style=flat-square&logo=jsonwebtokens&logoColor=white)](https://jwt.io)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)](LICENSE)
[![Live](https://img.shields.io/badge/Live-learnquest.qzz.io-6C63FF?style=flat-square)](https://learnquest.qzz.io)

[Live Demo](https://learnquest.qzz.io) -- [Frontend Repo](https://github.com/MrNawir/LearnQuest-Frontend) -- [API Reference](#api-reference)

## Team Members — Group 7
- **Ibrahim Abdu** — Project Leader, Backend Architecture & Integration
- **Bradley Murimi** — Backend Developer Lead(Auth & Gamification)
- **Joyce Njogu** — Frontend Developer Lead
- **Julius Mutinda** — Frontend Developer (Auth & Learning)
- **Ephrahim Otieno** — Full Stack Developer (Community Features)
- **Craig Omore** — Full Stack Developer (Content & Admin)

---

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Getting Started](#getting-started)
- [Environment Variables](#environment-variables)
- [Database](#database)
- [API Reference](#api-reference)
- [Authentication & Authorization](#authentication--authorization)
- [Gamification Engine](#gamification-engine)
- [Project Structure](#project-structure)
- [Deployment](#deployment)
- [Demo Accounts](#demo-accounts)
- [Team](#team)
- [License](#license)

---

## Overview

LearnQuest is a crowdsourced learning platform that transforms online education into an engaging, game-like experience. The backend is a RESTful API built with Flask that handles user authentication, learning path management, progress tracking, a full gamification engine (XP, badges, streaks, leaderboards, challenges), community discussions, quizzes, content moderation, and role-based administration.

The API serves a React/TypeScript single-page application and is deployed behind Nginx with Gunicorn on a Debian VPS, proxied through Cloudflare.

---

## Architecture

```
                                 HTTPS (Cloudflare)
                                       |
                                  [ Nginx ]
                                  /        \
                     Static Files           /api/*
                    (React Build)             |
                                        [ Gunicorn ]
                                             |
                                      [ Flask App ]
                                      /     |     \
                              Routes   Services   Models
                                             |
                                      [ PostgreSQL ]
```

**Request lifecycle:**

1. Client sends HTTPS request to `learnquest.qzz.io`
2. Cloudflare terminates public TLS, forwards to origin over Full SSL
3. Nginx routes `/api/*` to Gunicorn (port 5000), serves static frontend for all other paths
4. Flask processes the request through JWT middleware, route handlers, service logic, and ORM queries
5. Response returns as JSON with appropriate status codes

---

## Features

| Category | Capabilities |
|----------|-------------|
| **Authentication** | JWT-based registration/login, role-based access control (Learner, Contributor, Admin), token refresh, password hashing with Werkzeug |
| **Learning Paths** | Full CRUD for paths, modules, and resources; category/difficulty filtering; full-text search with PostgreSQL ILIKE; creator attribution |
| **Progress Tracking** | Enrollment management, per-resource completion with XP rewards, percentage-based progress calculation, time tracking |
| **Gamification** | 10 earnable badges with automatic criteria checking, XP/points system, daily streak tracking with bonus multipliers, weekly/monthly/seasonal challenges, global leaderboards with period filtering |
| **Quizzes** | Module-linked quizzes with multiple question types, timed attempts, scoring with explanations, perfect-score tracking for badges |
| **Community** | Threaded comments on learning paths, nested replies, edit window (15 min), soft-delete, XP rewards for participation |
| **Content Pipeline** | Contributors create paths via API, paths enter pending queue, admins approve/reject with feedback, XP awarded to creator on approval |
| **File Uploads** | PDF resource upload (up to 20MB), secure filename handling, direct file serving |
| **Admin Dashboard** | Platform statistics (users, paths, growth metrics), user management (role changes, suspension, deletion), content moderation queue, report handling |

---

## Tech Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **Framework** | Flask 3.x | Lightweight WSGI web framework |
| **ORM** | Flask-SQLAlchemy | Database abstraction and model layer |
| **Migrations** | Flask-Migrate (Alembic) | Schema version control |
| **Authentication** | Flask-JWT-Extended | JSON Web Token auth with role decorators |
| **CORS** | Flask-CORS | Cross-origin request handling |
| **Database** | PostgreSQL 16 | Production relational database |
| **WSGI Server** | Gunicorn | Production application server |
| **Reverse Proxy** | Nginx | TLS termination, static file serving, request routing |
| **CDN/DNS** | Cloudflare | DNS, DDoS protection, SSL proxy |

---

## Getting Started

### Prerequisites

- Python 3.8 or higher
- PostgreSQL (production) or SQLite (development)
- pipenv, pip, or uv

### Installation

```bash
# Clone the repository
git clone git@github.com:MrNawir/LearnQuest-Backend.git
cd LearnQuest-Backend

# Install dependencies
pipenv install
# OR: pip install flask flask-sqlalchemy flask-migrate flask-cors flask-jwt-extended python-dotenv gunicorn psycopg2-binary

# Configure environment
cp .env.example .env
# Edit .env with your database credentials (see Environment Variables below)

# Activate the virtual environment
pipenv shell

# Seed the database with demo data
python seed_data.py

# Start the development server
python run.py
```

The API will be available at **http://localhost:5000**. Verify with:

```bash
curl http://localhost:5000/api/health
# {"status": "healthy", "message": "LearnQuest API is running!"}
```

### Reset Database

To reset to a clean state with fresh demo data:

```bash
# SQLite (development)
rm -f instance/learnquest.db && python seed_data.py

# PostgreSQL (production)
# Drop and recreate the database, then re-seed
```

---

## Environment Variables

Create a `.env` file in the project root:

| Variable | Description | Default |
|----------|-------------|---------|
| `FLASK_APP` | Application entry point | `run.py` |
| `FLASK_ENV` | Environment mode | `development` |
| `SECRET_KEY` | Flask secret key for sessions | `dev-secret-key` |
| `JWT_SECRET_KEY` | JWT signing secret | `jwt-secret-key` |
| `DATABASE_URL` | Database connection string | `sqlite:///learnquest.db` |
| `CORS_ORIGINS` | Allowed CORS origins (comma-separated) | `*` |

**Example `.env` for production:**

```env
FLASK_APP=run.py
FLASK_ENV=production
SECRET_KEY=<random-64-char-hex>
JWT_SECRET_KEY=<random-64-char-hex>
DATABASE_URL=postgresql://user:password@localhost:5432/learnquest_db
CORS_ORIGINS=https://learnquest.qzz.io
```

---

## Database

### Schema Overview

```
users
  |-- learning_paths (creator_id)
  |     |-- modules
  |     |     |-- resources
  |     |     |-- quizzes
  |     |           |-- questions
  |     |-- enrollments (user_progress)
  |-- user_badges
  |-- resource_completions
  |-- quiz_attempts
  |-- comments
  |-- reports
  |-- notifications

badges, achievements, challenges (platform-wide)
leaderboard (materialized from user XP)
```

### Models

| Model | Key Fields | Purpose |
|-------|-----------|---------|
| **User** | username, email, role, xp, points, streak_days, status | User accounts with gamification stats |
| **LearningPath** | title, category, difficulty, creator_id, is_published, is_approved | Crowdsourced course content |
| **Module** | title, order, xp_reward, learning_path_id | Ordered sections within a path |
| **Resource** | title, resource_type (video/article/pdf), url, module_id | Individual learning materials |
| **UserProgress** | user_id, learning_path_id, completed_resources, progress_percentage | Enrollment and completion tracking |
| **ResourceCompletion** | user_id, resource_id, xp_earned, time_spent | Per-resource completion records |
| **Badge** | name, badge_type (bronze/silver/gold/platinum/special) | Earnable achievement badges |
| **UserBadge** | user_id, badge_id, earned_at | Junction table for earned badges |
| **Challenge** | title, challenge_type, xp_reward, start_date, end_date | Time-bound platform challenges |
| **Quiz** | title, module_id, passing_score, time_limit | Assessments linked to modules |
| **Question** | question_text, question_type, options, correct_answer | Quiz questions with explanations |
| **Comment** | content, user_id, learning_path_id, parent_id | Threaded discussion comments |
| **Report** | reporter_id, target_type, reason, status | Content moderation reports |

---

## API Reference

All endpoints are prefixed with `/api`. Protected endpoints require a `Bearer` token in the `Authorization` header.

### Authentication

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `POST` | `/auth/register` | -- | Register a new user (default role: learner) |
| `POST` | `/auth/login` | -- | Login, returns JWT access token |
| `GET` | `/auth/me` | JWT | Get current authenticated user |

**Example -- Login:**

```bash
curl -X POST https://learnquest.qzz.io/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "alex@example.com", "password": "demo123"}'
```

```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "user": { "id": 3, "username": "alex_learner", "role": "learner", "xp": 450 }
}
```

### Users

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `GET` | `/users/` | -- | List all users |
| `GET` | `/users/<id>` | -- | Get user by ID |
| `PUT` | `/users/profile` | JWT | Update current user's profile |
| `GET` | `/users/<id>/stats` | -- | Get user statistics |

### Learning Paths

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `GET` | `/learning-paths/` | -- | List all published and approved paths |
| `GET` | `/learning-paths/<id>` | -- | Get path details with modules and resources |
| `GET` | `/learning-paths/my-paths` | JWT | Get paths created by the current user |
| `POST` | `/learning-paths/` | JWT (contributor+) | Create a new learning path |
| `POST` | `/learning-paths/<id>/modules` | JWT (creator) | Add a module to a path |
| `POST` | `/learning-paths/modules/<id>/resources` | JWT | Add a resource to a module |
| `POST` | `/learning-paths/<id>/rate` | JWT | Rate a path (1-5 stars) |
| `GET` | `/learning-paths/search?q=<query>` | -- | Full-text search across paths, modules, resources |
| `POST` | `/learning-paths/upload-pdf` | JWT | Upload a PDF resource (multipart, max 20MB) |
| `GET` | `/learning-paths/pdf/<filename>` | -- | Serve an uploaded PDF file |

### Progress and Enrollment

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `POST` | `/progress/enroll/<path_id>` | JWT | Enroll in a learning path |
| `GET` | `/progress/my-paths` | JWT | Get all enrolled paths with progress |
| `GET` | `/progress/path/<path_id>` | JWT | Get progress for a specific path |
| `POST` | `/progress/complete-resource/<id>` | JWT | Mark resource as completed (+10 XP) |
| `POST` | `/progress/complete-module/<id>` | JWT | Mark module as completed (+XP) |

### Quizzes

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `GET` | `/quizzes/module/<module_id>/quiz` | -- | Get quiz for a module |
| `GET` | `/quizzes/<id>` | -- | Get quiz by ID with questions |
| `POST` | `/quizzes/<id>/submit` | JWT | Submit quiz answers, returns score and XP |
| `GET` | `/quizzes/<id>/attempts` | JWT | Get user's attempts for a quiz |

### Gamification

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `GET` | `/gamification/badges` | -- | List all platform badges |
| `GET` | `/gamification/badges/<user_id>` | -- | Get badges earned by a user |
| `POST` | `/gamification/badges/check` | JWT | Check and award any newly earned badges |
| `GET` | `/gamification/leaderboard` | -- | Get ranked leaderboard (`?period=weekly\|all_time&limit=50`) |
| `GET` | `/gamification/leaderboard/me` | JWT | Get current user's rank and surrounding users |
| `GET` | `/gamification/challenges` | -- | List active challenges |
| `GET` | `/gamification/achievements` | -- | List all achievements |
| `GET` | `/gamification/achievements/progress` | JWT | Get user's progress toward each achievement |
| `POST` | `/gamification/xp/add` | JWT | Manually add XP (admin/system use) |
| `POST` | `/gamification/streak/update` | JWT | Update daily learning streak |
| `GET` | `/gamification/streak/status` | JWT | Get current streak status |

### Comments and Discussions

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `GET` | `/comments` | -- | Get comments (`?learning_path_id=X`, paginated) |
| `POST` | `/comments` | JWT | Post a comment (+5 XP) |
| `PUT` | `/comments/<id>` | JWT | Edit own comment (within 15-minute window) |
| `DELETE` | `/comments/<id>` | JWT | Soft-delete own comment |

### Admin

All admin endpoints require the `admin` role.

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/admin/stats` | Platform-wide statistics (users, paths, growth) |
| `GET` | `/admin/pending` | Learning paths awaiting approval |
| `POST` | `/admin/approve/<path_id>` | Approve a path (+100 XP to creator) |
| `POST` | `/admin/reject/<path_id>` | Reject a path with reason |
| `GET` | `/admin/users` | List users (`?role=&search=&page=&per_page=`) |
| `PUT` | `/admin/users/<id>/role` | Change user role (learner or contributor only) |
| `PUT` | `/admin/users/<id>/suspend` | Suspend or reactivate a user |
| `DELETE` | `/admin/users/<id>` | Delete a user |
| `GET` | `/admin/reports` | Get moderation reports (`?status=pending`) |
| `POST` | `/admin/reports/<id>/dismiss` | Dismiss a report |
| `POST` | `/admin/reports/<id>/action` | Take action on a report |

---

## Authentication and Authorization

The API uses **JSON Web Tokens (JWT)** for stateless authentication.

**Token flow:**

1. Client sends `POST /api/auth/login` with email and password
2. Server validates credentials, returns a signed JWT
3. Client includes the token in subsequent requests via `Authorization: Bearer <token>`
4. Protected routes extract the user identity from the token and verify role permissions

**Role hierarchy:**

```
Admin
  |-- Can manage all users, approve/reject content, view platform stats
  |-- Cannot be assigned via API (only direct DB or existing admin)
  |
Contributor
  |-- Can create learning paths, upload resources
  |-- Inherits all Learner capabilities
  |
Learner (default)
  |-- Can enroll, complete lessons, earn XP/badges, participate in discussions
```

---

## Gamification Engine

The gamification system is designed to drive engagement through multiple feedback loops:

### XP and Points

| Action | XP Reward |
|--------|----------|
| Complete a resource | +10 XP |
| Complete a module | +50 XP |
| Post a comment | +5 XP |
| Earn a new badge | +50 XP per badge |
| Path approved (creator) | +100 XP |
| Quiz completion | Variable (based on score) |

### Badges (10 total)

| Badge | Type | Criteria |
|-------|------|----------|
| First Steps | Bronze | Complete 1 resource |
| Path Finder | Silver | Complete 1 full learning path |
| Week Warrior | Silver | Maintain a 7-day streak |
| Streak Legend | Platinum | Maintain a 30-day streak |
| Quiz Master | Gold | Score 100% on 5 quizzes |
| Social Butterfly | Bronze | Post 10 comments |
| Code Ninja | Platinum | Complete 50 resources |
| Mentor | Gold | Create 1 learning path (contributor) |
| Early Bird | Bronze | Special/seasonal |
| Data Science Month | Special | Seasonal challenge badge |

Badges are checked automatically via `POST /api/gamification/badges/check`, which is called by the frontend after each lesson completion.

### Streaks

The streak system tracks consecutive days of learning activity. Streak bonuses multiply XP gains and unlock the Week Warrior (7 days) and Streak Legend (30 days) badges.

### Leaderboard

Global rankings computed from user XP, filterable by `weekly` or `all_time` periods. Each entry includes rank, username, avatar, XP, and level (XP / 1000 + 1).

---

## Project Structure

```
LearnQuest-Backend/
|
|-- app/
|   |-- __init__.py              # App factory, extension init, blueprint registration
|   |-- models/
|   |   |-- user.py              # User model (roles, XP, streaks, status)
|   |   |-- learning_path.py     # LearningPath, Module, Resource models
|   |   |-- gamification.py      # Badge, UserBadge, Achievement, Challenge, Leaderboard
|   |   |-- quiz.py              # Quiz, Question, QuizAttempt
|   |   |-- progress.py          # UserProgress, ResourceCompletion
|   |   |-- comment.py           # Comment (threaded replies, soft-delete)
|   |   |-- notification.py      # Notification model
|   |   +-- report.py            # Report model for content moderation
|   |
|   |-- routes/
|   |   |-- auth.py              # /api/auth/* -- registration, login, token
|   |   |-- users.py             # /api/users/* -- profiles, stats
|   |   |-- learning_paths.py    # /api/learning-paths/* -- CRUD, search, PDF upload
|   |   |-- resources.py         # /api/resources/* -- resource management, downloads
|   |   |-- gamification.py      # /api/gamification/* -- badges, leaderboard, XP, streaks
|   |   |-- quizzes.py           # /api/quizzes/* -- quiz CRUD, submission, scoring
|   |   |-- progress.py          # /api/progress/* -- enrollment, completion tracking
|   |   |-- comments.py          # /api/comments/* -- threaded discussions
|   |   +-- admin.py             # /api/admin/* -- stats, approvals, user management
|   |
|   |-- services/
|   |   |-- streak_service.py    # Streak calculation and bonus logic
|   |   +-- leaderboard_service.py # Leaderboard computation and caching
|   |
|   +-- utils/                   # Shared decorators and helper functions
|
|-- uploads/
|   +-- pdfs/                    # Uploaded PDF learning resources
|
|-- tests/
|   +-- test_admin.py            # Admin endpoint test suite
|
|-- seed_data.py                 # Database seeding (users, paths, badges, quizzes)
|-- run.py                       # Development server entry point
|-- wsgi.py                      # Production WSGI entry point (Gunicorn)
|-- Pipfile                      # Python dependency manifest
+-- .env                         # Environment configuration (not committed)
```

---

## Deployment

The production environment runs on a Debian VPS with the following stack:

```
Cloudflare (DNS + Proxy + SSL)
       |
Nginx (reverse proxy, TLS origin cert, static files)
       |
Gunicorn (WSGI, systemd service: learnquest-api)
       |
Flask Application
       |
PostgreSQL 16
```

### Production commands

```bash
# Restart the API service
sudo systemctl restart learnquest-api

# Reload Nginx configuration
sudo systemctl reload nginx

# View API logs
sudo journalctl -u learnquest-api -f

# Check service status
sudo systemctl status learnquest-api
```

### Nginx configuration highlights

- `/api/*` proxied to `127.0.0.1:5000`
- All other routes serve the React SPA with `try_files $uri /index.html`
- `client_max_body_size 20M` for PDF uploads
- Gzip compression on text, CSS, JSON, and JavaScript
- 30-day cache headers on static assets

---

## Demo Accounts

All demo accounts use the password: **`demo123`**

| Role | Email | Username | Capabilities |
|------|-------|----------|-------------|
| **Admin** | admin@learnquest.com | admin_user | Full platform management, approve/reject paths, manage users |
| **Contributor** | creator@learnquest.com | demo_creator | Create learning paths via Creator Studio, upload resources |
| **Learner** | alex@example.com | alex_learner | Browse paths, complete lessons, earn XP and badges |
| **Learner** | sarah@example.com | sarah_jenkins | Clean account (no completions) -- ideal for demo |
| **Contributor** | mike@example.com | mike_teacher | Secondary contributor account |

### Demo Flow -- Badge Unlock

1. Log in as `sarah@example.com` / `demo123` (clean learner account)
2. Navigate to any learning path and click **Enroll**
3. Open a lesson and click **Complete Lesson**
4. The "First Steps" badge unlocks automatically with a toast notification
5. Visit the Achievements page to see the badge in your collection

### Demo Flow -- Creator to Admin Approval

1. Log in as `creator@learnquest.com` / `demo123`
2. Open **Creator Studio** and click **Create New Path**
3. Fill in details, add modules with video URLs or upload PDFs
4. Click **Publish** -- the path enters the admin approval queue
5. Log in as `admin@learnquest.com` / `demo123`
6. Open **Admin Dashboard** and navigate to **Pending Approvals**
7. Review and **Approve** the path -- it becomes visible to all learners

---

## Team -- Group 7

| Name | Role |
|------|------|
| **Ibrahim Abdu** | Project Leader, Backend Architecture and Integration |
| **Bradley Murimi** | Backend Developer (Auth and Gamification) |
| **Joyce Njogu** | Frontend Developer Lead |
| **Julius Mutinda** | Frontend Developer (Auth and Learning) |
| **Ephrahim Otieno** | Full Stack Developer (Community Features) |
| **Craig Omore** | Full Stack Developer (Content and Admin) |

---

## License

This project is licensed under the [MIT License](LICENSE).
