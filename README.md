# Intro

# API Design Thought process

Read [notes.md](notes.md)

# Run Swagger UI locally to interact

Run a swagger ui container locally to check out the Open API specs defined.
```bash

docker run -d -p 8080:8080 `
>>   -v "C:\Users\kbm\work\2026\neo-cqi-api-design\neo-cqi-api-contracts:/usr/share/nginx/html/api" `
>>   -e URL=api/cqi-compliance-api.yaml `
>>   swaggerapi/swagger-ui

```

Once the UI loads, you can switch out the api files to see other APIs.

Current CQI OpenAPI contracts in `neo-cqi-api-contracts/`:

- `cqi-reference-data-api.yaml`
- `cqi-compliance-api.yaml`
- `cqi-limits-api.yaml`
- `cqi-dose-rate-changes-api.yaml`
- `cqi-device-usage-api.yaml`
- `cqi-syringe-usage-api.yaml`
- `cqi-infusion-story-api.yaml`
- `cqi-guardian-alert-api.yaml`
- `cqi-report-preferences-api.yaml`
- `cqi-data-quality-api.yaml`
- `cqi-device-logs-api.yaml`
