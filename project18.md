# AUTOMATING INFRASTRUCTURE WITH IAC USING TERRAFORM PART 3 – REFACTORING
## INTRODUCTION
In continuation to [Project 17](https://github.com/nicedozie4u/Project17/blob/main/project17.md), the entire code is refactored inorder to simplify the code using a Terraform tool called **Module**.

The following outlines detailed step taken to achieve this:


## STEP 1: Refactoring The Codes Using Module

- Creating a folder called **modules**
- Creating the following folders inside the **modules** folder to combine resources of the similar type: **ALB, VPC, Autoscaling, Security, EFS, RDS, Compute**
- Creating the following files for each of the folders: **main.tf, variables.tf and output.tf**

**pbl folder structure**

![](./images/module00.png)

- Refactoring the code for **VPC** folder:

**outputs.tf**

![](./images/vpc%20output.png)

- Refactoring the code for **ALB** folder:

**variables.tf**

![](./images/alb%20variables.png)

**outputs.tf**

![](./images/alb%20outputs.png)

- Refactoring the code for **Autoscaling** folder:

**variables.tf**

![](./images/autoscaling%20variables.png)

- Refactoring the code for **security** folder:

**outputs.tf**

![](./images/security%20outputs.png)

- Refactoring the code for **EFS** folder:

**variables.tf**

![](./images/efs%20variables.png)

- Refactoring the code for **RDS** folder:

**variables.tf**

![](./images/rds%20variables.png)

- Refactoring the code the root **main.tf** folder:

![](./images/root%20main.png)
![](./images/root%20main02.png)

- The complete code structure is stored in this [repo](https://github.com/nicedozie4u/PBL-project-18)

## STEP 2: Executing The Terraform Plan

- First initialize terraform
**terraform init**

![](./images/terraform%20init.png)

- To ensure the validation of the whole setup, running the command **terraform validate**

![](./images/terraform%20validate.png)

- Testing the configuration by running the command **terraform plan**

![](./images/terraform%20plan01.png)
![](./images/terraform%20plan02.png)
![](./images/terraform%20plan03.png)
![](./images/terraform%20plan04.png)
![](./images/terraform%20plan05.png)
![](./images/terraform%20plan06.png)
![](./images/terraform%20plan07.png)
![](./images/terraform%20plan08.png)
![](./images/terraform%20plan09.png)
![](./images/terraform%20plan10.png)
![](./images/terraform%20plan11.png)
![](./images/terraform%20plan11.png)

**terraform apply**

![](./images/terraform%20apply00.png)

![](./images/terraform%20apply01.png)

![](./images/terraform%20apply03.png)

![](./images/terraform%20apply%20complete.png)

- Confirm infrastructure created 

![](./images/ec2-instances.png)

![](./images/sg.png)

![](./images/vpc.png)

![](./images/subnets.png)

![](./images/rtb.png)



## Configuring A Backend On The S3 Bucket
By default the Terraform state is stored locally, to store it remotely on AWS using S3 bucket as the backend and also making use of DynamoDB as the State Locking the following setup is done:
- Create a file called **main.tf** in the root directory and enter the following code:
```
resource "aws_s3_bucket" "terraform-state" {
  bucket = "dozie2-dev-terraform-bucket"
  force_destroy = true
}
resource "aws_s3_bucket_versioning" "version" {
  bucket = aws_s3_bucket.terraform-state.id
  versioning_configuration {
    status = "Enabled"
  }
}
resource "aws_s3_bucket_server_side_encryption_configuration" "first" {
  bucket = aws_s3_bucket.terraform-state.id
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}
```
- Adding the following code which creates a DynamoDB table to handle locks and perform consistency checks:
```
resource "aws_dynamodb_table" "terraform_locks" {
  name         = "terraform-locks"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"
  attribute {
    name = "LockID"
    type = "S"
  }
}
```
- Terraform expects that both S3 bucket and DynamoDB resources are already created before configuring the backend

![](./images/create%20s3bucket.png)


- Create a file called **backend.tf** and enter the following code to configure the backend:
```
terraform {
  backend "s3" {
    bucket         = "dozie2-dev-terraform-bucket"
    key            = "global/s3/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```

![](./images/create%20backend%20file.png)


![](./images/)