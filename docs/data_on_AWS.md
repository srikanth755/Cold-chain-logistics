# Steps to follow to put the data on AWS EC2 instance.

- Launch the EC2 instance and get the .pem key and public ip. Save it for the later purpose.

- Connect to the EC2 instance with the below commands
 - icacls C:\keys\fde-key.pem /inheritance:r
 - icacls C:\keys\fde-key.pem /grant:r "%username%:R" (Get the username using echo %username% and adjust the pem key path properly)
 - ssh -i C:\keys\fde-key.pem ec2-user@<EC2_PUBLIC_IP> (Get the publi ip from the aws console for the instance)

- Install docker on the EC2 
sudo yum update -y
sudo yum install -y docker
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker ec2-user
exit

Reconnect and check the docker version to verify it is installed

- Prepare the project locally
Mainly 2 files Dockerfile and .dockerignore (check in the codebase)

- Copy files to EC2
ssh -i C:\keys\fde-key.pem ec2-user@<EC2_PUBLIC_IP> "mkdir -p ~/app ~/data"
scp -i C:\keys\fde-key.pem -r data\raw ec2-user@<EC2_PUBLIC_IP>:~/data/
scp -i C:\keys\fde-key.pem -r src Dockerfile .dockerignore requirements.txt ec2-user@<EC2_PUBLIC_IP>:~/app/

- Build and run on EC2
ssh -i C:\keys\fde-key.pem ec2-user@<EC2_PUBLIC_IP>
cd ~/app
docker build -t logistics-analysis .

START THE SQL SERVER ON EC2 
docker network create fde-net
docker run -d --name mssql --network fde-net \
  -e ACCEPT_EULA=Y \
  -e MSSQL_SA_PASSWORD='YourStr0ng!Pass' \
  mcr.microsoft.com/mssql/server:2022-latest
docker ps

UPDATE THE .ENV FILE ON EC2
nano ~/.env
SQL_SERVER_HOST=mssql
SQL_SERVER_PORT=1433
SQL_ADMIN_USER=sa
SQL_ADMIN_PASSWORD=YourStr0ng!Pass
chmod 600 ~/.env

RUN THE INGESTION
docker run --rm -v ~/data:/app/data logistics-analysis

- verify and mount
ls ~/data/raw                      # confirm data arrived
docker images                      # confirm image built
docker run --rm -v ~/data:/app/data logistics-analysis ls /app/data/raw   # confirm mount
scp -i C:\keys\fde-key.pem -r ec2-user@<EC2_PUBLIC_IP>:~/data/output .\output


