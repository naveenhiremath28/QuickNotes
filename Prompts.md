
```
Interview me in detail using the AskUserQuestionTool about literally anything: technical implementation, UI & UX, concerns, tradeoffs, etc. but make sure the questions are not obvious. Be very in-depth and continue interviewing me continually until it's complete, then write the spec to the file
```

```
Interview me in detail using the AskUserQuestionTool about literally anything: technical implementation. but make sure the questions are not obvious. Be very in-depth (ask upto 5-10 questions)and continue interviewing me continually until it's complete, write the interviewed questions and summary progress at the same time to a file along with the plan.
```

export OTEL_SERVICE_NAME=finternet-units-workflow
export SERVICE_VERSION=1.0.0
export OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4318
export OTEL_EXPORTER_OTLP_HEADERS=authorization=246da7cc-626d-48ba-b7a4-eed53c9ee0ed



### 1. Pie Chart Duplicate Entry

- **Description:** 5 USDC appears twice in the pie chart despite the user only having 7 USDC total.
### 2. Dashboard Data Source Incorrect

- **Description:** Dashboard is not pulling from the groupby (composite) view. It should show 7 USDC and 0.048 SOL units correctly aggregated. (the entire dashboard should come from the groupby view (compsite) -> means there should be 7 USDC units and 0.048 units of SOL.)
refer tokenSearch api that we are calling its using groupBy tokenClass field, so we will get aggregated response
## Interview Instructions
Interview me in detail using the AskUserQuestionTool about literally anything: technical implementation. but make sure the questions are not obvious. Be very in-depth (ask upto 5 questions)and continue interviewing me continually until it's complete, write the interviewed questions and summary progress at the same time to a file along with the plan.