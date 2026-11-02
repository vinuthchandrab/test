AI Security Review Q&A (Project: POC-on-AI-Service-Now-Integration)
 Q1. Please analyze this project code for code security.  
Asked: Find which AI model from Azure Foundry is used, whether it can create/use other models, vulnerabilities, safety, Azure security violations, and do analysis only.
Answer:  
•
The code uses Azure OpenAI via Spring AI (spring-ai-azure-openai-spring-boot-starter in pom.xml).  
•
Model selection is runtime-configured through:
◦
spring.ai.azure.openai.chat.options.deployment-name: ${AZURE_OPENAI_MODEL} (application.yml)
◦
plus Docker default: AZURE_OPENAI_MODEL: ${AZURE_OPENAI_MODEL:-gpt-4o-mini} (docker-compose.yml)
•
No code path was found that creates Azure model deployments/subscriptions. It consumes existing deployment(s).
•
Main security concerns identified:
1.
Potential secret-like string present in application.yml comment (should be treated as possible exposure and rotated if real).
2.
Webhook auth can be effectively optional if token not configured.
3.
Raw response/stack-trace logging may expose sensitive data.
4.
Broad exception fallback behavior can hide real failures.
5.
Actuator endpoints/health detail exposure may leak operational data if publicly reachable.
6.
No clear app-layer auth/authz for business endpoints in shown code.
•
Conclusion: acceptable as POC, not production-safe without hardening.
 Q2. Which AI model is configured in this code? Is it capable of deploying an unconfigured model? Does it run infinitely and consume AI tokens without API triggers?
Answer:  
•
No fixed model is hardcoded in Java.  
•
The model/deployment is taken from AZURE_OPENAI_MODEL.  
•
Docker Compose provides default deployment name gpt-4o-mini when env var is absent.  
•
Code does not deploy models.  
•
No evidence of autonomous infinite AI loop; AI calls are triggered by request-driven flows.
 Q3. Can an attacker create deployments for different AI models? Do we need separate subscription/API key per model? Can this code create subscriptions/deployments and use Astra/Solo?
Answer:  
•
This code cannot create subscriptions or deployments.  
•
To use different models, Azure deployments must already exist.  
•
Usually no separate subscription/API key per model is required; keys are typically resource-level.  
•
This app can call different deployments if AZURE_OPENAI_MODEL is changed and target deployment exists/accessible.
 Q4. Are deployments automatic without human intervention? What access is needed? If resources/deployments are deleted, can attacker recreate?
Answer:  
•
Not automatic in this codebase.  
•
Deployment creation needs Azure control-plane permissions and explicit provisioning path (portal/CLI/API/IaC/CI).  
•
Runtime app secrets (endpoint, API key, deployment name) are generally for inference usage, not deployment creation by themselves.  
•
If resources are deleted, recreation is possible only for identities still holding sufficient Azure permissions.
 Q5. How to find who created new deployments?
Answer:
 Use Azure Activity Log and filter operation:
•
Microsoft.CognitiveServices/accounts/deployments/write
 (also check delete operations as needed).
Check fields:
•
Caller  
•
Timestamp  
•
Correlation ID  
•
Resource ID  
•
Status/initiator details
Sample KQL:
AzureActivity
| where OperationNameValue =~ "Microsoft.CognitiveServices/accounts/deployments/write"
| project TimeGenerated, Caller, CallerIpAddress, ResourceGroup, ResourceId, CorrelationId, ActivityStatusValue
| order by TimeGenerated desc
 Q6. I think gpt-4o-mini is hardcoded. Where configured? Can it use GPT-6 Astra / GPT-5 Solo without modifying code?
Answer:  
•
gpt-4o-mini appears as Docker Compose default fallback:
◦
AZURE_OPENAI_MODEL: ${AZURE_OPENAI_MODEL:-gpt-4o-mini}
•
The application can use another model deployment without code change by setting AZURE_OPENAI_MODEL to that deployment name, provided it exists and is authorized.
 Q7. For using different model, must separate subscriptions be created?
Answer:  
•
Generally no.  
•
Same subscription/resource can host multiple deployments.  
•
Separate subscriptions are organizational/isolation choice, not model requirement.
 Q8. How to create new deployment without human intervention in this code?
Answer:  
•
Not supported in this code.  
•
Requires separate automation outside app (Terraform/Bicep/ARM/CLI/REST in CI/CD) with proper Azure RBAC.
 Q9. If AZURE_OPENAI_MODEL changes dynamically, does it create deployment dynamically?
Answer:  
•
No.  
•
It only changes which existing deployment name is called.  
•
If deployment does not exist, request fails.
 Q10. Different models have different costs. Need separate API key?
Answer:  
•
Usually no separate key per model.  
•
Billing differs by model usage, but key is generally bound to Azure OpenAI resource.  
•
Separate keys/resources are optional for security/billing isolation policies.
 Confirmed Config Note (from your IDE selection)
docker-compose.yml includes: AZURE_OPENAI_MODEL: ${AZURE_OPENAI_MODEL:-gpt-4o-mini}
So gpt-4o-mini is the default deployment name fallback only, overrideable via environment value.

Get Outlook for Mac
