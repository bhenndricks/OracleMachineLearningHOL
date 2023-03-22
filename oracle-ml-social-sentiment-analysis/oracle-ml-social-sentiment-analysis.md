# Introduction
In this 90-minute workshop, you will learn how to use Oracle Machine Learning (OML) with Oracle Database 23c to analyze social media sentiment in real-time. This hands-on workshop will walk you through the entire process, from collecting social media data to building and evaluating a machine learning model for sentiment analysis. Attendees will gain experience with the latest OML features and understand how this technology can address the challenges of understanding public opinion on trending topics.

## Prerequisites
* This lab requires an [Oracle Cloud account](https://www.oracle.com/cloud/free/). You may use your own cloud account, a cloud account that you obtained through a trial, a Free Tier account, or a LiveLabs account.
* Oracle Database 23c with Oracle Machine Learning
* Oracle SQL Developer or another SQL client
* Basic knowledge of SQL and machine learning concepts
* Access to a social media API, such as Twitter API, for data collection

## Agenda
1. Introduction (10 minutes)
* Workshop objectives and overview
* Introduction to Oracle Machine Learning in 23c
* Viral challenge: real-time social media sentiment analysis

2. Collecting and Exploring Social Media Data (15 minutes)
* Task 1: Collect social media data using an API
* Task 2: Load and explore the dataset to understand the content and features

3. Data Preparation (15 minutes)
* Task 1: Clean and preprocess social media data
* Task 2: Feature extraction from text data
* Task 3: Split data into training and testing sets

4. Building a Sentiment Analysis Model (20 minutes)
* Task 1: Choose an appropriate algorithm for sentiment analysis
* Task 2: Train the model using the training dataset
* Task 3: Optimize model parameters

5. Evaluating and Deploying the Model (20 minutes)
* Task 1: Test the model using the testing dataset
* Task 2: Evaluate the model's performance
* Task 3: Deploy the model for real-time sentiment analysis

6. Wrap-up and Q&A (10 minutes)
* Recap of the workshop and key takeaways
* How Oracle Machine Learning can solve other viral challenges
* Q&A session

## Workshop Objectives
By the end of this workshop, attendees will be able to:

* Collect and explore social media data using Oracle Database 23c and Oracle Machine Learning
* Perform data cleaning, preprocessing, and feature extraction from text data
* Train, optimize, and evaluate a sentiment analysis model using Oracle Machine Learning
* Understand how OML can be applied to analyze public opinion on trending topics
* Differentiate Oracle Machine Learning from competing solutions

This workshop offers a hands-on experience with Oracle Machine Learning in Oracle Database 23c and highlights its ability to analyze real-time social media sentiment, a viral and trending topic. It showcases the latest features and differentiators, offering a high potential for social media engagement and differentiation from competing solutions.

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
# Lab 2: Data Preparation (15 minutes)

## Task 1: Clean and preprocess social media data

1. Clean the text data by removing URLs, special characters, and converting to lowercase:
````
<copy>
UPDATE tweets
SET full_text = REGEXP_REPLACE(LOWER(full_text), '(http|https)://[^\s]*', ''); -- Remove URLs

UPDATE tweets
SET full_text = REGEXP_REPLACE(full_text, '[^a-zA-Z0-9\s]', ''); -- Remove special characters
</copy>
````

2. Create a table with tokenized words from the tweets:
````
<copy>
CREATE TABLE tweet_words AS (
    SELECT tweet_id, REGEXP_SUBSTR(full_text, '[^ ]+', 1, LEVEL) AS word
    FROM tweets
    CONNECT BY REGEXP_SUBSTR(full_text, '[^ ]+', 1, LEVEL) IS NOT NULL
        AND PRIOR tweet_id = tweet_id
        AND PRIOR SYS_GUID() IS NOT NULL
);
</copy>
````
## Task 2: Feature extraction from text data

1. Remove stop words from the tweet_words table. You can create a table with common stop words and use it to filter out the stop words from the tweet_words table:

````
<copy>
CREATE TABLE stop_words (
    word VARCHAR2(50)
);

-- Insert common stop words into the stop_words table

DELETE FROM tweet_words
WHERE word IN (SELECT word FROM stop_words);
</copy>
````

2. Create a table with term frequency-inverse document frequency (TF-IDF) values for each word in each tweet:

````
<copy>
CREATE TABLE tfidf AS (
    SELECT a.tweet_id, a.word, a.tf * b.idf AS tfidf
    FROM (
        SELECT tweet_id, word, COUNT(*) AS tf
        FROM tweet_words
        GROUP BY tweet_id, word
    ) a
    JOIN (
        SELECT word, LOG( (SELECT COUNT(DISTINCT tweet_id) FROM tweet_words) / COUNT(DISTINCT tweet_id) ) AS idf
        FROM tweet_words
        GROUP BY word
    ) b ON a.word = b.word
);
</copy>
````

## Task 3: Split data into training and testing sets

1. Add a column to the tweets table to store sentiment labels, which will be used as target variables for the model:
````
<copy>
ALTER TABLE tweets
ADD (sentiment_label NUMBER);
</copy>
````
2. Manually label a sample of tweets with sentiment values (e.g., 1 for positive, 0 for neutral, and -1 for negative):

````
<copy>
UPDATE tweets
SET sentiment_label = 1
WHERE tweet_id IN ('tweet_id1', 'tweet_id2', ...);

UPDATE tweets
SET sentiment_label = 0
WHERE tweet_id IN ('tweet_id3', 'tweet_id4', ...);

UPDATE tweets
SET sentiment_label = -1
WHERE tweet_id IN ('tweet_id5', 'tweet_id6', ...);
</copy>
````

3. Create training and testing datasets using a 70:30 split:
````
<copy>
CREATE TABLE training_data AS (
    SELECT *
    FROM tweets
    WHERE DBMS_RANDOM.VALUE <= 0.7
);

CREATE TABLE testing_data AS (
    SELECT *
    FROM tweets
    WHERE DBMS_RANDOM.VALUE > 0.7
);
</copy>
````
# Lab 3: Building a Sentiment Analysis Model (20 minutes)

## Task 1: Choose an appropriate algorithm for sentiment analysis

1. For this task, we will use the Support Vector Machines (SVM) algorithm for sentiment analysis, as it works well with high-dimensional text data.

## Task 2: Train the model using the training dataset

1. Create a settings table for the SVM model:
````
<copy>
BEGIN
    DBMS_DATA_MINING.CREATE_MODEL_SETTINGS_TABLE(
        settings_table_name => 'svm_settings'
    );
END;
/

INSERT INTO svm_settings (setting_name, setting_value)
VALUES (dbms_data_mining.algo_name, dbms_data_mining.algo_svm);
</copy>
````

2. Create the SVM model using the training dataset and the TF-IDF features:

````
<copy>
        yattr => 'sentiment_label',
        xtype => dbms_data_mining_transform.attr_text,
        xl_settings_table => 'svm_settings'
    );

    DBMS_DATA_MINING.CREATE_MODEL(
        model_name => 'svm_model',
        mining_function => dbms_data_mining.classification,
        data_table_name => 'tfidf',
        case_id_column_name => 'tweet_id',
        target_column_name => 'sentiment_label',
        settings_table_name => 'svm_settings',
        xform_list => v_xl_instance
    );
END;
/
</copy>
````
## Task 3: Optimize model parameters
1. Use Oracle Data Miner or Grid Search to optimize the model parameters such as the kernel type and complexity factor.

# Lab 4: Evaluating and Deploying the Model (20 minutes)

## Task 1: Test the model using the testing dataset

1. Apply the SVM model to the testing dataset to obtain sentiment predictions:
````
<copy>
CREATE TABLE testing_predictions AS (
    SELECT a.tweet_id, b.sentiment_label, PREDICTION(svm_model USING *) AS predicted_sentiment
    FROM tfidf a
    JOIN testing_data b ON a.tweet_id = b.tweet_id
    WHERE a.tweet_id IN (SELECT tweet_id FROM testing_data)
    GROUP BY a.tweet_id, b.sentiment_label
);
</copy>
````
## Task 2: Evaluate the model's performance

1. Calculate the accuracy, precision, recall, and F1-score for the SVM model:
````
<copy>
SELECT
    COUNT(*) AS total,
    SUM(CASE WHEN sentiment_label = predicted_sentiment THEN 1 ELSE 0 END) AS correct,
    SUM(CASE WHEN sentiment_label = 1 AND predicted_sentiment = 1 THEN 1 ELSE 0 END) AS true_positive,
    SUM(CASE WHEN sentiment_label = 1 AND predicted_sentiment <> 1 THEN 1 ELSE 0 END) AS false_negative,
    SUM(CASE WHEN sentiment_label <> 1 AND predicted_sentiment = 1 THEN 1 ELSE 0 END) AS false_positive,
    SUM(CASE WHEN sentiment_label <> 1 AND predicted_sentiment <> 1 THEN 1 ELSE 0 END) AS true_negative
FROM testing_predictions;
</copy>
````
2. Compute the accuracy, precision, recall, and F1-score using the values obtained from the above query.

## Task 3: Deploy the model for real-time sentiment analysis

1. Create a PL/SQL function that takes a tweet text as input and returns the predicted sentiment using the SVM model:
````
<copy>
CREATE OR REPLACE FUNCTION predict_sentiment(p_tweet_text IN VARCHAR2) RETURN NUMBER
IS
    v_sentiment_label NUMBER;
    v_clean_text VARCHAR2(4000);
    v_tweet_id VARCHAR2(20) := 'temp_tweet';
BEGIN
    -- Clean and preprocess the input tweet text
    SELECT REGEXP_REPLACE(LOWER(p_tweet_text), '(http|https)://[^\s]*', '') INTO v_clean_text FROM DUAL;
    SELECT REGEXP_REPLACE(v_clean_text, '[^a-zA-Z0-9\s]', '') INTO v_clean_text FROM DUAL;

    -- Insert the cleaned text into the tweets table
    INSERT INTO tweets (tweet_id, full_text) VALUES (v_tweet_id, v_clean_text);

    -- Tokenize the input tweet text
    INSERT INTO tweet_words (tweet_id, word)
    SELECT v_tweet_id, REGEXP_SUBSTR(v_clean_text, '[^ ]+', 1, LEVEL) AS word
    FROM DUAL
    CONNECT BY REGEXP_SUBSTR(v_clean_text, '[^ ]+', 1, LEVEL) IS NOT NULL;

    -- Remove stop words from the tokenized input tweet text
    DELETE FROM tweet_words
    WHERE tweet_id = v_tweet_id
    AND word IN (SELECT word FROM stop_words);

    -- Calculate TF-IDF for the input tweet text
    INSERT INTO tfidf (tweet_id, word, tfidf)
    SELECT a.tweet_id, a.word, a.tf * b.idf AS tfidf
    FROM (
        SELECT v_tweet_id AS tweet_id, word, COUNT(*) AS tf
        FROM tweet_words
        WHERE tweet_id = v_tweet_id
        GROUP BY word
    ) a
    JOIN (
        SELECT word, LOG( (SELECT COUNT(DISTINCT tweet_id) FROM tweet_words WHERE tweet_id <> v_tweet_id) / COUNT(DISTINCT tweet_id) ) AS idf
        FROM tweet_words
        WHERE tweet_id <> v_tweet_id
        GROUP BY word
    ) b ON a.word = b.word;

    -- Apply the SVM model to obtain the predicted sentiment
    SELECT PREDICTION(svm_model USING *) INTO v_sentiment_label
    FROM tfidf
    WHERE tweet_id = v_tweet_id
    GROUP BY tweet_id;

    -- Clean up temporary data
    DELETE FROM tweets WHERE tweet_id = v_tweet_id;
    DELETE FROM tweet_words WHERE tweet_id = v_tweet_id;
    DELETE FROM tfidf WHERE tweet_id = v_tweet_id;

    RETURN v_sentiment_label;
END;
/
</copy>
````

2. Use the predict_sentiment function in your applications to perform real-time sentiment analysis on social media data. You can test the function by passing a sample tweet text as input:

````
<copy>
SELECT predict_sentiment('This workshop is amazing!') AS predicted_sentiment FROM DUAL;
</copy>
````


3. This will return the predicted sentiment label (1 for positive, 0 for neutral, -1 for negative) for the given tweet text.

## **Acknowledgements**
* **Author(s)** - Blake Hendricks
* **Contributor(s)** -
* **Last Updated By/Date** -