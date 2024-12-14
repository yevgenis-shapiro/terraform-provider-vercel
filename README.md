# Vercel
Vercel provides the developer tools and cloud infrastructure to build, scale, and secure a faster, more personalized web

![gitlab-vercel-logos 531a59e bd202d7cc72359805f60aa231b7dfba9](https://github.com/user-attachments/assets/c3308609-abab-4651-96c6-9f2d56374ad9)




## Configuration
### Providers
The Cloudflare API key is used to authenticate into my account. Local environmental variable is used:
```
terraform {
  required_version= ">= 1.3.0"
  required_providers {
    vercel= {
      source = "vercel/vercel"
      version= "~> 0.11.4"
    }
  }
}
```

### Vercel Variable
The Terraform state is stored in AWS S3 alongside with my other IaC.

```
provider "vercel" {
  api_token: var.vercel_api_token
}

variable "vercel_api_token" {
  type     = string
  sensitive= true
}
```

### Vercel Project
First create a project on vercel with vercel_project
```
resource "vercel_project" "test_project" {
  name     = "test_blog"
  framework= "hugo"

  git_repository: {
    type= "github"
    repo= "yevgenis-shapiro/vercel_hugo_blog"
  }
}
```

### Vercel Environment Variables
Environment variables can also be set here such as which version of NodeJS to be used, which version of Hugo to be used and etc
```
 resource "vercel_project_environment_variable" "test_env" {
  project_id= vercel_project.test_project.id
  key       = "HUGO_VERSION"
  value     = "0.110.0"
  target    = ["production"]
}
```

### Vercel Deploy the site
This Terraform IaC will configure the remaining settings for this zone, such as enabling cache, WAF and DDoS protections.

```
terraform plan -out tfplan.out
terraform apply tfplan.out
```

Modify local `hostfile` with the following records for testing:

