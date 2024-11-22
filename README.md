# AWS-CLI-VPC
Building a multi AZ VPC Architecture


## 1.Criar uma VPC

## 1.1 Criar uma nova VPC com um bloco CIDR /16:
 aws ec2 create-vpc --cidr-block 192.0.0.0/16
  
  
## 2.Adicionar uma Tag à VPC

aws ec2 create-tags --resources vpc-0b886de99d2130633 --tags Key=Name,Value=PraticalVPC
 
 
## 3. Habilitar DNS Hostnames e Suporte a DNS para a VPC

aws ec2 modify-vpc-attribute --vpc-id vpc-0b886de99d2130633 --enable-dns-hostnames

## 5.Habilitar Suporte a DNS:
 aws ec2 modify-vpc-attribute --vpc-id vpc-abc12345 --enable-dns-support

## 6.Criar Sub-redes na VPC
## 6.1 Criar uma Sub-rede em uma zona de disponibilidade específica (us-east-1a):

aws ec2 create-subnet --vpc-id vpc-0b886de99d2130633 --cidr-block 192.0.1.0/24 --availability-zone us-east-1a


## 5.Adicionar Tags às Sub-redes

aws ec2 create-tags --resources subnet-07df7998a8fe93b4a --tags Key=Name,Value=PublicSubnet


## 6.Criar Gateway de Internet

 aws ec2 create-internet-gateway

## 7.Adicionar uma tag ao Gateway de Internet:

aws ec2 create-tags --resources igw-061393fae44311300 --tags Key=Name,Value=Pratical-IG

## 8. Associar o Gateway de Internet à VPC:

aws ec2 attach-internet-gateway --internet-gateway-id igw-061393fae44311300 --vpc-id vpc-0b886de99d2130633


## 9. Alocar IP Elástico para o NAT Gateway

aws ec2 allocate-address --domain vpc

   
## 10.Criar o NAT Gateway usando o IP Elástico alocado
aws ec2 create-nat-gateway --subnet-id subnet-07df7998a8fe93b4a --allocation-id eipalloc-081e2a2d8c5ad3be9

## 11.Adicionar uma tag ao NAT Gateway:
aws ec2 create-tags --resources nat-03a2cca7596319169 --tags Key=Name,Value=NateGatewayAZ1

  
## 12.Criar Tabelas de Roteamento

aws ec2 create-route-table --vpc-id vpc-0b886de99d2130633

## 13.Adicionar uma tag à Tabela de Roteamento:
aws ec2 create-tags --resources rtb-019395246943bf730 --tags Key=Name,Value=PrivateRouteTable

## 14.Criar rotas para sub-redes públicas e privadas:
  - Para a rota pública (Gateway de Internet):

aws ec2 create-route --route-table-id rtb-019395246943bf730 --destination-cidr-block 0.0.0.0/0 --gateway-id igw-061393fae44311300

  - Para a rota privada (NAT Gateway):

aws ec2 create-route --route-table-id rtb-0a9ab1eb37f110bbc --destination-cidr-block 0.0.0.0/0 --gateway-id nat-03a2cca7596319169

## 15.Associar Tabelas de Roteamento com Sub-redes

aws ec2 associate-route-table --route-table-id rtb-019395246943bf730 --subnet-id subnet-07df7998a8fe93b4a

## 16.Associar a tabela de roteamento privada à sub-rede privada:

aws ec2 associate-route-table --route-table-id rtb-0a9ab1eb37f110bbc --subnet-id subnet-0de289a45c7b2b6f6
    
## 17.Criar um Grupo de Segurança

aws ec2 create-security-group --group-name pratical-security-group --description "web server" --vpc-id vpc-0b886de99d2130633

## 19.Adicionar uma tag ao Grupo de Segurança:

aws ec2 create-tags --resources sg-0dda9473271188921 --tags Key=Name,Value=pratical-security-group

## 20.Adicionar regras de ingresso ao Grupo de Segurança:
   - Permitir HTTP (porta 80) de qualquer IP:

aws ec2 authorize-security-group-ingress --group-id sg-0dda9473271188921 --protocol tcp --port 80 --cidr 0.0.0.0/0
 
   - Permitir SSH (porta 22) de qualquer lugar:
   
aws ec2 authorize-security-group-ingress --group-id sg-0dda9473271188921 --protocol tcp --port 22 --cidr 0.0.0.0/0

## 21.Criar um Par de Chave

22.Criar um Par de Chaves para a Instância EC2:

aws ec2 create-key-pair --key-name MinhakeyPair --query 'MinhaKeyPair' --output text > MinhakeyPair.pem

## 23.Lançar uma Instância EC2


aws ec2 run-instances --image-id ami-0166fe664262f664c --instance-type t2.micro --count 1 --subnet-id subnet-07df7998a8fe93b4a --security-group-ids sg-0dda9473271188921 --associate-public-ip-address --key-name MinhaKeyPair

## 24. Tag the EC2 Instance:

aws ec2 create-tags --resources i-0cb80aea6ecd455a3 --tags Key=Name,Value=Pratical-demo

## Aceder a instancia atravez de um cliente ssh

ssh -i "MinhaKeyPair" ec2-user@ipdainstancia


## 25: Instalar o Apache (HTTPD)
sudo yum install -y httpd

## 26: Adicionar o utilizador ec2-user ao grupo apache
sudo usermod -a -G apache ec2-user

## 27: Alterar a propriedade e permissões dos diretórios
sudo chown -R ec2-user:apache /var/www

sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;

find /var/www -type f -exec sudo chmod 0664 {} \;

## 28: Baixar e extrair o website estático
cd /tmp

wget https://github.com/edgarjose/static-website/archive/refs/heads/main.zip

unzip main.zip

cd static-website-main

cp -r festava_live/* /var/www/html/

## 29 : Limpar ficheiros temporários
rm -rf festava_live main.zip

## 30 : Configurar o Apache para iniciar automaticamente e iniciar o serviço
sudo systemctl enable httpd

sudo systemctl start httpd

