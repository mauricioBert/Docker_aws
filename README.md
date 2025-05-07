# Projeto AWS com Docker
Este projeto demonstra como configurar uma infraestrutura escalável na AWS utilizando Docker, Auto Scaling Group, Load Balancer, EFS e RDS para hospedar uma aplicação WordPress.

---
🔧 Próximos passos:
- Criação da VPC
- Configuração do RDS
- Criação do EFS
- Setup do Launch Template
- Auto Scaling Group
- Load Balancer
- Deploy do WordPress com Docker Compose

---

## 1. Criação da VPC (Virtual Private Cloud)

Para iniciar a estrutura do ambiente, criamos uma VPC personalizada. Siga os passos abaixo:

1. Acesse o console da AWS e vá até a seção **VPC**.
2. Clique em **Criar VPC**.
3. Selecione **VPC com sub-redes públicas e privadas**.
4. Preencha os campos com as seguintes configurações:
   - **Nome da VPC**: por exemplo, `vpc-projeto-docker`
   - **Bloco CIDR**: `10.0.0.0/16`
   - **Sub-redes públicas e privadas**: distribuídas em múltiplas Zonas de Disponibilidade (AZs)
   - **NAT Gateway**: ativado para acesso externo a partir de sub-redes privadas
   - **DNS Hostnames e DNS Resolution**: ativados

![image](https://github.com/user-attachments/assets/8aaedf6b-e3ec-499c-a855-2e7fc0820ba7)
![image](https://github.com/user-attachments/assets/136072cd-624e-40cf-bf00-9dc2c6002d9a)
A estrutura final deverá conter:

- ✅ 1 VPC
- ✅ 2 sub-redes públicas
- ✅ 2 sub-redes privadas
- ✅ 1 NAT Gateway
- ✅ 1 Internet Gateway
- ✅ Tabelas de roteamento configuradas para internet e NAT

> 
> Exemplo:
![image](https://github.com/user-attachments/assets/042cff34-72a6-4fb5-b8ca-2f1274ff43bd)


## 2. Criação dos Grupos de Segurança (Security Groups)

Acesse: **EC2 → Grupos de Segurança → Criar grupo de segurança**

Certifique-se de:
- Nomear cada grupo de segurança conforme o serviço
- Informar uma descrição clara
- Selecionar a **VPC** criada anteriormente

---

### 🛡️ Grupos de Segurança Criados

| Serviço               | Nome     | Regras de Entrada                                                                 | Regras de Saída                                                      |
|-----------------------|----------|------------------------------------------------------------------------------------|----------------------------------------------------------------------|
| EC2                   | SgEc2   | - HTTP - TCP - 80 - (SgClb) | - MYSQL/Aurora - TCP - 3306 - (SgRds) <br>- NFS - TCP - 2049 - (SgEfs) <br>- HTTP - TCP - 80 - (SgClb) <br>- All Traffic - All - All - 0.0.0.0/0                           |
| CLB |SgClb   | - HTTP - TCP- 80- 0.0.0.0/0 - All Traffic                                              | - HTTP - TCP - 80 - (SgEc2)                                 |
| EFS                   | SgEfs   | - NFS - TCP - 2049 - (SgEc2)                                               | - NFS - TCP - 2049 - (SgEc2)                                 |
| RDS            | SgRds   | - MYSQL/Aurora - TCP - 3306 - (SgEc2)                                      | - MYSQL/Aurora - TCP - 3306 - (SgEc2)                      |

---

#### EC2  
![SG EC2](https://github.com/user-attachments/assets/ffa531d0-c6cb-488e-85a2-a5b7b73a5899)

#### Classic Load Balancer  
![SG CLB](https://github.com/user-attachments/assets/c0c59120-6f44-47b3-af64-299649c5fd71)

#### EFS  
![SG EFS](https://github.com/user-attachments/assets/3f017f49-bb38-4757-b65f-07f8014d2d08)

#### RDS MySQL  
![SG RDS](https://github.com/user-attachments/assets/68900d36-c93b-411d-a431-5264f8fb7e4e)

#### Visão Geral  
![Visão Geral SG](https://github.com/user-attachments/assets/f10df0f3-bc10-4080-8d9d-174c092bab75)

---

🔧 Próximo passo: **Criação do banco de dados RDS MySQL**


## 3. Criação do Banco de Dados RDS MySQL

🔧 Acesse **Aurora and RDS → Criar banco de dados**

### ⚙️ Configurações Iniciais

1. **Selecione o mecanismo do banco de dados**:  
   - ✅ **MySQL**

   ![MySQL Mechanism](https://github.com/user-attachments/assets/88518594-cc2c-432f-a9c2-b7db150edcba)

2. **Escolha o modelo de uso gratuito**:

   ![Free Tier](https://github.com/user-attachments/assets/f666c53c-4ab7-48e7-81f6-5dac88ec72d3)

---

### 🛠️ Configurações do banco

Altere os seguintes campos:

- Nome do banco
- Nome de usuário
- Senha
- Substitua o grupo de segurança pelo **SG-RDS** criado anteriormente

![Configuração 1](https://github.com/user-attachments/assets/3af861c6-c999-4b21-b639-818127061cc6)
![Configuração 2](https://github.com/user-attachments/assets/62b20e36-2547-4765-82c2-4b6cd97ee59d)
![Configuração 3](https://github.com/user-attachments/assets/4f12a590-3158-4795-b72c-209b2ffead20)
![Configuração 4](https://github.com/user-attachments/assets/ef3c3443-fd64-45d9-8a59-70d7d129e2d8)

---

### ✅ Finalizando

Após revisar as configurações, clique em **Criar banco de dados**.

📌 **Importante**:  
Anote o **endpoint** gerado e a **senha do banco**. Você irá utilizá-los no arquivo `docker-compose.yml`.

![Endpoint Banco](https://github.com/user-attachments/assets/accbf540-7b22-4cc5-af51-18a049bdca53)

---

🔜 Próximo passo: **Criação do sistema de arquivos EFS**

## 4. Criação do EFS (Elastic File System)

🔧 Acesse **Elastic File System → Criar sistema de arquivos**

---

### 🛠️ Etapas de configuração

1. Crie o EFS, escolha um nome e selecionando a VPC correta do projeto:

   ![image](https://github.com/user-attachments/assets/62664fd8-7768-4b28-bf41-a9613758a1a1)

   

---

### 🌐 Configuração de Rede

2. Com o EFS criado, acesse o sistema de arquivos → **Aba "Rede" → Gerenciar**  
   - Selecione a VPC do projeto  
   - Crie **2 destinos de montagem**, atribuindo-os às **sub-redes privadas** da sua VPC  
   - Associe também o **Grupo de Segurança SG-EFS** criado anteriormente

![EFS Detalhes](https://github.com/user-attachments/assets/586f5f94-efcb-4c0f-b39c-3026b6801df3)
---

### 📌 Salvando o ponto de montagem

3. Retorne à página do EFS e clique em **Anexar**  
   - Copie o comando de montagem gerado pelo assistente de montagem EFS  
   - Ele será usado na configuração do container com Docker

   ![Montagem EFS](https://github.com/user-attachments/assets/d5329dae-63b6-4f4a-879e-001d0702cacf)

---

💡 Exemplo de comando de montagem:

```bash
sudo mount -t efs -o tls fs-0492b12a70426e2c0:/ efs
```
## 5. Criação do Load Balancer (CLB)

🔧 Acesse **EC2 → Load Balancers → Criar Load Balancer**

---

### 🔽 Etapas para criação do Classic Load Balancer

1. Na tela de seleção de tipo de Load Balancer, escolha **"Clássico"**:

   ![Selecionar CLB](https://github.com/user-attachments/assets/92b68d09-b610-428f-b530-64ec31349b28)

2. Configure:
   - **Nome** do Load Balancer
   - **Esquema voltado para internet**
   - **VPC do projeto**
   - **Sub-redes públicas** da VPC

   ![Configuração CLB](https://github.com/user-attachments/assets/cc7e09d7-dfb5-4018-945d-90bc95b5dda2)

3. Selecione o **Security Group SG-CLB** criado anteriormente:

   ![SG CLB](https://github.com/user-attachments/assets/8a876d47-607f-4bdd-b36a-23fb1821c1ee)

4.  **Resultado: **:
![image](https://github.com/user-attachments/assets/c59fca79-c3e6-4e2e-915c-7a40ca1d1bc6)


## 6. Desenvolvimento do `docker-compose.yml` e do Script `User-Data`

---

### 🐳 Docker Compose

Utilize o exemplo abaixo como base para configurar o ambiente do WordPress. Lembre-se de **substituir** o `WORDPRESS_DB_HOST` e o `WORDPRESS_DB_PASSWORD` com os dados do seu banco RDS MySQL.

```yaml
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
```
```yaml
#!/bin/bash

echo "Atualizando pacotes do sistema..."
yum update -y

echo "Instalando pacotes necessários..."
yum install -y ca-certificates wget amazon-efs-utils

echo "Instalando o Docker..."
yum install -y docker

echo "Iniciando e habilitando o serviço Docker..."
systemctl enable docker
systemctl start docker

echo "Corrigindo o nome do usuário para o grupo Docker..."
usermod -aG docker ec2-user

echo "Instalando o Docker Compose..."
curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

echo "Criando diretório de montagem do EFS..."
mkdir -p /data

echo "Montando o sistema de arquivos EFS..."
mount -t efs -o tls fs-0032019253f7836e3:/ /data

echo "Baixando o arquivo docker-compose.yml do GitHub..."
wget -O /home/ec2-user/docker-compose.yml https://raw.githubusercontent.com/mauricioBert/Docker_aws/refs/heads/main/docker-compose.yml

echo "Ajustando permissões do arquivo docker-compose.yml..."
chown ec2-user:ec2-user /home/ec2-user/docker-compose.yml

echo "Iniciando os containers com Docker Compose..."
cd /home/ec2-user && docker-compose up -d

echo "Configuração concluída com sucesso!"
```
```yaml
# =======================
# Lembrete:
# =======================
# ⚠️ Não se esqueça de fazer as seguintes alterações:
# 
# 1. Troque o link raw do docker-compose.yml para o link correto do seu repositório.
# 2. Verifique e altere o ponto de montagem do EFS (caso necessário).
#
# Exemplo:
#  - raw link: https://raw.githubusercontent.com/seu-usuario/repositorio/branch/docker-compose.yml
#  - ponto de montagem: fs-xxxxxxxx:/ (verifique a ID correta do seu EFS)
#
# =======================
```
## Montagem do Launch Template 🚀

🔧 Acesse **EC2 → Modelos de Execução → Criar modelo de execução**

---

### 🔽 Passos para configurar o Launch Template

1. Defina um **nome** e uma **descrição** para o modelo e marque a opção **‘Orientação sobre o Auto Scaling’**.

2. Selecione a **imagem do Amazon Linux**.

3. Escolha o tipo de instância **t2.micro**.

![image](https://github.com/user-attachments/assets/46c03042-4c74-4658-b2d6-1eb70deda9d5)
4. Em **configurações de rede**, não selecione nenhuma sub-rede e use o **Security Group das instâncias** (SG-EC2).

![image](https://github.com/user-attachments/assets/dd5be0ef-2ed3-4e14-b085-6cc1d3d713a9)
5. Caso necessário, adicione as **tags de recurso** na seção específica.

6. Na parte de **Detalhes Avançados**, insira o script de **user data** no campo correspondente.

![image](https://github.com/user-attachments/assets/93924c3a-2b74-49f0-8fb0-e50d83c3e028)
7. Clique em **Criar modelo** para finalizar.


## Criação do Auto Scaling Group



🔧 Acesse **EC2 → Grupos de Auto Scaling → Criar Grupo do Auto Scaling**

---

### 🔽 Passos para criar o Auto Scaling Group

1. Dê um **nome** para o ASG e selecione o **Modelo de Execução (Launch Template)** que foi criado anteriormente, na versão **latest**.
![image](https://github.com/user-attachments/assets/2f3cc924-02db-4f7e-a63e-b2e33b3dd8d4)
2. Escolha a **VPC** do projeto e as **2 sub-redes privadas**. Mantenha a opção **‘Melhor esforço equilibrado’** selecionada.
![image](https://github.com/user-attachments/assets/54228483-7e39-4534-9da5-fdaf9ac062b9)
3. Anexe o **Load Balancer Clássico** criado para o projeto.

4. Ative a opção de **verificações de integridade do ELB**.
![image](https://github.com/user-attachments/assets/9057539d-17f9-4407-8c5f-c56f9ba00a83)

5. Defina a **capacidade desejada** de instâncias e os limites de **capacidade mínima e máxima** (para testes, sugiro usar 2, 2 e 4, respectivamente).
![image](https://github.com/user-attachments/assets/ca0794e1-cec3-435e-a227-5ee2cbd2b9ef)
6. (Opcional) Ative a **coleta de métricas de grupo** no **CloudWatch** .

7. (Opcional) Adicione uma **etiqueta** para identificar as instâncias criadas pelo ASG.
![image](https://github.com/user-attachments/assets/954c6b85-e94a-4a35-8efb-026ed91b5708)
8. Revise as configurações e clique em **Criar Auto Scaling Group**.

---

Agora, verifique se as instâncias foram criadas em **EC2 → Instâncias**. Em seguida, tente acessar a página do WordPress através do **DNS do Load Balancer**.

💡 Após alguns minutos, as instâncias devem estar configuradas e o acesso ao WordPress liberado. Caso não consiga acessar, revise as configurações feitas até o momento.

