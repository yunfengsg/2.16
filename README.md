provider "aws" {
  region = "ap-southeast-1"
}

# Example Lambda function
resource "aws_lambda_function" "example" {
  function_name = "yunfengsg-topmovies-api"
  runtime       = "nodejs18.x"
  handler       = "index.handler"
  role          = aws_iam_role.lambda_exec.arn

  filename = "lambda_function_payload.zip" # Replace with your Lambda ZIP file
}

# IAM Role for Lambda execution
resource "aws_iam_role" "lambda_exec" {
  name = "lambda_exec_role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action    = "sts:AssumeRole"
        Effect    = "Allow"
        Principal = {
          Service = "lambda.amazonaws.com"
        }
      }
    ]
  })
}

resource "aws_iam_role_policy" "lambda_policy" {
  role = aws_iam_role.lambda_exec.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action   = ["logs:CreateLogGroup", "logs:CreateLogStream", "logs:PutLogEvents"]
        Effect   = "Allow"
        Resource = "arn:aws:logs:*:*:*"
      }
    ]
  })
}

# CloudWatch Alarm for Lambda Errors
resource "aws_cloudwatch_metric_alarm" "lambda_error_alarm" {
  alarm_name          = "LambdaErrorAlarm"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 1
  metric_name         = "Errors"
  namespace           = "AWS/Lambda"
  period              = 300
  statistic           = "Sum"
  threshold           = 1

  alarm_description = "This alarm triggers if there is at least 1 error in the Lambda function during a 5-minute period."
  actions_enabled   = true

  alarm_actions = [
    "arn:aws:sns:us-east-1:123456789012:example-topic" # Replace with a valid SNS topic ARN
  ]

  dimensions = {
    FunctionName = aws_lambda_function.example.function_name
  }
}
