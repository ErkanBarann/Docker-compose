
## 📚 **Project Assignment: Deploying a Flask Application with Docker, Gunicorn, and Nginx**

### **Project Description:**
In this project, you are expected to deploy a car rental application developed with Python Flask to an EC2 instance using Docker, Gunicorn, Nginx, and Terraform. Your application should include the following steps:

1. **Flask Application:** A simple car rental platform developed with Python Flask. This platform allows users to rent and view vehicles.
2. **Docker Compose:** The application should be structured with multiple services, each running independently. The services include:
   - **app**: Flask application with Gunicorn
   - **mysql**: MySQL database
   - **nginx**: Nginx server acting as a reverse proxy for the application.
3. **Nginx Configuration:** Nginx should be configured to route external requests to the Flask application. It can also be configured to support SSL certificates.

### **Project Steps:**

1. **Writing the Dockerfile:**
   - A `Dockerfile` will be created to build a Docker image for your Flask application.
   - It should install the application dependencies and launch it with Gunicorn.
   - The `Dockerfile` should include:
     - Using the Python 3.9 base image
     - Installing necessary dependencies
     - Running the Flask application using Gunicorn

   **Example Dockerfile:**
   ```dockerfile
   FROM python:3.9
   WORKDIR /app
   COPY . /app
   RUN pip install --no-cache-dir -r requirements.txt
   EXPOSE 8000
   CMD ["gunicorn", "-b", "0.0.0.0:8000", "wsgi:app"]
   ```

2. **Docker Compose File:**
   - Docker Compose will manage the `mysql`, `app` (Flask + Gunicorn), and `nginx` (Reverse Proxy) services.
   - Each service should be configured independently.

   **Example Docker Compose:**
   ```yaml
   version: '3'
   services:
     app:
       build: .
       container_name: flask_app
       environment:
         - DB_HOST=mysql
         - DB_USER=root
         - DB_PASSWORD=rootpassword
         - DB_NAME=arac_kiralama
       depends_on:
         - mysql
       networks:
         - app-network
       ports:
         - "8000:8000"

     mysql:
       image: mysql:5.7
       container_name: mysql_container
       environment:
         MYSQL_ROOT_PASSWORD: rootpassword
         MYSQL_DATABASE: arac_kiralama
       networks:
         - app-network
       ports:
         - "3306:3306"

     nginx:
       image: nginx:latest
       container_name: nginx_reverse_proxy
       volumes:
         - ./nginx.conf:/etc/nginx/nginx.conf
       ports:
         - "80:80"
       depends_on:
         - app
       networks:
         - app-network

   networks:
     app-network:
       driver: bridge
   ```

3. **Nginx Configuration:**
   - Nginx will receive incoming requests and route them to the Flask application running with Gunicorn.

   **Example nginx.conf:**
   ```nginx
   server {
       listen 80;
       server_name ${DOMAIN_NAME} www.${DOMAIN_NAME};

       location / {
           proxy_pass http://app:8000;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
       }
   }
   ```

4. **Deploy to EC2 with Terraform:**
   - Terraform will start an EC2 instance on AWS and run Docker Compose on it.

   **Example Terraform:**
   ```hcl
   provider "aws" {
     region = "us-east-1"
   }

   resource "aws_instance" "flask_app" {
     ami           = "ami-0c55b159cbfafe1f0"
     instance_type = "t2.micro"

     tags = {
       Name = "FlaskAppInstance"
     }

     user_data = <<-EOF
       #!/bin/bash
       apt update -y
       apt install -y docker.io
       apt install -y docker-compose
       cd /home/ubuntu
       git clone https://github.com/ErkanBarann/Docker-compose.git
       cd Docker-compose
       docker-compose up -d
     EOF
   }
   ```

### **Assignment Requirements:**

1. **Create Dockerfile and Docker Compose Files:**
   - A `Dockerfile` must be written to run the Flask application with Docker.
   - The `docker-compose.yml` file must configure MySQL, Nginx, and Flask services.

2. **Run the Application:**
   - The application should be started using Docker Compose.
   - Ensure Nginx is working correctly with the Flask application.

3. **Deploy with Terraform to EC2:**
   - An EC2 instance should be launched using Terraform, and Docker Compose should be executed on it.

4. **Upload Project to a Repository:**
   - All files must be uploaded to a GitHub repository.
