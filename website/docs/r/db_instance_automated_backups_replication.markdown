---
subcategory: "RDS (Relational Database)"
layout: "aws"
page_title: "AWS: aws_db_instance_automated_backups_replication"
description: |-
  Enables replication of automated backups to a different AWS Region.
---

# Resource: aws_db_instance_automated_backups_replication

Manage cross-region replication of automated backups to a different AWS Region. Documentation for cross-region automated backup replication can be found at:

* [Replicating automated backups to another AWS Region](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ReplicateBackups.html)

-> **Note:** This resource has to be created in the destination region.

## Example Usage

```terraform
resource "aws_db_instance_automated_backups_replication" "default" {
  source_db_instance_arn = "arn:aws:rds:us-west-2:123456789012:db:mydatabase"
  retention_period       = 14
}
```

## Encrypting the automated backup with KMS

```terraform
resource "aws_db_instance_automated_backups_replication" "default" {
  source_db_instance_arn = "arn:aws:rds:us-west-2:123456789012:db:mydatabase"
  kms_key_id             = "arn:aws:kms:us-east-1:123456789012:key/12345678-1234-1234-1234-123456789012"
}
```

## Example including a RDS DB instance

```terraform
provider "aws" {
  region = "us-east-1"
}

provider "aws" {
  region = "us-west-2"
  alias  = "replica"
}

resource "aws_db_instance" "default" {
  allocated_storage       = 10
  identifier              = "mydb"
  engine                  = "postgres"
  engine_version          = "13.4"
  instance_class          = "db.t3.micro"
  name                    = "mydb"
  username                = "masterusername"
  password                = "mustbeeightcharacters"
  backup_retention_period = 7
  storage_encrypted       = true
  skip_final_snapshot     = true
}

resource "aws_kms_key" "default" {
  description = "Encryption key for automated backups"

  provider = "aws.replica"
}

resource "aws_db_instance_automated_backups_replication" "default" {
  source_db_instance_arn = aws_db_instance.default.arn
  kms_key_id             = aws_kms_key.default.arn

  provider = "aws.replica"
}
```

## Argument Reference

The following arguments are supported:

* `kms_key_id` - (Optional, Forces new resource) The AWS KMS key identifier for encryption of the replicated automated backups. The KMS key ID is the Amazon Resource Name (ARN) for the KMS encryption key in the destination AWS Region, for example, `arn:aws:kms:us-east-1:123456789012:key/AKIAIOSFODNN7EXAMPLE`.
* `pre_signed_url` - (Optional, Forces new resource) A URL that contains a [Signature Version 4](https://docs.aws.amazon.com/general/latest/gr/signature-version-4.html) signed request for the [`StartDBInstanceAutomatedBackupsReplication`](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_StartDBInstanceAutomatedBackupsReplication.html) action to be called in the AWS Region of the source DB instance.
* `retention_period` - (Optional, Forces new resource) The retention period for the replicated automated backups, defaults to `7`.
* `source_db_instance_arn` - (Required, Forces new resource) The Amazon Resource Name (ARN) of the source DB instance for the replicated automated backups, for example, `arn:aws:rds:us-west-2:123456789012:db:mydatabase`.

## Attributes Reference

In addition to all arguments above, the following attributes are exported:

* `id` - The Amazon Resource Name (ARN) of the replicated automated backups.

## Timeouts

`aws_db_instance_automated_backups_replication` provides the following [Timeouts](https://www.terraform.io/docs/configuration/blocks/resources/syntax.html#operation-timeouts)
configuration options:

- `create` - (Default `75m`) How long to wait before RDS successfully starts replication of automated backups to a different AWS Region.
- `delete` - (Default `75m`) How long to wait before RDS stops automated backup replication for a DB instance.

## Import

RDS instance automated backups replication can be imported using the `arn`, e.g.,

```
$ terraform import aws_db_instance_automated_backups_replication.default arn:aws:rds:us-east-1:123456789012:auto-backup:ab-faaa2mgdj1vmp4xflr7yhsrmtbtob7ltrzzz2my
```
