# 🐦 SheeshAI: AI Tweet Bot

![Python](https://img.shields.io/badge/Python-3.9-3776AB?logo=python&logoColor=white)
![Tweepy](https://img.shields.io/badge/Tweepy-Twitter%20API-1DA1F2?logo=x&logoColor=white)
![OpenAI](https://img.shields.io/badge/OpenAI-GPT--3-412991?logo=openai&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?logo=jupyter&logoColor=white)

**SheeshAI** is a Twitter bot I built in **April 2022** that used OpenAI's GPT-3 to run
its own Twitter account. It searched for the latest trending topics and tweeted about them,
posted jokes, and replied to anyone who tagged it, based on what they had written.

> The project is stored in this repository as **`TweetBot.zip`**, exactly as originally
> uploaded on 23 April 2022. This README explains what's inside and how it works.

## Contents

- [Features](#features)
- [What's in the zip](#whats-in-the-zip)
- [How it works](#how-it-works)
- [The topics](#the-topics)
- [Code walkthrough](#code-walkthrough)
- [Running it](#running-it)
- [Known issues](#known-issues)
- [Tech stack](#tech-stack)

## Features

- 🔥 **Tweets about trending topics.** Looked up what was trending on Twitter and wrote a tweet about it.
- 😂 **Writes jokes.** Generated and posted jokes.
- 💬 **Replies when tagged.** When someone mentioned the bot, it read their tweet and replied with
  a response written for what they had said.
- 🎲 **Random topic tweets.** Picked a topic such as technology, coding, music, nature, India or
  the share market and tweeted about it.

> **Note:** the code saved in `TweetBot.zip` covers the random-topic tweets and jokes. The
> trending-topics and reply features aren't included in this zipped code.

## What's in the zip

| File | What it is |
|---|---|
| `aibot.py` | **The bot.** Picks a topic, generates a tweet with OpenAI and posts it |
| `.ipynb_checkpoints/TweetBOT-checkpoint.ipynb` | The first experiment: a notebook that connected to Twitter and posted a test tweet, "BOT in making" |
| `README.md` | The original one-line readme ("AITweetBot") |
| `.env/` | A Python virtual environment (installed packages). Not part of the project's code |

## How it works

### Random topics and jokes (`aibot.py`)

```
 ┌──────────────────────┐     ┌──────────────────────┐     ┌──────────────────────┐
 │ 1. Pick a random     │     │ 2. Ask OpenAI GPT-3  │     │ 3. Post the reply    │
 │    topic from a list │ ──► │    to write a tweet  │ ──► │    to Twitter        │
 │    (meme, music...)  │     │    (text-davinci-001)│     │    (Tweepy)          │
 └──────────────────────┘     └──────────────────────┘     └──────────────────────┘
```

1. **Connect to Twitter.** The bot logs in to the Twitter API with OAuth 1.0a credentials
   (an API key and secret plus an access token and secret) using the
   [Tweepy](https://www.tweepy.org) library.
2. **Connect to OpenAI.** It sets an OpenAI API key for the `openai` Python library.
3. **Choose a topic.** It picks one prompt at random from a built-in list of nine (see below).
4. **Generate the tweet.** It sends that prompt to OpenAI's **`text-davinci-001`** GPT-3
   model through the Completions API, allowing up to 256 tokens in the reply.
5. **Post it.** It prints the generated text and posts it as a new tweet with `api.update_status`.

Every run posts one new tweet. Running it on a schedule (for example with cron or a task
scheduler) turns it into a bot that tweets regularly by itself.

### Trending topics and replies

These two features follow the same pattern: get some text from Twitter, ask GPT-3 to write
about it, and post the result.

- **Trending topics.** Fetch the current trending topics from the Twitter API, choose one,
  and send it to GPT-3 as the prompt for a new tweet.
- **Replying when tagged.** Check the bot's mentions for new tweets that tag it, send each
  tweet's text to GPT-3 to write a reply, and post that reply as a response in the same
  conversation.

As noted under [Features](#features), the code for these two isn't in `TweetBot.zip`.

## The topics

| Hashtag | Prompt sent to GPT-3 |
|---|---|
| #meme | post a meme on engineer |
| #technology | tweet something cool for #technology |
| #India | tweet something cool for India |
| Coding | tweet something cool for coding |
| #Music | tweet something cool about Music |
| #Nature | tweet something about Nature |
| #ShareMarket | Tweet facts about nifty |
| weather | what's the weather today in delhi |
| #joke | tell me a joke |

Each topic has a hashtag alongside its prompt, but only the prompt is sent; the hashtags
aren't added to the tweet.

## Code walkthrough

The heart of `aibot.py` (credentials left out):

```python
import tweepy
import openai
import random

# 1. Log in to Twitter
auth = tweepy.OAuthHandler(api_key, api_secret)
auth.set_access_token(access_key, access_secret)
api = tweepy.API(auth)

# 2. Log in to OpenAI
openai.api_key = openai_key

# 3. Pick a random topic
prompts = [
    {"hashtag": "#meme", "text": "post a meme on engineer"},
    {"hashtag": "#technology", "text": "tweet something cool for #technology"},
    # ...seven more topics
]
chosen_prompt = random.choice(prompts)

# 4. Ask GPT-3 to write the tweet
response = openai.Completion.create(
    engine="text-davinci-001",
    prompt=chosen_prompt["text"],
    max_tokens=256,
)
text = response.choices[0].text

# 5. Post it
print(text)
api.update_status(text)
```

The notebook (`TweetBOT-checkpoint.ipynb`) is the earlier step: it runs the Twitter login
cell by cell and finishes by posting "BOT in making", which confirmed the connection worked
before the OpenAI part was added.

## Running it

1. Download **`TweetBot.zip`** from this repository and extract it.
2. Install the libraries (the bot uses the pre-1.0 OpenAI library):

   ```bash
   pip install "tweepy>=4,<5" "openai<1"
   ```

3. Get your own credentials:
   - a Twitter/X developer app with **read and write** access: API key, API secret, access token and access token secret;
   - an OpenAI API key.
4. Put them into `aibot.py` in place of the existing values. The keys in the original file
   have expired. Better still, read them from environment variables (`os.environ`) so they
   never end up in your code.
5. Run the bot:

   ```bash
   python aibot.py
   ```

> **It won't run unmodified today.** See the known issues below.

## Known issues

- **The GPT-3 model has been retired.** OpenAI no longer offers `text-davinci-001` or the
  old `openai.Completion` interface. Current versions of the `openai` library use a client
  and the Chat Completions API with a current model instead.
- **Twitter/X API changes.** Posting with Tweepy's v1.1 `update_status` is no longer
  available on most access tiers. The current approach is the v2 endpoint through
  `tweepy.Client(...).create_tweet(text=...)`, which needs an X developer account.
- **Credentials were written into the code.** The original files contained the API keys
  directly. They have expired, but keeping keys in environment variables is the safe approach.
- **No length check.** A 256-token reply can be longer than a tweet allows, which would make
  the post fail.
- **The zip includes a whole virtual environment** (`.env/`), which makes it about 6.5 MB.
  Only `aibot.py` and the notebook are needed.

## Tech stack

| Area | Tools |
|---|---|
| Language | Python 3.9 |
| Twitter/X | Tweepy (OAuth 1.0a, API v1.1) |
| Text generation | OpenAI GPT-3 (`text-davinci-001`, Completions API) |
| Experimentation | Jupyter Notebook |
