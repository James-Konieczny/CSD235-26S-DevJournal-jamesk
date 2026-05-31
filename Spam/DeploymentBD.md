# Deployment Brain Dump
**Goal:**

I am responsible for the research on how to deploy the project.
1. Read all the files in the project that can guide you on what it shall or not be changed in the project for the development
2. Search online for examples on how to do this deployment
3. Search for good and safe domain options to deploy the application
4. Present the options and alterations to the Group Leader and Sponsor

- Private server (test deployment) for Jenifer viewing
- Get OHS Remote up on a server for public access (offical launch)

**Pieces:**
- Backend – the “engine” (FastAPI Python code)
- Frontend – the “dashboard and steering wheel” (React app)
- Database – the “memory” (MySQL)
- Environment variables (secrets like API keys)

## Steps to consider

1. Rent a computer in the cloud

Sign up for a service like Render, Railway, Heroku, AWS, or Azure.
They give you a virtual machine (or container) with a public IP address. Many have free tiers good enough for a test deployment.

2. Set up a database in the cloud

Instead of using MySQL inside a local Docker container, you create a managed MySQL database from the same cloud provider. They give you a connection string (looks like mysql://username:password@host:port/dbname).

3. Put your backend on the cloud computer

You package your backend into a Docker container (the previous team already has a Dockerfile).

You upload that container to a container registry (like Docker Hub) or directly deploy it on a platform that understands Docker.

You tell the cloud computer to run your container, passing in all the environment variables (Stripe keys, OpenAI key, database URL, etc.) as secrets – never hardcoded.

4. Put your frontend on a static hosting service

Your React frontend is just static files (HTML, CSS, JS) after running npm run build.
You can upload the dist/ folder to a service like Netlify, Vercel, or even the same cloud provider’s static storage (e.g., AWS S3 + CloudFront).
These services give you a public URL (e.g., https://your-app.netlify.app).

5. Connect frontend to backend

You change the frontend’s API base URL (currently hardcoded to http://localhost:8000) to the public URL of your deployed backend (e.g., https://your-backend.onrender.com/api/v1).
Because the frontend runs in a browser, that backend URL must be accessible from the internet and must have CORS configured to allow requests from the frontend’s URL.

6. Update Stripe webhook

Stripe needs to call your backend when a payment succeeds. In the Stripe Dashboard, you change the webhook endpoint URL to your backend’s public URL + /api/v1/payments/webhook.
You also make sure the webhook secret in your environment variables matches.

7. (Optional) Get a domain name and SSL

Platforms usually give you a something.onrender.com or something.netlify.app URL with automatic HTTPS (SSL). That’s fine for a test deployment. For a real product, you’d buy a domain like ohsremote.com and point it to your servers.

## How this applies
The previous team left a docker-compose.yml for local development only.

1. Choose a platform (Render.com is very beginner‑friendly – it supports both Docker containers for the backend and static sites for the frontend, plus a managed PostgreSQL – but you use MySQL. Railway.app is another good choice).

2. Prepare a production‑ready Dockerfile (the existing one likely works with small tweaks).

3. Set up a cloud MySQL database (e.g., Aiven, ClearDB, or the platform’s own MySQL service).

4. Deploy the backend as a web service.

5. Deploy the frontend as a static site.

6. Configure environment variables on the platform (all the STRIPE_*, OPENAI_API_KEY, SMTP_*, SECRET_KEY, DATABASE_URL).

7. Update the frontend’s API URL (you can either hardcode it temporarily for the test deployment or, as recommended, add a Vite environment variable like VITE_API_URL).

8. Update the Stripe webhook URL in the Stripe Dashboard.

9. Test the whole flow end‑to‑end on the live URLs.

## Short plan for a test deployment (Jennifer just needs to see it)
1. Sign up for Render.com (free tier).
2. Create a new Web Service – point it to your GitHub repo (Capstone347/ohs_backend).
3. Render will automatically detect the Dockerfile.
4. Add all environment variables from your .env.docker file.
5. Change DATABASE_URL to point to a Render MySQL database (they offer it as an add‑on).
6. Create a Static Site on Render – point it to your frontend repo, set build command npm run build, publish directory dist.
7. Under “Environment Variables”, add VITE_API_URL = the URL of your deployed backend.
8. Rebuild the frontend so it picks up that variable.
9. Update the Stripe webhook URL in the Stripe Dashboard to https://your-backend.onrender.com/api/v1/payments/webhook.
10. Give Jennifer the frontend URL (e.g., https://ohs-frontend.onrender.com).
