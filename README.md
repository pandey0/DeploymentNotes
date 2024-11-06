# DeploymentNotes


### STATIC REACT WEBSITE DEPLOYMENT 
---


### **a) Deployment Using Vercel**

Vercel is a platform designed to deploy and host modern web applications with minimal configuration, especially for frameworks like React.

#### **Steps to Deploy Using Vercel:**

1. **Set up a React Project with Vite**:
   - **Vite** is a build tool that bundles your React project in a way that’s optimized for modern browsers. It’s faster and more efficient than older tools like Webpack.
   - Create a React app using **Vite**:
     ```bash
     npm create vite@latest my-react-app --template react
     cd my-react-app
     npm install
     ```
   - After developing your application, build it for production:
     ```bash
     npm run build
     ```

2. **Push the Code to GitHub**:
   - Once your React project is ready and tested locally, **push the code** to a GitHub repository.
     ```bash
     git init
     git add .
     git commit -m "Initial commit"
     git remote add origin <repository-url>
     git push -u origin main
     ```

3. **Connect GitHub to Vercel**:
   - Go to [Vercel](https://vercel.com) and sign up/login.
   - Click **"New Project"**, then select **GitHub** as the Git provider.
   - Connect your GitHub account to Vercel by authorizing it. Vercel will automatically list your repositories.
   - Choose the React project repository that you want to deploy.

4. **Deploy Your Application**:
   - Once you select the repository, Vercel automatically detects the React (Vite) project and prepares the deployment. It will automatically run `npm run build` and deploy the production version.
   - Vercel will provide a **preview URL** for your app. You can view the live version before linking any custom domains.

5. **Add a Custom Domain (Optional)**:
   - If you want to use a custom domain (e.g., `www.mysite.com`), purchase one from a domain provider like **GoDaddy** or **Google Domains**.
   - In the Vercel dashboard, go to the project settings and select **Domains**.
   - Follow the instructions provided by Vercel to point your domain’s **DNS settings** to Vercel’s servers (usually adding a **CNAME** or **A record**).
   
6. **Automatic SSL Certificates**:
   - Vercel will automatically provision an **SSL certificate** for your domain using **Let’s Encrypt** once the domain is properly configured. This ensures that your application is served over **HTTPS**.
   
7. **CI/CD Pipeline**:
   - Vercel provides **Continuous Deployment (CD)**. Any time you **push new changes to your GitHub repository**, Vercel automatically rebuilds and redeploys your application. This happens seamlessly without any manual intervention.
   - This is powered by **Continuous Integration (CI)**, where the code is tested, built, and deployed automatically whenever changes are made in the GitHub repository.

#### **Drawbacks of Vercel**:
- Vercel may become expensive if your application requires a lot of resources or if you exceed the free tier’s usage limits. Heavy traffic or large projects may incur additional costs.
  
---

### **b) Deployment on Virtual Machine (VM) (AWS, GCP, DigitalOcean)**

Deploying an application on a **Virtual Machine (VM)** gives you full control over the environment, but requires more manual configuration than platforms like Vercel.

#### **Steps to Deploy on a Virtual Machine (VM)**:

1. **Create a Virtual Machine (VM)**:
   - In the cloud provider’s dashboard (AWS, GCP, or DigitalOcean), create a new **Virtual Machine** or **Instance**. For example, on AWS, you can use **EC2**.
   - Choose an **Ubuntu** image or another Linux-based system.
   - Select an instance type that suits your needs (e.g., a small instance for low traffic, or a larger instance for more demanding workloads).
   - Assign a **Public IP address** to your instance, so it can be accessed over the internet.

2. **SSH into the VM**:
   - After your VM is created, connect to it via **SSH**:
     ```bash
     ssh -i /path/to/your-key.pem ubuntu@your-vm-public-ip
     ```

3. **Install Node.js and NPM**:
   - Install **Node.js** and **npm** on the VM so that you can run your React app.
   - For **Ubuntu**, you can install Node.js using the following commands:
     ```bash
     sudo apt update
     sudo apt install nodejs npm
     ```

4. **Transfer Source Code to the VM**:
   - You can either use **Git** to clone your repository onto the VM or **upload the files** using **SFTP** or **scp**:
     ```bash
     git clone https://github.com/yourusername/your-repository.git
     ```

5. **Build and Start the Application**:
   - Navigate to the project directory on the VM and install the necessary dependencies:
     ```bash
     cd your-project-directory
     npm install
     ```
   - Build the project for production:
     ```bash
     npm run build
     ```
   - Start the application (on Ubuntu, use **pm2** or **forever** to run the app persistently):
     ```bash
     sudo npm run serve
     ```
   - Your application should now be accessible via your VM’s **public IP address**.

6. **Configure Nginx as a Reverse Proxy**:
   - For better performance and SSL support, install and configure **Nginx** as a reverse proxy:
     ```bash
     sudo apt install nginx
     ```
   - Configure Nginx to forward incoming HTTP requests to your Node.js app. Edit the Nginx config file:
     ```bash
     sudo nano /etc/nginx/sites-available/default
     ```
   - Add the following configuration:
     ```nginx
     server {
         listen 80;
         server_name your-vm-public-ip;

         location / {
             proxy_pass http://localhost:3000;  # Node.js app runs on port 3000
             proxy_http_version 1.1;
             proxy_set_header Upgrade $http_upgrade;
             proxy_set_header Connection 'upgrade';
             proxy_set_header Host $host;
             proxy_cache_bypass $http_upgrade;
         }
     }
     ```
   - Restart Nginx to apply the changes:
     ```bash
     sudo systemctl restart nginx
     ```

7. **Configure SSL with Certbot**:
   - To secure your application with HTTPS, use **Certbot** to generate an SSL certificate:
     ```bash
     sudo apt install certbot python3-certbot-nginx
     sudo certbot --nginx -d your-domain.com
     ```
   - Certbot will automatically obtain an SSL certificate and configure Nginx to serve your app over HTTPS.

#### **Drawbacks of Using a VM**:
- **Manual Setup**: You need to handle the setup, configuration, scaling, and monitoring of the VM yourself.
- **Scaling Issues**: A single VM may not be able to handle high traffic. For scaling, you would need to manually configure load balancers, auto-scaling groups, or horizontally scale across multiple VMs.

---

### **c) Deployment Using CDN and Object Storage (Ideal for Static Apps)**

When deploying a static site like a React app, using a **Content Delivery Network (CDN)** combined with **Object Storage** is a highly efficient and scalable approach.

#### **Steps to Deploy Using CDN and Object Storage**:

1. **Create a Static Build of the React App**:
   - First, run the following command to generate a production build of your React app:
     ```bash
     npm run build
     ```
   - This will generate a `build/` or `dist/` folder containing your static assets (HTML, CSS, JS).

2. **Upload Files to an Object Storage (e.g., S3, Bunny.net)**:
   - Upload the contents of the `build/` or `dist/` folder to an object storage service like **Amazon S3**, **Google Cloud Storage**, or **Bunny.net**.
   - For S3, you can use the **AWS Management Console** or the **AWS CLI** to upload the files:
     ```bash
     aws s3 cp build/ s3://your-bucket-name/ --recursive
     ```

3. **Set Up a CDN to Serve the Files**:
   - Create a **CDN distribution** to cache and serve your static files to users across multiple edge locations globally.
   - For example, on **Cloudflare**, you can point your S3 bucket to Cloudflare’s CDN by creating a **CNAME record** for your domain that points to the S3 bucket’s endpoint.
   - If using **Bunny.net**, it will automatically act as the CDN and serve your content from edge locations.

4. **Link Custom Domain**:
   - If you want to use a custom domain (e.g., `www.mysite.com`), configure the **DNS** settings at your domain registrar (GoDaddy, Google Domains, etc.) to point to the CDN or object storage.

5. **Configure HTTPS**:
   - With most CDNs (like Cloudflare), HTTPS is automatically enabled. For **S3** with CloudFront, you can use

 AWS's **SSL certificates** through **AWS Certificate Manager**.

#### **Benefits of CDN + Object Storage**:
- **Scalability**: CDN and object storage scale effortlessly, handling high traffic without needing manual intervention.
- **Global Performance**: CDN caches files at edge locations close to users, ensuring faster load times.
- **Low Maintenance**: No need to manage servers, databases, or complex configurations.
  
#### **Drawbacks**:
- **Dynamic Content**: This method is ideal for **static sites**. If you need server-side processing or dynamic content, you might need to combine this with serverless functions or a backend solution.

---

### Summary

1. **Vercel**: Best for React apps with automatic deployments and scaling. Easy to use but can get expensive with higher usage.
2. **VM Deployment (AWS, GCP, DigitalOcean)**: Offers full control over the environment, but requires manual setup and scaling management.
3. **CDN + Object Storage**: Ideal for static websites like React apps. Fast, scalable, and cost-efficient, but not suitable for dynamic applications.


Sure! Let's focus specifically on how to deploy a static React app using **Bunny.net** as the CDN and **Bunny Object Storage** for hosting the static files. I'll break it down step-by-step to give you a detailed guide.

### **D) Deployment Using Bunny.net CDN and Object Storage (for Static React Apps)**

This method is highly efficient for deploying static React apps, offering low-latency delivery via the CDN and scalability through Bunny’s object storage.

---

#### **Step-by-Step Deployment Using Bunny.net**

##### **1. Build Your React Application**
Before uploading your React app to Bunny.net's object storage, you need to prepare a production-ready build.

- In your React project, run the following command to generate a **production build**:
  ```bash
  npm run build
  ```
- This will create a `build/` or `dist/` directory containing all the static files: HTML, CSS, JavaScript, images, etc.

##### **2. Sign Up for Bunny.net**
If you haven't already, sign up for an account on **Bunny.net**:
- Go to [Bunny.net](https://bunny.net) and create an account.
  
##### **3. Create a Storage Zone in Bunny.net**
Now that you have your Bunny.net account set up, you need to create a **Storage Zone** where you'll store the static assets (files from your React build).

- In the Bunny.net dashboard, go to the **Storage** tab.
- Click on **Create Storage Zone**.
- Choose a name for your storage zone (e.g., `my-react-app`).
- Select the region closest to your target audience for optimal performance (e.g., **EU**, **US**, etc.).
- Once the zone is created, you'll get a **storage endpoint URL** (e.g., `my-react-app.bunnycdn.com`).

##### **4. Upload Files to Bunny Object Storage**
Now, you’ll upload the files from your React build (`build/` or `dist/` folder) to Bunny.net's object storage.

There are several ways to do this:
- **Using Bunny.net’s web interface**:
  - Go to the **Storage Zone** you created.
  - Use the **upload** feature to drag and drop files from your local `build/` folder into the Bunny.net storage.
  
- **Using FTP/SFTP**:
  - Bunny.net provides **FTP/SFTP access** to upload files. You'll find the FTP credentials in the **Storage Zone** settings.
  - Connect via your preferred FTP client (e.g., FileZilla) and upload the `build/` folder's contents.

- **Using Bunny’s API (Optional)**:
  - If you prefer to automate the deployment, you can use **Bunny's API** to upload files directly to the storage zone.

##### **5. Set Up a CDN with Bunny.net**
Once the files are uploaded to Bunny Object Storage, the next step is to configure the **CDN** to serve these files from edge locations closer to your users.

- After uploading, your React app will already be publicly accessible through Bunny.net's **CDN**. Bunny.net caches files at edge locations across the world for fast delivery.
  
  The URL to access your files will look like this:
  ```
  https://your-storage-zone-name.bunnycdn.com
  ```

  For example, if your storage zone is called `my-react-app`, the public URL for your index.html file might be:
  ```
  https://my-react-app.bunnycdn.com/index.html
  ```

- **Make sure to update your index.html**:
  - If you’re using a custom domain, you will want to link this URL to your domain. For now, Bunny.net provides you with the URL as a CDN-backed public endpoint.

##### **6. Link a Custom Domain to Bunny.net (Optional)**

To use your custom domain (e.g., `www.mysite.com`), you’ll need to configure DNS settings.

- In your domain registrar (e.g., GoDaddy, Google Domains), go to the DNS settings for your domain.
- Create a **CNAME** record:
  - **Host**: `www`
  - **Points to**: `your-storage-zone-name.bunnycdn.com`
- Save the changes, and within a few minutes, your domain will be pointing to the Bunny.net CDN.

##### **7. Configure SSL for HTTPS (Optional)**

Bunny.net automatically provides **SSL** certificates for your domain. However, you’ll need to ensure that HTTPS is enabled:

- If you're using **Bunny.net's SSL**, simply enable the **SSL feature** in your Bunny.net dashboard.
- If you're using a custom domain, you can also upload your own SSL certificate or let Bunny.net handle it for you by using their **free SSL** feature.

Once the certificate is applied, your app will be served securely over **HTTPS**.

##### **8. Update Your React App’s Base URL (if Needed)**

If you’ve configured a custom domain, make sure your React app is properly configured to reference assets through the correct domain.

- In your `index.html` or other references, ensure that URLs to assets (like images, JS, and CSS files) are either **relative** or correctly reference the **CDN URL** (e.g., `https://my-react-app.bunnycdn.com/`).
- Alternatively, you can use environment variables in React to define the **base URL** for assets.

Example for setting a custom public URL in React:
```javascript
// src/config.js
export const BASE_URL = 'https://my-react-app.bunnycdn.com';
```

This ensures that React references your CDN for assets when it’s built.

##### **9. Cache Control and Purging (Optional)**

Bunny.net allows you to set **cache rules** for how long files should stay cached on their edge nodes. You can configure these cache settings via the **Bunny.net dashboard**.

- You can also purge the cache when updating your app to ensure that users get the latest version of your app.

##### **10. Test Your Deployment**
Once everything is set up:
- Visit your custom domain (e.g., `www.mysite.com`), or the Bunny.net CDN URL (`your-storage-zone-name.bunnycdn.com`).
- Your React app should load instantly, served from Bunny.net’s global edge locations.
- Check that all resources are loading correctly and the app is functioning as expected.

---

### **Summary of Steps Using Bunny.net for CDN + Object Storage**

1. **Build the React App**:
   - Run `npm run build` to generate the production build.
   
2. **Sign Up and Create a Storage Zone on Bunny.net**:
   - Set up a storage zone in Bunny.net to store your static files.

3. **Upload Static Files to Bunny Object Storage**:
   - Upload the files from the `build/` folder to Bunny’s storage via the web interface, FTP, or API.

4. **Configure the CDN**:
   - Bunny.net automatically serves your static assets via its CDN. The files are cached at edge locations globally for fast access.

5. **Set Up Custom Domain (Optional)**:
   - Configure DNS settings to point your custom domain to Bunny.net’s CDN endpoint.

6. **Enable SSL (Optional)**:
   - Bunny.net provides free SSL certificates for secure HTTPS access.

7. **Test the Application**:
   - Visit your domain or Bunny.net URL to check the deployment and ensure the app is being served correctly.

---

### **Benefits of Using Bunny.net for CDN + Object Storage**
- **Global Performance**: Bunny.net’s CDN caches your app's assets at edge locations, ensuring fast load times for users around the world.
- **Scalability**: The CDN and object storage are highly scalable, making it ideal for handling large volumes of traffic without additional setup.
- **Low Maintenance**: Once deployed, you don't need to worry about server management or infrastructure.
- **Cost-effective**: Bunny.net offers affordable pricing with a pay-as-you-go model for bandwidth and storage usage.

---

### **Drawbacks**
- **Static Only**: This method works well for static sites. If your application requires server-side processing (e.g., dynamic content), you may need to integrate with other services (e.g., serverless functions, databases, etc.).
- **Setup Complexity**: While the process is simple, you need to manage custom domains, DNS, and SSL, which can be complex for beginners.

Overall, deploying static React apps with **Bunny.net** CDN and **Object Storage** provides excellent performance, scalability, and low maintenance, making it an ideal solution for static web applications.
