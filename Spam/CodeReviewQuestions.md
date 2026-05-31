# Code Review Questions - Starting Code

What is a Alembic?
- Alembic is a tool for managing changes to your database schema (its structure) over time
- You'll constantly need to add new tables, rename columns, or change data types. Alembic allows you to track these changes in migration scripts
- Git but for databases basicly

What is ngrok?
- A tool that creates a secure, public URL that tunnels directly to a web service running on your local computer
- Incredibly useful for:
    - **Testing webhooks:** Services like Stripe or GitHub need to send data to a public URL. ngrok gives you one for your local app to receive them.
    - **Sharing work-in-progress:** You can instantly share a demo of your local build with a client or teammate without deploying to a server.
    - **Debugging external APIs:** It allows you to inspect live traffic from the internet to your local machine.

What is Stripe?
- Stripe is a technology company 
- Provides a platform for businesses to accept payments over the internet
- Stripe handles virtually everything related to a financial transaction:
    - **Processing Credit Cards:** Safely handling card information.
    - **Managing Subscriptions:** Setting up recurring billing for services.
    - **Fraud Prevention:** Using machine learning (via their product, Radar) to identify and block suspicious payments.
    - **Sending Payouts:** Automatically transferring funds to sellers or service providers.
- Stripe is the engine that allows a web app to become a real business by moving money safely and reliably.