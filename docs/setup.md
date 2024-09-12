# Setting Up and Securing Your AWS Account

In this guide, we will cover how to create a free tier AWS account, secure the root user by enabling multi-factor authentication (MFA), and establish essential IAM users for various roles, such as emergency access, workloads, and administrators. Securing your AWS account is vital to ensuring safe and compliant operations in the cloud.

## AWS Setup Instructions

This guide provides instructions for creating and securing an AWS environment required for the lab:

### Steps:
1. **Sign up for an AWS Free Tier account**.
2. **Secure your AWS account root user**.
3. **Create an IAM user for emergency access**.
4. **Create an IAM user for workloads that cannot use IAM roles**.
5. **Install the Terraform AWS Provider**.
6. **Install Ansible**.
7. **Grant permissions required to use EC2 Instance Connect**.

After completing the steps above, proceed with the [Pivoting Lab Walkthrough Guide](./README.md) to deploy the lab.

For a breakdown of the Terraform configurations, see the [Terraform Configuration Guide](./terraform.md).

---

## Sign up for an AWS Account

If you don't have an AWS account yet, follow these steps:

1. **Open the AWS signup page**: [AWS Signup](https://portal.aws.amazon.com/billing/signup)
2. **Follow the on-screen instructions**:
   - Enter your email, password, and account name.
   - Provide your contact information.
   - Select the AWS **Free Tier** option.
3. **Verify your identity**: AWS will call your provided phone number. Enter the verification code using the phone keypad.
4. **Choose a support plan**: Select the Free Basic Support plan unless you need additional features.
5. **Complete the process**: AWS will send a confirmation email once your account is ready.

> **Note**: After signing up, an AWS root user will be created. This user has full access to your account. For security, you should only use the root user for tasks that require it and create a separate IAM user for daily administrative tasks.

For more details, visit [Getting Started with AWS](https://docs.aws.amazon.com/linux/al2023/ug/aws.html).

---

## Securing Your AWS Root User

### Why Secure the Root User?

The root user has access to all AWS services and resources, making it the most powerful user in your account. AWS strongly recommends not using the root user for everyday tasks. Instead, create IAM users or roles with appropriate permissions for your needs.

For tasks that require root user credentials, visit [Tasks that require root user credentials](https://docs.aws.amazon.com/IAM/latest/UserGuide/root-user-tasks.html).

### Steps to Secure the AWS Root User

1. **Sign in as Root**: 
   - Go to the [AWS Management Console](https://console.aws.amazon.com/).
   - Select **Root user**, enter your AWS account email, and complete the sign-in process.
   
2. **Enable Multi-Factor Authentication (MFA)**:
   - Go to **My Account** > **Security credentials**.
   - Scroll to the **Multi-Factor Authentication (MFA)** section and click **Assign MFA device**.
   - Choose **Authenticator app**, scan the QR code using a compatible app (such as Google Authenticator), and enter the two generated MFA codes.
   - Set a **Device Name**: e.g., `RootUser_MFA_GoogleAuth`.
   - Click **Add MFA**.

   > **Important**: Make a secure backup of the QR code or secret configuration key. Alternatively, enable multiple MFA devices for your root user account to avoid losing access.

   For detailed instructions, see [Enable a virtual MFA device for your root user](https://docs.aws.amazon.com/IAM/latest/UserGuide/enable-virt-mfa-for-root.html).

---

## Setting Up IAM Users

To manage AWS resources securely, you should set up IAM users for different roles. Below are instructions to create IAM users for emergency access, workloads, and administrators.

---

### Create an IAM User for Emergency Access

An emergency access user is a backup user in case you lose access to your Identity Provider (IdP). Follow these steps to create one:

1. **Open the IAM Console**: [IAM Console](https://console.aws.amazon.com/iam/)
2. In the navigation pane, choose **Users** > **Create users**.
3. **Specify user details**:
   - User name: `EmergencyAccess`
   - **Optional**: Provide AWS Management Console access.
   - **Console password**: Autogenerate password (disable password reset at next sign-in).
4. Click **Next** to set permissions.
5. **Set permissions**:
   - Choose **Add user to group** > **Create group**.
   - Group name: `EmergencyAccessGroup`
   - Attach policy: `AdministratorAccess`
6. Click **Create group** and then **Next**.
7. **Review and create** the user.
8. **Download the .csv file** containing the login credentials and store it securely.

For more details, visit [Create an IAM user for emergency access](https://docs.aws.amazon.com/IAM/latest/UserGuide/getting-started-emergency-iam-user.html).

---

### Create an IAM User for Workloads

For workloads that cannot use IAM roles (e.g., legacy systems), create an IAM user as follows:

1. **Open the IAM Console**: [IAM Console](https://console.aws.amazon.com/iam/)
2. In the navigation pane, choose **Users** > **Create users**.
3. **Specify user details**:
   - User name: `WorkloadUser`
   - **Optional**: Provide AWS Management Console access.
   - **Console password**: Autogenerate password.
4. Click **Next** to set permissions.
5. **Set permissions**:
   - Choose **Add user to group** > **Create group**.
   - Group name: `WorkloadGroup`
   - Attach policy: `AdministratorAccess`
6. Click **Create group** and then **Next**.
7. **Review and create** the user.
8. **Download the .csv file** containing the login credentials and store it securely.

For more details, visit [Create an IAM user for workloads](https://docs.aws.amazon.com/IAM/latest/UserGuide/getting-started-workloads.html).

---

### Enable IAM Identity Center and Create an Admin User

#### Step 1: Enable IAM Identity Center

1. **Open the IAM Identity Center Console**: [IAM Identity Center Console](https://console.aws.amazon.com/singlesignon)
2. Under **Enable IAM Identity Center**, choose **Enable with AWS Organizations**.

   > For more details, see [Enabling IAM Identity Center](https://docs.aws.amazon.com/singlesignon/latest/userguide/get-set-up-for-idc.html).

---

#### Step 2: Grant Administrative Access to a User

1. **Create a Permission Set**:
   - In IAM Identity Center, go to the **Permission sets** section.
   - Click **Create permission set** > **Create custom permission set**.
   - Name it `AdminAccess` and attach the `AdministratorAccess` policy.

2. **Create an Admin User**:
   - In IAM Identity Center, go to the **Users** section.
   - Click **Add user**.
   - Enter user details for the admin (e.g., `terlbrown`).

3. **Create a Group for Admins**:
   - In IAM Identity Center, go to the **Groups** section.
   - Click **Create group** and name it `Admins`.

4. **Assign the Admin User to the Group**:
   - In IAM Identity Center, go to the **Users** section.
   - Select the admin user (e.g., `terlbrown`) and click **Add to group**.
   - Assign the user to the `Admins` group.

5. **Assign Permissions to the Admin Group**:
   - In IAM Identity Center, go to the **AWS accounts** section.
   - Select the AWS account.
   - Click **Assign users or groups** and assign the `Admins` group to the `AdminAccess` permission set.

---

### Enable MFA for IAM Identity Center Users

1. **Open the IAM Identity Center Console**: [IAM Identity Center Console](https://console.aws.amazon.com/singlesignon)
2. In the left navigation pane, choose **Users**.
3. Select a user, go to the **MFA devices** tab, and choose **Register MFA device**.
4. Follow the instructions to register the device.

For more information, visit [Register an MFA device for users](https://docs.aws.amazon.com/singlesignon/latest/userguide/how-to-register-device.html).

---

## Best Practices for IAM Users

- Use IAM roles or federated identities when possible instead of IAM users with long-term credentials.
- Update access keys regularly if you use programmatic access with IAM users. For more details, visit [Update IAM user access keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/Using_RotateAccessKey.html).
- Enable MFA for all IAM users.

For more security best practices, visit [Security best practices in IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html).

