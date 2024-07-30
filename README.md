## Objective

Design and implement a real-time data processing pipeline in AWS to manage a simulated e-commerce platform's inventory data. The pipeline captures inventory events, processes them in real-time, and updates the inventory system accordingly.

## Problem Statement

Your e-commerce platform needs to track real-time inventory changes for its products. Every time a product is added, removed, or its quantity changes, an event is generated. Your task is to design a real-time processing system that will capture these events, process them, and update a real-time inventory system.

## Requirements

- **Data Source**: A mock script you'll design will simulate inventory events by generating and publishing them to an AWS Kinesis stream.
- **Event Processing**: Use AWS EventBridge to capture events from the Kinesis stream and forward them for processing.
- **Data Transformation and Storage**:
  - Process the real-time stream of data to calculate the latest inventory state for each product.
  - Store the processed data in DynamoDB, ensuring that inventory counts remain consistent and accurate.
- **Inventory Events Include**:
  - Product added to inventory.
  - Product removed from inventory.
  - Product quantity changes (increment or decrement).

## Architecture

![Architecture Diagram](images/image22.png)

## Can we invoke two Lambda functions from EventBridge?

No, refer to the screenshot below for more information.

![EventBridge and Lambda](images/image3.png)

---

## Step 1: Mock Data Generator

[Mock Data Generator Colab Notebook](https://colab.research.google.com/drive/1JbkrwxnE5dcusBi5-wML_dWLJgGGNEPk#scrollTo=me6Y9IkmaZMc)

---

## Step 2: Create Kinesis Data Stream

![Kinesis Data Stream](images/image7.png)

Trigger the mock data and you may encounter the following error:



> Exception occurs: An error occurred (AccessDeniedException) when calling
the PutRecord operation: User:
arn:aws:iam::381491939671:user/aws-de-user is not authorized to perform:
kinesis:PutRecord on resource:
arn:aws:kinesis:us-east-1:381491939671:stream/e-commerce-streaming
because no identity-based policy allows the kinesis:PutRecord action

So from kinesis we can't directly add permission , To resolve this
issue, you need to update the IAM policy attached to the IAM user
aws-de-user to include permissions for kinesis:PutRecord on the specific
Kinesis stream
arn:aws:kinesis:us-east-1:381491939671:stream/e-commerce-streaming.

**Step 3:**
Go to Iam user and update the IAM policy for the user to
make access to this. Because you are using access key and secret key of
this user to access the kinesis from external source(google collab here)

![](images/image19.png)

Above we have given kinesis all access to the IAM user aws-de-user for
all kinesis streams, but what if we want to give only record access into
only one specific kinesis stream.

Go to add permission, add inline permission and create a json with below
json string and save.

> {

\"Version\": \"2012-10-17\",

\"Statement\": \[

{

\"Effect\": \"Allow\",

\"Action\": \"kinesis:PutRecord\",

\"Resource\":
\"arn:aws:kinesis:us-east-1:381491939671:stream/eCommerceStream\"

}

\]

}

Just give arn of the specific resource in the Resource key

Now we have data inserted from a mock data generator into Kinesis shard.

![](images/image1.png)

**Step 4: Create a lambda function that will connect to kinesis stream
directly and read data and update dynamo DB**

![](images/image2.png)

**Step 5: Set up AWS EventBridge to capture the events from the Kinesis
stream.**

![](images/image17.png)

Connect Kinesis stream to lambda that we created in step 4

This lambda itself can send sns or alerts if the threshold value is
crossed. Eg: Quantity exceeds 1000, send SNS.

Step 6: Create DynamoDB to store the inventory details

![](images/image13.png)

**Can we delete from Kinesis**

To delete items from an Amazon Kinesis stream, you need to understand
that Kinesis streams are designed for append-only operations. You can\'t
directly delete individual records from a Kinesis stream. Instead,
records in a Kinesis stream expire based on the stream\'s retention
period, which can be set between 24 hours and 365 days.

Step 7: Remember to give necessary permissions to lambda, and
eventbridge pipe.

Eventbridge IAM policy

![](images/image8.png)

Lambda function IAM policy

![](images/image18.png)

Data inserted from mock data generator

![](images/image14.png)

Kinesis stream has data

![](images/image9.png)

Through event bridge to lambda and from lambda data landed in dynamoDB

![](images/image16.png)

Did some updates, insert from mock data generator

![](images/image10.png)

Into shrad 1 two additional records

![](images/image15.png)

Others 3 in shrad2

![](images/image21.png)

After one add event

sample_dict = {

\'Watch\': 6

}

![](images/image5.png)

Now remove Oven

![](images/image20.png)

![](images/image6.png)

Oven removed from dynamoDB

![](images/image11.png)

Updating quantity

![](images/image12.png)

Quantity of phone updated

![](images/image4.png)




## Conclusion

In this project, we successfully designed and implemented a real-time data processing pipeline for an e-commerce platform's inventory data using AWS services. The pipeline efficiently captures, processes, and updates inventory events in real-time, ensuring the inventory system remains accurate and up-to-date. Key accomplishments include:

1. **Real-Time Data Capture and Processing**: Utilizing AWS Kinesis for streaming inventory events and AWS EventBridge for event-driven processing allowed us to handle real-time data efficiently.
2. **Data Transformation and Storage**: We transformed the streaming data to calculate the latest inventory state and stored the processed data in DynamoDB, ensuring consistency and accuracy.
3. **Robust Architecture**: The architecture, comprising Kinesis, EventBridge, Lambda, and DynamoDB, provided a scalable and reliable solution for real-time inventory management.
4. **Error Handling and Security**: By separating the event generation, processing, and storage into distinct components, we ensured better error handling and easier debugging. Additionally, proper IAM policies and permissions were set up to secure the pipeline.
5. **CICD Integration**: Integrating CICD using AWS CodeBuild streamlined the deployment process, allowing for continuous integration and delivery of changes.

Overall, the project demonstrates the effective use of AWS services to build a real-time, scalable, and secure data processing pipeline for managing e-commerce inventory data.
