# Google Cloud Pub/Sub: Qwik Start – Console
- Cloud Pub/Sub is Google Cloud's fully managed, real-time messaging service that allows you to send and receive messages between independent applications. The "Qwik Start – Console" lab walks you through the basics using the Google Cloud Console UI (no code/CLI required).

## 🔑 Core Concepts
- TermDescriptionTopicA named resource to which messages are sent by publishersSubscriptionA named resource representing the stream of messages from a topicPublisherAn app that creates and sends messages to a topicSubscriberAn app that receives messages from a subscriptionMessageThe data sent through the system (+ optional attributes)

## 🏗️ How It Works
Publisher → [Topic] → [Subscription] → Subscriber
- A publisher sends a message to a Topic
Pub/Sub stores it and fans it out to all Subscriptions attached to that topic
Subscribers pull (or receive via push) those messages and acknowledge them
Only after acknowledgment is the message removed from the queue


## 🖥️ What the "Console" Lab Covers
 - Step 1 – Create a Topic
   - Navigate to Pub/Sub → Topics in the Cloud Console
   - Click Create Topic → give it a name (e.g., MyTopic)

- Step 2 – Create a Subscription
   - Go to your topic → click Create Subscription
   - Name it (e.g., MySub), choose Pull delivery type
   - This links the subscription to your topic

- Step 3 – Publish a Message
   - Inside the topic → go to Messages tab
   - Click Publish Message → type a message body (e.g., Hello World)
   - Hit Publish

- Step 4 – Pull the Message
   - Go to the Subscription → click Pull
   - You'll see your message appear with its message ID, data, and publish time
   - Acknowledge it to remove it from the queue


📦 Delivery Types
- TypeHow it worksPullSubscriber calls the Pub/Sub API to fetch messages manuallyPushPub/Sub sends HTTP POST requests to your subscriber's endpoint
The Console lab uses Pull — easier to test without an endpoint.

## ✅ Key Takeaways

- Pub/Sub decouples producers from consumers (they don't need to know about each other)
-Messages are durably stored until acknowledged (up to 7 days by default)
- It supports at-least-once delivery — a message may be delivered more than once
- Scales automatically — no infrastructure to manage
- Used in event-driven architectures, streaming pipelines (with Dataflow), IoT, logging, etc.


🔗 Real-World Use Cases

- Log ingestion → stream app logs to BigQuery via Pub/Sub + Dataflow
- Microservices communication → decouple services in a distributed system
- IoT data ingestion → devices publish sensor data to topics
- Task queues → async background job processing
