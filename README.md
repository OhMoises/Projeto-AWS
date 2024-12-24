# Atividade Docker - AWS

![image](https://github.com/user-attachments/assets/ce0379b2-9f1c-49fc-9cee-8ebffba36211)

## Introdução

Esté projeto consiste em rodar o WordPress em instancias EC2 da AWS, criando uma infraestrutura como na imagem acima.

## Passo 1: Criar uma VPC
- **Acesse o Console da AWS:** Entre na sua conta da AWS e vá para o serviço VPC.
- Clique em "Create VPC".
- Selecione a opção "VPC e muito mais", para criarmos uma vpc com as sub-reds necessarias e as tabelas de rotas.
- preencha os detalhes necessários como nome da VPC e etc, para o bloco CIDR vamos usar o endereço IP: "10.0.0.0/16" e podemos deixar o resto das configurações padrões já como vem quando apertamos em criar VPC.
  ![VPC _ us-east-1 - Google Chrome 24_12_2024 11_17_40](https://github.com/user-attachments/assets/cdc24270-703b-41b1-9de3-cb63b6288b69)
## Passo 2: Criação do grupo de segurança
- Na barra de busca no topo da página procure por grupo de segurança.
- Clique em "Criar Grupo de Segurança, vamos criar quatro grupos de segurança.
- Um grupo para a EC2, um grupo para o RDS, um grupo para a EFS e um grupo para o LoadBalancer.

 ### Para o EC2:
- Entrada 
  | Tipo         | Protocolo|  Porta     |      Tipo de Origem         |
  |--------------|----------|------------|-----------------------------|
  |     HTTP     |    TCP   |    80      |    qualquer origem (0.0.0.0/0)     |
  |     SSH      |    TCP   |    22      |    qualquer origem (0.0.0.0/0)     |
  |    HTTPS     |    TCP   |    443     |    qualquer origem (0.0.0.0/0)     |
  
![VPC _ us-east-1 - Google Chrome 24_12_2024 14_15_45](https://github.com/user-attachments/assets/d438d6aa-aff9-47c5-b5d4-50456a287ed2)

- Saída
  | Tipo          | Protocolo|  Porta     |      Tipo de Origem         |
  |---------------|----------|------------|-----------------------------|
  | Todo tráfego  |   Todos  |   Tudo     |        0.0.0.0/0            |
  | MySQL/Aurora  |   TCP    |   2206     |  Grupo de Segurança da RDS  |
  |    NFS        |   TCP    |   2049     |  Grupo de Segurança da EFS  |

![VPC _ us-east-1 - Google Chrome 24_12_2024 14_25_31](https://github.com/user-attachments/assets/df71dab7-e98a-4b31-b858-f107c3ac3f28)

  ### Para o RDS MySql:
- Entrada 
  | Tipo         | Protocolo|  Porta     |      Tipo de Origem         |
  |--------------|----------|------------|-----------------------------|
  | MySql/Aurora |    TCP   |   3306     | Grupo de Segurança da EC2   |

![VPC _ us-east-1 - Google Chrome 24_12_2024 14_37_40](https://github.com/user-attachments/assets/d327b916-afe7-470e-b676-dfc60f482923)

  ### Para o EFS:
- Entrada
  | Tipo         | Protocolo|  Porta     |      Tipo de Origem         |
  |--------------|----------|------------|-----------------------------|
  |    NFS       |    TCP   |   2049     |  Grupo de Segurança da EC2  |

![grupo-sg-EFS](https://github.com/user-attachments/assets/a59e5164-b242-4a33-b498-bb2f8d1afc36)

  ### Para o LoadBalancer:
- Entrada
  | Tipo         | Protocolo|  Porta     |      Tipo de Origem         |
  |--------------|----------|------------|-----------------------------|
  |     HTTP     |    TCP   |    80      |         0.0.0.0/0           |

![grupo-sg-LB](https://github.com/user-attachments/assets/3cafa8a7-2200-41c8-802a-f2bc1a9b7242)

- Saída
  | Tipo         | Protocolo|  Porta     |      Tipo de Origem         |
  |--------------|----------|------------|-----------------------------|
  | Todo tráfego |   TCP    |   Tudo     |         0.0.0.0/0           |
  |    HTTP      |   TCP    |     80     | Grupo de Segurança da EC2   |

![grupo-sg-LB-saida](https://github.com/user-attachments/assets/a25def72-f737-443b-8d04-626f5e3d26a4)

## Passo : Criar o banco de dados RDS
- Na barra de busca no topo da página, digite "RDS" e selecione RDS na lista de serviços.
- No painel de navegação à esquerda, clique em Databases.
- Clique em Create database (Criar banco de dados).
- Na página Create database, selecione Standard create para mais opções de configuração.
- Selecionar o Tipo de Mecanismo em Opções do mecanismo, escolha o tipo de banco de dados que deseja usar (por exemplo, MySQL, PostgreSQL, etc.), na seção Templates, escolha (Nível gratuito) se você estiver elegível para isso.
- Em DB instance class, selecione db.t3.micro para uso no nível gratuito, em Storage (Armazenamento), defina um valor como 20 GB (ou o mínimo permitido), em DB instance identifier, insira um nome exclusivo para sua instância.
- Defina um nome de usuário e uma senha para o usuário master do banco de dados. Anote essas credenciais, pois você precisará delas para se conectar ao banco.
- Escolha uma VPC Security Group que permita acesso ao seu banco de dados, adicione regras que permitam conexões na porta padrão do banco (por exemplo, 3306 para MySQL).
- Revise todas as configurações que você fez e clique em Criar banco de dados no final da página.
## Passo 3: Criar o EFS
- Na barra de busca no topo da página, digite "EFS" e selecione EFS na lista de serviços.
- No console do EFS, clique no botão "Criar sistema de arquivos".
- Nome: Insira um nome para o sistema de arquivos. VPC: Escolha a VPC onde você deseja criar o EFS.
- Após configurar as opções desejadas, clique em "Criar" para criar o sistema de arquivos.
## Passo 4: Criação do grupo de segurança
- Na barra de busca no topo da página procure por grupo de segurança.
- Clique em "Criar Grupo de Segurança, de um nome para o grupo de segurança, associe a VPC que criamos.
- Regras de entradas vamos deixar tipo: SSH, protoloco: TCP, intervalo de portas: 22 e origem: Qualquer local-IPv4.
- Regas de saída, tipo: Todo o tráfego, protoloco: Tudo, intervalo de portas: Tudo e destino: Qualquer local-IPv4.
## Passo 5: Criar EC2, Bastion-Host
- Na barra de busca no topo da página, digite "EC2" e selecione EC2 na lista de serviços.
- Clique em "Executar Instância" escolha uma AMI (Amazon Machine Image), como Amazon Linux 2023 ou Ubuntu.
- Selecione o tipo de instância t2.micro para o nível gratuito.
- Se você não tiver um par de chaves, crie um para acessar sua instância via SSH.
- Em configurações de rede selecione a VPC que criamos antés, adicione uma subnet publica e ativa a função de atribuir IP publico automaticamente.
- Selecione o grupo de segurança que criamos.
## Passo 6: Criando maquinas EC2,
- Para crias nossas EC2 vamos serguir o mesmo passo anterior a unica mudança é que vamos usar uma subnet privada.
- E vamos desabilitar a função de atribuir IP publico automaticamente.
- O grupo de segurança é o mesmo .
- Por ultimo vamos nos dados do usuario e vamos subir um script para nossa EC2 ser lançada já com o docker e com os dados do usuario do RDS que criamos.
- O script que vamos usar é este que está logo abaixo, esse script so serve para amazon linux.
  ![script](https://github.com/user-attachments/assets/201ded95-d8e9-4def-8f17-0f0950d73f51)
-   Clique em "Executar Instância" para ser criado a EC2.
## Passo x: Criar Nat gateway
- Na barra de busca no topo da página procure por NAT gateway.
- 

