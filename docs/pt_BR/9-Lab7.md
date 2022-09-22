# EKS Security Workshop

[**< Voltar**](./8-Lab6.md)

# Protecting the infrastructure & Image Security

[Aqua Trivy](https://aquasecurity.github.io/trivy/v0.32/) (tri pronunciado como gatilho, vy pronunciado como inveja) é um scanner de segurança abrangente. É confiável, rápido, extremamente fácil de usar e funciona onde você precisar.

O Trivy possui diferentes scanners que procuram diferentes problemas de segurança e diferentes alvos onde podem encontrar esses problemas.

Alvos:

    Imagem do contêiner
    Sistema de arquivo
    Repositório Git (remoto)
    Cluster ou recurso do Kubernetes

Scanners:

    Pacotes de SO e dependências de software em uso (SBOM)
    Vulnerabilidades conhecidas (CVEs)
    Configurações incorretas de IaC
    Informações confidenciais e segredos


## Como instalar localmente: 

```bash
rpm -ivh https://github.com/aquasecurity/trivy/releases/download/v0.32.0/trivy_0.32.0_Linux-64bit.rpm
```

Verificar  configurações incorretas em arquivos de IaC como Terraform (para verificar a criação do cluster EKS) e Dockerfile

Comando: 
```bash
trivy config [SEU_IAC_DIR]
```

No nosso workshop podemos aplicar no diretório eks-security-workshop/terraform

```bash
trivy config eks-security-workshop/terraform
```
## Como instalar em nosso EKS Cluster

### Kubernetes Cluster Scanning - trivy operator (instalação com helm):

Primeiro, vamos adicionar o repositório Aqua Security Helm à nossa lista de repositórios Helm:

```bash
helm repo add aqua https://aquasecurity.github.io/helm-charts/
```

Em seguida, atualizaremos todos os nossos repositórios do Helm. Mesmo que você tenha acabado de adicionar um novo repositório aos gráficos existentes, geralmente é uma boa prática ter acesso às alterações mais recentes:


```bash
helm repo update
 ```

Por fim, podemos instalar o Helm Chart do operador Trivy em nosso cluster:

```bash
helm install trivy-operator aqua/trivy-operator \
   --namespace trivy-system \
   --create-namespace \
   --set="trivy.ignoreUnfixed=true" \
   --version v0.0.3
 ```

Você pode certificar-se de que o operador está instalado corretamente através do seguinte comando:

```bash
kubectl get deployment -n trivy-system 
```

O Trivy começará automaticamente a verificar seus recursos do Kubernetes. Por exemplo, você pode visualizar relatórios de vulnerabilidade com o seguinte comando:

```bash
kubectl get vulnerabilityreports --all-namespaces -o wide 
```

E então você pode acessar os detalhes do seu scan de segurança:

```bash
kubectl describe  vulnerabilityreports <name of one of the above reports> 
```

O mesmo processo pode ser aplicado para acessar o Configauditreports:

```bash
kubectl get configauditreports --all-namespaces -o wide 
```
## Scan de Images

### Como criar um pipeline de CI/CD para verificação de vulnerabilidades de contêiner com Trivy e AWS Security Hub

Nesse Lab iremos seguir os steps desse [blog post](https://aws.amazon.com/blogs/security/how-to-build-ci-cd-pipeline-container-vulnerability-scanning-trivy-and-aws-security-hub/) 

1. Objetivo

Neste Lab, vocês irão criar um pipeline de integração contínua e entrega contínua (CI/CD) usando o AWS Developer Tools, bem como o scanner de vulnerabilidade de containers de código aberto da Aqua Security, o Trivy. Você criará duas imagens do Docker, uma com vulnerabilidades e outra sem, para conhecer os recursos do Trivy e como enviar todas as informações de vulnerabilidade ao AWS Security Hub.

Muitas implantações de containers utilizam o Docker. O Docker usa Dockerfiles para definir os comandos que você usa para criar a imagem do Docker que forma a base do seu container. Em vez de aplicar o patch no local, você reescreve seu Dockerfile para apontar para imagens de base mais atualizadas, dependências ou ambas e rebuildar (reconstruir) a imagem do Docker. O Trivy permite que você saiba quais dependências na imagem do Docker são vulneráveis ​​e qual versão dessas dependências não são mais vulneráveis, permitindo que você entenda rapidamente o que corrigir para voltar a um estado seguro.

2. Arquitetura da solução:

<p align="left"> 
<img src="../static/9.1-trivy9.png">
</p>

Veja como a solução funciona, conforme mostrado na Figura 1:

* Os desenvolvedores enviam Dockerfiles e outros códigos para o [AWS CodeCommit](https://aws.amazon.com/codecommit/features/).
* O [AWS CodePipeline](https://aws.amazon.com/codepipeline/features/?nc=sn&loc=2) inicia automaticamente uma compilação do [AWS CodeBuild](https://aws.amazon.com/codebuild/features/?nc=sn&loc=2) que usa uma [build specification](https://docs.aws.amazon.com/codebuild/latest/userguide/build-spec-ref.html) (arquivo de especificação de compilação) para instalar o Trivy, compilar uma imagem do Docker e verificá-la durante o tempo de execução.
* O AWS CodeBuild envia os logs de compilação quase em tempo real para um grupo do [Amazon CloudWatch Logs](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html).
* O Trivy verifica todas as vulnerabilidades e as envia ao AWS Security Hub, independentemente da gravidade.
* Se nenhuma vulnerabilidade crítica for encontrada, as imagens do Docker serão consideradas como aprovadas na verificação e enviadas ao [Amazon Elastic Container Registry (ECR)](https://aws.amazon.com/ecr/), para que possam ser implantadas.

> Observação: o CodePipeline oferece suporte a diferentes fontes, como Amazon Simple Storage Service (Amazon S3) ou GitHub. Se você estiver confortável com esses serviços, sinta-se à vontade para substituí-los por este passo a passo da solução.

> Para implantar a solução rapidamente, você usará um modelo do [AWS CloudFormation]() para implantar todos os serviços necessários.

3. Pré-requisitos

* Você deve ter o Security Hub habilitado na região da AWS onde você implanta esta solução. No AWS Management Console, vá para AWS Security Hub e selecione Enable Security Hub.
*adicionar esse step aqui com print*
* Você deve ter a integração do Aqua Security habilitada no Security Hub na região onde você implanta esta solução. Para fazer isso, acesse o console do AWS Security Hub e, à esquerda, selecione Integrations, pesquise Aqua Security e selecione Accept Findings.
*adicionar esse step aqui com print*

4. Configuração

Para esta etapa, você implantará o modelo do CloudFormation e fará a configuração preliminar do repositório do CodeCommit.

* Baixe o modelo do CloudFormation do GitHub nesse [link](https://github.com/aws-samples/aws-security-hub-scan-with-trivy/blob/master/TrivyVulnScan-to-SecHub.yaml) e crie uma nova stack do CloudFormation.
* Depois que a stack do CloudFormation for concluída, acesse o console do CloudFormation e selecione a guia Resources para ver os recursos criados, conforme mostrado na figura abaixo.

<p align="left"> 
<img src="../static/9.2-trivy2.png">
</p>

Configurando o repositório do CodeCommit:

Os repositórios do CodeCommit precisam de pelo menos um arquivo para inicializar seu branch master. Sem um arquivo, você não pode usar um repositório do CodeCommit como fonte para o CodePipeline. Para criar um arquivo de amostra, faça o seguinte.

* Vá para o console do CodeCommit e, à esquerda, selecione Repositórios e, em seguida, selecione seu repositório do CodeCommit.
* Role até a parte inferior da página, selecione a lista suspensa Adicionar arquivo e, em seguida, selecione Criar arquivo.
* Na tela Criar um arquivo, insira readme no corpo do texto, nomeie o arquivo readme.md, insira seu nome como Nome do autor e seu endereço de e-mail e selecione Confirmar alterações, conforme mostrado na abaixo.

<p align="left"> 
<img src="../static/9.3-trivy3.png">
</p>

*tentar fazer um gif com essas açoes*



5. Simule uma imagem com vulnerabilidade

6. Analise seu container com vulnerabilidade no AWS Security Hub

7. Push uma imagem sem vulnerabilidade 


## Aprenda mais:

### GitOps:
Outra forma bastante eficaz de instalar e operar o Trivy é utilizando o ArgoCD (esse Addon GitOps pode ser facilmente habilita via EKS Blueprints).Segue o passo-a-passo para configuração via ArgoCD: https://aquasecurity.github.io/trivy/v0.32/tutorials/kubernetes/gitops/

[**Próximo >**](./10-Lab8.md)