### 🧍‍♂️ **IAM User**

- A **permanent identity** in your AWS account.
    
- Has **long-term credentials** (username, password, access keys).
    
- Used for **individuals or applications** that consistently need access.
    

---

### 🎭 **IAM Role**

- A **temporary identity** with **no credentials of its own**.
    
- Can be **assumed by AWS services, users, or other accounts**.
    
- Grants **temporary security credentials** (rotated automatically).
    
- Used for:
    
    - EC2/Lambda access to AWS services (e.g., S3, DynamoDB)
        
    - Cross-account access
        
    - Federated users
        

✅ The **role itself is permanent**, but **credentials it provides are temporary**.

---

### 🔧 **Terraform Example**

You saw how to:

- Create an **IAM role for EC2** with S3 read access.
    
- Attach a **trust policy** to allow EC2 to assume the role.
    
- Bind a **managed policy** for S3 access.
    
- Create an **instance profile** to attach the role to an EC2 instance.
    

---

### 🗂️ Key Concepts

|Concept|Description|
|---|---|
|**IAM Role**|Permanent object in AWS granting temporary permissions|
|**Assume Role**|Action by which a user/service takes on a role’s permissions|
|**Trust Policy**|Defines **who can assume** the role|
|**Instance Profile**|Binds a role to an EC2 instance|

---

Let me know if you'd like a diagram, follow-up with Lambda, or want to build this into a module!