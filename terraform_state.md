## Terraform state and remote state

If you list the contents of your directory, you'll see a file called
terraform.tfstate. This file is called the Terraform state file, and it
contains information on the resources that Terraform is managing.

As you can imagine, the information in this file is important for all
developers working with your Terraform-managed resources. So instead of using a
local file, it would be better to store this in a remote location. We recommend
using an S3 bucket for remote state. That bucket should be very locked down,
since this file can contain sensitive information about your deployment.

In the file `backend/main.tf`, we have a configuration for an S3
bucket that can be used as a backend for Terraform state. For the purposes of
this tutorial, you can create that bucket by changing into `backend/main.tf`
directory and running `tofu plan -out tfplan` and `tofu apply tfplan`. Remember
that Terraform uses all the `.tf` files in the current directory, but not in
subdirectories. Be sure to remember the name you choose for the bucket, we will need it later
to tell terraform where to find our state file.

To use that configuration as our backend, we need to configure the `terraform`
block in our `main.tf` file. Change the existing block to something like this:

```hcl
# versions.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.0"
    }
  }
# bucket to store the state
  backend "s3" {
    key    = "terraform.tfstate"
    region = "us-west-2"   # needs to match region of the state bucket made previously 
  }
}

# variables.tf
variable "bucket_name" {
  description = "The name of the S3 bucket"
  type        = string
}

# main.tf
resource "aws_s3_bucket" "my_bucket" {
  bucket = var.bucket_name
}

# outputs.tf
output "bucket_name" {
  value = aws_s3_bucket.my_bucket.bucket
}
```

In this, we're keeping some of the configuration in outside of the main
Terraform. That's because some information (in our case, the bucket name) is
potentially sensitive and shouldn't be kept in a public repository. 

To keep the state bucket's name out of a public repository, we will use an environmental variable `TF_STATE_BUCKET` that we can pass into our `terraform init command`:
Note, this is just one way to pass this information in, you can also pass in an HCL file that has the backend bucket name.

```
export TF_STATE_BUCKET=BUCKET-NAME-YOU-PICKED
tofu init -backend-config="bucket=$TF_STATE_BUCKET"
```


You will then get a message that looks like this:

```
Do you want to copy existing state to the new backend?
  Pre-existing state was found while migrating the previous "local" backend to the
  newly configured "s3" backend. No existing state was found in the newly
  configured "s3" backend. Do you want to copy this state to the new "s3"
  backend? Enter "yes" to copy and "no" to start with an empty state.
```

Type "yes" to migrate your local tfstate file to the cloud.

Now when you run `aws s3 ls` you should see 2 buckets listed.
One bucket should be the name of the bucket you are using to track the state file, and the other bucket name should be the one you created with your modified `main.tf` file.
You don't need to pass the environmental variable in for future commands since it will store the backend information in the `.terraform/` directory.
