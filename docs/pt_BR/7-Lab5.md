# EKS Security Workshop

[**< Voltar**](./6-Lab4.md)

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


Como instalar: 

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

Kubernetes Cluster Scanning - trivy operator (instalação com helm)

Primeiro, vamos adicionar o repositório Aqua Security Helm à nossa lista de repositórios Helm:

```bash
helm repo add aqua https://aquasecurity.github.io/helm-charts/
```

Em seguida, atualizaremos todos os nossos repositórios do Helm. Mesmo que você tenha acabado de adicionar um novo repositório aos gráficos existentes, geralmente é uma boa prática ter acesso às alterações mais recentes:

helm repo update

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

Aprenda mais:
CI/CD:
O trivy possui diversas formas de implementação e uma bastante interessante é de adicionar a verificação de vulnerabilidade na sua Pipeline de CI/CD, nesse blog post há um passo-a-passo para implementar uma pipeline com AWS Codepipeline: https://aws.amazon.com/blogs/containers/scanning-images-with-trivy-in-an-aws-codepipeline/

GitOps:
Outra forma bastante eficaz de instalar e operar o Trivy é utilizando o ArgoCD (esse Addon GitOps pode ser facilmente habilita via EKS Blueprints).Segue o passo-a-passo para configuração via ArgoCD: https://aquasecurity.github.io/trivy/v0.32/tutorials/kubernetes/gitops/

[**Próximo >**](./8-Lab6.md)