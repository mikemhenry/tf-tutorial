## Terraform state and remote state

If you list the contents of your directory, you'll see a file called
terraform.tfstate. This file is called the Terraform state file, and it
contains information on the resources that Terraform is managing.

As you can imagine, the information in this file is important for all
developers working with your Terraform-managed resources. So instead of using a
local file, it would be better to store this in a remote location. We recommend
using an S3 bucket for remote state. That bucket should be very locked down,
since this file can contain sensitive information about your deployment.

In the file `modules/s3backend/main.tf`, we have a configuration for an S3
bucket that can be used as a backend for Terraform state. For the purposes of
this tutorial, you can create that bucket by changing into `modules/s3backend`
directory and running `tofu plan -out tfplan` and `tofu apply tfplan`. Remember
that Terraform uses all the `.tf` files in the current directory, but not in
subdirectories.

To use that configuration as our backend, we need to configure the `terraform`
block in our `main.tf` file. Change the existing block to something like this:

```hcl
```

In this, we're keeping some of the configuration in outside of the main
Terraform. That's because some information (in our case, the bucket name) is
potentially sensitive and shouldn't be kept in a public repository. 
