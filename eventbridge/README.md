# EventBridge Configuration

All EventBridge rules for this lab were configured manually in the AWS Console.

They include:
- A rule detecting root logins via CloudTrail.
- A rule triggering SNS alerts on Access Key creation and GetCallerIdentity API calls.

These configurations were created manually to emphasize the event-driven detection pipeline:
**CloudTrail → EventBridge → SNS → Email Alert**.

No JSON rule definitions are included here, since this project focuses on conceptual understanding and verification rather than IaC automation.

Refer to the main README section **“Monitoring with CloudTrail & EventBridge”** for detailed steps and screenshots.
