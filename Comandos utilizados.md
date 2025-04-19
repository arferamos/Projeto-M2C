Instale a ferramenta CLI eksctl 


curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp


sudo cp /tmp/eksctl /usr/bin


eksctl version
​
Instale a ferramenta CLI kubectl 


curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.18.9/2020-11-02/bin/linux/amd64/kubectl


chmod +x ./kubectl


mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin

echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc


kubectl version --short --client
​
Instale a ferramenta CLI helm:

curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3

chmod 700 get_helm.sh

./get_helm.sh

helm version
​
Crie um usuário IAM eks-user com AdministratorAccess policy.

Crie uma Access Key para o eks-user

Usando o Terraform, provisione o S3 e a tabela DynamoDB a serem usados pela Aplicação 

cd /home/ec2-user/environment/human-gov-infrastructure/terraform

terraform show

terraform plan

terraform apply
​

## Crie um EKS Cluster
eksctl create cluster --name humangov-cluster --region us-east-1 --nodegroup-name standard-workers --node-type t3.medium --nodes 1

​
## Conecte-se ao cluster EKS usando a configuração do kubectl 
aws eks update-kubeconfig --name humangov-cluster

​
## Verifique a conectividade do Cluster
kubectl get svc

kubectl get nodes

## Crie uma política IAM.
Baixe uma política IAM para o AWS Load Balancer Controller que permita que ele faça chamadas às APIs da AWS em seu nome.
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/refs/heads/main/docs/install/iam_policy.json
​
## Crie um provedor de identidade IAM OIDC para o seu cluster com o eksctl
eksctl utils associate-iam-oidc-provider --cluster humangov-cluster --approve
​
Crie uma função IAM e uma conta de serviço Kubernetes com o nome aws-load-balancer-controller no namespace kube-system para o AWS Load Balancer Controller e adicione uma anotação à conta de serviço Kubernetes com o nome da IAM role.

## Instale o AWS Load Balancer Controller usando o Helm V3 ou posterior, ou aplicando um manifesto Kubernetes
Adicione o repositório eks-charts

helm repo add eks https://aws.github.io/eks-charts
​
Atualize seu repositório local para garantir que você tenha os charts mais recentes.

helm repo update eks
​
Instale o AWS Load Balancer Controller.

### Verifique que o controller está instalado.
kubectl get deployment -n kube-system aws-load-balancer-controller
​
Um exemplo de output é o seguinte:

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE

aws-load-balancer-controller   2/2     2            2           84s


### Crie uma Role & Service Account para fornecer aos pods acesso às tabelas S3 e DynamoDB.
eksctl create iamserviceaccount \
  --cluster=humangov-cluster \
  --name=humangov-pod-execution-role \
  --role-name HumanGovPodExecutionRole \
  --attach-policy-arn=arn:aws:iam::aws:policy/AmazonS3FullAccess \
  --attach-policy-arn=arn:aws:iam::aws:policy/AmazonDynamoDBFullAccess \
  --region us-east-1 \
  --approve
​
### Coloque a aplicação em container
Mude para o diretório da aplicação.
​
### Crie um Repositório ECR
Crie um novo repositório ECR público chamado humangov-app, build and push a imagem Docker para o repositório ECR criado 

Recupere o token de autenticação e autentique o seu cliente Docker no seu registro. Use o AWS CLI:

<get this command from View push commands in ECR>
​
docker build -t humangov-app .
​
Após a conclusão do build, adicione uma tag à sua imagem, para que você possa fazer o push da imagem para este repositório:
  
<get this command from View push commands in ECR>
​
Execute o seguinte comando para fazer o push desta imagem para o novo repositório AWS que você criou:
  
<get this command from View push commands in ECR>
​
### Implemente a Aplicação para Cada Estado
Crie o arquivo humangov-california.yaml no diretório human-gov-application/src . 
Substitua o caminho da imagem do ECR e o nome do Bucket da AWS pelos nomes dos seus recursos, conforme mostrado em vermelho abaixo:
apiVersion: apps/v1
kind: Deployment
metadata:
  name: humangov-python-app-california
spec:
  replicas: 1
  selector:
    matchLabels:
      app: humangov-python-app-california
  template:
    metadata:
      labels:
        app: humangov-python-app-california
    spec:
      serviceAccountName: humangov-pod-execution-role
      containers:
      - name: humangov-python-app-california
        image: public.ecr.aws/l4c0j8h9/humangov-app:latest
        env:
        - name: AWS_BUCKET
          value: "humangov-california-s3-tim7"
        - name: AWS_DYNAMODB_TABLE
          value: "humangov-california-dynamodb"
        - name: AWS_REGION
          value: "us-east-1"
        - name: US_STATE
          value: "california"

---

apiVersion: v1
kind: Service
metadata:
  name: humangov-python-app-service-california
spec:
  type: ClusterIP
  selector:
    app: humangov-python-app-california
  ports:
    - protocol: TCP
      port: 8000
      targetPort: 8000

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: humangov-nginx-reverse-proxy-california
spec:
  replicas: 1
  selector:
    matchLabels:
      app: humangov-nginx-reverse-proxy-california
  template:
    metadata:
      labels:
        app: humangov-nginx-reverse-proxy-california
    spec:
      containers:
      - name: humangov-nginx-reverse-proxy-california
        image: nginx:alpine
        ports:
        - containerPort: 80
        volumeMounts:
        - name: humangov-nginx-config-california-vol
          mountPath: /etc/nginx/
      volumes:
      - name: humangov-nginx-config-california-vol
        configMap:
          name: humangov-nginx-config-california

---

apiVersion: v1
kind: Service
metadata:
  name: humangov-nginx-service-california
spec:
  selector:
    app: humangov-nginx-reverse-proxy-california
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80

---

apiVersion: v1
kind: ConfigMap
metadata:
  name: humangov-nginx-config-california
data:
  nginx.conf: |

    events {
      worker_connections 1024;
    }

    http {

      server {
        listen 80;

        location / {
          proxy_set_header Host $host;
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          proxy_pass http://humangov-python-app-service-california:8000; # App container
        }
      }
    }
  
  proxy_params: |
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

​
kubectl apply -f humangov-california.yaml




 Crie um novo domínio no Route 53 (os domínios .click geralmente são os mais baratos)
 Crie um Certificado para o ALB
    
  Acesse o AWS Certificate Manager e solicite um novo certificado público para `*.yourdomain.click`.
    
 Implemente as Ingress Rules
    
 Crie um arquivo YAML de Ingress (`humangov-ingress-all.yaml`) que inclua regras para cada estado. **Certifique-se de substituir o Certificate ARN em vermelho pelo seu Certificate ARN.**
    
    ```yaml
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: humangov-python-app-ingress
      annotations:
        alb.ingress.kubernetes.io/scheme: internet-facing
        alb.ingress.kubernetes.io/target-type: ip
        alb.ingress.kubernetes.io/group.name: frontend
       ** alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:us-east-1:xxxxxxxxxxx:certificate/xxxxxxxxxxx**
        alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}, {"HTTPS":443}]'
        alb.ingress.kubernetes.io/ssl-redirect: '443'
      labels:
        app: humangov-python-app-ingress
    spec:
      ingressClassName: alb
      rules:
        - host: california.humangov.click
          http:
            paths:
            - path: /
              pathType: Prefix
              backend:
                service:
                  name: humangov-nginx-service-california
                  port:
                    number: 80
    ```
    
    Aplique o ingress:
    
    ```bash
    kubectl apply -f humangov-ingress-all.yaml
    
    ```
    
Verifique o Ingress Controller**
    
    ```bash
    kubectl get ingress
    
    ```
    
Crie uma entrada de alias (Record Type: A) no Route 53 que aponte o subdomínio para o domínio do ALB.
Exemplo: `california.humangov.click` → `dualstack.k8s-frontend-ded5adda2e-875432214.us-east-1.elb.amazonaws.com` 
    
Teste a aplicação 
Repita o processo para implementar a aplicação do HumanGov na Flórida no mesmo cluster**.
    
Provisione o DynamoDB e o bucket S3 para a aplicação na Flórida com o Terraform e anote os nomes dos recursos. Abra o arquivo Terraform `human-gov-infrastructure/terraform/variables.tf`  usando o Cloud9 Editor e adicione `florida` à lista de estados.
    
    ```bash
    variable "states" {
      description = "The list of state names"
      default     = ["california","florida"]
    }
    ```
    
    Aplique a configuração do Terraform.
    
    ```bash
    cd /home/ec2-user/environment/human-gov-infrastructure/terraform
    terraform plan
    terraform apply
    ```
    
Duplique o arquivo de implementação do Kubernetes
    
    ```bash
    cd /home/ec2-user/environment/human-gov-application/src
    cp humangov-california.yaml humangov-florida.yaml
    ```
    
Abra o arquivo `humangov-florida.yaml` e substitua todas as entradas `california` por `florida` usando a função de busca e substituição do Cloud9.
Atualize o nome do AWS_BUCKET para o nome do bucket da Flórida no arquivo `humangov-florida.yaml`. Salve o arquivo.
Implemente a aplicação do HumanGov na Flórida.
    
    ```bash
    kubectl apply -f humangov-florida.yaml
    ```
    
Abra o arquivo `humangov-ingress-all.yaml` no Cloud9 e adicione a Ingress rule para o estado da Flórida. **Certifique-se de substituir o Certificate ARN em vermelho pelo seu Certificate ARN.**
    
    ```yaml
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: humangov-python-app-ingress
      annotations:
        alb.ingress.kubernetes.io/scheme: internet-facing
        alb.ingress.kubernetes.io/target-type: ip
        alb.ingress.kubernetes.io/group.name: frontend
        **alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:us-east-1:xxxxxxxx:certificate/xxxxxxxxxxx**
        alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}, {"HTTPS":443}]'
        alb.ingress.kubernetes.io/ssl-redirect: '443'
      labels:
        app: humangov-python-app-ingress
    spec:
      ingressClassName: alb
      rules:
        - host: california.humangov.click
          http:
            paths:
            - path: /
              pathType: Prefix
              backend:
                service:
                  name: humangov-nginx-service-california
                  port:
                    number: 80
        **- host: florida.humangov.click
          http:
            paths:
            - path: /
              pathType: Prefix
              backend:
                service:
                  name: humangov-nginx-service-florida
                  port:
                    number: 80**
    ```
    
Implemente as mudanças ingress
    
    ```bash
    kubectl apply -f humangov-ingress-all.yaml
    ```
    
Adicione a entrada DNS do Route 53 para o estado da Flórida.
    
Teste a aplicação 

Analise os recursos no nível do Kubernetes e o ALB no nível da Console AWS.

```bash
kubectl get pods
kubectl get deployment
kubectl get svc
kubectl get ingress
```

## Passo Final: Destruindo o ambiente

1. Exclua o Kubernetes Ingress
    
kubectl delete -f humangov-ingress-all.yaml
    ```
    
Exclua os recursos restantes da aplicação no Kubernetes.
    
bash
    kubectl delete -f humangov-california.yaml
    kubectl delete -f humangov-florida.yaml
    ```
    
Exclua o EKS Cluster usando a CLI
    
bash
    eksctl delete cluster --name humangov-cluster --region us-east-1

