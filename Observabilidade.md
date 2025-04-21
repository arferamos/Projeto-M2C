# Observabilidade

## Grafana exibe métricas e alertas: 
O Grafana roda no EKS ou via Amazon Managed Grafana.
Ele se conecta ao Prometheus como data source sendo possível criar dashboards personalizados com gráficos e tabelas, monitorar o estado da infraestrutura e das aplicações em tempo real e configurar alertas visuais no próprio Grafana.

## Prometheus coleta e armazena as métricas: 
O Prometheus roda dentro do cluster EKS via helm, scrapeia os endpoints /metrics periodicamente e armazena os dados em seu banco interno (time series database).
Com base nas métricas, é possível definir regras de alerta (ex: CPU > 85% por 5 minutos).

## Dentro do cluster EKS é necessário instalar os exporters abaixo:
node_exporter: coleta métricas dos nós (CPU, memória, etc).

kube-state-metrics: coleta o estado dos recursos do Kubernetes (pods, deployments, etc.).

cadvisor: coleta métricas de containers em execução.

Esses componentes expõem métricas em formato Prometheus (/metrics endpoint), para leitura.

## Alertmanager dispara alertas:
Quando uma regra de alerta é acionada no Prometheus, ele envia o alerta para o Alertmanager.

O Alertmanager é responsável por evitar alertas duplicados e agrupar alertas semelhantes, rotear de forma decisiva para onde mandar os alertas. 

O alerta pode ser enviado para um webhook para o AWS SNS.


## AWS SNS (Simple Notification Service)
O SNS atua como um canal de distribuição de alertas e o Alertmanager envia os alertas como mensagens utilizando o webhook para um tópico SNS.

O SNS pode então:
Enviar email para operadores/devs.

Disparar mensagens SMS.

Acionar um bot no Slack via Lambda.

Invocar uma Lambda function para ações automáticas (ex: escalar pods, abrir ticket, etc.).

Obs: Na arquitetura proposta, o caminho para o SNS será para envio por emails.

