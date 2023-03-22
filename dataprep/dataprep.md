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