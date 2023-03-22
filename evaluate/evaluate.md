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