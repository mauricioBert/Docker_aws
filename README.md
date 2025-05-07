# Projeto AWS com Docker

## Criação da VPC

![image](https://github.com/user-attachments/assets/042cff34-72a6-4fb5-b8ca-2f1274ff43bd)
![image](https://github.com/user-attachments/assets/8aaedf6b-e3ec-499c-a855-2e7fc0820ba7)
![image](https://github.com/user-attachments/assets/136072cd-624e-40cf-bf00-9dc2c6002d9a)

## Criar os Grupos de Segurança

### EC2
![image](https://github.com/user-attachments/assets/ffa531d0-c6cb-488e-85a2-a5b7b73a5899)

### Classic Load Balancer
![image](https://github.com/user-attachments/assets/c0c59120-6f44-47b3-af64-299649c5fd71)

### EFS
![image](https://github.com/user-attachments/assets/3f017f49-bb38-4757-b65f-07f8014d2d08)

### RDS Mysql
![image](https://github.com/user-attachments/assets/68900d36-c93b-411d-a431-5264f8fb7e4e)

### Visão geral do SG:
![image](https://github.com/user-attachments/assets/f10df0f3-bc10-4080-8d9d-174c092bab75)

## Criação do RDS
acesse a pagina do Rds e crie um novo banco de dados

![image](https://github.com/user-attachments/assets/88518594-cc2c-432f-a9c2-b7db150edcba)

![image](https://github.com/user-attachments/assets/f666c53c-4ab7-48e7-81f6-5dac88ec72d3)

Altere esses campos:
![image](https://github.com/user-attachments/assets/3af861c6-c999-4b21-b639-818127061cc6)
![image](https://github.com/user-attachments/assets/62b20e36-2547-4765-82c2-4b6cd97ee59d)
![image](https://github.com/user-attachments/assets/4f12a590-3158-4795-b72c-209b2ffead20)
![image](https://github.com/user-attachments/assets/ef3c3443-fd64-45d9-8a59-70d7d129e2d8)

após essas configurações crie o banco de dados.
OBS: Salve o endpoint e a senha do banco, pois vc irá usar futuramente!
![image](https://github.com/user-attachments/assets/accbf540-7b22-4cc5-af51-18a049bdca53)

## Criar o EFS
![image](https://github.com/user-attachments/assets/2df445dd-c6ce-4b62-af20-5fb0783d48bd)
![image](https://github.com/user-attachments/assets/586f5f94-efcb-4c0f-b39c-3026b6801df3)

Salve o ponto de Montagem pois você irá utilizar futuramente
![image](https://github.com/user-attachments/assets/d5329dae-63b6-4f4a-879e-001d0702cacf)

## Criação do Load Balancer
![image](https://github.com/user-attachments/assets/92b68d09-b610-428f-b530-64ec31349b28)
![image](https://github.com/user-attachments/assets/cc7e09d7-dfb5-4018-945d-90bc95b5dda2)
![image](https://github.com/user-attachments/assets/8a876d47-607f-4bdd-b36a-23fb1821c1ee)
![image](https://github.com/user-attachments/assets/c59fca79-c3e6-4e2e-915c-7a40ca1d1bc6)

## Desenvolvimento do Docker-Compose.yml e do User-Data
### Docker
Use esse como exemplo:
Não se esqueça de trocar o endpoint e tambem a senha do seu banco
services:
  wordpress:
    image: wordpress:latest
    container_name: wordpress
    restart: always
    ports:
      - "80:80"
    environment:
      WORDPRESS_DB_HOST: wordpress.clcoysmgg2k7.us-east-1.rds.amazonaws.com:3306
      WORDPRESS_DB_USER: admin
      WORDPRESS_DB_PASSWORD: 997956257
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - /data:/var/www/html

Após você criar o seu docker-compose use o Raw do github para ficar publico esse arquivo, o link deve se parecer com isso:
 https://raw.githubusercontent.com/mauricioBert/Docker_aws/refs/heads/main/docker-compose.yml


 ### User-Data
 #!/bin/bash

# Atualiza os pacotes do sistema
yum update -y

# Instala pacotes necessários
yum install -y ca-certificates wget amazon-efs-utils

# Instala o Docker
yum install -y docker

# Inicia e habilita o Docker
systemctl enable docker
systemctl start docker

# Nome do usuário
usermod -aG docker ec2-user

# Instala o Docker Compose manualmente
curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

# Cria diretório de montagem do EFS
mkdir -p /data

# Monta o EFS (corrigido: o nome 'efs' estava sobrando antes de /data)
mount -t efs -o tls fs-06ed311412509e68e:/ /data

# Baixa o docker-compose.yml do GitHub
wget -O /home/ec2-user/docker-compose.yml https://raw.githubusercontent.com/mauricioBert/Docker_aws/refs/heads/main/docker-compose.yml

# Ajusta permissões do arquivo para o usuário EC2
chown ec2-user:ec2-user /home/ec2-user/docker-compose.yml

# Executa o Docker Compose
cd /home/ec2-user && docker-compose up -d


## Montagem do Launch Template
![image](https://github.com/user-attachments/assets/b7ef8fc8-f0b3-40cb-a529-00d7b60d1ed4)
![image](https://github.com/user-attachments/assets/dd5be0ef-2ed3-4e14-b085-6cc1d3d713a9)
![image](https://github.com/user-attachments/assets/93924c3a-2b74-49f0-8fb0-e50d83c3e028)

## Criação do Auto Scaling Group
![image](https://github.com/user-attachments/assets/2f3cc924-02db-4f7e-a63e-b2e33b3dd8d4)
![image](https://github.com/user-attachments/assets/54228483-7e39-4534-9da5-fdaf9ac062b9)
![image](https://github.com/user-attachments/assets/9057539d-17f9-4407-8c5f-c56f9ba00a83)
![image](https://github.com/user-attachments/assets/ca0794e1-cec3-435e-a227-5ee2cbd2b9ef)
Pule a etapa 5 e 6
![image](https://github.com/user-attachments/assets/954c6b85-e94a-4a35-8efb-026ed91b5708)
