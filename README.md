How to use an existing AWS VPC
===
This requires an existing VPC with a predefined subnet and routing table including the IWG (Internet Gateway)

### Step 1: Create the VPC
Update the following fields in the vpc/main.tf.

```terraform
resource "aws_vpc" "default" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = "true"
  tags = { owner = "john.miller@d2iq.com" }
}
```

| Resource Type | Resource Name | Field | Default Value |
| --- | --- | --- | --- |
| aws_vpc | default | cidr_block | 10.0.0.0/16 |
| aws_vpc | default | tags | { owner = john.miller@d2iq.com } |
| aws_subnet | public | cidr_block | 10.0.0.0/22 |

```bash
cd vpc
terraform init
terraform plan -out vpc.tfplan
terraform apply "vpc.tfplan"
```
The terraform apply will output the vpc_id which is needed in step two.

### Step 2: Create the DC/OS cluster
Update the following variable fields as needed.

| Variable Name | Default Value |
| --- | --- |
| cidr_block | 10.0.0.0/16 |
| tags | { owner = john.miller@d2iq.com } |
| ssh_public_key | ~/.ssh/id_mesosphere.pub |

```bash
cd dcos
terraform init
terraform plan -var "admin_ips=[\"0.0.0.0/0\"]" -var ssh_public_key="~/.ssh/id_mesosphere.pub" -var vpc_id="vpc-03f137ac52c76eed2" -var tags="john.miller@d2iq.com" -out=dcos.tfplan
terraform apply dcos.tfplan
```

### Step 3: Remove DC/OS cluster
```bash
cd dcos
terraform destroy -auto-approve -var "admin_ips=[\"0.0.0.0/0\"]" -var ssh_public_key="~/.ssh/id_mesosphere.pub" -var vpc_id="vpc-03f137ac52c76eed2" -var tags="{ owner = john.miller@d2iq.com }" 
```

### Step 4: Removing the VPC
```bash
cd vpc
terraform destroy -auto-approve
```