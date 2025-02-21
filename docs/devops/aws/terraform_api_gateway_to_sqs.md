# Configuring API Gateway to SQS Queue Using Terraform

First we need to create SQS queue

```tf
resource "aws_sqs_queue" "queue" {
  name                       = "sqs-queue"
  visibility_timeout_seconds = 300 # 5 minutes
}
```

Now we need to define an IAM role so API Gateway has the necessary permissions to SendMessage to our SQS queue.

```tf
data "aws_iam_policy_document" "api_gateway_assume_role" {
  statement {
    effect = "Allow"

    principals {
      identifiers = ["apigateway.amazonaws.com"]
      type        = "Service"
    }

    actions = [
      "sts:AssumeRole"
    ]
  }
}

resource "aws_iam_role" "api_gateway" {
  name               = "api-gateway-role"
  assume_role_policy = data.aws_iam_policy_document.api_gateway_assume_role.json
}

data "aws_iam_policy_document" "api_gateway_policy_statement" {
  statement {
    sid    = "SqsAll"
    effect = "Allow"
    actions = [
      "sqs:*",
    ]
    resources = [
      "arn:aws:sqs:${data.aws_region.current.name}:${data.aws_caller_identity.current.account_id}:sqs-queue",
    ]
  }
}

resource "aws_iam_policy" "api_gateway" {
  name   = "api-gateway-policy"
  policy = data.aws_iam_policy_document.api_gateway_policy_statement.json
}

resource "aws_iam_role_policy_attachment" "api_gateway" {
  role       = aws_iam_role.api_gateway.name
  policy_arn = aws_iam_policy.api_gateway.arn
}
```

Now define an API Gateway REST API. We will host the POST method at the root `/` of the API.

```tf
resource "aws_api_gateway_rest_api" "api" {
  name        = "sqs-api"
  description = "POST records to SQS queue"
}
```

Create an API Gateway POST method for the API

```tf
resource "aws_api_gateway_method" "post_api" {
  rest_api_id      = aws_api_gateway_rest_api.api.id
  resource_id      = aws_api_gateway_rest_api.api.root_resource_id
  api_key_required = false
  http_method      = "POST"
  authorization    = "NONE"
}
```

Now we can define the API gateway integration that will forward records into the SQS queue.

```tf
resource "aws_api_gateway_integration" "api_integration" {
  rest_api_id             = aws_api_gateway_rest_api.api.id
  resource_id             = aws_api_gateway_rest_api.api.root_resource_id
  http_method             = aws_api_gateway_method.post_api.http_method
  type                    = "AWS"
  integration_http_method = "POST"
  passthrough_behavior    = "NEVER"
  credentials             = aws_iam_role.api_gateway.arn
  uri                     = "arn:aws:apigateway:${data.aws_region.current.name}:sqs:path/${aws_sqs_queue.queue.name}"

  request_parameters = {
    "integration.request.header.Content-Type" = "'application/x-www-form-urlencoded'"
  }

  request_templates = {
    "application/json" = "Action=SendMessage&MessageBody=$input.body"
  }
}
```

For the FIFO queue, the request template should be

```tf
resource "aws_api_gateway_integration" "api_integration" {
  ...
  request_templates = {
    "application/json" = <<EOF
#set($deduplicationId = $input.json('$.email'))
Action=SendMessage&MessageGroupId=default&MessageDeduplicationId=$deduplicationId&MessageBody=$input.body
EOF
  }
}
```

We should define a basic 200 handler for successful requests with a custom response message. Layer on more responses as needed.

Notice the response_templates value below, which is what the service will return as a `200` status code and message. The selection_pattern is nothing more than a regex pattern to match any 2XX status codes that come back from SQS, which will then return a `200` and the json message `{"message": "Success"}` to the client from the API Gateway request.

```tf
resource "aws_api_gateway_method_response" "post_api_success" {
  rest_api_id = aws_api_gateway_rest_api.api.id
  resource_id = aws_api_gateway_rest_api.api.root_resource_id
  http_method = aws_api_gateway_method.post_api.http_method
  status_code = 200

  response_models = {
    "application/json" = "Empty"
  }
}

resource "aws_api_gateway_integration_response" "post_api_success_response" {
  rest_api_id       = aws_api_gateway_rest_api.api.id
  resource_id       = aws_api_gateway_rest_api.api.root_resource_id
  http_method       = aws_api_gateway_method.post_api.http_method
  status_code       = aws_api_gateway_method_response.post_api_success.status_code
  selection_pattern = "^2[0-9][0-9]"

  response_templates = {
    "application/json" = "{\"message\": \"Success!\"}"
  }

  depends_on = ["aws_api_gateway_integration.api"]
}
```

And lastly, create the API Gateway deployment.

```tf
resource "aws_api_gateway_deployment" "api" {
  rest_api_id = aws_api_gateway_rest_api.api.id
  stage_name  = "main"

  depends_on = [
    aws_api_gateway_integration.api,
  ]
}
```
