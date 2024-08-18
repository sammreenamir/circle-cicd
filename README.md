CI/CD Pipeline Setup with CircleCI for 
Deploying a Static HTML Page
Introduction
This guide walks you through setting up a Continuous Integration and Continuous Deployment 
(CI/CD) pipeline using CircleCI to automatically deploy an HTML page to an Nginx server. This 
setup is ideal for beginners and includes detailed steps on configuring CircleCI, handling SSH keys,
and verifying the deployment.
Prerequisites
1. CircleCI Account: Sign up for a free CircleCI account at CircleCI.
2. GitHub Repository: Have a GitHub repository where you will store your project files.
3. Nginx Server: Have a server with Nginx installed to which you will deploy your HTML 
page.
Step 1: Prepare Your Project
1. Create Your HTML File:
• Create an index.html file with your desired content.
• Ensure it is ready for deployment.
2. Set Up Your GitHub Repository:
• Create a new repository on GitHub (e.g., circleci-html-project).
• Add your index.html file to this repository.
Step 2: Create a CircleCI Configuration
1. Create .circleci Directory:
• On your local machine, navigate to your project directory.
• Create a directory named .circleci:
mkdir .circleci
2. Create config.yml File:
• Inside the .circleci directory, create a file named config.yml:
nano .circleci/config.yml
Add the following content to config.yml:
version: 2.1
jobs:
 build:
 docker:
 - image: circleci/node:latest
 steps:
 - checkout
 - run:
 name: Build Project
 command: echo "Building the project..."
 test:
 docker:
 - image: circleci/node:latest
 steps:
 - checkout
 - run:
 name: Run Tests
 command: echo "Running tests..."
 deploy:
 docker:
 - image: circleci/python:3.8 # Docker image with SSH 
capabilities
 steps:
 - checkout
 - run:
 name: Install SSH Client
 command: sudo apt-get update && sudo apt-get install -y 
openssh-client
 - run:
 name: Deploy index.html to Nginx
 command: |
 # Create SSH key for authentication
 mkdir -p ~/.ssh
 echo "$SSH_PRIVATE_KEY" > ~/.ssh/id_rsa
 chmod 600 ~/.ssh/id_rsa
 ssh-keyscan -H your_server_ip >> ~/.ssh/known_hosts
 # Use SCP to copy the file to the remote server
 scp -i ~/.ssh/id_rsa -o StrictHostKeyChecking=no 
index.html user@your_server_ip:/var/www/html/index.html
 # Optionally, restart Nginx on the server
 ssh -i ~/.ssh/id_rsa -o StrictHostKeyChecking=no 
user@your_server_ip "sudo systemctl restart nginx"
workflows:
 version: 2
 build_test_deploy:
 jobs:
 - build
 - test:
 requires:
 - build
 - deploy:
 requires:
 - test
3. Push Configuration to GitHub:
• Commit and push the .circleci/config.yml file to your GitHub repository:
git add .circleci/config.yml
git commit -m "Add CircleCI configuration"
git push origin main
Step 3: Configure CircleCI
1. Link Your GitHub Repository:
• Log in to CircleCI and go to the dashboard.
• Click “Add Projects” and select your GitHub repository.
• Click “Set Up Project” to link CircleCI with your repository.
2. Add SSH Key to CircleCI:
• Go to your CircleCI project settings.
• Under "Project Settings" > "Environment Variables," add SSH_PRIVATE_KEY with
your private SSH key. This key is used to authenticate with your remote server.
Step 4: Verify Deployment
1. Trigger a Build:
• CircleCI will automatically trigger a build when you push changes to your repository.
• Check the CircleCI dashboard to see the status of your build and deployment.
2. Verify the Deployment:
• Open your browser and navigate to http://localhost (if testing locally) or 
http://your_server_ip (if testing on a remote server).
• You should see the content of your index.html page.
Troubleshooting
1. Check CircleCI Logs:
• If the deployment fails, review the logs in CircleCI to identify issues.
2. Verify SSH Key Setup:
• Ensure that your SSH key is correctly added to CircleCI and has the necessary 
permissions on your server.
3. Check File Paths and Permissions:
• Confirm that the file paths and permissions on the server are correct.
Conclusion
You’ve now set up a CI/CD pipeline with CircleCI to automatically deploy an HTML page to an 
Nginx server. This setup allows you to automate the deployment process, making it easier to 
manage updates and changes to your website
