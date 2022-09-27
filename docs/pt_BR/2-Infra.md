# EKS Security Workshop

[**< Voltar**](./1-Prepare.md)

### Exporte variáveis ​​que serão utilizadas pelo workshop. 

```bash
export ACCOUNT_ID=$(aws sts get-caller-identity --output text --query Account)
export AWS_REGION=$(curl -s 169.254.169.254/latest/dynamic/instance-identity/document | jq -r '.region')
export AZS=($(aws ec2 describe-availability-zones --query 'AvailabilityZones[].ZoneName' --output text --region $AWS_REGION))
export CLUSTER_NAME='eks-security-workshop'
export TF_VAR_aws_region="${AWS_REGION}"
```

Exporte as variáveis no perfil do bash

```bash
echo "export ACCOUNT_ID=${ACCOUNT_ID}" | tee -a ~/.bash_profile
echo "export AWS_REGION=${AWS_REGION}" | tee -a ~/.bash_profile
echo "export AZS=(${AZS[@]})" | tee -a ~/.bash_profile
echo "export CLUSTER_NAME=${CLUSTER_NAME}" | tee -a ~/.bash_profile
echo "export TF_VAR_aws_region=${TF_VAR_aws_region}" | tee -a ~/.bash_profile
aws configure set default.region ${AWS_REGION}
aws configure get default.region
```

# Provisionando seu cluster EKS

Para o provisionamento do cluster, vamos usar o [**EKS Blueprints**](https://aws.amazon.com/blogs/containers/bootstrapping-clusters-with-eks-blueprints/). EKS Blueprints é uma coleção de módulos de infraestrutura como código (IaC) que ajuda a configurar e implantar clusters EKS consistentes e com addons instalados. Você pode usar EKS Blueprints para inicializar facilmente um cluster EKS com addons do Amazon EKS, bem como uma ampla variedade de addons populares de código aberto, incluindo Prometheus, Karpenter, Nginx, Traefik, AWS Load Balancer Controller, Fluent Bit, Keda , Argo CD e muito mais. O EKS Blueprints também ajuda a implementar controles de segurança relevantes necessários para operar cargas de trabalho de várias equipes no mesmo cluster.

* Para isso vamos testar a versão do Terraform:

``` 
terraform -version
```

* Caso a saida seja superior a versão 1.2.9, por exemplo:
```
Terraform v1.3.0
on linux_amd64
```

* Executar:
```
sudo yum -y downgrade terraform-1.3.0-1.x86_64 terraform-1.2.9-1.x86_64
```
> OBS: A versão do Terraform deve ser inferior a 1.3.0 para garantir as compatibilidades dos modulos

### Etapa 1: clone o repositório usando o comando abaixo

```bash
git clone https://github.com/juwisnie/eks-security-workshop.git
```

### Etapa 2: execute o Terraform INIT

Inicialize um diretório de trabalho com arquivos de configuração

```bash
cd ~/environment/eks-security-workshop/terraform/
terraform init
```

### Etapa 3: executar o Terraform PLAN

Verifique os recursos criados por esta execução

```bash
terraform plan
```

### Passo 4: Finalmente, Terraform APPLY

para criar recursos

```bash
terraform apply --auto-approve
```

## Configurar o kubectl e testar o cluster

Os detalhes do cluster EKS podem ser extraídos da saída do terraform ou do Console AWS para obter o nome do cluster. O comando a seguir atualiza o kubeconfig em sua máquina local onde você executa comandos kubectl para interagir com seu cluster EKS.

### Etapa 5: execute o comando update-kubeconfig

`~/.kube/config`arquivo é atualizado com detalhes do cluster:

```bash
aws eks --region ${AWS_REGION} update-kubeconfig --name ${CLUSTER_NAME}
```

### Etapa 6: Liste todos os worker nodes executando o comando abaixo

```bash
kubectl get nodes
```
Agora que nosso Cluster EKS já está no ar podemos iniciar nossos labs de segurança!

[**Próximo >**](./3-Lab1.md)