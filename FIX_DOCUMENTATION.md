# Test Hang Fix Documentation

## Problem Identified

The grading tests were hanging for 59 minutes before GitHub Actions cancelled the workflow. 

### Root Cause

The issue was in `grading-folder/backend/server.js` (line 37):

```javascript
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

When the test file (`tests/backend/api-endpoints.test.js`) tried to `require()` the server.js file to get the Express app for testing, **the entire file executed**, including the `app.listen()` call. This caused the server to start listening on a port, which made the `require()` call hang indefinitely because the server kept running.

### Timeline
- **10:32:48** - Test started: "📋 Running: API Endpoints"
- **10:32:48 - 11:31:55** - Test hung for ~59 minutes in `beforeAll()` hook
- **11:31:55** - GitHub Actions hit 60-minute job timeout and cancelled
- **Result**: Score 0/64, orphaned node processes (PIDs 3251, 3271, 3259)

## Solution Applied

### 1. Fixed `grading-folder/backend/server.js`

Changed the server file to only start the server when run directly, not when imported:

```javascript
const PORT = process.env.PORT || 5000;

// Only start server if this file is run directly (not imported for testing)
if (require.main === module) {
  app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
  });
}

// Export app for testing
module.exports = app;
```

**How it works:**
- `require.main === module` is `true` only when the file is run directly (e.g., `node server.js`)
- `require.main === module` is `false` when the file is imported (e.g., `require('./server.js')`)
- This allows tests to import the app without starting the server

### 2. Enhanced `tests/backend/api-endpoints.test.js`

Added better error logging to help debug future issues:

```javascript
console.log(`🔍 Attempting to load: ${appPath}`);
app = require(appPath);
console.log(`✅ Loaded app from: ${appPath}`);
```

If loading fails:
```javascript
console.warn(`⚠️  Failed to load ${appPath}: ${err.message}`);
```

## Why This Is Important

This is a **classic Node.js/Express testing pattern**. Students must:

1. **Export the Express app** without starting the server
2. **Start the server conditionally** only when the file is run directly
3. This allows:
   - Running the server normally: `node server.js`
   - Testing the server: `require('./server.js')` in tests

## Testing the Fix

To verify the fix works:

```bash
# Run the tests
npm test tests/backend/api-endpoints.test.js
```

The test should now:
- ✅ Load the app successfully
- ✅ Complete within 30 seconds (TEST_TIMEOUT)
- ✅ Not hang or create orphan processes

## Prevention

**For Future Students:**

All Express server files MUST follow this pattern:

```javascript
// ... app setup code ...

if (require.main === module) {
  const PORT = process.env.PORT || 5000;
  app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
  });
}

module.exports = app;
```

**For Grading System:**

The test file now has:
- ✅ Timeout protection (`beforeAll(..., TEST_TIMEOUT)`)
- ✅ Better error messages with emojis for visibility
- ✅ Detailed logging of which path was loaded
- ✅ Graceful handling when app fails to load

## Files Modified

1. ✅ `grading-folder/backend/server.js` - Added conditional server start + module export
2. ✅ `tests/backend/api-endpoints.test.js` - Enhanced error logging

## Result

- Tests will no longer hang indefinitely
- Clear error messages help identify issues quickly
- Students can run their servers normally AND test them
- GitHub Actions won't timeout waiting for hung tests
