# How To Deploy

1. **Code Readiness & Preparation**

Before deployment, the code must be production-ready: (reference `/StartingCode.md` file)

Environment Configuration: Move all secrets (API keys, database credentials) to environment variables. Backend uses a `.env` file for local development and reads configuration through `pydantic`-settings. Never commit this file to Git.

Build the Frontend: Run `npm run build` to create an optimized `dist` folder containing the static React application.

Test Locally: Ensure everything works as expected using `npm run preview` for the frontend and `uvicorn main:app --reload` for the backend.

Version Control: Push the final code to Git repository (most cloud platforms integrate directly with these services).

------

2. **Choose a Cloud Service Provider**

Select a provider that fits needs in terms of cost, scalability, and management overhead.

General Guidance: For a full-stack web app, need a static hosting solution for the React frontend and a compute platform for the FastAPI backend, plus a managed database for MySQL.

------

3. **Provision Database (First Step in Cloud)**
Set up the database first so that there is a connection string ready for the backend.

Create a MySQL instance on the chosen cloud provider. Copy the internal connection string (e.g., `DATABASE_URL`) provided by the service, which often includes username, password, and hostname.

Run Migrations: will need to execute the Alembic migrations on this production database. Some platforms do this automatically on deploy via a `startCommand`, while others may require you to run it manually.

------

4. **Deploy the FastAPI Backend**

Platforms: can deploy to a PaaS (e.g., Render, Railway, Heroku), a Container Service (e.g., AWS ECS, Google Cloud Run), or a Virtual Machine (most control, e.g., DigitalOcean Droplet, AWS EC2).

Steps:

Connect Repository: Link the GitHub repository containing the backend code.

Configure Build Settings: For a Python/FastAPI app, typically specify a build command (`pip install -r requirements.txt`) and a start command (`uvicorn main:app --host 0.0.0.0 --port $PORT`). The use of `$PORT` is critical, as cloud platforms assign a port dynamically.

Set Environment Variables: In the cloud dashboard, add all the variables from the `.env` file, including the `DATABASE_URL` from step 3, Stripe keys, OpenAI keys, etc.

Deploy: Initiate the deployment. Most platforms will automatically redeploy when you push new code to the connected branch.

------

5. **Deploy the React Frontend**

Platforms: Use a Static Web Hosting service like Netlify, Vercel, AWS S3+CloudFront, or a PaaS that supports static sites.

Steps:

Connect Repository: Link the GitHub repository with the frontend code.

Configure Build Settings: Set the Build Command to `npm run build` and the Publish Directory to the output folder (usually `dist` for Vite projects).

Set Environment Variables: If the frontend uses environment variables (prefixed with VITE_), add them here.

Deploy: The platform will build and deploy the site, providing you with a public URL

------

6. **Connect Frontend to Backend & Finalize**

Update API URL: In the frontend environment variables, set the `VITE_API_BASE_URL` to the public URL of the deployed FastAPI backend.

Trigger Rebuild: After changing this variable, trigger a new deployment of the frontend so it can build with the correct API endpoint.

Configure CORS: In the FastAPI backend, ensure the `CORS_ORIGINS` environment variable includes the URL of the deployed frontend to allow requests from the domain.

Configure Custom Domain: Configure a custom domain for both the frontend and backend in the cloud provider's dashboard.

