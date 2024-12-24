# Atividade Docker - AWS

![image](https://github.com/user-attachments/assets/ce0379b2-9f1c-49fc-9cee-8ebffba36211)

## Introdução

Esté projeto consiste em rodar o WordPress em instancias EC2 da AWS, criando uma infraestrutura como na imagem acima.

## Passo 1: Criar uma VPC

- **Acesse o Console da AWS:** Entre na sua conta da AWS e vá para o serviço VPC.
- Clique em "Create VPC".
- Selecione a opção "VPC e muito mais", para criarmos uma vpc com as sub-reds necessarias e as tabelas de rotas.
- preencha os detalhes necessários como nome da VPC e etc, para o bloco CIDR vamos usar o endereço IP: "10.0.0.0/16" e podemos deixar o resto das configurações padrões já como vem quando apertamos em criar VPC.
- Usando a opção de "Criar VPC e muito mais" a AWS criar a VPC e criar também as Sub-redes, Tabelas de Rotas e a Conexções de rede, então não precisamos criar sub-redes e rotas somente configura elas.
## Passo 2: Criar o banco de dados RDS

- Na barra de busca no topo da página, digite "RDS" e selecione RDS na lista de serviços.
- No painel de navegação à esquerda, clique em Databases.
- Clique em Create database (Criar banco de dados).
- Na página Create database, selecione Standard create para mais opções de configuração.
- Selecionar o Tipo de Mecanismo em Opções do mecanismo, escolha o tipo de banco de dados que deseja usar (por exemplo, MySQL, PostgreSQL, etc.), na seção Templates, escolha (Nível gratuito) se você estiver elegível para isso.
- Em DB instance class, selecione db.t3.micro para uso no nível gratuito, em Storage (Armazenamento), defina um valor como 20 GB (ou o mínimo permitido), em DB instance identifier, insira um nome exclusivo para sua instância.
- Defina um nome de usuário e uma senha para o usuário master do banco de dados. Anote essas credenciais, pois você precisará delas para se conectar ao banco.
- Escolha uma VPC Security Group que permita acesso ao seu banco de dados, adicione regras que permitam conexões na porta padrão do banco (por exemplo, 3306 para MySQL).
- Revise todas as configurações que você fez e clique em Criar banco de dados no final da página.
## 
