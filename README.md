# AUTOMATE-INFRASTRUCTURE-WITH-IAC-USING-TERRAFORM-PART-3-REFACTORING-USING-MODULES
-----------------------------
### Exploring terraform modules for refactoring code

### Introducing Backend on S3

Each Terraform configuration can specify a backend, which defines where and how operations are performed, where state snapshots are stored, etc. Take a peek into what the states file looks like. It is basically where terraform stores all the state of the infrastructure in json format. So far, we have been using the default backend, which is the local backend – it requires no configuration, and the states file is stored locally. This mode can be suitable for learning purposes, but it is not a robust solution, so it is better to store it in some more reliable and durable storage.

Another useful option that is supported by S3 backend is State Locking – it is used to lock your state for all operations that could write state. This prevents others from acquiring the lock and potentially corrupting your state. State Locking feature for S3 backend is optional and requires another AWS service – DynamoDB.

* Add S3 and DynamoDB resource blocks before deleting the local state file.

* Update terraform block to introduce backend and locking.

* Re-initialize terraform

* Delete the local tfstate file and check the one in S3 bucket.

* Add outputs.

* terraform apply

* Create a file and name it backend.tf. Add the below code and replace the name of the S3 bucket you created in [Project-16](https://github.com/lateef-taiwo/AUTOMATE-INFRASTRUCTURE-WITH-IAC-USING-TERRAFORM-PART-1).

        resource "aws_s3_bucket" "terraform_state" {
        bucket = "savvy-dev-terraform-bucket"
        # Enable versioning so we can see the full revision history of our state files
        versioning {
            enabled = true
        }
        # Enable server-side encryption by default
        server_side_encryption_configuration {
            rule {
            apply_server_side_encryption_by_default {
                sse_algorithm = "AES256"
            }
            }
        }
        }

You must be aware that Terraform stores secret data inside the state files. Passwords, and secret keys processed by resources are always stored in there. Hence, you must consider to always enable encryption. You can see how we achieved that with `server_side_encryption_configuration`.

Next, we will create a DynamoDB table to handle locks and perform consistency checks. In previous projects, locks were handled with a local file as shown in `terraform.tfstate.lock.info`. Since we now have a team mindset, causing us to configure S3 as our backend to store state file, we will do the same to handle locking. Therefore, with a cloud storage database like DynamoDB, anyone running Terraform against the same infrastructure can use a central location to control a situation where terraform is running at the same time from multiple different people.

* Dynamo DB resource for locking and consistency checking:
