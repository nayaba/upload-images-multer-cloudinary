# Cloudinary Sign Up

You can get the **CLOUDINARY_CLOUD_NAME**, **CLOUDINARY_API_KEY**, and **CLOUDINARY_API_SECRET** by following these steps:

---

### **1. Sign Up or Log In to Cloudinary**
- Go to [Cloudinary's website](https://cloudinary.com/).
- If you don’t already have an account, **sign up for free**.
- If you already have an account, log in.

---

### **2. Access Your Cloudinary Dashboard**
- After logging in, you'll be redirected to your **Cloudinary Dashboard**.
- In the dashboard, you’ll see the **Account Details** section on the home page. This contains:
  - **Cloud Name**
  - **API Key**
  - **API Secret**

---

### **3. Locate Your Credentials**
Here’s where you find each:
- **CLOUDINARY_CLOUD_NAME**: This is the **Cloud Name** visible in your dashboard.
- **CLOUDINARY_API_KEY**: This is the **API Key** listed in the same section.
- **CLOUDINARY_API_SECRET**: This is the **API Secret**. Click the "Reveal" button if it’s hidden.

---

### **4. Copy and Add Keys to `.env`**
- Copy each key from the dashboard.
- Add them to your `.env` file like this:
  ```plaintext
  CLOUDINARY_CLOUD_NAME=your_cloud_name
  CLOUDINARY_API_KEY=your_api_key
  CLOUDINARY_API_SECRET=your_api_secret
  ```

---

Once you’ve added these credentials to your `.env` file, restart your server to apply the environment variables.
