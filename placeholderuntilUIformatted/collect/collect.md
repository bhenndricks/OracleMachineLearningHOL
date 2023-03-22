# Lab 1: Collecting and Exploring Social Media Data (15 Minutes)

## Task 1: Collect social media data using an API
Sign up for a Twitter Developer account and create a new project to obtain your API keys and access tokens. You will need the following:

* API key
* API key secret
* Access token
* Access token secret

2. Install Tweepy, a Python library for accessing the Twitter API:

````
<copy>
pip install tweepy
</copy>
````

3. Create a Python script to collect tweets related to a specific keyword or hashtag:
python

````
<copy>
import tweepy
import csv

# Replace with your own credentials
API_KEY = 'your_api_key'
API_SECRET_KEY = 'your_api_secret_key'
ACCESS_TOKEN = 'your_access_token'
ACCESS_SECRET = 'your_access_secret'

auth = tweepy.OAuthHandler(API_KEY, API_SECRET_KEY)
auth.set_access_token(ACCESS_TOKEN, ACCESS_SECRET)

api = tweepy.API(auth)

hashtag = 'your_hashtag'
tweets_count = 1000

tweets = tweepy.Cursor(api.search_tweets, q=hashtag, lang='en', tweet_mode='extended').items(tweets_count)

with open('tweets.csv', 'w', newline='', encoding='utf-8') as f:
    writer = csv.writer(f)
    writer.writerow(['tweet_id', 'created_at', 'user', 'full_text'])

    for tweet in tweets:
        writer.writerow([tweet.id_str, tweet.created_at, tweet.user.screen_name, tweet.full_text]) 
</copy>
````
* **Note:** Replace your_hashtag with your desired hashtag and run the script to collect the tweets.

## Task 2: Load and explore the dataset to understand the content and features

1. Use Oracle SQL Developer or another SQL client to connect to your Oracle Database 23c instance.

2. Create a table to store the collected tweets:

````
<copy>
CREATE TABLE tweets (
    tweet_id VARCHAR2(20) PRIMARY KEY,
    created_at TIMESTAMP,
    user_name VARCHAR2(50),
    full_text VARCHAR2(4000)
);
</copy>
````

3. Load the collected tweets from tweets.csv into the tweets table using SQL*Loader or Oracle Data Pump.

4. Explore the dataset by running basic queries to understand the content:

````
<copy>
SELECT COUNT(*) FROM tweets; -- Get the total number of tweets

SELECT user_name, COUNT(*) AS num_tweets
FROM tweets
GROUP BY user_name
ORDER BY num_tweets DESC; -- Get the top users by the number of tweets
</copy>
````