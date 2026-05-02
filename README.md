# my-npm-app-jfog-demo

cloude chat: https://claude.ai/share/27a1ff10-b29f-4374-a64e-2ec9ef6e0d3a

markdown# my-app — JFrog Artifactory CI/CD Pipeline

A simple Node.js project that builds and pushes artifacts to JFrog Artifactory using GitHub Actions.

---

## Project Structure
my-app/
├── .github/
│   └── workflows/
│       └── build-push.yml
├── src/
│   └── index.js
├── package.json
└── README.md

---

## Prerequisites

- GitHub repository
- JFrog Artifactory instance running at `http://52.20.197.39:8082`
- Artifactory local repository named `generic-local` (Generic type)
- Artifactory Access Token generated from your Artifactory profile

---

## JFrog Artifactory Setup

### 1. Create Local Repository
1. Open `http://52.20.197.39:8082`
2. Login with admin credentials
3. Go to **Administration → Repositories → New Local Repository**
4. Select **Generic** as package type
5. Set Repository Key as `generic-local`
6. Click **Save & Finish**

### 2. Generate Access Token
1. Click your profile icon (top right)
2. Go to **Edit Profile**
3. Under **Identity Tokens**, click **Generate Token**
4. Copy the full token (long JWT string starting with `eyJ...`)

---

## GitHub Secrets Setup

Go to your GitHub repository → **Settings → Secrets and variables → Actions → New repository secret**

Add the following secrets:

| Secret Name | Value |
|---|---|
| `JFROG_URL` | `http://52.20.197.39:8082` |
| `JFROG_ACCESS_TOKEN` | Your copied Artifactory token |

---

## Project Files

### src/index.js
```js
console.log("Hello from my-app v1.0.0");
```

### package.json
```json
{
  "name": "my-app",
  "version": "1.0.0",
  "description": "Sample Node.js app with JFrog Artifactory CI/CD",
  "main": "src/index.js",
  "scripts": {
    "build": "npm pack",
    "test": "echo 'Tests passed!'"
  },
  "author": "",
  "license": "MIT"
}
```

### .github/workflows/build-push.yml
```yaml
name: Build & Push to JFrog Artifactory

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  build-and-push:
    name: Build and Publish to Artifactory
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Setup JFrog CLI
        uses: jfrog/setup-jfrog-cli@v4
        env:
          JF_URL: ${{ secrets.JFROG_URL }}
          JF_ACCESS_TOKEN: ${{ secrets.JFROG_ACCESS_TOKEN }}

      - name: Ping Artifactory
        run: jf rt ping

      - name: Install dependencies
        run: npm install

      - name: Run tests
        run: npm test

      - name: Build package
        run: npm pack

      - name: Upload artifact to Artifactory
        run: |
          jf rt upload \
            "*.tgz" \
            "generic-local/my-app/${{ github.run_number }}/" \
            --build-name="my-app" \
            --build-number="${{ github.run_number }}"

      - name: Publish Build Info
        run: |
          jf rt build-collect-env "my-app" "${{ github.run_number }}"
          jf rt build-publish "my-app" "${{ github.run_number }}"

      - name: Print artifact location
        run: |
          echo "Artifact uploaded to:"
          echo "${{ secrets.JFROG_URL }}/artifactory/generic-local/my-app/${{ github.run_number }}/"
```

---

## Push to GitHub

```bash
# Initialize git repo
git init

# Add all files
git add .

# Commit
git commit -m "Initial commit with JFrog Artifactory CI/CD pipeline"

# Connect to your GitHub repository
git remote add origin https://github.com/<your-username>/my-app.git

# Push to main branch
git branch -M main
git push -u origin main
```

---

## Verify Pipeline

1. Go to your GitHub repository
2. Click the **Actions** tab
3. You should see the workflow **Build & Push to JFrog Artifactory** running
4. Wait for all steps to go green

---

## Verify Artifact in Artifactory

1. Open `http://52.20.197.39:8082`
2. Go to **Artifactory → Artifacts**
3. Browse to `generic-local → my-app → <build-number>`
4. You should see `my-app-1.0.0.tgz` uploaded successfully

---

## Artifact Path in Artifactory
generic-local/
└── my-app/
└── 1/
└── my-app-1.0.0.tgz

---

## Notes

- Every push to `main` branch triggers the pipeline automatically
- Build number increments with every GitHub Actions run
- Access Token does not require a username — token alone is sufficient for authentication
- Make sure port `8082` is open in your EC2 Security Group inbound rules for the pipeline to reach Artifactory