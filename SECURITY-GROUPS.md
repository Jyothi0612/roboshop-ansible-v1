# RoboShop security groups

`security-groups.yaml` creates only the 11 service groups using
the [RoboShop README](https://github.com/daws-92s/roboshop-documentation#security-groups).
`roboshop-common` must already exist; this playbook does not create or modify it.
Keep shared SSH access configured there. Attach both the common group and the
appropriate service group to every service instance, as in `roboshop.yaml`.

Launch your `roboshop-shell` controller with the existing common group first,
then run this playbook before provisioning the service instances.

Install prerequisites in the Python environment used by Ansible:

```sh
ansible-galaxy collection install amazon.aws
python3 -m pip install boto3 botocore
```

Use configured AWS credentials or an instance role with permission to manage
security groups. Run:

```sh
ansible-playbook -i inventory.ini security-groups.yaml \
  -e 'vpc_id=vpc-REPLACE_ME'
```

Replace the placeholder with the VPC containing `roboshop-common` and your EC2 instances; for
the current EC2 playbook, which does not specify a subnet, use the default VPC
in `us-east-1`. For a custom VPC, set `vpc_subnet_id` in the EC2 creation task to
a subnet in that VPC.

The first loop creates all service groups without removing existing rules.
The second loop applies the full rule list, so group references already exist.
`group_name` identifies the source group allowed to connect to the destination
group on the specified port. Dispatch has no service inbound rules; its SSH
access comes from the common group. Outbound IPv4 traffic is allowed for package
downloads and service connections.

The final pass uses `purge_rules: true`: existing inbound rules not listed here
are removed. Ensure the common group is attached to existing instances before
running, since service-specific SSH rules will be removed.

RabbitMQ management access is optional and disabled by default. To allow TCP
15672 from your management client IP, append:

```sh
-e 'rabbitmq_management_enabled=true ssh_cidr=YOUR_PUBLIC_IP/32'
```

This only opens the port; the RabbitMQ management plugin must also be enabled.

The security group playbook always creates all 11 service groups. The EC2
playbook's `instances` variable separately controls which servers are created.
