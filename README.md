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

## Passo 3: Criar Nat gateway
- No painel de navegação do VPC, clique em "Gateways NAT", clique em "Criar um gateway NAT".
  - ### Configurar o Gateway:
  - Nome (opcional): Especifique um nome para o gateway NAT (exemplo: "MeuNATGateway").
  - Sub-rede: Selecione a sub-rede pública onde o NAT Gateway será criado. O NAT Gateway deve estar em uma sub-rede que tenha acesso à internet.
  - Atribuir um IP Elástico: Selecione "Atribuir automaticamente" para que a AWS atribua um endereço IP elástico ao seu NAT Gateway ou escolha um IP elástico existente.
   ![NAT gateway](https://github.com/user-attachments/assets/1b41a616-85a7-4a65-ae92-da6e18e99d08)
- Clique em "Criar gateway NAT" para finalizar a criação.
- Atualizar as Tabelas de Roteamento
- Após criar o NAT Gateway, você precisa atualizar a tabela de rotas da sub-rede privada para direcionar o tráfego da internet através do NAT Gateway.
    1. No console do VPC, clique em "Tabelas de Roteamento" no painel de navegação.
    2. Selecione a tabela de roteamento associada à sua sub-rede privada.
    3. Clique em "Editar rotas".
    4. Adicionar uma nova rota:
        - Destino: 0.0.0.0/0 (para permitir todo o tráfego da internet).
        - Target (Alvo): Selecione o seu NAT Gateway recém-criado.
    5. Clique em "Salvar rotas".
       ![Chat _ Daily PB - Outubro 2024 _ DevSecOps _ Microsoft Teams - Google Chrome 26_12_2024 09_28_21](https://github.com/user-attachments/assets/48305fad-865e-4873-87be-947da0ca476d)
## Passo 4: Criar o banco de dados RDS
- Na barra de busca no topo da página, digite "RDS" e selecione RDS na lista de serviços.
- No painel de navegação à esquerda, clique em Databases.
- Clique em Create database (Criar banco de dados).
- Na página Create database, selecione Standard create para mais opções de configuração.
- Selecionar o Tipo de Mecanismo em Opções do mecanismo, escolha o tipo de banco de dados que deseja usar (por exemplo, MySQL, PostgreSQL, etc.), na seção Templates, escolha (Nível gratuito) se você estiver elegível para isso.
- Em DB instance class, selecione db.t3.micro para uso no nível gratuito, em Storage (Armazenamento), defina um valor como 20 GB (ou o mínimo permitido), em DB instance identifier, insira um nome exclusivo para sua instância.
- Defina um nome de usuário e uma senha para o usuário master do banco de dados. Anote essas credenciais, pois você precisará delas para se conectar ao banco.
- Escolha uma VPC Security Group que permita acesso ao seu banco de dados, adicione regras que permitam conexões na porta padrão do banco (por exemplo, 3306 para MySQL).
- Revise todas as configurações que você fez e clique em Criar banco de dados no final da página.
## Passo 5: Criar o EFS
- Na barra de busca no topo da página, digite "EFS" e selecione EFS na lista de serviços.
- No console do EFS, clique no botão "Criar sistema de arquivos".
- Nome: Insira um nome para o sistema de arquivos. VPC: Escolha a VPC onde você deseja criar o EFS.
- Após configurar as opções desejadas, clique em "Criar" para criar o sistema de arquivos.
## Passo 6: Criar EC2 (Instância)
- Vamos precisar criar três EC2, uma vai servir como nossa Bastion-Host e as outras duas sera nossas maquinas virtuais.
- Na barra de busca no topo da página, digite "EC2" e selecione EC2 na lista de serviços.
- Clique em "Executar Instância" escolha uma AMI (Amazon Machine Image), como Amazon Linux 2023 ou Ubuntu.
- Selecione o tipo de instância t2.micro para o nível gratuito.
- Se você não tiver um par de chaves, crie um para acessar sua instância via SSH.
- Em configurações de rede selecione a VPC que criamos antés, adicione uma subnet publica e ativa a função de atribuir IP publico automaticamente.
- Selecione o grupo de segurança que criamos.
- Essas configurações são para nossa EC2 Bastion-host.
- Nas outras EC2 que vamos criar utilizaremos as mesmas configurações da Bastion-host a unica mudança vai ser a utilização de uma subnet privada.
- E vamos desabilitar a função de atribuir IP publico automaticamente.
- O grupo de segurança é o mesmo .
- Por ultimo vamos nos dados do usuario e vamos subir um script para nossaa EC2 ser lançada já com o docker e com os dados do usuario do RDS que criamos.
- O script que vamos usar é este que está logo abaixo, esse script so serve para amazon linux.
  ![script](https://github.com/user-attachments/assets/201ded95-d8e9-4def-8f17-0f0950d73f51)
-   Clique em "Executar Instância" para ser criado a EC2.
## Passo 7: Criar o Load Balancer (Balanceador de Carga)
- Configurar o Load Balancer
  - Nome: Insira um nome para o seu Load Balancer.
  - Scheme: Escolha entre "Internet-facing" (para acesso público) ou "Internal" (para acesso privado).
  - IP address type: Escolha entre IPv4 ou dualstack (IPv4 e IPv6)
 ![Load balancing](https://github.com/user-attachments/assets/64cdaeb6-f791-4a2b-9be0-189610f99f92)
- Configurar Listeners e roteamento:
  - O listener padrão para um Application Load Balancer é a porta 80 (HTTP). Você pode adicionar um listener para HTTPS (porta 443) se desejar.
  - Clique em "Add Listeners" se precisar adicionar mais listeners.
![LB](https://github.com/user-attachments/assets/071f1b7e-64bf-4d0d-92ef-c68b5b42f554)
- Selecionar VPC, Sub-redes e Configurar Grupos de Segurança:
  - Selecione a VPC onde o Load Balancer será criado.
  - Escolha as sub-redes onde o Load Balancer deve ser implantado. É recomendável selecionar sub-redes em diferentes zonas de disponibilidade para alta disponibilidade.
  - Configurar Grupos de Segurança: Selecione o grupo de segurança existente que criamos antes que permita tráfego nas portas que você configurou nos listeners (por exemplo, portas 80 e 443), certifique-se de permitir tráfego de entrada nas portas apropriadas.
![LB-VPC](https://github.com/user-attachments/assets/f74d720d-a0ed-4ad0-9ce4-d3e2f4cbc826)
  - Revise todas as configurações que você fez.
  - Clique em "Create Load Balancer". O processo pode levar alguns minutos.
- Testar o Load Balancer
    - Após a criação, você verá seu Load Balancer listado no console, anote o DNS do Load Balancer, que será algo como my-load-balancer-1234567890.us-east-1.elb.amazonaws.com.
    - Abra um navegador e acesse esse DNS para verificar se ele está funcionando corretamente e se está roteando o tráfego para suas instâncias EC2.
## Passo 8: Criar Auto-Scaling
- Antes de criar um Auto Scaling Group, você precisa de um modelo de lançamento que define como as instâncias devem ser criadas.
- O modelo a ser usado vai ser o mesmo do passo 6, vamos utilizar as mesmas configurações.
    ### Criar um Auto Scaling Group
        - No painel de navegação do EC2, clique em "Auto Scaling Groups" e clique em "Create Auto Scaling group".
        - 1. Configurar o Auto Scaling Group:
          - Auto Scaling group name: Insira um nome para o grupo.
          - Launch template: Selecione o modelo de lançamento criado anteriormente.
          - Version: Escolha a versão do modelo mais recente.
        - 2. Configurar Capacidade:
          - Desired capacity: Defina a capacidade desejada (número inicial de instâncias).
          - Minimum capacity: Defina o número mínimo de instâncias que deseja manter.
          - Maximum capacity: Defina o número máximo de instâncias que podem ser criadas.
        - 3. Configurar Zonas de Disponibilidade
          - Selecione as zonas de disponibilidade onde as instâncias serão lançadas. É recomendável escolher múltiplas zonas para alta disponibilidade.
        - 4. Revisar e Criar
          -  Revise todas as configurações feitas e certifique-se de que tudo está correto, clique em "Create Auto Scaling group". O processo pode levar alguns minutos.
        - 5. Verificar o Auto Scaling Group
          - Após a criação, você verá seu Auto Scaling Group listado no console. Você pode monitorar a saúde das instâncias e verificar se elas estão escalando conforme configurado.
  
  





