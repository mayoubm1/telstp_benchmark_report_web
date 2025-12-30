# Tawasol Life Sciences Technology Hub - Deployment Guide

This guide provides instructions for deploying the Tawasol Life Sciences Technology Hub web application, which is built using React, Vite, and deployed via Vercel.

## 1. Project Setup

The project is a standard React application.

1.  **Install Dependencies:** Navigate to the project root directory and install the required packages.
    ```bash
    cd tawasol-life-sciences-hub
    pnpm install
    # or npm install
    # or yarn install
    ```

2.  **Local Development:** To run the application locally for testing:
    ```bash
    pnpm run dev
    ```
    The application will typically be available at `http://localhost:5173`.

## 2. Supabase Integration

The application is configured to connect to your Supabase backend for real-time data (M2-3M content, etc.).

The credentials are set in `/src/utils/supabaseClient.js`:

*   **Supabase URL:** `https://vrfyjirddfdnwuffzqhb.supabase.co`
*   **Supabase Anon Key:** `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...`

**Action Required:** Ensure these credentials are also set as **Environment Variables** in your Vercel project settings for the production deployment.

| Variable Name | Value |
| :--- | :--- |
| `VITE_SUPABASE_URL` | `https://vrfyjirddfdnwuffzqhb.supabase.co` |
| `VITE_SUPABASE_ANON_KEY` | `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...` |

## 3. Deployment to Vercel (Recommended)

The project is already linked to your Vercel account and GitHub repository (`mayoubm1/tawasol-life-sciences-hub`).

1.  **Push to GitHub:** Ensure all your local changes are committed and pushed to the `main` branch of your repository.
    ```bash
    git add .
    git commit -m "Final local changes before deployment"
    git push origin main
    ```
    Vercel is configured to automatically build and deploy on every push to the `main` branch.

2.  **Manual Deployment (If needed):** If the automatic deployment fails, you can trigger a manual deployment using the Vercel CLI (if installed and logged in) or directly from the Vercel dashboard.
    ```bash
    # From the project root directory
    vercel deploy --prod
    ```

## 4. Final Checks

*   **Full-Screen Globe:** The globe is configured to consume the full screen.
*   **Microsites:** Clicking a hub should now open a dedicated microsite component (`GlobalHubPage` or `EgyptHubPage`).
*   **AI Assistant:** The Hayah AI assistant is included and ready for backend integration (using your provided Mistral API key).
*   **Thematic Visuals:** Egyptian thematic images and a video background are integrated for a stunning UI/UX.

If you encounter any issues, please refer to the Vercel deployment logs or contact support.
