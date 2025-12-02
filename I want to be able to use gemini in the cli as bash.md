<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# I want to be able to use gemini in the cli as bash commands but need it to use the vertex AI model that I'm hosting in GCP in Belgium. Please give me instructions of how to set this up in my linux environment (github codespace).

Great! I have comprehensive information. Now let me provide you with detailed setup instructions for using Gemini CLI with your Vertex AI model in GCP Belgium.

## Set Up Gemini CLI with Vertex AI (GCP Belgium) in GitHub Codespaces

Here's the complete setup process:

### Step 1: Install Gemini CLI

```bash
npm install -g @google-gemini/gemini-cli
```

Or if you prefer using a package manager:

```bash
# Using curl
curl -L https://github.com/google-gemini/gemini-cli/releases/download/latest/gemini-cli-linux-x64 -o gemini
chmod +x gemini
sudo mv gemini /usr/local/bin/
```


### Step 2: Set Up GCP Authentication

You have three authentication options for Vertex AI. I recommend **Option A** (ADC) as it's most straightforward for Codespaces:

#### **Option A: Application Default Credentials (ADC) - Recommended**

1. **Authenticate with gcloud**:
```bash
gcloud init
gcloud auth application-default login
```

2. **Set your GCP project** (replace with your project ID):
```bash
gcloud config set project YOUR_GCP_PROJECT_ID
```

3. **Set environment variables** (add to `~/.bashrc` or create `.env` in your Codespace):
```bash
export GOOGLE_CLOUD_PROJECT="YOUR_GCP_PROJECT_ID"
export GOOGLE_CLOUD_LOCATION="europe-west1"  # Brussels/Belgium region
export GOOGLE_GENAI_USE_VERTEXAI=true
```

**Note**: For Belgium, use `europe-west1` as your location (or `europe-west1-b` for zone-specific).

#### **Option B: Service Account (For Automated Workflows)**

This is better for CI/CD or long-running processes:

1. **Create a service account in GCP Console**:
    - Go to IAM \& Admin → Service Accounts
    - Click "Create Service Account"
    - Grant it the "Vertex AI User" role (`roles/aiplatform.user`)
2. **Create and download a JSON key**:
    - Click on the service account
    - Go to "Keys" tab
    - Create a new JSON key
    - Download it to your Codespace
3. **Set the authentication environment variable**:
```bash
export GOOGLE_APPLICATION_CREDENTIALS="/path/to/service-account-key.json"
export GOOGLE_CLOUD_PROJECT="YOUR_GCP_PROJECT_ID"
export GOOGLE_CLOUD_LOCATION="europe-west1"
export GOOGLE_GENAI_USE_VERTEXAI=true
```


### Step 3: Make Environment Variables Persistent in GitHub Codespaces

Create or edit `~/.bashrc`:

```bash
cat >> ~/.bashrc << 'EOF'

# Gemini CLI + Vertex AI Configuration
export GOOGLE_CLOUD_PROJECT="YOUR_GCP_PROJECT_ID"
export GOOGLE_CLOUD_LOCATION="europe-west1"
export GOOGLE_GENAI_USE_VERTEXAI=true

# If using service account (optional):
# export GOOGLE_APPLICATION_CREDENTIALS="/workspace/path/to/service-account-key.json"
EOF

# Reload bashrc
source ~/.bashrc
```


### Step 4: Verify Setup

```bash
# Check gcloud is authenticated
gcloud auth list

# Check your project and location are set
gcloud config list

# Test Gemini CLI with your Vertex AI model
gemini "What is 2+2?"
```


### Step 5: Use Gemini in Bash Commands

Now you can use Gemini as bash commands:

```bash
# Simple query
gemini "Explain OAuth2 flow"

# Piping output to Gemini
cat mycode.py | gemini "Review this Python code and suggest improvements"

# Using Gemini for code generation
gemini "Write a Python function to parse JSON files"

# Debugging with Gemini
gemini "I'm getting this error: $(python myscript.py 2>&1). How do I fix it?"

# Interactive mode
gemini chat

# Using with files
gemini "Summarize this: $(cat document.txt)"
```


### Step 6 (Optional): Create Bash Aliases for Common Tasks

Add to your `~/.bashrc`:

```bash
# Quick Gemini alias
alias ai="gemini"

# Code review with Gemini
alias review="gemini 'Review this code and suggest improvements:'"

# Generate documentation
alias docs="gemini 'Generate comprehensive documentation for:'"

# Debug helper
alias debug="gemini 'This error occurred:'"
```

Then use them like:

```bash
ai "explain closures in JavaScript"
cat myfunction.py | review
```


### Troubleshooting

**Issue: "API keys are not supported by this API"**

- Solution: Use ADC (Option A) or Service Account (Option B) instead of API keys

**Issue: Location errors or model not found**

```bash
# List available models in your region
gcloud ai models list --location=europe-west1

# Verify your location supports Gemini
gcloud compute regions list | grep europe
```

**Issue: Permission denied**

```bash
# Grant yourself Vertex AI User role
gcloud projects add-iam-policy-binding YOUR_PROJECT_ID \
  --member=user:YOUR_EMAIL@gmail.com \
  --role=roles/aiplatform.user
```

**Issue: Codespace loses environment variables between sessions**

- Use the `.env` file in your Codespace root
- Or add to `.devcontainer.json` environment variables section
- Or update `~/.bashrc` (persists between sessions by default in newer Codespaces)


### For Advanced Use: Direct Bash Functions

You can also create a wrapper function for more control:

```bash
cat >> ~/.bashrc << 'EOF'

# Advanced Gemini wrapper with model selection
gemini_ask() {
  local prompt="$@"
  local model="${GEMINI_MODEL:-gemini-2.5-flash}"
  
  curl -X POST \
    -H "Authorization: Bearer $(gcloud auth print-access-token)" \
    -H "Content-Type: application/json" \
    "https://europe-west1-aiplatform.googleapis.com/v1/projects/${GOOGLE_CLOUD_PROJECT}/locations/${GOOGLE_CLOUD_LOCATION}/publishers/google/models/${model}:generateContent" \
    -d "{
      \"contents\": {
        \"role\": \"user\",
        \"parts\": [{\"text\": \"${prompt}\"}]
      }
    }" | jq -r '.candidates[^0].content.parts[^0].text'
}

EOF
source ~/.bashrc
```

Then use it: `gemini_ask "Your question here"`

### Key Points for Your Setup

✅ **Belgium Region**: Use `europe-west1` (Liège, Belgium region for GCP)
✅ **Vertex AI vs Gemini API**: You're using Vertex AI (correct choice for your hosted model)
✅ **Model Format**: Specify your custom model or use `gemini-2.5-flash`, `gemini-2.0-pro`, etc.
✅ **Token Caching**: Gemini CLI automatically optimizes tokens for your region
✅ **Persistence**: Environment variables survive Codespaces stops/starts if added to `~/.bashrc`

Need help with custom model endpoint routing or workflow automation next?
<span style="display:none">[^1][^10][^11][^12][^13][^14][^15][^16][^17][^18][^19][^2][^20][^3][^4][^5][^6][^7][^8][^9]</span>

<div align="center">⁂</div>

[^1]: https://geminicli.com/docs/get-started/authentication/

[^2]: https://docs.ag2.ai/latest/docs/user-guide/models/google-vertexai/

[^3]: https://docs.cloud.google.com/vertex-ai/docs/authentication

[^4]: https://docs.cloud.google.com/vertex-ai/generative-ai/docs/start/quickstart

[^5]: https://ai.google.dev/gemini-api/docs/migrate-to-cloud

[^6]: https://stackoverflow.com/questions/76223871/how-to-get-authorization-token-for-googles-vertex-ai-service

[^7]: https://ai-sdk.dev/providers/ai-sdk-providers/google-vertex

[^8]: https://cloud.google.com/vertex-ai

[^9]: https://aicodingtools.blog/en/gemini-cli/cli/authentication

[^10]: https://codelabs.developers.google.com/gemini-cli-deep-dive

[^11]: https://github.com/google-gemini/gemini-cli/blob/main/docs/get-started/authentication.md

[^12]: https://docs.cloud.google.com/vertex-ai/docs/training/neural-architecture-search/environment-setup

[^13]: https://docs.cloud.google.com/gemini/docs/codeassist/gemini-cli

[^14]: https://codelabs.developers.google.com/gemini-cli-hands-on

[^15]: https://discuss.google.dev/t/authentication-issue-with-google-vertex-ai-api/140886

[^16]: https://cloud.google.com/blog/topics/developers-practitioners/gemini-cli-custom-slash-commands

[^17]: https://www.cloudskillsboost.google/paths/280/course_templates/892/video/425580?locale=it

[^18]: https://github.com/google-gemini/gemini-cli

[^19]: https://console.cloud.google.com/vertex-ai/publishers/google/model-garden/gemini-2.5-pro

[^20]: https://codeassist.google

