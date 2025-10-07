# CloudTrail Configuration

CloudTrail was configured manually in the AWS Console to:
- Record **all management and data events** in all regions.
- Send logs to a dedicated **S3 bucket** with restricted access.
- Integrate with **EventBridge** for event-driven alerts.

The goal was to validate the monitoring and alerting pipeline through a hands-on approach.

No IaC or JSON exports are included here — this lab focuses on conceptual security design and incident validation rather than automated deployment.
