# AWS setup

**To start instance:**

`terraform apply -var 'public_key_path=~/.ssh/id_rsa.pub' -var 'aws_access_key=YOUR_ACCESS_KEY' -var 'aws_secret_key=YOUR_SECRET_KEY' -var 'aws_instance_type=t2.micro'`

**To destroy resources:**

`terraform destroy -var 'public_key_path=~/.ssh/id_rsa.pub' -var 'aws_access_key=YOUR_ACCESS_KEY' -var 'aws_secret_key=YOUR_SECRET_KEY' -var 'aws_instance_type=t2.micro'`

**Access this new ubuntu instace:**
`ssh ubuntu@0.0.0.0`
Replace 0.0.0.0 with you aws instance's public IP.
