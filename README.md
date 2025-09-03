# Static Website Hosting with S3 Backup on AWS

This project demonstrates how to host a static website on an AWS EC2 instance (Ubuntu) using Nginx, with regular backups to an S3 bucket. You can restore your website from S3 at any time.

## Prerequisites

- AWS account with permission to create EC2 instances, S3 buckets, and IAM roles/policies.
- Basic knowledge of Linux (Ubuntu).
- AWS CLI basics.

## 1. Setting up the EC2 Instance

1. Launch an Ubuntu EC2 instance.
2. SSH into the instance using its public IP:
   ```bash
   ssh ubuntu@<EC2_Public_IP>
   ```
3. Install Nginx and set up your website:
   ```bash
   sudo apt update && sudo apt install nginx -y      # Install Nginx
   sudo systemctl start nginx                        # Start Nginx server
   sudo rm -rf /var/www/html/*                       # Remove default Nginx index.html
   sudo cp -r /home/ubuntu/website/* /var/www/html/  # Copy your website files
   sudo systemctl restart nginx                      # Restart Nginx to apply changes
   ```

## 2. S3 Bucket Setup for Backup

1. Create an S3 bucket named `tj330-static-website-backup` (you can replace this bucket name with your bucket name).
2. Create an IAM policy with the following permissions for your bucket:

   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Action": [
           "s3:PutObject",
           "s3:GetObject",
           "s3:ListBucket"
         ],
         "Resource": [
           "arn:aws:s3:::tj330-static-website-backup",
           "arn:aws:s3:::tj330-static-website-backup/*"
         ]
       }
     ]
   }
   ```

3. Create an IAM role (e.g., `EC2S3BackupRole`) and attach the above policy. Assign this role to your EC2 instance.

## 3. Backup Website Files to S3

1. Install AWS CLI in the EC2 instance:
   ```bash
   sudo snap install aws-cli --classic
   ```
2. Upload website files to S3:
   ```bash
   aws s3 cp /home/ubuntu/website/ s3://tj330-static-website-backup/ --recursive
   ```
   ![Image](https://github.com/tj330/Static_Website_AWS/blob/main/images/s3-bucket.png?raw=true)
3. Restore files from S3:
   ```bash
   aws s3 sync s3://tj330-static-website-backup/ /var/www/html/ --delete
   sudo systemctl restart nginx
   ```

## 4. Accessing the Website

Once Nginx is running and your website files are in place, you can access your static website by entering your EC2 instance's public IP address in your browser:

```text
http://<EC2_Public_IP>
```
![Image](https://github.com/tj330/Static_Website_AWS/blob/main/images/website.png?raw=true)

**Note:**  
- Make sure your EC2 instance's security group allows inbound traffic on port 80 (HTTP).
- Replace `<EC2_Public_IP>` with the actual public IP of your EC2 instance.

## Troubleshooting

- **Permission denied**: Ensure your EC2 instance has the correct IAM role.
- **Nginx not serving files**: Check file permissions and Nginx status (`systemctl status nginx`).

## References

- [AWS EC2 Documentation](https://docs.aws.amazon.com/ec2/)
- [AWS S3 Documentation](https://docs.aws.amazon.com/s3/)
- [Nginx Documentation](https://nginx.org/en/docs/)

---

## Summary
By following these instructions, users can easily deploy a static site, set up secure backups using AWS IAM roles and policies, and quickly recover or redeploy the site in case of failure. The documentation is suitable for AWS beginners and intermediate users, emphasizing practical commands, security best practices, and troubleshooting tips to ensure a smooth deployment and backup workflow.
