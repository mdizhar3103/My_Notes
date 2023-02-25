AWS Networking
-------------

|Security Group| Network Access Control List |
|--------------|--------------------|
|Instance Level|Subnet Level|
|Allow rules only| Allow and deny rules |
|Evaluates all rules before allowing traffic| Rules processed in numeric order |
|Stateful: Return traffic automatically allowed regardles of any rules | Stateless: Return traffic must be explicitly allowed by rules
|Applies to instances associated with Security Group| Applies to all instances in subnets associated with NACL |

> Note: 
> - Every time if you create an EC2 instance a new default Security group is created, we can select the existing Security group to avoid creation of new default Security group.
> - Security groups belongs to VPC 

Traffic flow to instance:
------------------------
```bash
Internet Gateway (VPC) --> Router (Route Table) --> Subnet --> NACL --> Security Group --> Instance 
```

**Elastic IP**
- Elastic IP is used because public ip keeps changing, and to avoid that we can use the Elastic IP and it wil not change unless we release it.
- Elastic IP is free per instance running (note)
- If Elastic IP is not attached to running instance you will be charged.


