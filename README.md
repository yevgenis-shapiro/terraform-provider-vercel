# Vercel
Vercel provides the developer tools and cloud infrastructure to build, scale, and secure a faster, more personalized web

![gitlab-vercel-logos 531a59e bd202d7cc72359805f60aa231b7dfba9](https://github.com/user-attachments/assets/c3308609-abab-4651-96c6-9f2d56374ad9)




## Configuration
### Cloudflare
The Cloudflare API key is used to authenticate into my account. Local environmental variable is used:
```
echo 'export CLOUDFLARE_API_KEY="123"' >> ~/.bashrc.alex
echo 'export CLOUDFLARE_EMAIL="myemail@domain.com"' >> ~/.bashrc.alex
```

### Terraform Backend
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

## Prerequisites
First setup the zone in Cloudflare
```
terraform init
terraform apply -target cloudflare_zone.lexdsolutions
```

DNS is required to be hosted by Cloudflare. I need to migrate my existing records from AWS Route53 to Cloudflare.

 - Manually port records over to Cloudflare DNS
 - Update DNS registrar to use Cloudflare Name Servers
 - Ensure Top Level domain (lexdsolutions.com) record is **proxied** through CloudFlare

## Deploy Remaining CloudFlare Configuration
This Terraform IaC will configure the remaining settings for this zone, such as enabling cache, WAF and DDoS protections.

```
terraform plan -out tfplan.out
terraform apply tfplan.out
```

### Local Testing
Due to the lag in DNS replication on the internet, to test the Cloudflare protections, I must modify my local `hostfile` to point to the CloudFlare's edge IPs for my DNS.

First get the IP by quering CloudFlare Name Server
```
alex@LEXD-PC:~$ nslookup
> server bob.ns.cloudflare.com  <------- Set to query Cloudflare
Default server: bob.ns.cloudflare.com
Address: 173.245.59.104#53
Default server: bob.ns.cloudflare.com
Address: 172.64.33.104#53
Default server: bob.ns.cloudflare.com
Address: 108.162.193.104#53
Default server: bob.ns.cloudflare.com
Address: 2606:4700:58::adf5:3b68#53
Default server: bob.ns.cloudflare.com
Address: 2803:f800:50::6ca2:c168#53
Default server: bob.ns.cloudflare.com
Address: 2a06:98c1:50::ac40:2168#53
> lexdsolutions.com  <------- query my record
Server:         bob.ns.cloudflare.com
Address:        173.245.59.104#53

Name:   lexdsolutions.com
Address: 104.21.51.90  <------- Take note of this record (can be any from this response)
Name:   lexdsolutions.com
Address: 172.67.177.252
Name:   lexdsolutions.com
Address: 2606:4700:3034::ac43:b1fc
Name:   lexdsolutions.com
Address: 2606:4700:3037::6815:335a
```

Modify local `hostfile` with the following records for testing:
```
104.21.51.90 lexdsolutions.com
104.21.51.90 www.lexdsolutions.com
```

## Cloudflare Tunnel
Background: Cloudflare tunnel allows me to selfhost my blog and not on AWS. Due to CGNAT limitation, I do not have a dedicated public IP available to use.

Terraform is used to setup the tunnel and once it is setup:
 - Manually log onto Cloudflare dashboard
 - Look for the tunnel
 - Inside the configuration, it provides the command to run `cloudflared` as a container, for example:
    ```
    $ docker run -d --restart unless-stopped cloudflare/cloudflared:latest tunnel --no-autoupdate run --token abc123
    ```

<!-- BEGIN_TF_DOCS -->
## Requirements

| Name | Version |
|------|---------|
| <a name="requirement_cloudflare"></a> [cloudflare](#requirement\_cloudflare) | ~> 4.25.0 |
| <a name="requirement_random"></a> [random](#requirement\_random) | 3.6.0 |

## Providers

| Name | Version |
|------|---------|
| <a name="provider_cloudflare"></a> [cloudflare](#provider\_cloudflare) | ~> 4.25.0 |
| <a name="provider_random"></a> [random](#provider\_random) | 3.6.0 |