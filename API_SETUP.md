# 🔑 API Configuration Setup

**IMPORTANT:** You need a Google Gemini API key to use this application.

## Steps to Configure:

### 1. Get your Gemini API Key
- Go to: https://makersuite.google.com/app/apikey
- Sign in with your Google account
- Click "Create API Key"
- Copy the generated key

### 2. Create .env file
Create a file named `.env` (without .example) in the project root with:

```env
GEMINI_API_KEY=paste_your_api_key_here
GEMINI_MODEL=gemini-2.0-flash
```

### 3. Replace the placeholder
Replace `paste_your_api_key_here` with your actual API key from step 1.

### 4. Restart the application
After creating .env, restart the server:

```bash
# Stop current server (Ctrl+C)
# Then run:
python main.py
```

## Verification

When the API key is configured correctly, you'll see:
```
✅ Gemini API client initialized with model: gemini-2.0-flash
```

When NOT configured, you'll see:
```
⚠️  WARNING: GEMINI_API_KEY not set! Will use mock data.
```

## Security Note

- Never commit your `.env` file to Git
- The `.env` file is already in `.gitignore`
- Keep your API key private
