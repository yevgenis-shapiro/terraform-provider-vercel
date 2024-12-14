# Vercel
Vercel provides the developer tools and cloud infrastructure to build, scale, and secure a faster, more personalized web

![gitlab-vercel-logos 531a59e bd202d7cc72359805f60aa231b7dfba9](https://github.com/user-attachments/assets/c3308609-abab-4651-96c6-9f2d56374ad9)




## Configuration
### Providers
The Cloudflare API key is used to authenticate into my account. Local environmental variable is used:
```
echo 'export CLOUDFLARE_API_KEY="123"' >> ~/.bashrc.alex
echo 'export CLOUDFLARE_EMAIL="myemail@domain.com"' >> ~/.bashrc.alex
```

### Vercel Backend
The Terraform state is stored in AWS S3 alongside with my other IaC.

Note: When I first played with this Cloudflare provider, I was using a local state, but once I added the `backend.tf`, I had to run the following command to migrate my state over to S3. The following commands were used:

```shell
# First auth into AWS
# See: https://github.com/88lexd/lexd-solutions/tree/main/misc-scripts/python-aws-assume-role)
$ assume-role --c cred.yml -r roles.yml

$ export AWS_PROFILE=lexd-admin

# Migrate state over to S3
$ terraform init -migrate-state
$ terraform plan
```

## Vercel Project
First setup the zone in Cloudflare
```
terraform init
terraform apply -target cloudflare_zone.lexdsolutions
```

## Vercel Environment Variables
DNS is required to be hosted by Cloudflare. I need to migrate my existing records from AWS Route53 to Cloudflare.

 - Manually port records over to Cloudflare DNS
 - Update DNS registrar to use Cloudflare Name Servers
 - Ensure Top Level domain (lexdsolutions.com) record is **proxied** through CloudFlare

## Vercel Deploy the site
This Terraform IaC will configure the remaining settings for this zone, such as enabling cache, WAF and DDoS protections.

```
terraform plan -out tfplan.out
terraform apply tfplan.out
```

Modify local `hostfile` with the following records for testing:

