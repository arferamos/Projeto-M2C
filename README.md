# Projeto-M2C

# Apresentação da Demanda
O Projeto M2C "Move to Cloud" vem do direcional executivo e tem por objetivo mover toda a aplicação do Datacenter on-premises para a Cloud com objetivo na descontinuidade dos Datacenter próprios.
Futuramente, toda a parte de armazenamento também será movida para a Cloud.

Na fase 1 desse projeto, o objetivo é migrar parte da solução atual para a nuvem tornando a solução em modelo híbrido para aproveitar a escalabilidade e flexibilidade da nuvem sem abandonar investimentos existentes.

A arquitetura atual da solução é baseada e um fluxo de caixa que é composta por dois serviços rodando em VMs em Datacenter on-premises.

Serviço que faz o Controle de Lançamentos e Serviço do Consolidado Diário. 


## MIT License
https://github.com/arferamos/Projeto-M2C/blob/main/LICENSE


## Macro Escopo
Atualmente o parque de servidores de nossos Datacenter on-premises estão em processo de desativação devido a falta de suporte e as aplicações estão sendo migradas para a Nuvem.
A arquitetura de integração dos sistemas legado está baseado em uma arquitetura já obsoleta e sem suporte do fornecedor com o custo muito alto de manutenção e spare parts comprometido. 
O objetivo dessa demanda é prover a atualização tecnológica do barramento de serviços com soluções em container na cloud, com desenvolvimento LowCode para uma maior produtividade e seguir a metodologia DevOps mantendo a solução em ambiente hibrido sem abandonar investimentos existentes em seu ambente de DR e também de armazenamento em storages.

![image](https://github.com/user-attachments/assets/92993449-46d2-4d3c-a1e3-6a5a2b233086)



## Premissas de Arquitetura
Migração do ambiente On-premises para a Cloud deve ser realizada de forma faseada sem impacto no ambiente atual assegurando a eficiência e performance na comunicação dos sistemas.
Todo acesso no novo ambiente será realizado de forma segura e eficiente aos servidores e serviços utilizando protocolos SSH e HTTPS. 

Garantir a escalabilidade e alta disponibilidade "HA" do ambiente proposto.

Garantir as funcionalidades do sistema pelo DR mantendo o percentual acordado de serviço up no Datacenter Corporativo 2.

Manter a solução atual de Storage que está instalada dentro dos Datacenters On-premises com objetivo em manter a solução no formato híbrido para aproveitar a escalabilidade e flexibilidade da nuvem sem abandonar investimentos existentes.


## Volumetria:
A volumetria atual não sofrerá alterações, mas necessitará da criação de uma infraestrutura paralela para conviver com a infraestrutura atual, até que todos os requisitos funcionais tenham sido migrados, testados e homologados dentro do novo ambiente.

Para quantificar a quantidade de transações que entram no Barramento de serviços, temos que considerar a coluna "Transações Entrada" na tabela. Ou seja, uma aplicação consumindo o serviço.
A coluna "Orquestrações Internas" representam todo o fluxo executado pelos serviços consumindo as camadas internas do serviço xpto.

![image](https://github.com/user-attachments/assets/d99cc509-1af2-4ead-a2bb-21b3fb7f4a2f)

![image](https://github.com/user-attachments/assets/3e8d1e8e-6937-4b22-a488-72c2588134ff)


## Cenário As-Is:
Arquitetura sem escalabilidade em ambiente virtualizado, está em “End-of-Service” e obsoleto devido a versões e contrato com fornecedores sem renovação "End-of-Suporte"


## Arquitetura AS-IS
![Cluster segregado-Segregado Projeto SAS Prod To-be_3Nodes](https://github.com/user-attachments/assets/32aa705c-8fd0-4da1-b44a-967c124c8d95)


## Storage H.A com GFS sendo utilizado a solução Huawei, Dell ou Hitachi Vantara na qual tenho experiência nas 3 soluções:
![image](https://github.com/user-attachments/assets/f2a9b5a2-3368-4814-95ed-a49228ca9f4d)

Obs: Nas soluções para storage desses players, é possível tambem utilizar NAS para file server e Object Storage.

## Desaster Recovery
Disaster Recovery plan com link dedicado utilizando mpls, fiber channel com raio de até 300km entre Datacenters On-premises.

![image](https://github.com/user-attachments/assets/033c6449-92f0-412f-b2e1-7b207dc398c3)

# Proposta de Arquitetura para o cenário To-be
## Objetivo: 
Implantar a aplicação SaaS com subdomínios na AWS usando EKS (Elastic Kubernetes Service).

Infraestrutura:
Configuração da VPC com seus derivados e do cluster EKS.
Uso do Cloud9 com kubectl e eksctl para gerenciamento.

Containerização:
Aplicação containerizada com Docker.
Imagens enviadas para o Amazon ECR (Elastic Container Registry).

Orquestração com Kubernetes:
Criação de manifestos de deployment e service.
Definição de pods, recursos e variáveis de ambiente.

Gerenciamento de domínio e tráfego:
Route 53 usado para zona hospedada e resolução de domínio.

Application Load Balancer (ALB) configurado com ingress.

Segurança:
AWS Certificate Manager usado para provisionar certificados SSL.
Terminação SSL feita no ALB para tráfego seguro.

### Resultado final:
Aplicação altamente disponível, segura e escalável, acessível por domínio seguro.
Utilizando metodologia DevOps com Kubernetes na AWS, Route53 para domínios, roteamento de tráfego, SSL e containerização.



# Arquitetura To-be
![Desenho de arquitetura To-be](https://github.com/user-attachments/assets/8fd57647-5713-4ebe-83d5-7ea92e4e3627)




## Risco e Restrições
### Migrar os serviços do Legado para a arquitetura de integração meta da empresa
A implantação do novo ambiente será realizada através de Janela de Manutenção com GMUD aprovada dentro do Comitê Executivo de Mudanças que serão planejadas por serviços que estão rodando nos servidores xpto e cada uma delas em horário de baixo tráfego e em final de semana.

Mediante reunião realizada entre Gerente de Projetos, Lider Técnico, Capacity, Segurança, Integração e Arquitetura, todo o alinhamento necessário foi realizado.

Nesse formato acima, todos os envolvidos no projeto entende que a infraestrutura não sofrerá impacto durante a Janela de Manutenção.

## Por que não utilizar somente o S3 para os Dados ?
Todos os detalhes estão em DynamoDB vs S3.md

Favor checar no link abaixo:

https://github.com/arferamos/Projeto-M2C/blob/main/DynamoDB%20vs%20S3.md


## Observabilidade
Favor checar no link abaixo:

https://github.com/arferamos/Projeto-M2C/blob/main/Observabilidade.md


## Comandos utilizados para deploy do Projeto
Favor checar em Comando utilizados.md

https://github.com/arferamos/Projeto-M2C/blob/main/Comandos%20utilizados.md


## Evidências da Solução
Favor checar em Evidências.md

https://github.com/arferamos/Projeto-M2C/blob/main/Evid%C3%AAncias.md


## Plano Orçamentário FinOps
Favor checar em Plano FinOps.md clicando no link abaixo:


https://github.com/arferamos/Projeto-M2C/blob/main/FinOps.md

# Autor
Arlindo Ferreira da Silva Ramos

https://www.linkedin.com/in/arlindo-ramos/







