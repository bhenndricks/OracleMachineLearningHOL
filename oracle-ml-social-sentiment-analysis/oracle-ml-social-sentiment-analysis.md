# Introduction
Welcome to this sentiment analysis workshop using Oracle's technology stack! In this interactive, hands-on workshop, you will learn how to harness the power of Oracle's advanced tools and technologies to build a sentiment analysis model, deploy it as a REST API, and integrate it into a dashboard or application.

Throughout this 90-minute workshop, we will explore various Oracle technologies, including Oracle Autonomous Database, Oracle Text, Oracle Data Mining, OML4SQL, Oracle Data Miner, and Oracle Application Express (APEX). You will gain practical experience working with these technologies, as well as valuable insights into how they can be applied to real-world sentiment analysis use cases.

The workshop is divided into four labs, each focused on different aspects of the sentiment analysis process:
1. Data Preparation and Feature Extraction
2. Model Training and Evaluation
3. Model Deployment and Real-time Sentiment Analysis
4. Integration with a Dashboard or Application

By the end of the workshop, you will have a comprehensive understanding of the sentiment analysis process and how to implement it using Oracle's technology stack. You will also have hands-on experience creating a sentiment analysis model, exposing it through a REST API, and integrating it into a dashboard or application.

No matter your level of expertise, this workshop is designed to be accessible and engaging for everyone. So, let's dive in and get started on this exciting journey into the world of sentiment analysis using Oracle's technology stack!


## Prerequisites
* Participants need access to [Oracle Cloud account](https://www.oracle.com/cloud/free/) with privileges to create and manage Oracle Autonomous Database instances. You may use your own cloud account, a cloud account that you obtained through a trial, a Free Tier account, or a LiveLabs account.
* Oracle SQL Developer: Participants should have Oracle SQL Developer installed on their machines. This will be used to connect to the Oracle Autonomous Database and execute SQL queries. You can download it from the [Oracle SQL Developer website](https://www.oracle.com/tools/downloads/sqldev-downloads.html).
* Oracle Data Miner: Oracle Data Miner is a component of SQL Developer, and participants should have it enabled. You can follow the [instructions in the official documentation](https://docs.oracle.com/en/database/oracle/oracle-data-miner/4.2/odmug/enabling-Oracle-data-miner.html) to enable Oracle Data Miner.
* Oracle Application Express (APEX): Participants need access to an Oracle APEX environment connected to their Oracle Autonomous Database instance. You can sign up for a free APEX workspace at the [Oracle APEX website](https://apex.oracle.com/) or follow the [instructions in the official documentation](https://docs.oracle.com/en/database/oracle/application-express/21.1/htmdb/configuring-autonomous-database.html) to configure APEX on Autonomous Database.
* Oracle REST Data Services (ORDS): Participants should have ORDS installed and configured to work with their Oracle Autonomous Database instance. You can download ORDS from the [Oracle REST Data Services website](https://www.oracle.com/tools/downloads/rest-data-services-downloads.html) and follow the [instructions in the official documentation](https://docs.oracle.com/en/database/oracle/oracle-rest-data-services/21.3/aelig/installing-Oracle-REST-data-services.html) to install and configure it.
* Dataset: Participants will use the Sentiment 140 labeled dataset for sentiment analysis. This dataset should have text data and corresponding sentiment labels (e.g., positive, neutral, negative). Participants will import this dataset into the Oracle Autonomous Database during the workshop. #provide link here when dataset is approved
* Basic SQL and PL/SQL Knowledge: Participants should have a basic understanding of SQL and PL/SQL, as the workshop involves writing and executing SQL queries and working with PL/SQL packages.
* Familiarity with Oracle's Technology Stack: It's beneficial if participants have some familiarity with Oracle Text, Oracle Data Mining, and OML4SQL, although the workshop will provide guidance on using these technologies.

## Agenda
1. Introduction (10 minutes)
* Workshop objectives and overview
* Introduction to sentiment analysis and its applications
* Overview of Oracle technologies used in the workshop

2. Lab 1: Data Preparation and Feature Extraction (20 minutes)
* Importing the labeled dataset into Oracle Autonomous Database
* Exploring the dataset and understanding its structure
* Using Oracle Text to preprocess and extract features from the text data
* Creating a table for storing the processed data

3. Lab 2: Model Training and Evaluation (25 minutes)
* Introduction to Oracle Data Mining and OML4SQL
* Creating a sentiment analysis model using Oracle Data Miner
* Evaluating the model's performance using common metrics
* Interpreting the results and identifying potential improvements

4. Lab 3: Model Deployment and Real-time Sentiment Analysis (20 minutes)
* Introduction to Oracle REST Data Services (ORDS) and Oracle Application Express (APEX)
* Deploying the sentiment analysis model as a REST API using ORDS
* Testing the REST API with sample text inputs
* Performing real-time sentiment analysis using the deployed API

5. Lab 4: Integration with a Dashboard or Application (15 minutes)
* Designing a simple dashboard or application using Oracle APEX
* Integrating the sentiment analysis REST API into the dashboard or application
* Testing the integration and observing the results

6. Wrap-up (10 minutes)
* Review of workshop objectives and key concepts covered in each lab
* Reflection on hands-on experience and real-world applications of sentiment analysis
* Discussion of potential extensions and adaptations of the workshop techniques
* Encouragement to provide feedback and share experiences on social media



## Workshop Objectives
By the end of this workshop, attendees will be able to:

* Understand sentiment analysis and its real-world applications: Introduce participants to the concept of sentiment analysis, its importance, and its various use cases in industries such as marketing, customer service, finance, and more.
* Gain hands-on experience with Oracle's technology stack: Provide participants with practical experience using Oracle technologies, including Oracle Autonomous Database, Oracle Text, Oracle Data Mining, OML4SQL, Oracle Data Miner, and Oracle Application Express (APEX), to implement sentiment analysis.

* Learn the end-to-end process of sentiment analysis: Guide participants through the entire process of sentiment analysis, from data preparation and feature extraction to model training, evaluation, deployment, and integration into a dashboard or application.

* Develop skills in deploying machine learning models as REST APIs: Teach participants how to deploy their sentiment analysis models as REST APIs using Oracle REST Data Services (ORDS), enabling real-time sentiment analysis and integration with various applications.

* Integrate sentiment analysis into a dashboard or application: Demonstrate how to design a simple dashboard or application using Oracle APEX and integrate the sentiment analysis REST API, allowing participants to observe the results of their analysis in a real-world context.

* Promote knowledge sharing and networking: Encourage participants to share their experiences, learn from each other, and build connections within the workshop community.


# Lab 1: Collecting and Exploring Social Media Data (15 Minutes)

## Task 1: Set up Oracle Autonomous Database and import the dataset.
1. Create an Oracle Autonomous Database instance.

* Sign in to your Oracle Cloud account.
* Navigate to the Autonomous Database section and create a new instance.
* Download the wallet file and save the necessary credentials.


2.  Import the dataset into the Autonomous Database instance.

* Use Oracle SQL Developer or another SQL client to connect to the Autonomous Database using the wallet file and credentials.
* Create a table to store the dataset:

````
<copy>
CREATE TABLE sentiment_data ( id NUMBER, sentiment_label NUMBER, text VARCHAR2(4000) );
</copy>
````

* Import the data using SQL scripts, external tables, or the client's import functionality.

Option 1: SQL scripts with external tables
1. First, upload the Sentiment140 CSV file to Oracle Cloud Object Storage or another accessible storage location.
2. Next, create a directory object in the Oracle Autonomous Database to point to the storage location. Replace <your_credentials> with the necessary access information, and <your_storage_url> with the URL of the storage location.

````
<copy>
CREATE OR REPLACE DIRECTORY sentiment_data_dir AS '<your_storage_url>' CREDENTIALS '<your_credentials>';
</copy>
````

3. Now, create an external table that maps to the Sentiment140 CSV file:

````
<copy>
CREATE TABLE sentiment_data_ext ( sentiment_label NUMBER, tweet_id NUMBER, tweet_date DATE, query VARCHAR2(100), user_handle VARCHAR2(100), tweet_text VARCHAR2(4000) ) ORGANIZATION EXTERNAL ( TYPE ORACLE_LOADER DEFAULT DIRECTORY sentiment_data_dir ACCESS PARAMETERS ( RECORDS DELIMITED BY NEWLINE FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"' MISSING FIELD VALUES ARE NULL ) LOCATION ('sentiment140.csv') );
</copy>
````

4. Finally, create a table to store the imported data and insert the data from the external table:

````
<copy>
CREATE TABLE sentiment_data AS SELECT * FROM sentiment_data_ext;
</copy>
````

Option 2: SQL*Loader utility
1. Install the Oracle Instant Client and SQL*Loader utility on your local machine if you haven't already.
2. Create a control file named sentiment140.ctl with the following content. Replace <your_username> and <your_password> with your Oracle Autonomous Database credentials, and <your_connection_string> with the appropriate connection string.

````
<copy>
load data infile 'sentiment140.csv' append into table sentiment_data fields terminated by ',' optionally enclosed by '"' ( sentiment_label, tweet_id, tweet_date "to_date(:tweet_date, 'YYYY-MM-DD HH24:MI:SS')", query, user_handle, tweet_text )
</copy>
````

3. Run the SQL*Loader utility from the command prompt or terminal:
````
<copy>
sqlldr <your_username>/<your_password>@<your_connection_string> control=sentiment140.ctl
</copy>
````
This will load the Sentiment140 dataset into the sentiment_data table in your Oracle Autonomous Database.

Either of these options will import the Sentiment140 dataset into your Oracle Autonomous Database, allowing you to proceed with the workshop steps.

## Task 2: Explore the dataset using Oracle Data Miner.

1. Connect Oracle Data Miner to your Oracle Autonomous Database instance.
* Open Oracle Data Miner.
* Create a new connection using the wallet file and credentials for your Autonomous Database instance.

2. Explore the dataset.
* Create a new Data Miner project.
* Drag and drop the dataset table onto the Data Miner canvas.
* Use Data Miner's built-in tools to visualize and explore the dataset.

# Lab 2: Preprocessing and Feature Extraction (30 minutes)

## Task 1: Perform text preprocessing and feature extraction using Oracle Text and Oracle Data Mining

1. Create a new table with preprocessed text data.

* Connect to your Oracle Autonomous Database using SQL Developer or another SQL client.

* Preprocess the text data using SQL and built-in text processing functions. For example, you can use the following SQL statement to create a new table with preprocessed text data:

````
<copy>
CREATE TABLE preprocessed_tweets AS
SELECT 
    id, -- Retain the tweet ID from the original table
    sentiment_label, -- Retain the sentiment label from the original table
    REGEXP_REPLACE(
        REGEXP_REPLACE(
            REGEXP_REPLACE(
                LOWER(tweet_text), -- Convert the tweet text to lowercase
                'http[s]?://(?:[a-zA-Z]|[0-9]|[$-_@.&+]|[!*\\(\\),]|(?:%[0-9a-fA-F][0-9a-fA-F]))+', '' -- Remove URLs
            ), 
            '(@[A-Za-z0-9_]+)', '' -- Remove mentions (e.g., "@username")
        ), 
        '[^a-zA-Z0-9\s]', '' -- Remove special characters
    ) AS preprocessed_text -- Store the preprocessed text in a new column
FROM
    raw_tweets; -- Original table containing tweet ID, sentiment_label, and tweet_text columns
</copy>
````

* Write and execute SQL queries to preprocess the text data using Oracle Text:

````
<copy>
BEGIN ctx_ddl.create_preference('SENTIMENT_DATA_LEXER', 'BASIC_LEXER'); ctx_ddl.set_attribute('SENTIMENT_DATA_LEXER', 'PRINTJOINS', '_'); END; / CREATE INDEX sentiment_data_idx ON sentiment_data(text) INDEXTYPE IS CTXSYS.CONTEXT PARAMETERS ('LEXER SENTIMENT_DATA_LEXER');
</copy>
````

* Store the preprocessed data in a new table in the Autonomous Database:
````
<copy>
CREATE TABLE sentiment_data_preprocessed AS SELECT id, sentiment_label, ctx_doc.tokens(text, 'SENTIMENT_DATA_LEXER') AS tokens FROM sentiment_data;
</copy>
````

2. Extract features from the preprocessed text using Oracle Data Mining.

* Write and execute SQL queries to extract features such as Term Frequency-Inverse Document Frequency (TF-IDF) using Oracle Data Mining:
````
<copy>
BEGIN dbms_data_mining_transform.create_transform( transform_name => 'sentiment_data_transform', mining_attribute_name => 'tokens', transform_type => dbms_data_mining_transform.token_to_term, target_table_name => 'sentiment_data_tfidf' ); END; / BEGIN dbms_data_mining_transform.execute_transform( transform_name => 'sentiment_data_transform', data_table_name => 'sentiment_data_preprocessed' ); END;
</copy>
````


* Store the extracted features in a new table in the Autonomous Database.

To store the extracted features in a new table in the Autonomous Database, you'll first need to extract features from the preprocessed text using Oracle Machine Learning for SQL (OML4SQL) and then create a new table with the extracted features.

* Prepare the preprocessed text data for feature extraction. You can use the `DBMS_DATA_MINING_TRANSFORM` package to perform text tokenization and feature extraction, like the following example:

````
<copy>
BEGIN
   DBMS_DATA_MINING_TRANSFORM.TRANSFORM(
      'preprocessed_tweets', -- Source table with preprocessed text data
      'preprocessed_tweets_features', -- New table to store extracted features
      DBMS_DATA_MINING_TRANSFORM.CASE_ID, -- Column to be used as a unique identifier
      'sentiment_label', -- Target column with sentiment labels
      DBMS_DATA_MINING_TRANSFORM.NAIVE_BAYES, -- Algorithm to be used for feature extraction
      NULL,
      DBMS_DATA_MINING_TRANSFORM.AUTO); -- Let the package decide on the best feature extraction settings
END;
/
</copy>
````
Note: In this example, we're using the TRANSFORM() procedure of the DBMS_DATA_MINING_TRANSFORM package to extract features from the preprocessed_tweets table (containing the preprocessed text data) and store them in a new table named preprocessed_tweets_features. The sentiment_label column is used as the target column, and the Naive Bayes algorithm is used for feature extraction.

* Execute the PL/SQL block to perform feature extraction and create the new table with the extracted features. Once the block is executed, the new table `preprocessed_tweets_features` will be created and populated with the extracted features.

Now you have a new table containing the extracted features, which can be used for model training and evaluation in the subsequent steps of the workshop.



# Lab 3: Model Training and Evaluation (25 minutes)

## Task 1: Train a Support Vector Machines (SVM) model for sentiment classification using OML4SQL.

1.  Write and execute OML4SQL queries to train the SVM model on the dataset with extracted features:

````
<copy>
DECLARE v_model_settings dbms_data_mining.model_settings; BEGIN v_model_settings := dbms_data_mining.algo_name_svm || dbms_data_mining.prep_auto; dbms_data_mining.create_model( model_name => 'SENTIMENT_SVM_MODEL', mining_function => dbms_data_mining.classification, data_table_name => 'sentiment_data_tfidf', case_id_column_name => 'id', target_column_name => 'sentiment_label', settings_table_name => v_model_settings ); END; /
</copy>
````

2. Evaluate the SVM model using OML4SQL.

* Write and execute SQL queries to apply the SVM model to the test dataset and calculate the model's performance metrics (e.g., accuracy, F1-score, etc.):

````
<copy>
SELECT sentiment_label, prediction(SENTIMENT_SVM_MODEL USING *) predicted_sentiment_label FROM sentiment_data_tfidf; -- Calculate performance metrics using the actual and predicted sentiment labels
</copy>
````

# Lab 4: Model Deployment and Real-time Sentiment Analysis (25 minutes)

## Task 1: Create a REST API to expose the sentiment analysis model.

1. Create a PL/SQL package that wraps the sentiment analysis model.

* Write a PL/SQL package that includes a function to preprocess text, extract features, and apply the trained SVM model to predict sentiment.

````
<copy>
CREATE OR REPLACE PACKAGE sentiment_analysis AS FUNCTION predict_sentiment(p_text IN VARCHAR2) RETURN VARCHAR2; END sentiment_analysis; /
</copy>
````

2. Implement the package function.

* Write the implementation of the predict_sentiment function using Oracle Text, Oracle Data Mining, and OML4SQL.

````
<copy>
CREATE OR REPLACE PACKAGE BODY sentiment_analysis AS FUNCTION predict_sentiment(p_text IN VARCHAR2) RETURN VARCHAR2 IS v_predicted_sentiment_label VARCHAR2(32); BEGIN -- Preprocess text, extract features, and apply the SVM model -- (Implementation details depend on your specific data preprocessing and feature extraction steps) RETURN v_predicted_sentiment_label; END predict_sentiment; END sentiment_analysis; /
</copy>
````

3. Create an Oracle REST Data Services (ORDS) module to expose the sentiment analysis package as a REST API.

* Install and configure ORDS to work with your Autonomous Database instance if you haven't already.

* Create an ORDS module with a GET or POST method that maps to the `sentiment_analysis.predict_sentiment` function.

## Task 2: Integrate the REST API into a dashboard or application. CHALLENGE

1. Create an Oracle Application Express (APEX) application.

* If you haven't already, install and configure Oracle APEX to work with your Autonomous Database instance.

* Create a new APEX application with a dashboard page to display the sentiment analysis results.

2. Call the sentiment analysis REST API from the APEX application.

* Use APEX's built-in tools to create a form or similar UI element that allows users to input text for sentiment analysis.

* Configure the form to call the REST API and display the sentiment prediction (positive, neutral, or negative) on the dashboard page.

To configure the form to call the REST API and display the sentiment prediction on the dashboard page, you'll need to follow these steps:

* Create an HTML form and a result section on your dashboard page. Here's a simple example:
````
<copy>
<!DOCTYPE html>
<html>
<head>
    <title>Sentiment Analysis Dashboard</title>
</head>
<body>
    <h1>Sentiment Analysis Dashboard</h1>
    <form id="sentiment-form">
        <label for="tweet-text">Enter tweet text:</label>
        <input type="text" id="tweet-text" name="tweet-text" required>
        <button type="submit">Analyze Sentiment</button>
    </form>
    <div id="result"></div>
    <script src="main.js"></script>
</body>
</html>
</copy>
````

* Create a JavaScript file (e.g., main.js) to handle the form submission, call the REST API, and display the sentiment prediction. Here's an example:

````
<copy>
document.getElementById("sentiment-form").addEventListener("submit", async (event) => {
    event.preventDefault();
    const tweetText = document.getElementById("tweet-text").value;
    const resultElement = document.getElementById("result");

    try {
        const response = await fetch("https://your-api-endpoint/predict", {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({ text: tweetText }),
        });

        if (!response.ok) {
            throw new Error("Error fetching sentiment prediction");
        }

        const prediction = await response.json();
        resultElement.textContent = `Sentiment: ${prediction.sentiment}`;
    } catch (error) {
        resultElement.textContent = "Error: Unable to fetch sentiment prediction";
    }
});
</copy>
````

* Replace `https://your-api-endpoint/predict` with the actual REST API endpoint that you have set up for the sentiment analysis model.

* In this example, when the user submits the form with the tweet text, the JavaScript code will send a request to the REST API with the tweet text. The API will return a JSON object with the sentiment prediction, which will then be displayed on the dashboard page.

Note: Make sure to replace the example API endpoint with the actual endpoint for your REST API. Also, adjust the JavaScript code according to your form and result element IDs, if necessary. Once you've done that, you'll have a functioning dashboard that calls the REST API and displays the sentiment prediction based on the input tweet text.

# Wrap Up

At this point, the workshop is complete, and participants should have a solid understanding of how to implement sentiment analysis using Oracle's technology stack, including Oracle Autonomous Database, Oracle Text, Oracle Data Mining, OML4SQL, Oracle Data Miner, and Oracle Application Express. They will have gained hands-on experience creating a sentiment analysis model, exposing it through a REST API, and integrating it into a dashboard or application.

## **Acknowledgements**
* **Author(s)** - Blake Hendricks
* **Contributor(s)** -
* **Last Updated By/Date** - 5/4/2023