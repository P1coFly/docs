Event name | Description
--- | ---
`AddFederatedUserAccounts` | Adding a user to a federation
`CreateAccessKey` | Creating a static key
`CreateApiKey` | Creating API keys
`CreateCertificate` | Adding a certificate for a federation
`CreateFederation` | Creating a federation
`CreateIamCookieForSubject` | Federated user login ^*^
`CreateKey` | Creating a key pair for a service account
`CreateServiceAccount` | Creating a service account
`DeleteAccessKey` | Deleting a static key
`DeleteApiKey` | Deleting API keys
`DeleteCertificate` | Deleting a certificate for a federation
`DeleteFederation` | Deleting a federation
`DeleteKey` | Deleting a key pair for a service account
`DeleteServiceAccount` | Deleting a service account
`DetectLeakedCredential` | Detecting a secret in a public source
`SetServiceAccountAccessBindings` | Assigning access permissions for a service account
`UpdateAccessKey` | Updating a static key
`UpdateApiKey` | Updating an API key
`UpdateCertificate` | Renewing a certificate
`UpdateFederation` | Updating a federation
`UpdateKey` | Updating a key pair
`UpdateServiceAccount` | Updating a service account
`UpdateServiceAccountAccessBindings` | Updating access permissions for a service account

\* The event is not logged in the audit log unless the [audit log collection scope](../../../audit-trails/concepts/trail.md#collecting-area) for the trail is `Organization`.