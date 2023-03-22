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