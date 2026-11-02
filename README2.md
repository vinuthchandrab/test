Summary: This project does not create Azure subscriptions or AI deployments; it only calls an existing Azure OpenAI deployment configured at runtime.
•
Model selection is via AZURE_OPENAI_MODEL; in Docker Compose, fallback is gpt-4o-mini (${AZURE_OPENAI_MODEL:-gpt-4o-mini}).
•
Changing AZURE_OPENAI_MODEL switches the target deployment name only; it does not create a deployment.
•
To use another model (e.g., GPT-6 Astra / GPT-5 Solo), that model must already be deployed in Azure OpenAI and accessible.
•
Separate subscription/API key per model is usually not required; keys are typically resource-level. Separate resources/keys are optional for isolation/governance.
•
No evidence of infinite autonomous AI token usage in this code; calls are request/event-driven.
•
Security posture is POC-level, not production-hardened. Key risks:
◦
possible secret-like value in config comments,
◦
webhook auth can be optional if token unset,
◦
potentially sensitive logging,
◦
broad exception fallbacks,
◦
broad actuator exposure if internet-accessible.
•
To find who created deployments, use Azure Activity Log on operation Microsoft.CognitiveServices/accounts/deployments/write and inspect Caller/timestamp/correlation ID.
