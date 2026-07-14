
# Component 7 — Amazon API Gateway

## Objective

The seventh component of this project was to expose the Lambda function through a public HTTPS endpoint using Amazon API Gateway.

Up until this stage, every submission had been tested manually from within the AWS Lambda console. API Gateway allows external clients, such as a website running in a user's browser, to invoke the Lambda function over HTTPS.

---

# AWS Services Used

- Amazon API Gateway
- AWS Lambda
- Amazon DynamoDB
- Amazon SES

---

# What Was Created

An HTTP API was created with the following configuration:

```text
API Name:
ContactAPI

API Type:
HTTP API

Route:
POST /contact

Integration:
contactFormHandler Lambda Function

Stage:
$default

Auto Deploy:
Enabled
```

The API automatically exposes the following endpoint:

```text
https://3rxwrsc561.execute-api.us-east-1.amazonaws.com/contact
```

---

# Why API Gateway?

Although the Lambda function already contained all of the application logic, it could only be executed manually inside AWS.

API Gateway provides a secure public entry point that allows external applications to invoke the Lambda over HTTPS.

For this project, an **HTTP API** was chosen because:

- it is cheaper than the traditional REST API
- lower latency
- easier to configure
- ideal for Lambda proxy integrations
- request/response mapping templates were unnecessary

---

# Resource-Based vs Identity-Based Policies

Creating the Lambda integration automatically attached a **resource-based policy** to the Lambda function.

This allowed API Gateway to invoke the function.

At the same time, the Lambda execution role already contained **identity-based policies** allowing it to interact with other AWS services.

The difference is:

| Policy Type | Controls |
|-------------|----------|
| Identity-based policy | What the Lambda is allowed to do to other AWS services |
| Resource-based policy | Who is allowed to invoke or access the Lambda |

Both permissions were required simultaneously:

```text
Browser
        │
        ▼
API Gateway
        │
(Resource-based policy allows invocation)
        ▼
Lambda
        │
(Identity-based policies allow DynamoDB & SES access)
        ▼
DynamoDB / SES
```

---

# CORS Configuration

Cross-Origin Resource Sharing (CORS) was configured so that only the hosted CloudFront website could communicate with the API.

Configuration:

```text
Allowed Origin:
https://doo0zc8l06gjf.cloudfront.net

Allowed Methods:
POST
OPTIONS

Allowed Headers:
content-type

Allow Credentials:
Disabled

Preflight Cache:
0 seconds
```

Restricting CORS to the CloudFront domain prevents arbitrary websites from making browser requests to the API.

---

# Stage Configuration

The API was deployed using the **$default** stage.

Unlike named stages such as:

```text
/dev
/test
/prod
```

the `$default` stage is not included within the URL path.

This allows the endpoint to remain:

```text
https://3rxwrsc561.execute-api.us-east-1.amazonaws.com/contact
```

instead of:

```text
https://3rxwrsc561.execute-api.us-east-1.amazonaws.com/prod/contact
```

Auto Deploy was enabled, meaning every API change is automatically published without requiring a manual deployment.

---

# Problems Encountered

## Using the Wrong URL

Initially, the API Route ARN was copied instead of the Invoke URL.

An ARN identifies AWS resources internally but cannot be accessed over the internet.

The correct endpoint is the API Gateway Invoke URL found under the deployed Stage.

---

## Stage Not Yet Deployed

Initially the API showed:

```text
Stage: -
```

This indicated that the route existed but had not yet been deployed.

Creating routes alone does not make an API reachable.

Once the `$default` stage was deployed with Auto Deploy enabled, the endpoint became publicly accessible.

---

# Verification

The API endpoint was tested directly using a POST request.

```bash
curl -X POST https://3rxwrsc561.execute-api.us-east-1.amazonaws.com/contact \
-H "Content-Type: application/json" \
-d '{"name":"API Test","email":"lewisorourke29@gmail.com","message":"Testing via API Gateway"}'
```

Response:

```json
{
    "message": "Submission received",
    "submissionId": "f4a215a9-15e9-4b3c-a44a-e52cb26938fc"
}
```

This successful response confirmed that:

- API Gateway successfully received the request.
- API Gateway invoked the Lambda function.
- Lambda successfully validated the request.
- Lambda wrote the submission into DynamoDB.
- Lambda successfully triggered Amazon SES.
- The API returned a successful response over HTTPS.

This was the first complete end-to-end test performed entirely outside of the AWS Lambda console.

---

# Security Considerations

Several security controls were deliberately implemented:

- Lambda can only be invoked by this API Gateway through its resource-based policy.
- Lambda retains least-privilege IAM permissions.
- CORS only allows requests from the CloudFront website.
- The API only exposes a single POST endpoint.
- Communication occurs entirely over HTTPS.

---

# Key Design Decisions

| Decision | Reason |
|----------|--------|
| HTTP API instead of REST API | Lower cost and lower latency |
| `$default` stage | Cleaner endpoint without `/prod` |
| Auto Deploy enabled | Eliminates manual deployments |
| Restricted CORS | Prevents browser requests from untrusted origins |
| Lambda Proxy Integration | Passes the HTTP request directly into Lambda |

---

# Lessons Learned

During this component I learned:

- API Gateway provides a public HTTPS endpoint for AWS services.
- HTTP APIs are cheaper and simpler than REST APIs for Lambda integrations.
- API Gateway automatically creates a resource-based policy allowing it to invoke Lambda.
- Resource-based and identity-based policies solve different security problems.
- Routes do not become publicly available until a stage has been deployed.
- The `$default` stage removes the stage name from the URL.
- CORS should be configured with the minimum required origins rather than allowing all websites.
- Testing with `curl` provides an independent verification that the entire application works outside of the AWS console.
