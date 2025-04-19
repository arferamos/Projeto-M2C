# Por que utilizar o DynamoDB e não somente o Amazon S3 ?
Devido a necessidade de acesso rápido e em tempo real a dados estruturados o DynamoDB oferece latência de milissegundos em leituras e gravações, ideal para aplicações que precisam de resposta rápida, como sistemas transacionais, dashboards em tempo real ou APIs.

DynamoDB gerencia automaticamente o escalonamento de leitura e escrita com provisionamento sob demanda, enquanto o S3 é mais focado em throughput bruto para arquivos.

No S3, cada alteração geralmente envolve sobrescrever o objeto inteiro, enquanto no DynamoDB, é possível atualizar apenas campos específicos de um item, com menor custo (nesse caso) e mais eficiência.  

# Como os dados serão enviados para o Storage on-premises ?
O AWS DataSync é uma ferramenta ideal para transferência eficiente de arquivos entre S3 e Storage local. Ele comprime, criptografa e transfere arquivos de forma segura e incremental sendo ideal pra volumes grandes.
Pode ser agendado ou executado sob demanda e funciona com agente local instalado no datacenter e S3 como origem.


# Fluxo para transferência de dados:
Exportar os dados do DynamoDB para o S3, com exportação nativa via console ou API com suporte ao formato Parquet. Depois disso, o DataSync leva esses dados do S3 até o Storage On-premises.


Conectividade: 

VPN ou AWS Direct Connect

Para que o DataSync ou qualquer outro serviço da AWS consiga alcançar o Storage local (on-premises), será necessário obter uma conexão segura:

1-	VPN Site-to-Site (mais simples e barato, boa para baixa/média carga).

2-	AWS Direct Connect (latência menor, ideal para transferências de alto volume ou missão crítica).

Devido a questões de latência, o AWS Direct Connect é melhor para o cenário em questão.


Com objetivo em deixar o ambiente com relação aos dados de forma híbrida. O cenário acima pode ser utilizado através de um link dedicado entre Datacenter local e Nuvem.


## Obs:
Poderia apesentar a solução diretamente utilizando S3 com suas opções de Lifecycle, mas o objetivo também é demonstrar o conhecimento técnico e de arquitetura em outras opções.
Se o objetivo seria em considerar apenas o S3, a recomendação mediante a reuniões de entendimento sobre a demanda, seria incialmente essa abaixo.

Dia 0   → Objeto criado no S3 Standard  
Dia 30  → Objeto movido para S3 Standard-IA (Infrequent Access)  
Dia 90  → Objeto movido para S3 Glacier Instant Retrieval  
Dia 180 → Objeto movido para S3 Glacier Deep Archive  
Dia 365 → Objeto excluído automaticamente.

