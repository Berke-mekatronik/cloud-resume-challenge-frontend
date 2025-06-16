# Cloud Resume Challenge
This is the project I completed as part of the Cloud Resume Challenge. It covers deploying a resume website fully on AWS using services such as S3, CloudFront, DynamoDB, Lambda, and API Gateway. You can reach to the project from there: [Berke Ozturk Resume](https://s3.domain-of-berke.com/)

Main project is belongs to Forrest Brazeal. It can be checked from this link: [Cloud Resume Challenge](https://cloudresumechallenge.dev/docs/the-challenge/)

# Preview
![Preview_of_resume](/img/Preview_of_resume.png)

# Features
* Python, HTML, CSS, JavaScript - Core programming and styling languages used
* AWS S3 - Hosts the static website (HTML, CSS, JavaScript files)
* Amazon Route 53 - Provides custom domain name and DNS management
* Amazon CloudFront - Distributes website globally with low latency
* AWS WAF & Shield - Protects the application from web attacks
* Amazon DynamoDB - Stores and updates visitor count data
* AWS Lambda - Serverless compute service to handle backend logic
* AWS IAM - Manages permissions and security roles for AWS services
* Amazon API Gateway - Connects frontend requests to the Lambda backend

# Steps of Project
Let's dive into the project.

## Step 1 - HTML
A basic HTML page was created to serve as an online resume:

* index.html
* contact.html
* others.html

## Step 2 - CSS
Resume styled with css, please check:

* style.css

## Step 3 - Static Website
An S3 bucket was created with public access blocked (except through CloudFront) and static website hosting enabled.

**Important settings:** Enable static website hosting, set index.html as the entry point.

1. Create S3 bucket
2. It should be same as domain name
3. Upload files
![S3-objects](/img/S3-objects.PNG)
4. Click Properties, find static website hosting part, click edit
![S3-static_hosting](/img/S3-static_hosting.PNG)
5. AWS will generate URl but it wont be accesible since we didn't setup our bucket to the public
6. Click Permission, click edit, change block settings
7. Edit bucket policy, add new statement (as below)

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowPublicRead",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::s3.domain-of-berke.com/*"
        }
    ]
}
```

## Step 4 - DNS
The domain name was registered or connected via Route53. An A record was created to point to the CloudFront distribution.

1. Create domain with Amazon Route 53
2. Domain name should be same as your bucket name
3. Hosted zone will create automatically
4. Create record in hosted zone
5. Record name: s3, value: domain-of-berke.com.s3-website-us-east-1.amazonaws.com with CNAME
![Route53-domain_settings](/img/Route53-domain_settings.PNG)

## Step 5 - HTTPS
CloudFront was configured to cache and deliver the content globally with low latency.

1. Choose origin domain as our bucket, use website endpoint
![CloudFront-origin_domain](/img/CloudFront-origin_domain.PNG)
2. Under default cache behavior, choose redirect HTTP to HTTPS
![CloudFront-cache_behaviour](/img/CloudFront-cache_behaviour.PNG)
3. Enable WAF
4. Under settings, choose alternate domain name as our website(s3.domain-of-berke.com) 
5. Add custom SSL certificate, if you don't have request
![CloudFront-ssl_settings](/img/CloudFront-ssl_settings.PNG)
6. Click request certificate, request public certificate
7. Enter fully qualified domain name, request
8. In request create cname record in route 53, click create records in route 53 
9. Comeback distribution, choose your certificate and create distribution
10. Copy your domain name from distribution
11. Paste in cname that you created before

## Step 6 - Javascript (Local Testing)
Initially, a basic visitor counter was implemented using localStorage to simulate visitor tracking locally.

```
<p id="visitor-counter">Visitors: Loading...</p>

<script>
    // Function to update and display visitor count
    function updateVisitorCounter() {
        let count = localStorage.getItem('visitorCount'); // Get the visitor count from local storage

        if (!count) { // If there is no count stored (first visit)
            count = 1; // Set count to 1
        } else {
            count = parseInt(count) + 1; // Convert count to an integer and increment it
        }

        localStorage.setItem('visitorCount', count); // Store the updated count in local storage
        document.getElementById('visitor-counter').innerText = `Visitors: ${count}`; // Update the displayed text
    }

    updateVisitorCounter(); // Call the function when the page loads
</script>
```

**Note:** Later, this was replaced with a real database-backed visitor counter.

## Step 7 - Database
A DynamoDB table named VisitorCounter was created to store the visitor count.

**Partition key:** visitor_count_id (Number)

**Attribute:** visitor_count

1. Create a table(AWS DynamoDB)
2. Enter table name
3. Enter partition key(number)
4. Create an attribute that stores visitor count
5. Click explore table items
6. Click create item
7. Click add new attribute(number)
![DynamoDB-items](/img/DynamoDB-items.PNG)

## Step 8 - Python
A Lambda function was developed to:

1. Create a Lambda function (AWS Lambda)
2. Enter function name
3. Choose runtime (Python 3.12)
4. Adjust your lambda function
5. Retrieve the current visitor count from DynamoDB.
6. Increment the count.
7. Save the updated count back into the table.

Below Lambda function got created, please check from visitor_counter.py   

```
import json
import boto3
    
dynamodb = boto3.resource('dynamodb')
table = (dynamodb.Table('VisitorCounter'))

# to see connection is working or not
#print(table.creation_date_time)

def lambda_handler(event, context):

    # getting item from table
    response = table.get_item(Key={
        'visitor_count_id': 0
    })

    # just for testing
    #visitor_count_id = response['Item']["visitor_count_id"]
    #print(visitor_count_id)

    # to reach item value
    visitor_count = response["Item"]["visitor_count"]
    visitor_count += 1
    print(visitor_count)

    # to update table item
    response = table.put_item(Item={
        "visitor_count_id": 0,
        "visitor_count": visitor_count
    })

    return visitor_count
```
## Step 9 - API
An API Gateway HTTP API was created.

Integration target: Lambda function

CORS settings applied:
* Allow Origin: *

The API Gateway endpoint was used by the JavaScript fetch call.

1. In Lambda function add triger
2. Select source as API Gateway
3. Create a new API, HTTP API
4. Allow all request to API gateway(CORS settings)
![APIGateway-cors_settings](/img/APIGateway-cors_settings.PNG)
5. Edit route settings, use HTTP GET
![APIGateway-route](/img/APIGateway-route.PNG)
6. After AWS API Gatewat settings, update javascript code

## Step 10 - Update JavaScript to Fetch Visitor Count via API Gateway
Final JavaScript to fetch live visitor count:

```
<script>
    // Function to update and display visitor count
    function updateVisitorCounter() {
        fetch('https://h1m9w0eed0.execute-api.us-east-1.amazonaws.com/default/VisitorCounter')
            .then(response => response.json())
  	        .then(data => {
            document.getElementById('visitor-counter').innerText = `Visitors: ${data["visitorcount"]}`;
        })
    }
    updateVisitorCounter();
</script>
```

# Final Notes

Project folder structure:
```
/ (root)
|-- index.html
|-- contact.html
|-- others.html
|-- style.css
|-- visitor_counter.py
|-- README.md
|-- /img (project screenshots)
```

S3 + CloudFront + Route53 setup tested successfully.

Lambda + API Gateway + DynamoDB integration works as expected.

Thanks for reviewing my project! 🚀