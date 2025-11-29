# XDK Python SDK

The official Python SDK for the X API (formerly Twitter API). This library provides a simple, intuitive interface to interact with X API v2 endpoints, featuring generated endpoint coverage, OAuth2 PKCE authentication, cursor-based pagination, and real-time streaming support.

[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Version](https://img.shields.io/badge/version-0.2.5--beta-green.svg)](https://github.com/xdevplatform/xdk)

## Features

- **Full X API v2 Coverage** — Access posts, users, spaces, direct messages, lists, communities, and more
- **Auto-Generated Coverage** — Lightweight clients for every documented X API v2 category
- **Lightweight Models** — Pydantic stubs that accept/return plain dictionaries (see “Working with Models” below)
- **Multiple Authentication Methods** — Support for Bearer Token and OAuth2 PKCE flows
- **Cursor-Based Pagination** — Elegant iteration over paginated results with `.pages()` and `.items()` methods
- **Real-Time Streaming** — Native support for streaming endpoints with generator-based iteration
- **Automatic Token Refresh** — OAuth2 tokens are automatically refreshed when expired

## Installation

Install using [uv](https://github.com/astral-sh/uv) (recommended):

```bash
uv add xdk
```

Or using pip:

```bash
pip install xdk
```

## Quick Start

### Bearer Token Authentication

For app-only authentication with read access:

```python
from xdk import Client

client = Client(bearer_token="YOUR_BEARER_TOKEN")

# Get a post by ID
post = client.posts.get_by_id(id="1234567890")
print(post)

# Search recent posts
results = client.posts.search_recent(query="python programming")
print(results)

# Get user by username
user = client.users.get_by_username(username="elonmusk")
print(user)
```

### OAuth2 PKCE Authentication

For user-context authentication with full read/write access:

```python
from xdk import Client

# Initialize with OAuth2 credentials
client = Client(
    client_id="YOUR_CLIENT_ID",
    client_secret="YOUR_CLIENT_SECRET",  # Optional for public clients
    redirect_uri="https://your-app.com/callback",
    scope="tweet.read tweet.write users.read offline.access"
)

# Get authorization URL for user to visit
auth_url, state = client.get_authorization_url()
print(f"Please visit: {auth_url}")

# After user authorizes, exchange the callback URL for tokens
token = client.fetch_token(authorization_response="https://your-app.com/callback?code=...")

# Now you can make authenticated requests
me = client.users.get_me()
print(f"Authenticated as: {me}")

# Create a post
response = client.posts.create(body={"text": "Hello from XDK!"})
```

### Using an Existing Token

```python
from xdk import Client

# Use a previously saved token
token = {
    "access_token": "...",
    "refresh_token": "...",
    "expires_at": 1234567890
}

client = Client(
    client_id="YOUR_CLIENT_ID",
    token=token
)

# Token will be automatically refreshed when expired
```

### Token Management

The client provides several properties and methods for managing OAuth2 tokens:

```python
from xdk import Client

client = Client(
    client_id="YOUR_CLIENT_ID",
    token=existing_token
)

# Access the current token dictionary
current_token = client.token
print(f"Token expires at: {current_token['expires_at']}")

# Get just the access token string
access_token = client.access_token

# Check if the token is expired
if client.is_token_expired():
    print("Token has expired")

# Manually refresh the token (usually automatic)
new_token = client.refresh_token()

# Access the underlying OAuth2 session for advanced use cases
oauth2_session = client.oauth2_session
```

### Working with Models

The SDK’s request and response classes are generated directly from the OpenAPI schema. For now they act as flexible stubs: they permit any fields and expose responses as untyped dictionaries. In day-to-day usage:

- Provide request bodies as plain dictionaries (or objects you convert with `dict()`).
- Expect response objects to behave like JSON/dict payloads.
- Add your own Pydantic models if you need strict validation.

The code samples below follow this pattern so they mirror actual runtime behaviour.

## API Reference

### Available Clients

The SDK is organized into sub-clients for each API category:

| Client | Description |
|--------|-------------|
| `client.posts` | Create, retrieve, delete, and search posts |
| `client.users` | User lookup, followers, following, blocking, muting, bookmarks |
| `client.direct_messages` | Send and receive direct messages |
| `client.spaces` | Twitter Spaces operations |
| `client.lists` | Create and manage lists |
| `client.communities` | Community operations |
| `client.community_notes` | Community Notes (Birdwatch) data |
| `client.trends` | Trending topics |
| `client.media` | Media upload and metadata |
| `client.stream` | Real-time streaming endpoints |
| `client.compliance` | Compliance batch jobs |
| `client.webhooks` | Account Activity API webhooks |
| `client.account_activity` | Account activity subscriptions |
| `client.usage` | API usage statistics |
| `client.news` | News-related endpoints |
| `client.connections` | User connections |
| `client.activity` | User activity data |
| `client.general` | General API information |

### Posts Operations

```python
# Get a single post
post = client.posts.get_by_id(
    id="1234567890",
    tweet_fields=["created_at", "public_metrics", "author_id"],
    expansions=["author_id"],
    user_fields=["username", "profile_image_url"]
)

# Get multiple posts
posts = client.posts.get_by_ids(ids=["123", "456", "789"])

# Search recent posts (last 7 days)
results = client.posts.search_recent(
    query="from:elonmusk",
    max_results=100,
    tweet_fields=["created_at", "public_metrics"]
)

# Search all posts (full archive - requires Academic Research access)
results = client.posts.search_all(query="python lang:en")

# Create a post (requires OAuth2)
response = client.posts.create(body={"text": "Hello, World!"})

# Delete a post (requires OAuth2)
client.posts.delete(id="1234567890")

# Get post counts
counts = client.posts.get_counts_recent(query="python", granularity="day")

# Get users who liked a post
liking_users = client.posts.get_liking_users(id="1234567890")

# Get users who reposted
reposted_by = client.posts.get_reposted_by(id="1234567890")

# Get quote tweets
quotes = client.posts.get_quoted(id="1234567890")

# Get post analytics (requires OAuth2)
analytics = client.posts.get_analytics(
    ids=["123", "456"],
    start_time="2024-01-01T00:00:00Z",
    end_time="2024-01-31T23:59:59Z",
    granularity="day"
)

# Get 28-hour insights (requires OAuth2)
insights = client.posts.get_insights28hr(
    tweet_ids=["123", "456"],
    granularity="hour",
    requested_metrics=["impressions", "engagements"]
)

# Get historical insights (requires OAuth2)
historical = client.posts.get_insights_historical(
    tweet_ids=["123", "456"],
    start_time="2024-01-01T00:00:00Z",
    end_time="2024-01-31T23:59:59Z",
    granularity="day",
    requested_metrics=["impressions", "engagements"]
)

# Get full archive post counts (requires Academic Research access)
counts_all = client.posts.get_counts_all(query="python", granularity="day")

# Get reposts of a post
reposts = client.posts.get_reposts(id="1234567890")

# Hide a reply (requires OAuth2)
client.posts.hide_reply(tweet_id="1234567890", body={"hidden": True})
```

### Users Operations

```python
# Get user by ID
user = client.users.get_by_id(id="12345")

# Get user by username
user = client.users.get_by_username(username="elonmusk")

# Get multiple users by IDs
users = client.users.get_by_ids(ids=["123", "456", "789"])

# Get multiple users by usernames
users = client.users.get_by_usernames(usernames=["elonmusk", "jack"])

# Get authenticated user (requires OAuth2)
me = client.users.get_me()

# Get user's followers
followers = client.users.get_followers(id="12345", max_results=100)

# Get users that a user is following
following = client.users.get_following(id="12345")

# Get user's posts
posts = client.users.get_posts(id="12345", max_results=50)

# Get user's timeline (requires OAuth2)
timeline = client.users.get_timeline(id="12345")

# Get user's mentions
mentions = client.users.get_mentions(id="12345")

# Get user's liked posts
liked = client.users.get_liked_posts(id="12345")

# Follow/unfollow users (requires OAuth2)
client.users.follow_user(id="MY_USER_ID", body={"target_user_id": "12345"})
client.users.unfollow_user(source_user_id="MY_USER_ID", target_user_id="12345")

# Mute/unmute users (requires OAuth2)
client.users.mute_user(id="MY_USER_ID", body={"target_user_id": "12345"})
client.users.unmute_user(source_user_id="MY_USER_ID", target_user_id="12345")

# Bookmarks (requires OAuth2)
bookmarks = client.users.get_bookmarks(id="MY_USER_ID")
client.users.create_bookmark(id="MY_USER_ID", body={"tweet_id": "12345"})
client.users.delete_bookmark(id="MY_USER_ID", tweet_id="12345")

# Like/unlike posts (requires OAuth2)
client.users.like_post(id="MY_USER_ID", body={"tweet_id": "12345"})
client.users.unlike_post(id="MY_USER_ID", tweet_id="12345")

# Repost/unrepost (requires OAuth2)
client.users.repost_post(id="MY_USER_ID", body={"tweet_id": "12345"})
client.users.unrepost_post(id="MY_USER_ID", source_tweet_id="12345")

# Search users (requires OAuth2)
search_results = client.users.search(query="python developer", max_results=50)

# Get blocked users (requires OAuth2)
blocked = client.users.get_blocking(id="MY_USER_ID")

# Get muted users (requires OAuth2)
muted = client.users.get_muting(id="MY_USER_ID")

# Block/unblock DMs (requires OAuth2)
client.users.block_dms(id="USER_ID")
client.users.unblock_dms(id="USER_ID")

# Get reposts of your content (requires OAuth2)
reposts_of_me = client.users.get_reposts_of_me()

# Lists operations
owned_lists = client.users.get_owned_lists(id="12345")
followed_lists = client.users.get_followed_lists(id="12345")
list_memberships = client.users.get_list_memberships(id="12345")

# Follow/unfollow lists (requires OAuth2)
client.users.follow_list(id="MY_USER_ID", body={"list_id": "LIST_ID"})
client.users.unfollow_list(id="MY_USER_ID", list_id="LIST_ID")

# Pin/unpin lists (requires OAuth2)
pinned = client.users.get_pinned_lists(id="MY_USER_ID")
client.users.pin_list(id="MY_USER_ID", body={"list_id": "LIST_ID"})
client.users.unpin_list(id="MY_USER_ID", list_id="LIST_ID")

# Bookmark folders (requires OAuth2)
folders = client.users.get_bookmark_folders(id="MY_USER_ID")
folder_bookmarks = client.users.get_bookmarks_by_folder_id(id="MY_USER_ID", folder_id="FOLDER_ID")
```

### Pagination

The SDK provides elegant cursor-based pagination through the `Cursor` class:

```python
from xdk import Client, Cursor, cursor

client = Client(bearer_token="YOUR_BEARER_TOKEN")

# Iterate over pages of results
for page in Cursor(client.users.get_followers, "12345", max_results=100).pages(limit=5):
    print(f"Got {len(page.data)} followers")

# Iterate over individual items
for user in Cursor(client.users.get_followers, "12345", max_results=100).items(limit=500):
    print(f"Follower: {user}")

# Using the cursor factory function for better type inference
followers_cursor = cursor(client.users.get_followers, "12345", max_results=100)
for follower in followers_cursor.items(250):
    print(follower)

# Paginate search results
for post in Cursor(client.posts.search_recent, query="python").items(limit=1000):
    print(post)
```

### Real-Time Streaming

Connect to X's streaming endpoints for real-time data:

```python
from xdk import Client

client = Client(bearer_token="YOUR_BEARER_TOKEN")

# Stream filtered posts (requires filter rules to be set up)
for post in client.stream.posts():
    print(f"New post: {post}")

# Stream 10% sample of all posts
for post in client.stream.posts_sample10(partition=1):
    print(f"Sample post: {post}")

# Stream 1% sample of all posts
for post in client.stream.posts_sample():
    print(f"Sample post: {post}")

# Stream all posts (firehose - requires elevated access)
for post in client.stream.posts_firehose(partition=1):
    print(f"Firehose post: {post}")

# Language-specific firehose streams (requires elevated access)
for post in client.stream.posts_firehose_en(partition=1):  # English
    print(f"English post: {post}")
for post in client.stream.posts_firehose_ja(partition=1):  # Japanese
    print(f"Japanese post: {post}")
for post in client.stream.posts_firehose_ko(partition=1):  # Korean
    print(f"Korean post: {post}")
for post in client.stream.posts_firehose_pt(partition=1):  # Portuguese
    print(f"Portuguese post: {post}")

# Likes streams (requires elevated access)
for like in client.stream.likes_firehose(partition=1):
    print(f"Like: {like}")
for like in client.stream.likes_sample10(partition=1):
    print(f"Sample like: {like}")

# Compliance streams (for data compliance)
for event in client.stream.posts_compliance(partition=1):
    print(f"Post compliance event: {event}")
for event in client.stream.users_compliance(partition=1):
    print(f"User compliance event: {event}")
for event in client.stream.likes_compliance():
    print(f"Likes compliance event: {event}")
for event in client.stream.labels_compliance():
    print(f"Labels event: {event}")

# Manage stream rules
rules = client.stream.get_rules()
print(f"Current rules: {rules}")

# Get rule counts
rule_counts = client.stream.get_rule_counts()
print(f"Rule counts: {rule_counts}")

client.stream.update_rules(body={
    "add": [{"value": "python lang:en", "tag": "python-posts"}]
})

# Dry run to test rules without applying
client.stream.update_rules(
    body={"add": [{"value": "test", "tag": "test"}]},
    dry_run=True
)

# Delete all rules
client.stream.update_rules(
    body={"delete": {"ids": ["rule_id_1", "rule_id_2"]}},
)

# Terminate all active streaming connections
client.connections.delete_all()
```

### Direct Messages

```python
# Get DM events for a conversation (requires OAuth2)
events = client.direct_messages.get_events_by_conversation_id(id="CONVERSATION_ID")

# Create a new DM conversation (requires OAuth2)
conversation = client.direct_messages.create_conversation(
    body={"participant_ids": ["123", "456"]}
)

# Send a DM to an existing conversation (requires OAuth2)
client.direct_messages.create_by_conversation_id(
    dm_conversation_id="CONVERSATION_ID",
    body={"text": "Hello everyone!"}
)

# Send a DM to a single participant (requires OAuth2)
client.direct_messages.create_by_participant_id(
    participant_id="USER_ID",
    body={"text": "Hello!"}
)

# Remove an event (requires OAuth2)
client.direct_messages.delete_events(event_id="EVENT_ID")
```

### Spaces

```python
# Get spaces by IDs
spaces = client.spaces.get_by_ids(ids=["SPACE_ID_1", "SPACE_ID_2"])

# Get a single space
space = client.spaces.get_by_id(id="SPACE_ID")

# Search spaces
results = client.spaces.search(query="tech talk")

# Get spaces by creator IDs
creator_spaces = client.spaces.get_by_creator_ids(user_ids=["USER_ID"])

# Get ticket buyers for a space (requires OAuth2)
buyers = client.spaces.get_buyers(id="SPACE_ID", max_results=100)

# Get posts shared in a space
posts = client.spaces.get_posts(id="SPACE_ID")
```

### Lists

```python
# Get a list by ID
list_data = client.lists.get_by_id(id="LIST_ID")

# Get list members
members = client.lists.get_members(id="LIST_ID")

# Get posts from a list
posts = client.lists.get_posts(id="LIST_ID")

# Create a list (requires OAuth2)
new_list = client.lists.create(body={"name": "My List", "description": "A test list"})

# Add member to list (requires OAuth2)
client.lists.add_member(id="LIST_ID", body={"user_id": "USER_ID"})

# Remove member from list (requires OAuth2)
client.lists.remove_member_by_user_id(id="LIST_ID", user_id="USER_ID")

# Update list metadata (requires OAuth2)
client.lists.update(id="LIST_ID", body={"name": "Renamed List"})

# Delete a list (requires OAuth2)
client.lists.delete(id="LIST_ID")
```

### Communities

```python
# Get a community by ID
community = client.communities.get_by_id(id="COMMUNITY_ID")

# Search communities
results = client.communities.search(query="python developers")
```

### Trends

```python
# Get trends for a location (WOEID)
trends = client.trends.get_by_woeid(woeid=1)  # 1 = Worldwide

# Get personalized trends (requires OAuth2)
trends = client.trends.get_personalized()

# Get AI-generated trend summaries
ai_trends = client.trends.get_ai(id="TREND_ID")
```

### Media

```python
# Upload media (requires OAuth2)
media = client.media.upload(body={"media_data": "base64_encoded_data"})

# Get media upload status
status = client.media.get_upload_status(media_id="MEDIA_ID")

# Get media by key
media = client.media.get_by_key(media_key="MEDIA_KEY")

# Get multiple media by keys
media_list = client.media.get_by_keys(media_keys=["KEY1", "KEY2"])

# Get media analytics (requires OAuth2)
analytics = client.media.get_analytics(
    media_keys=["KEY1", "KEY2"],
    start_time="2024-01-01T00:00:00Z",
    end_time="2024-01-31T23:59:59Z"
)

# Chunked upload workflow for large files
init = client.media.initialize_upload(body={
    "total_bytes": 1048576,
    "media_type": "video/mp4"
})
client.media.append_upload(id=init.media_id, body={
    "media_data": "base64_chunk",
    "segment_index": 0
})
result = client.media.finalize_upload(id=init.media_id)
```

### Compliance

```python
# Create compliance jobs (requires OAuth2)
job = client.compliance.create_jobs(body={
    "type": "tweets",
    "name": "my-compliance-job"
})

# Get compliance job by ID
job = client.compliance.get_jobs_by_id(id="JOB_ID")

# List compliance jobs
jobs = client.compliance.get_jobs(type="tweets")
```

### Community Notes

```python
# Search for posts eligible for community notes (requires OAuth2)
eligible = client.community_notes.search_eligible_posts(query="misinformation")

# Search community notes you've written (requires OAuth2)
my_notes = client.community_notes.search_written()

# Create a community note (requires OAuth2)
note = client.community_notes.create(body={
    "tweet_id": "1234567890",
    "text": "This needs context..."
})

# Delete a community note (requires OAuth2)
client.community_notes.delete(id="NOTE_ID")

# Evaluate a note (requires OAuth2)
client.community_notes.evaluate(body={"note_id": "NOTE_ID", "rating": "helpful"})
```

### Usage

```python
# Get API usage statistics (requires OAuth2)
usage = client.usage.get(days=7)
```

## Error Handling

The SDK raises standard `requests` exceptions for HTTP errors:

```python
from xdk import Client
import requests

client = Client(bearer_token="YOUR_BEARER_TOKEN")

try:
    post = client.posts.get_by_id(id="invalid_id")
except requests.exceptions.HTTPError as e:
    print(f"HTTP Error: {e.response.status_code}")
    print(f"Error details: {e.response.json()}")
except requests.exceptions.RequestException as e:
    print(f"Request failed: {e}")
```

For pagination errors:

```python
from xdk import Client, Cursor, PaginationError

client = Client(bearer_token="YOUR_BEARER_TOKEN")

try:
    # This will raise PaginationError because get_by_id doesn't support pagination
    for item in Cursor(client.posts.get_by_id, id="123").items():
        pass
except PaginationError as e:
    print(f"Method doesn't support pagination: {e}")
```

## Requirements

- Python 3.8+
- `requests` >= 2.25.0
- `requests-oauthlib` >= 1.3.0
- `pydantic` >= 2.0.0

## Development

Clone the repository and install development dependencies:

```bash
git clone https://github.com/xdevplatform/xdk.git
cd xdk/python
uv sync --dev
```

Run tests:

```bash
uv run pytest
```

Format code:

```bash
uv run black xdk tests
uv run ruff check --fix xdk tests
```

## Documentation

For complete API documentation, visit [X Developer Documentation](https://docs.x.com/xdks/python/overview).

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Links

- [X Developer Portal](https://developer.x.com/)
- [X API Documentation](https://developer.x.com/en/docs/twitter-api)
- [GitHub Repository](https://github.com/xdevplatform/xdk)
- [Report Issues](https://github.com/xdevplatform/xdk/issues)
