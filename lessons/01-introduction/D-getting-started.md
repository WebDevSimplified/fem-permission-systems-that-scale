---
title: "Getting Started"
description: "Step-by-step instructions to clone the workshop repository, set up the database, and run the demo application."
---

# Getting Started

Let's get the workshop project running on your machine. We'll clone the repository, set up a PostgreSQL database, and seed it with sample data.

## Prerequisites

Make sure you have the following installed:

- **Node.js** v20 or higher
- **Docker** (for running PostgreSQL)
- **Git**
- A code editor (VS Code recommended)

## 1. Clone the Repository

Clone the workshop repository and checkout the starting branch:

```bash
git clone https://github.com/WebDevSimplified/fem-permission-systems-that-scale-demo-project.git

cd fem-permission-systems-that-scale-demo-project

git checkout 1-basic-permissions
```

## 2. Install Dependencies

```bash
npm install
```

## 3. Start the Database

The project includes a Docker Compose file for PostgreSQL. Start it with:

```bash
docker compose up
```

This will start a PostgreSQL container.

> **Don't have Docker?** You can use any PostgreSQL instance. Just update the connection details in your environment variables.

## 4. Configure Environment Variables

Make sure your `.env` file contains the following variables:

```text
# .env
DB_PASSWORD=password
DB_USER=postgres
DB_NAME=permissions-demo
DB_HOST=localhost
DB_PORT=5432
```

## 5. Run Database Migrations

Apply the database schema using Drizzle:

```bash
npm run db:push
```

## 6. Seed the Database

Populate the database with sample users, projects, and documents:

```bash
npm run db:seed
```

This creates:

- **8 users** across two departments (Engineering and Marketing)
- **5 projects** with different department assignments
- **15+ documents** with various statuses and ownership

## 7. Start the Development Server

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) in your browser.

## 8. Verify It Works

You should see a login page with quick-login buttons for different users. Try logging in as different users to see the application:

If you can log in and see projects in the sidebar, you're all set!

## Troubleshooting

### Database connection errors

Make sure Docker is running and the PostgreSQL container is up:

```bash
docker compose ps
```

### Port conflicts

If port 5432 is already in use, update the port in your `.env` file.

### Seed script fails

Make sure you've run the migrations first (`npm run db:push`).

## Project Structure Overview

Here's the key parts of the codebase we'll be spending most of our time in:

```text
src/
├── actions/          # Server actions (form submissions)
├── app/              # Next.js app router pages
├── dal/              # Data access layer (queries, mutations)
```

## Next Steps

With the project running, let's explore the application structure and understand the data model we'll be working with.
