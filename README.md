# ⚙️ Project 2: Serverless “Contact” Form with AWS Lambda, API Gateway & DynamoDB

## 🎯 Goal

Enhance your Vue 3 portfolio with a fully functional “Contact” form that saves messages to AWS DynamoDB — all done serverlessly using Lambda + API Gateway (no traditional backend).

## 🧰 AWS Services Used 
- Service	Purpose
- Lambda	Handles form submissions (backend logic)
- API Gateway	Creates REST API endpoint for Vue frontend
- DynamoDB	Stores submitted messages
- IAM	Grants secure access between Lambda and DynamoDB

## 🧱 Step-by-Step Implementation


1️⃣ Create a DynamoDB Table

- Go to AWS Console → DynamoDB → Create table

- Table name: PortfolioMessages

- Partition key: id (String)

- Click Create table (leave other settings default).

<img width="2558" height="818" alt="image" src="https://github.com/user-attachments/assets/94b2511b-5153-4628-b9d2-68fc171de3b9" />


2️⃣ Create a Lambda Function
Example (Python 3.9)
```
import json, boto3, uuid

def lambda_handler(event, context):
    print("Event received:", event)

    # Handle both proxy and non-proxy integration cases
    body = event.get('body')
    if body:
        # If 'body' exists, it might be a string — so parse it
        try:
            body = json.loads(body)
        except Exception:
            pass
    else:
        # No 'body' key — use the event itself
        body = event

    dynamo = boto3.resource('dynamodb')
    table = dynamo.Table('PortfolioMessages')

    # Validate expected keys exist
    if not all(k in body for k in ("name", "email", "message")):
        print("Missing required keys:", body)
        return {
            'statusCode': 400,
            'headers': {'Access-Control-Allow-Origin': '*'},
            'body': json.dumps({'error': 'Missing name, email, or message in request'})
        }

    # Save data
    table.put_item(Item={
        'id': str(uuid.uuid4()),
        'name': body['name'],
        'email': body['email'],
        'message': body['message']
    })

    return {
        'statusCode': 200,
        'headers': {'Access-Control-Allow-Origin': '*'},
        'body': json.dumps({'status': 'Message saved successfully!'})
    }

```

<img width="2553" height="1167" alt="image" src="https://github.com/user-attachments/assets/896fedcf-94c3-4ae2-850f-db1ee0e8d2ae" />


✅ Steps:

1. Go to Lambda → Create function

2. Runtime: Python 3.9 (or Node.js 18 if you prefer)

3. Name: createMessage

4. Create function

5. Under Configuration → Permissions, click Execution role → Attach policy

6. Add AmazonDynamoDBFullAccess

7. Paste the code above in Code tab

8. Click Deploy

<img width="2542" height="1153" alt="image" src="https://github.com/user-attachments/assets/68257de6-4ba8-4709-b77f-78eff92e8499" />



3️⃣ Create an API Gateway

1.Go to API Gateway → Create API

- Choose REST API → Build

- Name: My Portfolio

2.Click Actions → Create Resource

- Resource name: (i created blank you can give name)

- Resource path: /create

3.Click Actions → Create Method → POST

- Integration type: Lambda Function

- Select your Lambda (createMessage)

- Save and allow permissions.

4.Enable CORS on the /contact resource:

- Actions → Enable CORS → Confirm.

5.Click Actions → Deploy API

-Deployment stage: create

Note down your endpoint URL, e.g.
https://mz5cgdzdr5.execute-api.us-east-1.amazonaws.com/create

<img width="2542" height="1150" alt="image" src="https://github.com/user-attachments/assets/4320d8b0-9f5b-4d74-b23b-c01271fafb8e" />


4️⃣ Connect Vue 3 Contact Form

In Vue 3 app:

```
<script setup>
  await fetch("https://mz5cgdzdr5.execute-api.us-east-1.amazonaws.com/create", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      name: form.name,
      email: form.email,
      message: form.message,
    }),
  });
  alert("Message sent successfully API!");
</script>
```

5️⃣ Test the Flow

✅ Go to Vue site → Fill in the form
✅ Submit → You should see “Message sent successfully!”
✅ Go to DynamoDB → PortfolioMessages → Explore table items → message appears 🎉

<img width="2492" height="1283" alt="image" src="https://github.com/user-attachments/assets/435f287d-c399-412d-9497-32e8fd6db5d0" />

<img width="2557" height="1162" alt="image" src="https://github.com/user-attachments/assets/366d7149-f1af-4aac-aa0d-4c6bfeedb453" />


