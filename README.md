# Customer Support Chatbot with Amazon Bedrock Flows

## What I Built

This project uses Amazon Bedrock Flows for a small online shop support chatbot. The flow classifies each customer message into one of three labels:

- `BUG_REPORT`
- `PLATFORM_QUESTION`
- `OTHER`

The Condition node sends each label down its own path, and each path ends at a separate Output node.

## AWS Resources

- Region: `us-east-1`
- Tool stack: `bug-report-tool-stack`
- DynamoDB table: `BugReports-a4820eb0`
- Lambda function: `create-bug-report-a4820eb0`
- Bedrock Agent: `cscab-bug-report-agent` (`5PRIQLUST6`)
- Bedrock Agent alias: `TYORSYKVDA`
- Bedrock Flow: `cscab-customer-support-flow` (`C61E4EEBFQ`)
- Bedrock Flow alias: `M0775IZWNU`
- Bedrock Flow version: `10`
- Flow input node: `FlowInputNode`
- Evaluation stack: `bug-report-testing-stack`
- Evaluation bucket: `udacity-agentic-engineer-c1-eval-502972339313`
- Evaluation job: `flow-eval-run-2`

## Flow Behavior

The classifier prompt is set up to return only the label. The Condition node checks exact matches for `BUG_REPORT`, `PLATFORM_QUESTION`, and `OTHER`.

Bug reports first go through a detail check. Complete reports go to a Bedrock Agent that collects:

- Bug description
- Steps to reproduce
- Environment information

When all three fields are available, the agent calls the `create_bug_report` Lambda-backed action group and writes a ticket to the DynamoDB `BugReports-*` table. If the steps or environment details are missing, the flow asks for them before creating a ticket.

Platform questions go to a Prompt node with `online_shop_faq.md` embedded in the prompt. Covered questions are answered from the FAQ. If the FAQ does not cover the question, the response directs the customer to human support at `1-800-555-0199`.

Other requests use a separate redirect path and send the customer to the same support number.

## Automated Tests

`flow-tests.json` covers:

- A complete bug report
- A bug report missing details
- A covered FAQ question
- An uncovered platform question
- An unrelated support request

The flow input node is named `FlowInputNode`, and `flow-tests.json` uses that exact value.

Run the dataset generator with the deployed flow and alias:

```bash
python generate-eval-dataset.py \
  --tests-json flow-tests.json \
  --flow-id <flow-id> \
  --flow-alias-id <flow-alias-id> \
  --region us-east-1 \
  --enable-trace
```

The script writes `output_eval_dataset.jsonl`, which is the file uploaded to S3 for Bedrock Evaluations.

## Evaluation Notes

The Bedrock Evaluation job `flow-eval-run-2` completed successfully.

- Evaluation job ARN: `arn:aws:bedrock:us-east-1:502972339313:evaluation-job/b75a1tuzfi05`
- Dataset S3 URI: `s3://udacity-agentic-engineer-c1-eval-502972339313/output_eval_dataset.jsonl`
- Results S3 prefix: `s3://udacity-agentic-engineer-c1-eval-502972339313/results/`
- Correctness score: `1.0` for all 5 test records
- Notes: the bug report path created a ticket when the message included the required details and asked for steps and environment when those details were missing. The platform path answered a tracking question from the FAQ and redirected a gift-wrapping question to phone support. The other-request path redirected a gift recommendation request to phone support.
- Written observations are also saved in `evidence/14-evaluation-observations.md`.

## Evidence Checklist

- `evidence/01-flow-diagram.png`: full flow diagram and routing
- `evidence/02-classifier-prompt.png`: classifier prompt configuration
- `evidence/03-condition-expressions.png`: Condition node expressions
- `evidence/04-agent-action-group.png`: Agent/action group configuration
- `evidence/05-bug-flow.png`: complete bug report flow response
- `evidence/06-bug-follow-up.png`: bug report follow-up response
- `evidence/07-bug-ticket-created.png`: DynamoDB item created by the bug report flow
- `evidence/08-faq-prompt.png`: FAQ Prompt node with embedded FAQ content
- `evidence/09-covered-question.png`: covered platform question response
- `evidence/10-uncovered-question.png`: uncovered platform question redirect
- `evidence/11-other-request-message.png`: other-request redirect response
- `evidence/11-flow-tests.json`: copy of the flow test cases used for evaluation
- `evidence/12-flow-eval-run-2-output.jsonl`: evaluation dataset output
- `evidence/13-evaluation-results.png`: Bedrock Evaluation result summary
- `evidence/14-evaluation-observations.md`: written observations on the evaluation results
