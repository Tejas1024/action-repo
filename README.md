# GitHub Webhook Monitor - webhook-repo

This Flask application receives GitHub webhook events, stores them in MongoDB, and displays them in a real-time UI.

## Features 

- ✅ Receives GitHub webhooks for Push, Pull Request, and Merge events
- ✅ Stores events in MongoDB with proper schema
- ✅ Auto-refreshing UI that polls every 15 seconds
- ✅ Clean and minimal design
- ✅ Proper error handling and data validation
- ✅ Prevents duplicate event display

## Prerequisites

- Python 3.8 or higher
- MongoDB (local installation or MongoDB Atlas account)
- Git
- ngrok (for local development to expose webhook endpoint)

## Installation

### 1. Clone the repository

```bash
git clone <your-webhook-repo-url>
cd webhook-repo
```

### 2. Create a virtual environment

```bash
# Windows
python -m venv venv
venv\Scripts\activate

# Linux/Mac
python3 -m venv venv
source venv/bin/activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Configure environment variables

Create a `.env` file in the root directory:

```bash
cp .env.example .env
```

Edit `.env` and add your MongoDB URI:

```
MONGO_URI=mongodb://localhost:27017/github_webhooks
```

**For MongoDB Atlas:**
```
MONGO_URI=mongodb+srv://username:password@cluster.mongodb.net/github_webhooks
```

## MongoDB Setup

### Option 1: Local MongoDB

1. Install MongoDB Community Edition from https://www.mongodb.com/try/download/community
2. Start MongoDB service:
   ```bash
   # Windows
   net start MongoDB
   
   # Linux
   sudo systemctl start mongod
   
   # Mac
   brew services start mongodb-community
   ```

### Option 2: MongoDB Atlas (Cloud - Recommended)

1. Go to https://www.mongodb.com/cloud/atlas/register
2. Create a free account
3. Create a new cluster (Free tier M0)
4. Go to Database Access → Add New Database User
   - Create username and password
   - Set privileges to "Read and write to any database"
5. Go to Network Access → Add IP Address
   - Click "Allow Access from Anywhere" (for development)
   - Add `0.0.0.0/0`
6. Go to Databases → Connect → Connect your application
   - Copy the connection string
   - Replace `<password>` with your database user password
   - Update your `.env` file

## Running the Application

### 1. Start the Flask application

```bash
python app.py
```

The application will run on `http://localhost:5000`

### 2. Expose your local server using ngrok

In a new terminal:

```bash
ngrok http 5000
```

You'll see output like:
```
Forwarding  https://abc123.ngrok.io -> http://localhost:5000
```

Copy the `https://` URL - this is your webhook URL.

## Setting up GitHub Webhooks

1. Go to your `action-repo` on GitHub
2. Navigate to **Settings** → **Webhooks** → **Add webhook**
3. Configure the webhook:
   - **Payload URL**: `https://your-ngrok-url.ngrok.io/webhook`
   - **Content type**: `application/json`
   - **Secret**: Leave empty (or add one if you want to validate webhooks)
   - **Which events**: Select individual events
     - ✅ Pushes
     - ✅ Pull requests
   - **Active**: ✅ Checked
4. Click **Add webhook**

## Testing the Application

### 1. View the UI

Open your browser and go to: `http://localhost:5000`

### 2. Test PUSH event

In your `action-repo`:
```bash
echo "Test commit" >> test.txt
git add .
git commit -m "Test push event"
git push origin main
```

### 3. Test PULL REQUEST event

```bash
# Create a new branch
git checkout -b feature-branch

# Make changes
echo "Feature" >> feature.txt
git add .
git commit -m "Add feature"
git push origin feature-branch

# Create PR on GitHub
# Go to your repo → Pull requests → New pull request
# Select feature-branch → main
# Create pull request
```

### 4. Test MERGE event

1. Go to your Pull Request on GitHub
2. Click "Merge pull request"
3. Confirm the merge

### 5. Check the UI

The UI should automatically refresh every 15 seconds and display:
- Push events
- Pull request events
- Merge events

## Project Structure

```
webhook-repo/
├── app.py                 # Main Flask application
├── requirements.txt       # Python dependencies
├── .env                  # Environment variables (not in git)
├── .env.example          # Example environment file
├── .gitignore            # Git ignore file
├── README.md             # This file
└── templates/
    └── index.html        # UI template
```

## API Endpoints

### POST /webhook
Receives GitHub webhook events

**Headers:**
- `X-GitHub-Event`: Event type (push, pull_request, etc.)

**Response:**
```json
{
  "status": "success",
  "message": "Event stored successfully"
}
```

### GET /api/events
Fetches events from MongoDB

**Query Parameters:**
- `last_timestamp` (optional): ISO format timestamp to fetch only newer events

**Response:**
```json
{
  "status": "success",
  "events": [
    {
      "_id": "...",
      "author": "username",
      "action": "push",
      "to_branch": "main",
      "from_branch": null,
      "timestamp": "2026-01-28T10:30:00.000Z"
    }
  ]
}
```

### GET /
Serves the main UI

## MongoDB Schema

```javascript
{
  _id: ObjectId,
  author: String,
  action: String,        // "push", "pull_request", or "merge"
  from_branch: String,   // null for push events
  to_branch: String,
  timestamp: Date
}
```

## Troubleshooting

### Webhooks not being received

1. Check if ngrok is running
2. Verify the webhook URL in GitHub settings
3. Check ngrok web interface at `http://127.0.0.1:4040` for incoming requests
4. Look at Flask console for errors

### MongoDB connection issues

1. Verify MongoDB is running (local) or connection string is correct (Atlas)
2. Check `.env` file has correct `MONGO_URI`
3. For Atlas: ensure IP whitelist includes `0.0.0.0/0`

### UI not updating

1. Open browser console (F12) and check for JavaScript errors
2. Verify `/api/events` endpoint is working
3. Check if events are being stored in MongoDB

### Events not showing in correct format

1. Check the webhook payload in ngrok interface
2. Verify the parsing logic in `app.py`
3. Check browser console for formatting errors

## Deployment Options

### Heroku

```bash
# Install Heroku CLI and login
heroku login

# Create app
heroku create your-app-name

# Add MongoDB addon
heroku addons:create mongolab:sandbox

# Deploy
git push heroku main
```

### Railway

1. Go to https://railway.app
2. Connect your GitHub repository
3. Add MongoDB database
4. Deploy automatically

### Render

1. Go to https://render.com
2. Create new Web Service
3. Connect your GitHub repository
4. Add environment variables
5. Deploy

## Important Notes

- The UI polls every 15 seconds as required
- Only new events are displayed (no duplicates)
- Events are shown in reverse chronological order (newest first)
- Merge events are captured when PR is closed with `merged: true`
- All timestamps are in UTC format with proper formatting

## Demo Video Checklist

When recording your demo video, show:

1. ✅ GitHub repository with webhooks configured
2. ✅ Flask application running
3. ✅ MongoDB (show connection or Atlas dashboard)
4. ✅ UI displaying events
5. ✅ Creating a push event and seeing it appear in UI
6. ✅ Creating a pull request and seeing it in UI
7. ✅ Merging the PR and seeing merge event in UI
8. ✅ Auto-refresh working (wait 15+ seconds)
9. ✅ Code walkthrough (app.py structure)
10. ✅ Show no duplicate events appearing

## Support

For issues or questions:
- Check GitHub webhook delivery status in repo settings
- Check ngrok request inspector at `http://127.0.0.1:4040`
- Review Flask console logs
- Check MongoDB for stored events

## License

This project is created for Techstax Developer Assessment Task.
