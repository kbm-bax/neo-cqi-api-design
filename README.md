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

Once the UI loads, you can swich out the api files to see other APIs.
