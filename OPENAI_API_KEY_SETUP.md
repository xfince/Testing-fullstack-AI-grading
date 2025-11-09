# OpenAI API Key Setup Guide

## Problem
You're seeing this error in your grading workflow:
```
❌ Error evaluating: GPT API error: Connection error.
```

**Root Cause**: The `OPENAI_API_KEY` is not set in your GitHub repository secrets.

---

## Solution: Add OpenAI API Key to GitHub Secrets

### Step 1: Get Your OpenAI API Key

1. Go to https://platform.openai.com/api-keys
2. Sign in with your OpenAI account
3. Click **"Create new secret key"**
4. Give it a name like `grading-system-key`
5. **Copy the key immediately** (you won't be able to see it again!)
6. The key should look like: `sk-proj-...` or `sk-...`

### Step 2: Add the Key to GitHub Secrets

1. Go to your GitHub repository:
   ```
   https://github.com/xfince/Testing-fullstack-AI-grading
   ```

2. Click on **Settings** (in the top navigation bar)

3. In the left sidebar, click **Secrets and variables** → **Actions**

4. Click the **"New repository secret"** button

5. Fill in the form:
   - **Name**: `OPENAI_API_KEY`
   - **Secret**: Paste your OpenAI API key (e.g., `sk-proj-...`)

6. Click **"Add secret"**

### Step 3: Verify the Secret Was Added

1. Go back to **Settings** → **Secrets and variables** → **Actions**
2. You should see `OPENAI_API_KEY` listed
3. ✅ If you see it, you're done!

---

## Testing the Fix

### Option 1: Re-run the Failed Workflow

1. Go to **Actions** tab in your GitHub repository
2. Find the failed workflow run
3. Click **"Re-run all jobs"**
4. Wait for the workflow to complete
5. Check that GPT evaluation now shows:
   ```
   ✅ Score: X/4
   Total API Calls: 14
   Total Tokens: XXXX
   ```

### Option 2: Push a New Commit

1. Make any small change (e.g., update README.md)
2. Commit and push:
   ```bash
   git add .
   git commit -m "Test GPT evaluation fix"
   git push origin main
   ```
3. Go to **Actions** tab and watch the new workflow run

---

## Verifying It Works

When the API key is set correctly, you should see in the workflow logs:

```
🤖 Starting GPT-4o Evaluation...

📋 Evaluating 14 criteria with GPT-4o

📦 Processing Batch 1/5 (2 criteria)...
   Evaluating: Project Planning & Problem Definition
   ✅ Score: 3/4
   Evaluating: Documentation & README
   ✅ Score: 2.5/4

...

✨ GPT Evaluation Complete!

📊 Statistics:
   Total API Calls: 14
   Total Tokens: 11427
   Time: 91.65s
```

---

## Common Issues & Solutions

### Issue 1: "OPENAI_API_KEY not set in repository secrets"
**Solution**: Follow Step 2 above to add the secret

### Issue 2: "Authentication error: Invalid or missing OPENAI_API_KEY"
**Solutions**:
- The API key might be invalid or expired
- Go to https://platform.openai.com/api-keys and create a new key
- Make sure you copied the entire key (starts with `sk-`)
- Update the GitHub secret with the new key

### Issue 3: "Quota exceeded: Your OpenAI API account has insufficient credits"
**Solutions**:
- Go to https://platform.openai.com/account/billing
- Add credits to your account
- Check your usage limits

### Issue 4: "Rate limit error: Too many requests"
**Solutions**:
- Wait a few minutes and re-run the workflow
- The grading system now has automatic retry logic with exponential backoff
- If it persists, check your API usage limits

### Issue 5: Network/Connection Errors
**Solutions**:
- Check https://status.openai.com for any outages
- The system will automatically retry transient errors up to 3 times
- If errors persist, wait and try again later

---

## Cost Estimation

Each grading run evaluates 14 criteria and uses approximately:
- **11,000-15,000 tokens** per run
- Using GPT-4o: ~$0.03-$0.05 per student grading
- 100 students = ~$3-$5 total

Make sure your OpenAI account has sufficient credits!

---

## Security Notes

✅ **DO**:
- Store the API key in GitHub Secrets (encrypted)
- Never commit the API key to git
- Keep your API key private

❌ **DON'T**:
- Share your API key publicly
- Commit the key to the repository
- Store it in plain text files

---

## Improvements Made (November 9, 2025)

The grading system has been enhanced with:

1. **Better Error Messages**: Now shows specific errors:
   - Authentication errors (invalid key)
   - Quota exceeded errors
   - Rate limit errors
   - Network/connection errors
   - OpenAI server errors

2. **Automatic Retries**: Transient network errors are automatically retried:
   - Up to 3 attempts
   - Exponential backoff (2s, 4s, 8s delays)
   - Only retries network/server errors, not auth/quota errors

3. **Detailed Logging**: Full error details are logged for debugging:
   - Error type, code, status
   - HTTP response details
   - Helpful troubleshooting messages

---

## Need Help?

If you're still having issues:

1. Check the workflow logs for the specific error message
2. Verify the API key is correct at https://platform.openai.com/api-keys
3. Check your billing/quota at https://platform.openai.com/account/billing
4. Ensure the secret name is exactly `OPENAI_API_KEY` (case-sensitive)

---

## Quick Reference

| Step | Action | URL |
|------|--------|-----|
| 1 | Get API Key | https://platform.openai.com/api-keys |
| 2 | Add to GitHub | Settings → Secrets → Actions → New secret |
| 3 | Test | Actions → Re-run failed workflow |
| 4 | Check Billing | https://platform.openai.com/account/billing |
| 5 | Check Status | https://status.openai.com |

---

**Next**: After adding the secret, re-run your workflow and it should work! 🚀
