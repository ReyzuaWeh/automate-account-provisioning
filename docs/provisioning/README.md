# Overview
Setting an n8n workflows that automatically create a user account — with the correct
default role — in each of our internal tools whenever a provisioning webhook is triggered.
Every workflow starts from the same base template (flowable-provisioning-workflow.json,
attached to this ticket) rather than being built from scratch.

CODE : *DEV-868*

# Sub Tasks
## Twenty CRM (DEV-869)
**Story**: As an operator, I want an n8n workflow that provisions a new TwentyCRM workspace member with the standard role, so a new hire gets CRM access without manual setup.

## Password Field
The `password` field sent in the webhook payload is **received but intentionally ignored**.

TwentyCRM's invite flow (`sendInvitations`) does not accept a pre-set password because there is no argument for it in the API. Instead, the invited user sets their own password when they accept the invite email. Any password value sent in the webhook is discarded, not stored, and not forwarded to TwentyCRM.

### How's the Workflow?
Choosing to use the same workflow like inviting in UI due to [problem](README.md#twenty-crm) found while working on it. Here is the detail.

![N8N Twenty CRM Workflow](../daily/20260913_DEV-869_Acceptances/twentycrm-workflow.png)

### Acceptance
1. [New member shows up](../daily/20260913_DEV-869_Acceptances/twentycrm-invited.png)
2. [Role assigned is Member](../daily/20260913_DEV-869_Acceptances/twentycrm-invited.png)
3. [Wrong secret](../daily/20260913_DEV-869_Acceptances/request.png)
4. [Password Note](README.md#password-field)

# Daily Progress
| Date | Summary | Evicdence |
| --- | --- | --- |
| 10 September 2026 | Create a base configuration, docker compose file, and github repository | [Github Repository](https://github.com/ReyzuaWeh/automate-account-provisioning) |
| 12 September 2026 | Planning to continue task for TwentyCRM config (DEV-869). However, there's an issue I found about role's permission | [Role Permission Problem](../daily/20260912/twentycrm-forbidden.png)
| 13 September 2026 | Targetting to fix TwentyCRM role's permission issue and completed DEV-869 if possible | [DEV-869 Complete](../daily/20260913_DEV-869_Acceptances/) |


# Development Struggle
## Twenty CRM
1. Forbidden Permission

![Role Permission Problem](../daily/20260912/twentycrm-forbidden.png)

Can't add member due to forbidden. It's actually has used api key from admin, still the problem still remain. Haven't found any solutiun for it now.

**SOLUTION**
>DO NOT USE `/graphql` ENDPOINT. USE `/metadata`
The problem I've got before is because I'm using `/graphql` endpoint and using `CreateWorkspaceMember` mutation. The solution is to use `/metadata` endpoint instead. The request body for invite should be like this:
```json
{
    "query": "mutation SendInvitations($emails: [String!]!, $roleId: UUID) {\n  sendInvitations(emails: $emails, roleId: $roleId) {\n    success\n    errors\n    result {\n      ... on WorkspaceInvitation {\n        id\n        email\n        roleId\n        expiresAt\n      }\n    }\n  }\n}",
    "variables": {
        "emails": [
            "{{ $json.body.employeeEmail }}"
        ]
    },
    "operationName": "SendInvitations"
}
```

>Note: Also, I find that admin api key has a limited acces than user admin. the token use user admin token.


