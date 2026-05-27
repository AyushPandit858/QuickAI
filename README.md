# QuickAI

QuickAI is a full-stack AI SaaS web application that gives users a single dashboard for generating written content, creating images, editing images, reviewing resumes, and sharing public AI-generated images with the community. The project uses a React frontend, an Express backend, Clerk authentication, Neon PostgreSQL for persistence, Gemini for text intelligence, ClipDrop for image generation, and Cloudinary for image storage and transformations.

## Table of Contents

- [How It Works](#how-it-works)
- [Tech Stack](#tech-stack)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Getting API Keys](#getting-api-keys)
  - [Clerk](#clerk)
  - [Google Gemini](#google-gemini)
  - [ClipDrop](#clipdrop)
  - [Cloudinary](#cloudinary)
  - [Neon PostgreSQL](#neon-postgresql)
- [Quick Start](#quick-start)
  - [1. Clone or open the project](#1-clone-or-open-the-project)
  - [2. Install dependencies](#2-install-dependencies)
  - [3. Configure environment variables](#3-configure-environment-variables)
  - [4. Create the database table](#4-create-the-database-table)
  - [5. Start the backend](#5-start-the-backend)
  - [6. Start the frontend](#6-start-the-frontend)
  - [7. Create accounts and run a demo](#7-create-accounts-and-run-a-demo)
- [Environment Variables](#environment-variables)
- [Project Structure](#project-structure)
- [Main Features](#main-features)
- [The AI Pipeline](#the-ai-pipeline)
- [Database Schema](#database-schema)
- [API Routes](#api-routes)
- [Deployment](#deployment)
- [Interview Notes](#interview-notes)

## How It Works

QuickAI works as a protected AI tools platform. A visitor lands on the public home page, signs in through Clerk, and then enters the `/ai` dashboard. Inside the dashboard, the user can access different AI tools such as article generation, blog title generation, image generation, background removal, object removal, and resume review.

The frontend collects the user's input and sends a request to the Express backend using Axios. Every protected request includes a Clerk bearer token. On the backend, Clerk verifies the user, custom authentication middleware checks whether the user has a premium plan, and the controller decides whether the requested feature is allowed.

For free users, text tools are limited using `free_usage` stored in Clerk private metadata. Premium users can access the image tools and resume review features. Each successful creation is stored in the Neon PostgreSQL `creations` table so the dashboard can show the user's recent work. Public image generations can also appear in the community gallery, where other users can like or unlike them.

## Tech Stack

### Frontend

- **React 19** is used to build the user interface as reusable components and pages.
- **Vite 7** is used as the frontend build tool because it gives fast local development, quick refreshes, and simple production builds.
- **React Router DOM 7** handles routing between the landing page, dashboard, AI tools, and community page.
- **Tailwind CSS 4** is used for styling with utility classes directly inside JSX.
- **Clerk React** provides authentication UI, user session management, and plan protection components.
- **Axios** handles HTTP requests from the frontend to the backend API.
- **React Hot Toast** displays success and error messages.
- **React Markdown** renders AI-generated markdown responses like articles, blog titles, and resume reviews.
- **Lucide React** provides clean icons for the dashboard and tool pages.

### Backend

- **Node.js** runs the backend JavaScript runtime.
- **Express 5** is used to create REST API routes for AI generation and user creation management.
- **Clerk Express** protects backend routes and verifies authenticated users.
- **Neon Serverless PostgreSQL** stores generated content, image URLs, publish status, likes, and timestamps.
- **OpenAI SDK** is used with Gemini's OpenAI-compatible endpoint, allowing the backend to call Gemini models with a familiar chat completions interface.
- **Axios** is used on the backend to call ClipDrop's image generation API.
- **Cloudinary** stores generated/uploaded images and performs AI image transformations such as background removal and object removal.
- **Multer** handles uploaded files such as images and PDF resumes.
- **pdf-parse** extracts text from uploaded PDF resumes before sending the text to Gemini for review.
- **dotenv** loads backend secrets from the `.env` file.
- **cors** allows the frontend to communicate with the backend during local development and deployment.

## Architecture

The project follows a client-server architecture:

```text
User Browser
    |
    | React + Vite frontend
    | Clerk session + Axios requests
    v
Express API Server
    |
    | Clerk middleware verifies authentication
    | Custom auth middleware checks free/premium usage
    v
Controllers
    |
    | Text tools -> Gemini
    | Image generation -> ClipDrop -> Cloudinary
    | Image editing -> Cloudinary transformations
    | Resume review -> Multer -> pdf-parse -> Gemini
    v
Neon PostgreSQL
```

The frontend is responsible for UI, form state, routing, authentication screens, loading states, and rendering generated output. The backend is responsible for security-sensitive work such as API key usage, plan enforcement, file uploads, AI provider calls, database writes, and community actions.

## Prerequisites

Before running the project locally, install or create the following:

- **Node.js 18 or higher**
- **npm**
- **Clerk account**
- **Google AI Studio Gemini API key**
- **ClipDrop API key**
- **Cloudinary account**
- **Neon PostgreSQL database**
- A modern browser such as Chrome, Edge, or Firefox

## Getting API Keys

### Clerk

Clerk is used for authentication and subscription-plan checks.

1. Create a Clerk application.
2. Copy the frontend publishable key.
3. Copy the backend secret key.
4. Configure a `premium` plan in Clerk if you want the premium feature checks to work as intended.
5. Add the publishable key to the client `.env` file and the secret key to the server `.env` file.

### Google Gemini

Gemini powers the article generator, blog title generator, and resume reviewer.

1. Go to Google AI Studio.
2. Create an API key.
3. Add it as `GEMINI_API_KEY` in the backend `.env` file.

The backend uses the OpenAI SDK with this Gemini-compatible base URL:

```js
https://generativelanguage.googleapis.com/v1beta/openai/
```

### ClipDrop

ClipDrop is used for text-to-image generation.

1. Create a ClipDrop account.
2. Generate an API key.
3. Add it as `CLIPDROP_API_KEY` in the backend `.env` file.

### Cloudinary

Cloudinary stores generated images and performs image transformations.

1. Create a Cloudinary account.
2. Copy the cloud name, API key, and API secret.
3. Add them to the backend `.env` file.

Cloudinary is used for:

- Uploading generated images from ClipDrop
- Removing backgrounds from uploaded images
- Removing selected objects from uploaded images
- Returning hosted image URLs to the frontend

### Neon PostgreSQL

Neon is the serverless PostgreSQL database used to store AI creations.

1. Create a Neon project.
2. Copy the database connection string.
3. Add it as `DATABASE_URL` in the backend `.env` file.
4. Create the `creations` table using the SQL shown in the [Database Schema](#database-schema) section.

## Quick Start

### 1. Clone or open the project

If you already have the project folder, open it in your terminal:

```bash
cd "QuickAI-Full-Stack"
```

The project contains two main folders:

```text
client
server
```

### 2. Install dependencies

Install backend dependencies:

```bash
cd server
npm install
```

Install frontend dependencies:

```bash
cd ../client
npm install
```

### 3. Configure environment variables

Create a `.env` file inside `server`:

```env
PORT=3000
DATABASE_URL=your_neon_database_url
CLERK_SECRET_KEY=your_clerk_secret_key
GEMINI_API_KEY=your_gemini_api_key
CLIPDROP_API_KEY=your_clipdrop_api_key
CLOUDINARY_CLOUD_NAME=your_cloudinary_cloud_name
CLOUDINARY_API_KEY=your_cloudinary_api_key
CLOUDINARY_API_SECRET=your_cloudinary_api_secret
```

Create a `.env` file inside `client`:

```env
VITE_CLERK_PUBLISHABLE_KEY=your_clerk_publishable_key
VITE_BASE_URL=http://localhost:3000
```

### 4. Create the database table

Open your Neon SQL editor and run:

```sql
CREATE TABLE IF NOT EXISTS creations (
  id SERIAL PRIMARY KEY,
  user_id TEXT NOT NULL,
  prompt TEXT NOT NULL,
  content TEXT NOT NULL,
  type TEXT NOT NULL,
  publish BOOLEAN DEFAULT FALSE,
  likes TEXT[] DEFAULT '{}',
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

### 5. Start the backend

From the `server` folder:

```bash
npm run server
```

The backend should start on:

```text
http://localhost:3000
```

You can test it by opening:

```text
http://localhost:3000
```

It should return:

```text
Server is Live!
```

### 6. Start the frontend

Open another terminal and run:

```bash
cd client
npm run dev
```

The frontend usually starts on:

```text
http://localhost:5173
```

### 7. Create accounts and run a demo

1. Open the frontend URL.
2. Sign up or sign in with Clerk.
3. Go to the AI dashboard.
4. Try the article generator or blog title generator first.
5. Use a premium account to test image generation, background removal, object removal, and resume review.
6. Generate a public image and check it in the community page.

## Environment Variables

### Client

| Variable | Purpose |
| --- | --- |
| `VITE_CLERK_PUBLISHABLE_KEY` | Public Clerk key used by the React app to initialize Clerk authentication. |
| `VITE_BASE_URL` | Backend API base URL used by Axios. For local development, use `http://localhost:3000`. |

### Server

| Variable | Purpose |
| --- | --- |
| `PORT` | Port where the Express server runs. Defaults to `3000` if not provided. |
| `DATABASE_URL` | Neon PostgreSQL connection string. |
| `CLERK_SECRET_KEY` | Clerk backend secret used by `@clerk/express`. |
| `GEMINI_API_KEY` | Gemini API key used for text generation and resume review. |
| `CLIPDROP_API_KEY` | ClipDrop API key used for text-to-image generation. |
| `CLOUDINARY_CLOUD_NAME` | Cloudinary cloud name. |
| `CLOUDINARY_API_KEY` | Cloudinary API key. |
| `CLOUDINARY_API_SECRET` | Cloudinary API secret. |

## Project Structure

```text
QuickAI-Full-Stack
├── client
│   ├── public
│   ├── src
│   │   ├── assets
│   │   ├── components
│   │   │   ├── AiTools.jsx
│   │   │   ├── CreationItem.jsx
│   │   │   ├── Footer.jsx
│   │   │   ├── Hero.jsx
│   │   │   ├── Navbar.jsx
│   │   │   ├── Plan.jsx
│   │   │   ├── Sidebar.jsx
│   │   │   └── Testimonial.jsx
│   │   ├── pages
│   │   │   ├── BlogTitles.jsx
│   │   │   ├── Community.jsx
│   │   │   ├── Dashboard.jsx
│   │   │   ├── GenerateImages.jsx
│   │   │   ├── Home.jsx
│   │   │   ├── Layout.jsx
│   │   │   ├── RemoveBackground.jsx
│   │   │   ├── RemoveObject.jsx
│   │   │   ├── ReviewResume.jsx
│   │   │   └── WriteArticle.jsx
│   │   ├── App.jsx
│   │   ├── index.css
│   │   └── main.jsx
│   ├── package.json
│   └── vite.config.js
│
├── server
│   ├── configs
│   │   ├── cloudinary.js
│   │   ├── db.js
│   │   └── multer.js
│   ├── controllers
│   │   ├── aiController.js
│   │   └── userController.js
│   ├── middlewares
│   │   └── auth.js
│   ├── routes
│   │   ├── aiRoutes.js
│   │   └── userRoutes.js
│   ├── package.json
│   └── server.js
│
└── README.md
```

## Main Features

### AI Article Writer

The user enters a topic and selects an article length. The frontend builds a prompt and sends it to `/api/ai/generate-article`. The backend calls Gemini, stores the generated article in Neon, updates free usage for free users, and returns markdown content to the frontend.

### Blog Title Generator

The user enters a keyword and selects a category. Gemini generates title ideas, and the result is saved as a `blog-title` creation.

### AI Image Generation

Premium users can generate images from text prompts. The backend sends the prompt to ClipDrop, receives the generated image as binary data, converts it to base64, uploads it to Cloudinary, stores the Cloudinary URL in Neon, and returns the URL to the frontend.

### Background Removal

Premium users upload an image. Multer receives the file, Cloudinary applies background removal, and the processed image URL is saved and returned.

### Object Removal

Premium users upload an image and provide a single object name. Cloudinary uses a generative remove transformation to remove that object from the image.

### Resume Review

Premium users upload a PDF resume. The backend extracts text using `pdf-parse`, sends that text to Gemini with a review prompt, stores the feedback, and returns markdown feedback to the frontend.

### Dashboard

The dashboard fetches the current user's saved creations from `/api/user/get-user-creations`. It shows total creations, active plan, and recent AI outputs.

### Community Gallery

The community page fetches published creations from `/api/user/get-published-creations`. Users can like or unlike public images through `/api/user/toggle-like-creation`.

## The AI Pipeline

### Text Generation Pipeline

```text
User prompt
  -> React form
  -> Axios request with Clerk token
  -> Express route
  -> Clerk authentication
  -> Free/premium usage check
  -> Gemini model
  -> Save result in Neon
  -> Return markdown response
  -> Render with React Markdown
```

### Image Generation Pipeline

```text
User prompt
  -> React form
  -> Express backend
  -> Premium plan check
  -> ClipDrop text-to-image API
  -> Convert image response to base64
  -> Upload to Cloudinary
  -> Store Cloudinary URL in Neon
  -> Show image in frontend
```

### Resume Review Pipeline

```text
PDF upload
  -> Multer file handling
  -> File size validation
  -> Extract text with pdf-parse
  -> Gemini resume feedback prompt
  -> Store review in Neon
  -> Render review in frontend
```

## Database Schema

The project expects a `creations` table:

```sql
CREATE TABLE IF NOT EXISTS creations (
  id SERIAL PRIMARY KEY,
  user_id TEXT NOT NULL,
  prompt TEXT NOT NULL,
  content TEXT NOT NULL,
  type TEXT NOT NULL,
  publish BOOLEAN DEFAULT FALSE,
  likes TEXT[] DEFAULT '{}',
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

### Column Meaning

| Column | Description |
| --- | --- |
| `id` | Unique creation ID. |
| `user_id` | Clerk user ID of the creator. |
| `prompt` | Prompt or task description used to generate the content. |
| `content` | Generated text, resume review, or Cloudinary image URL. |
| `type` | Creation type such as `article`, `blog-title`, `image`, or `resume-review`. |
| `publish` | Whether an image should appear in the community gallery. |
| `likes` | Array of Clerk user IDs who liked the creation. |
| `created_at` | Time when the creation was inserted. |
| `updated_at` | Time when the creation was last updated. |

## API Routes

### AI Routes

| Method | Route | Description |
| --- | --- | --- |
| `POST` | `/api/ai/generate-article` | Generates an article using Gemini. |
| `POST` | `/api/ai/generate-blog-title` | Generates blog title ideas using Gemini. |
| `POST` | `/api/ai/generate-image` | Generates an image using ClipDrop and stores it in Cloudinary. |
| `POST` | `/api/ai/remove-image-background` | Removes image background using Cloudinary. |
| `POST` | `/api/ai/remove-image-object` | Removes a selected object from an image using Cloudinary. |
| `POST` | `/api/ai/resume-review` | Reviews an uploaded PDF resume using Gemini. |

### User Routes

| Method | Route | Description |
| --- | --- | --- |
| `GET` | `/api/user/get-user-creations` | Fetches all creations for the logged-in user. |
| `GET` | `/api/user/get-published-creations` | Fetches public creations for the community page. |
| `POST` | `/api/user/toggle-like-creation` | Likes or unlikes a public creation. |

## Deployment

The project includes separate `vercel.json` files for the frontend and backend, so both parts can be deployed separately.

### Deploy Backend

1. Push the project to GitHub.
2. Create a new Vercel project for the `server` folder.
3. Set the framework preset to Other if needed.
4. Add all backend environment variables in Vercel.
5. Deploy the backend.
6. Copy the deployed backend URL.

### Deploy Frontend

1. Create another Vercel project for the `client` folder.
2. Add the frontend environment variables.
3. Set `VITE_BASE_URL` to the deployed backend URL.
4. Deploy the frontend.
5. Add the deployed frontend URL to Clerk's allowed origins or redirect settings if required.

### Local Laptop Deployment

To run the complete project on your laptop:

1. Install dependencies in both `client` and `server`.
2. Create both `.env` files.
3. Create the `creations` table in Neon.
4. Start the backend with `npm run server`.
5. Start the frontend with `npm run dev`.
6. Open `http://localhost:5173`.
7. Sign in and test the tools.

