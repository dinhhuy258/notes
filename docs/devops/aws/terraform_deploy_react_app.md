# Deploying a React Application on AWS Using Terraform

## 1. Create an S3 Bucket for Static Hosting

Amazon S3 will be used to store the React build files.

```tf
resource "aws_s3_bucket" "frontend" {
  bucket = 'frontend_bucket_name'
}
```

We need to block public access to ensure security.

```tf
resource "aws_s3_bucket_public_access_block" "frontend" {
  bucket = aws_s3_bucket.frontend.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}
```

Versioning allows you to recover previous versions of your files in case of accidental deletions or modifications.

```tf
resource "aws_s3_bucket_versioning" "frontend" {
  bucket = aws_s3_bucket.frontend.id
  versioning_configuration {
    status = "Enabled"
  }
}
```

Enable static website hosting

```tf
resource "aws_s3_bucket_website_configuration" "frontend_static_site" {
  bucket = aws_s3_bucket.frontend.id

  index_document {
    suffix = "index.html"
  }
}
```

## 2. Create an SSL Certificate with ACM

AWS ACM provides an SSL certificate to secure the application over HTTPS.

```tf
resource "aws_acm_certificate" "frontend" {
  domain_name               = local.frontend_domain_name
  subject_alternative_names = ["*.${local.frontend_domain_name}"]
  validation_method         = "DNS"

  lifecycle {
    create_before_destroy = true
  }

  # The certificate must be created in the us-east-1 region to be used with CloudFront
  provider = aws.virginia
}
```

## 3. Configure Route 53 for DNS and SSL Validation

Fetch the Hosted Zone

```tf
data "aws_route53_zone" "base_zone_name" {
  name = var.base_zone_name
}
```

Create DNS Records for ACM Validation

```tf
resource "aws_route53_record" "acm_validation" {
  for_each = {
    for dvo in aws_acm_certificate.frontend.domain_validation_options :
    dvo.domain_name => {
      name   = dvo.resource_record_name
      record = dvo.resource_record_value
      type   = dvo.resource_record_type
    }
  }

  allow_overwrite = true
  name            = each.value.name
  records         = [each.value.record]
  ttl             = 300
  type            = each.value.type
  zone_id         = data.aws_route53_zone.base_zone_name.zone_id
}
```

Validate ACM Certificate

```tf
resource "aws_acm_certificate_validation" "frontend" {
  certificate_arn         = aws_acm_certificate.frontend.arn
  validation_record_fqdns = [for record in aws_route53_record.acm_validation : record.fqdn]

  depends_on = [
    aws_acm_certificate.frontend,
    aws_route53_record.acm_validation,
  ]

  # The certificate must be created in the us-east-1 region to be used with CloudFront
  provider = aws.virginia
}

```

## 4. Deploy CloudFront as a CDN

Create an Origin Access Identity (OAI)

```tf
resource "aws_cloudfront_origin_access_identity" "frontend" {
  comment = "Origin access identity for frontend"
}
```

Fetch AWS-Managed Cache and Origin Request Policies

```tf
# AWS Managed Caching Policy - Found in AWS Management Console at CloudFront > Policies > Cache
data "aws_cloudfront_cache_policy" "caching_optimised" {
  name = "Managed-CachingOptimized"
}

# AWS Managed Caching Policy - Found in AWS Management Console at CloudFront > Policies > Origin request
data "aws_cloudfront_origin_request_policy" "cors_s3_origin" {
  name = "Managed-CORS-S3Origin"
}
```

Set Security Headers in CloudFront

```tf
resource "aws_cloudfront_response_headers_policy" "frontend_distribution" {
  name = "frontend-response-headers-policy"

  security_headers_config {
    strict_transport_security {
      access_control_max_age_sec = "31536000"
      include_subdomains         = true
      override                   = true
      preload                    = true
    }
    content_type_options {
      override = false
    }
    frame_options {
      frame_option = "SAMEORIGIN"
      override     = false
    }
  }
}
```

Enable WAF

```tf
resource "aws_wafv2_web_acl" "waf_cloudfront" {
  name        = "frontend-enable-aws-managed-rules"
  description = "Enable AWS managed ruleset and attached to Cloudfront distribution"
  scope       = "CLOUDFRONT"

  default_action {
    allow {}
  }

  rule {
    name     = "AWS-AWSManagedRulesCommonRuleSet"
    priority = 1

    override_action {
      count {}
    }

    statement {
      managed_rule_group_statement {
        name        = "AWSManagedRulesCommonRuleSet"
        vendor_name = "AWS"
      }
    }

    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "AWS-AWSManagedRulesCommonRuleSet"
      sampled_requests_enabled   = false
    }
  }

  rule {
    name     = "AWS-AWSManagedRulesAmazonIpReputationList"
    priority = 2

    override_action {
      none {}
    }

    statement {
      managed_rule_group_statement {
        name        = "AWSManagedRulesAmazonIpReputationList"
        vendor_name = "AWS"
      }
    }

    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "AWS-AWSManagedRulesAmazonIpReputationList"
      sampled_requests_enabled   = false
    }
  }

  rule {
    name     = "AWS-AWSManagedRulesAnonymousIpList"
    priority = 3

    override_action {
      none {}
    }

    statement {
      managed_rule_group_statement {
        name        = "AWSManagedRulesAnonymousIpList"
        vendor_name = "AWS"
      }
    }

    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "AWS-AWSManagedRulesAnonymousIpList"
      sampled_requests_enabled   = false
    }
  }

  rule {
    name     = "AWS-AWSManagedRulesKnownBadInputsRuleSet"
    priority = 4

    override_action {
      none {}
    }

    statement {
      managed_rule_group_statement {
        name        = "AWSManagedRulesKnownBadInputsRuleSet"
        vendor_name = "AWS"
      }
    }

    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "AWS-AWSManagedRulesKnownBadInputsRuleSet"
      sampled_requests_enabled   = false
    }
  }

  visibility_config {
    cloudwatch_metrics_enabled = false
    metric_name                = "AWS-WAF"
    sampled_requests_enabled   = false
  }

  provider = aws.virginia
}
```

Create CloudFront distribution

```tf
resource "aws_cloudfront_distribution" "frontend_distribution" {
  aliases = [
    local.frontend_domain_name,
    "www.${local.frontend_domain_name}",
  ]

  origin {
    domain_name = aws_s3_bucket.frontend.bucket_regional_domain_name
    origin_id   = "s3-frontend"

    s3_origin_config {
      origin_access_identity = aws_cloudfront_origin_access_identity.frontend.cloudfront_access_identity_path
    }
  }

  enabled             = true
  is_ipv6_enabled     = true
  comment             = "CloudFront distribution for frontend"
  default_root_object = "index.html"

  default_cache_behavior {
    cache_policy_id            = data.aws_cloudfront_cache_policy.caching_optimised.id
    origin_request_policy_id   = data.aws_cloudfront_origin_request_policy.cors_s3_origin.id
    allowed_methods            = ["GET", "HEAD"]
    cached_methods             = ["GET", "HEAD"]
    target_origin_id           = "s3-frontend"
    viewer_protocol_policy     = "redirect-to-https"
    response_headers_policy_id = aws_cloudfront_response_headers_policy.frontend_distribution.id
  }

  price_class = "PriceClass_100"

  restrictions {
    geo_restriction {
      restriction_type = "none"
      locations        = []
    }
  }

  web_acl_id = aws_wafv2_web_acl.waf_cloudfront.arn

  viewer_certificate {
    acm_certificate_arn            = aws_acm_certificate.frontend.arn
    cloudfront_default_certificate = false
    minimum_protocol_version       = "TLSv1.2_2021"
    ssl_support_method             = "sni-only"
  }
}
```

## 5. Configure Route 53 for CloudFront

```tf
resource "aws_route53_record" "frontend_record_ipv4" {
  zone_id = data.aws_route53_zone.base_zone_name.zone_id
  name    = local.frontend_domain_name
  type    = "A"

  alias {
    name                   = aws_cloudfront_distribution.frontend_distribution.domain_name
    zone_id                = aws_cloudfront_distribution.frontend_distribution.hosted_zone_id
    evaluate_target_health = false
  }
}

resource "aws_route53_record" "frontend_record_ipv6" {
  zone_id = data.aws_route53_zone.base_zone_name.zone_id
  name    = local.frontend_domain_name
  type    = "AAAA"

  alias {
    name                   = aws_cloudfront_distribution.frontend_distribution.domain_name
    zone_id                = aws_cloudfront_distribution.frontend_distribution.hosted_zone_id
    evaluate_target_health = false
  }
}

resource "aws_route53_record" "frontend_record_cname" {
  zone_id = data.aws_route53_zone.base_zone_name.zone_id
  name    = "www"
  type    = "CNAME"
  ttl     = "300"
  records = [local.frontend_domain_name]
}
```

## 6. Update S3 Bucket Policy to Allow CloudFront Access

```tf
data "aws_iam_policy_document" "frontend_policy_document" {
  statement {
    effect = "Allow"
    principals {
      type        = "AWS"
      identifiers = [aws_cloudfront_origin_access_identity.frontend.iam_arn]
    }
    actions = [
      "s3:GetObject",
    ]
    resources = [
      "${aws_s3_bucket.frontend.arn}/*"
    ]
  }
}

resource "aws_s3_bucket_policy" "frontend_policy" {
  bucket = aws_s3_bucket.frontend.id
  policy = data.aws_iam_policy_document.frontend_policy_document.json
}
```
