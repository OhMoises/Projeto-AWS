# Atividade Docker - AWS

![image](https://github.com/user-attachments/assets/ce0379b2-9f1c-49fc-9cee-8ebffba36211)

## Introdução

Esté projeto consiste em rodar o WordPress em instancias EC2 da AWS, criando uma infraestrutura como na imagem acima.

## Passo 1: Criar uma VPC

- **Acesse o Console da AWS:** Entre na sua conta da AWS e vá para o serviço VPC.
- Clique em "Create VPC".
- Selecione a opção "VPC e muito mais", para criarmos uma vpc com as sub-reds necessarias e as tabelas de rotas.
- preencha os detalhes necessários como nome da VPC e etc, para o bloco CIDR vamos usar o endereço IP: "10.0.0.0/16" e podemos deixar o resto das configurações padrões já como vem quando apertamos em criar VPC.
- Usando a opção de "Criar VPC e muito mais" a AWS criar a VPC e criar também as Sub-redes, Tabelas de Rotas e a Conexções de rede, então não precisamos criar sub-redes e nem rotas somente configurar elas.
## 
