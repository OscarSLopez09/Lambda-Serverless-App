# Let's start with a high-level overview.

I'm creating a serverless app that will be making calls to the News API website.
I will be introducing a little bit of machine learning in this app.
The app will be configured to fetch top news stories using a news API and then make calls to Amazon Comprehend service to determine the sentiment of news and categorize it as positive, negative, or neutral.
So, one thing to keep in mind.
Previously, you could run your own Python program on your machine learning program in any programming language to determine the news sentiment. Amazon created the service Amazon Comprehend, where you can pass text and determine a sentiment. Then, the news will be stored including the sentiment in Dynamo DB.
Once the news and the sentiment are stored in Dynamo DB, a user can query news by sentiment (e.g., positive, negative, neutral).
Also, we're creating a rule on Cloudwatch to trigger the Lambda function. 

<img src="https://github.com/OscarSLopez09/Lambda-Serverless-App/blob/main/Images/Capture.PNG" height="80%" width="80%" alt="Disk Sanitization Steps"/>

# The AWS services used in this project:

* AWS Lambda
* Amazon Comprehend
* Dynamo DB
* Amazon API Gateway
* AWS Cloudwatch

Before we jump into the actual code, we have one pre-requisite. We get all the top news stories from the website - newsapi.org.
To call news API, we need to get an API key from them.
First, we go to https://newsapi.org and create a login account using your email, then select Get API Key.
The API Key and the account is free.

<img src="https://github.com/OscarSLopez09/Lambda-Serverless-App/blob/main/Images/newsapi.PNG" height="80%" width="80%" alt="Disk Sanitization Steps"/>

Once you get the API we're going to be able to make calls to the API from the website newsapi.org
#

First, I'm going to build the Lambda function that is going to be used as the frontend. The lambda is going to be making call to News API 
and Amazon Comprehend service to get the sentiment, then is going to inserted in DynamoDB.

I'm going to code witch Cloud9 IDE. on the AWS console under service look for Cloud9 and select start environment and click on open.

* Create a directory with - mkdir NewsAPIReader
* Change directory -  cd NewsAPIReader
* Check the directory - ls -l
* Create the lambda file using python - nano lambda_function.py
<img src="https://github.com/OscarSLopez09/Lambda-Serverless-App/blob/main/Images/lambda0.PNG" height="80%" width="80%" alt="Disk Sanitization Steps"/>

* Once the editor is open I paste the code to it
```python
import requests
import json
import datetime
import boto3


#this lambda grabs today's headlines, does sentiment analysis using AWS Comprehend
#and saves the news along with sentiment into a dynamodb table
def lambda_handler(event, context):
    # TODO implement
    print("button pressed")
    print(event)
    print("just changing a print")
    if event['action']=='insert news':
        findNews()
    else:
        deleteNews()
    
    return 'End of News Sentiment IOT function'


def deleteNews():
    dynamodb = boto3.resource('dynamodb')
    table = dynamodb.Table('news')
    #Scanning the table to get all rows in one shot
    response =table.scan()
    if 'Items' in response:
        items=response['Items']
        for row in items:
            sentiment=row['sentiment']
            timestamp=row['timestamp']
            delresponse = table.delete_item(
                Key={
                'sentiment': sentiment,
                'timestamp':timestamp
                    }
                    ) 

def findNews():
    #News credit to newsapi.org
    #Fetch headlines using the API
    #IMPORTANT: Register in newsapi.org to get your own API key, it's super easy!
    response = requests.get("https://newsapi.org/v2/top-headlines?country=us&category=business&apiKey=84e3ef10a0b44603a7c64de19aca7848")
    d=response.json()
    if (d['status']) == 'ok':
        for article in d['articles']:
            print(article['title'])
            newsTitle=article['title']
            timestamp=article['publishedAt']
            sentiment=json.loads(getSentiment(newsTitle))
            print(sentiment['Sentiment'])
            insertDynamo(sentiment['Sentiment'],newsTitle,timestamp)

#getSentiment function calls AWS Comprehend to get the sentiment
def getSentiment(newsTitle):
    comprehend = boto3.client(service_name='comprehend')
    return(json.dumps(comprehend.detect_sentiment(Text=newsTitle, LanguageCode='en'), sort_keys=True))

#inserts headline along with sentiment into Dynamo    
def insertDynamo(sentiment,newsTitle,timestamp):
    print("inside insert dynamo function")
    dynamodb = boto3.resource('dynamodb')
    table = dynamodb.Table('news')
    response = table.put_item(
       Item={
        'sentiment': sentiment,
        'title': newsTitle,
        'timestamp' : timestamp
       }
       )
```
<img src="https://github.com/OscarSLopez09/Lambda-Serverless-App/blob/main/Images/lambda01.PNG" height="80%" width="80%" alt="Disk Sanitization Steps"/>

* Verify that the file was created - ls -lthr
<img src="https://github.com/OscarSLopez09/Lambda-Serverless-App/blob/main/Images/lambda02.PNG" height="80%" width="80%" alt="Disk Sanitization Steps"/>

* Install the requests external dependencies on the file - pip install requests -t .
<img src="https://github.com/OscarSLopez09/Lambda-Serverless-App/blob/main/Images/lambda03.PNG" height="80%" width="80%" alt="Disk Sanitization Steps"/>

* Zip all the files to uploaded to AWS Lambda - zip -r lambda.zip *
<img src="https://github.com/OscarSLopez09/Lambda-Serverless-App/blob/main/Images/lambda04.PNG" height="80%" width="80%" alt="Disk Sanitization Steps"/>

* Verify that zip file is created successfully - ls -lthr
<img src="https://github.com/OscarSLopez09/Lambda-Serverless-App/blob/main/Images/lambda05.PNG" height="80%" width="80%" alt="Disk Sanitization Steps"/>

* After running the AWS commands check the function on the AWS lambda console
<img src="https://github.com/OscarSLopez09/Lambda-Serverless-App/blob/main/Images/lambda06.PNG" height="90%" width="90%" alt="Disk Sanitization Steps"/>

* Click on the NewsReaderAPI function to see the code
<img src="https://github.com/OscarSLopez09/Lambda-Serverless-App/blob/main/Images/lambda07.PNG" height="80%" width="80%" alt="Disk Sanitization Steps"/>

* Proceed to create Dynamo DB table
<img src=""80%" width="80%" alt="Disk Sanitization Steps"/>

* On AWS console look for DynamoDB service
* Click on tables on left side of the screen, then click on create table
<img src="https://github.com/OscarSLopez09/Lambda-Serverless-App/blob/main/Images/lambda08.PNG" height="80%" width="80%" alt="Disk Sanitization Steps"/>

* On table details create name - news
* Create the partition key - sentiment
* Create a sort key - timestamp
<img src="https://github.com/OscarSLopez09/Lambda-Serverless-App/blob/main/Images/lambda09.PNG" height="80%" width="80%" alt="Disk Sanitization Steps"/>

* Scroll down and select - Create table
<img src="https://github.com/OscarSLopez09/Lambda-Serverless-App/blob/main/Images/lambda10.PNG" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://github.com/OscarSLopez09/Lambda-Serverless-App/blob/main/Images/lambda11.PNG" height="80%" width="80%" alt="Disk Sanitization Steps"/>

After the Dynamo DB table is create I will continue on the Second part of the project where I create the Cloudwatch rule and the backend Lambda function







