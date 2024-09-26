Here’s a modified version of your document with a cleaner, more straightforward format. I've adjusted the headers and reduced the bold text for clarity.

```markdown
# Troubleshooting SSH Connection Issues on EC2 Instances

This document provides a step-by-step guide for resolving SSH connection issues to EC2 instances, particularly when encountering a "Connection timed out" error. It also includes a cloud-init script to address potential firewall (UFW) misconfigurations that may block access.

## Problem Description

You may encounter an issue where you are unable to connect to your EC2 instance using SSH, despite having followed the proper setup steps. The error message typically reads:

```bash
ssh: connect to host [instance-public-IP] port 22: Operation timed out
```

This issue could arise due to misconfigurations in the firewall settings, security groups, or other networking components.

## Pre-Checks

Before troubleshooting further, ensure the following items are configured correctly:

### 1. Security Group Settings
- The EC2 instance’s Security Group should allow inbound traffic on port 22 (SSH) from your IP address.
    - **Verify**: Check the inbound rules of the security group attached to your instance. There should be a rule allowing traffic on port 22 (SSH) from your IP address or your organization’s IP range.

### 2. Instance Network Configuration
- Ensure the instance is in a public subnet and is associated with a public IP or Elastic IP.
    - **Verify**: The instance should be in a public subnet with a route to an Internet Gateway. Additionally, make sure the instance has a public IP that you are using to connect.

### 3. Correct SSH Key Pair
- Make sure you are using the correct key pair (PEM file) that corresponds to the key pair associated with the instance during its creation.
    - **Verify**: Double-check that the key file you’re using in your SSH command matches the key pair for the instance. Permissions for the key file should also be set to `chmod 400 [your-key.pem]`.

### Example SSH Command:
```bash
ssh -i "your-key.pem" ubuntu@[instance-public-IP]
```

If you're still unable to connect and facing "Connection timed out," the problem might be related to the instance’s firewall (UFW) settings.

## Resolving the Issue: Firewall (UFW) Misconfiguration

In some cases, the Uncomplicated Firewall (UFW) may block inbound traffic, including SSH, even though the security group settings are correct. If UFW is misconfigured, it may prevent access to the instance.

### Solution: Disable UFW via Cloud-Init Script

To resolve this, you can use cloud-init to stop and disable UFW by adding the following script to the instance’s user data.

### Steps to Apply the Fix:

1. **Stop the EC2 Instance**
    - Open the AWS EC2 Console and select the instance.
    - Click on **Actions > Instance State > Stop**.

2. **Add Cloud-Init Script to Disable UFW**
    - In the AWS Console, go to **Actions > Instance Settings > Edit user data**.
    - Add the following script in the User data field:

    ```yaml
    #cloud-config
    bootcmd:
      - cloud-init-per always fix_broken_ufw_1 sh -xc "/usr/sbin/service ufw stop >> /var/tmp/svc_$INSTANCE_ID 2>&1 || true"
      - cloud-init-per always fix_broken_ufw_2 sh -xc "/usr/sbin/ufw disable >> /var/tmp/ufw_$INSTANCE_ID 2>&1 || true"
    ```

3. **Start the EC2 Instance**
    - After saving the user data script, start the instance by clicking **Actions > Instance State > Start**.

4. **Verify Access**
    - Once the instance is running, attempt to SSH into the instance again:

    ```bash
    ssh -i "your-key.pem" ubuntu@[instance-public-IP]
    ```

You should now be able to connect successfully.

### Explanation of the Script

The provided Cloud-Config commands are designed to automatically configure UFW during the boot process of a cloud instance. They aim to address potential issues with UFW's state and ensure that it is disabled, preventing unwanted network traffic from reaching the instance.

### Logs

You can review the logs to confirm that the commands were executed successfully:

```bash
cat /var/tmp/svc_$INSTANCE_ID
cat /var/tmp/ufw_$INSTANCE_ID
```

## Additional Troubleshooting

If the issue persists even after disabling UFW, consider the following:

1. **Check VPC and Route Tables**: Ensure the instance's subnet is associated with a route table that has a route to an Internet Gateway.
2. **Check Network ACLs**: Ensure no Network ACLs are blocking inbound or outbound traffic on port 22.
3. **Use EC2 Serial Console**: If SSH access is still unavailable, you can use the EC2 Serial Console for further troubleshooting.

## Conclusion

This document provides a guide to resolving SSH connectivity issues related to UFW on EC2 instances. By following these instructions, you can restore access to your instance and prevent UFW from blocking connections in the future.

If further issues arise or additional help is needed, please contact your cloud support team.
```
