# EKS Security Workshop

[**< Voltar**](./2-Infra.md)

# Identity and Access Management

Nesta série de laboratórios, consideramos vários tópicos relacionados ao gerenciamento de identidade e acesso.

## 1.a - Introduction to Role Based Access Control (RBAC)

Este lab é reproduzido a partir do oficial [EKSWorkshop.com](https://www.eksworkshop.com/beginner/090_rbac/intro/) 

O que é RBAC?

De acordo com os documentos oficiais do kubernetes:

> O Role-based access control (controle de acesso baseado em função) (RBAC) é um método de regular o acesso a recursos de computador ou rede com base nas funções de usuários individuais dentro de uma empresa.

### Os principais componentes lógicos do RBAC são:

__Entity (Entidade)__

Um grupo, usuário ou conta de serviço (uma identidade que representa um aplicativo que deseja executar determinadas operações (ações) e requer permissões para isso).

__Resource (Recurso)__

Um pod, serviço ou secret (segredo) que a entity (entidade) deseja acessar usando determinadas operações.

__Role (Função)__

Usado para definir regras para as ações que a entity (entidade) pode realizar em vários recursos.

__Role binding (Vinculação de papéis)__

Isso anexa (vincula) uma role (função) a uma entity (entidade), informando que o conjunto de regras define as ações permitidas pela entidade anexada nos recursos especificados.

Existem dois tipos de roles (funções): Role e ClusterRole e suas respectivas associações (RoleBinding, ClusterRoleBinding). Eles diferenciam entre autorização em um namespace ou em todo o cluster.

__Namespace__

Os namespaces são uma excelente maneira de criar limites de segurança, eles também fornecem um escopo exclusivo para nomes de objetos, como o nome 'namespace' indica. Eles devem ser usados ​​em ambientes multilocatários para criar clusters kubernetes virtuais no mesmo cluster físico.

## Objetivos para este módulo

Neste módulo, vamos explorar o k8s RBAC criando um usuário do IAM chamado rbac-user que é autenticado para acessar o cluster EKS, mas está autorizado apenas (via RBAC) a listar, obter e observar pods e implantações no ' rbac-test' namespace.

Para conseguir isso, vamos criar um usuário do IAM, mapear esse usuário para uma função do kubernetes e, em seguida, realizar ações do kubernetes no contexto desse usuário.

## Instalar pods de teste

Neste tutorial, vamos demonstrar como fornecer acesso limitado a pods em execução no namespace rbac-test para um usuário chamado rbac-user.

Para fazer isso, vamos primeiro criar o namespace rbac-test e, em seguida, instalar o nginx nele:

```bash
kubectl create namespace rbac-test
kubectl create deploy nginx --image=nginx -n rbac-test
```
Para verificar se os pods de teste foram instalados corretamente, execute:

```bash
kubectl get all -n rbac-test
```
A saída deve ser semelhante a:

<sub>
NAME                       READY   STATUS    RESTARTS   AGE
pod/nginx-5c7588df-8mvxx   1/1     Running   0          48s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   1/1     1            1           48s

NAME                             DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-5c7588df   1         1         1       48s
</sub>

## Crie um usuário

Observação Para simplificar, neste capítulo, salvaremos as credenciais em um arquivo para facilitar a alternância entre os usuários. Nunca faça isso em produção ou com credenciais que tenham acesso privilegiado; Não é uma prática recomendada de segurança armazenar credenciais no sistema de arquivos.

De dentro do terminal do Cloud9, crie um novo usuário chamado rbac-user e gere/salve credenciais para ele:

Ao executar a etapa anterior, você deve obter uma resposta semelhante a:

<sub>
{
    "AccessKey": {
        "UserName": "rbac-user",
        "Status": "Active",
        "CreateDate": "2019-07-17T15:37:27Z",
        "SecretAccessKey": < AWS Secret Access Key > ,
        "AccessKeyId": < AWS Access Key >
    }
}
</sub> 

Para facilitar a alternância entre o usuário administrador com o qual você criou o cluster e esse novo rbac-user, execute o seguinte comando para criar um script que, quando originado, define o usuário ativo como rbac-user: 

<sub>
cat << EoF > rbacuser_creds.sh
export AWS_SECRET_ACCESS_KEY=$(jq -r .AccessKey.SecretAccessKey /tmp/create_output.json)
export AWS_ACCESS_KEY_ID=$(jq -r .AccessKey.AccessKeyId /tmp/create_output.json)
EoF
</sub> 

## Mapear um usuário do IAM para K8s

Em seguida, definiremos um usuário do k8s chamado rbac-user e mapearemos para sua contraparte de usuário do IAM. Execute o seguinte para obter o ConfigMap existente e salve em um arquivo chamado aws-auth.yaml:

```bash
kubectl get configmap -n kube-system aws-auth -o yaml | grep -v "creationTimestamp\|resourceVersion\|selfLink\|uid" | sed '/^  annotations:/,+2 d' > aws-auth.yaml
```

Em seguida, anexe o mapeamento rbac-user ao configMap existente: 

<sub>
cat << EoF >> aws-auth.yaml
data:
  mapUsers: |
    - userarn: arn:aws:iam::${ACCOUNT_ID}:user/rbac-user
      username: rbac-user
EoF
</sub>

Alguns dos valores podem ser preenchidos dinamicamente quando o arquivo é criado. Para verificar tudo preenchido e criado corretamente, execute o seguinte: 

```bash
cat aws-auth.yaml
```

E a saída deve refletir esse rolearn e userarn preenchidos, semelhante a: 

<sub>
apiVersion: v1
kind: ConfigMap
metadata:
  name: aws-auth
  namespace: kube-system
data:
  mapUsers: |
    - userarn: arn:aws:iam::123456789:user/rbac-user
      username: rbac-user
</sub> 

Em seguida, aplique o ConfigMap para aplicar este mapeamento ao sistema: 

```bash
kubectl apply -f aws-auth.yaml
```

## Teste o novo usuário

Até agora, como operador de cluster, você acessava o cluster como usuário administrador. Vamos agora ver o que acontece quando acessamos o cluster como o usuário rbac recém-criado. 

Emita o seguinte comando para originar as variáveis ​​de ambiente do usuário do AWS IAM do usuário rbac-user: 

```bash
. rbacuser_creds.sh
```

Ao executar o comando acima, você definiu as variáveis ​​ambientais da AWS que devem substituir o usuário ou função admin padrão. Para verificar se substituímos as configurações de usuário padrão, execute o seguinte comando: 

```bash
aws sts get-caller-identity
```

Você deve ver algo semelhante ao abaixo, onde agora estamos fazendo chamadas de API como rbac-user: 

<sub>
{
    "Account": <AWS Account ID>,
    "UserId": <AWS User ID>,
    "Arn": "arn:aws:iam::<AWS Account ID>:user/rbac-user"
}
</sub>

Agora que estamos fazendo chamadas no contexto do rbac-user, vamos fazer rapidamente uma solicitação para obter todos os pods: 

```bash
kubectl get pods -n rbac-test
```

Você deve obter uma resposta de volta semelhante a: 

<sub>
No resources found.  Error from server (Forbidden): pods is forbidden: User "rbac-user" cannot list resource "pods" in API group "" in the namespace "rbac-test"
</sub>

Já criamos o rbac-user, então por que recebemos esse erro? 

Apenas criar o usuário não dá a ele acesso a nenhum recurso no cluster. Para conseguir isso, precisaremos definir uma role e, em seguida, vincular o usuário a essa role.  

## Criando a Role e Binding

Como mencionado anteriormente, temos nosso novo usuário rbac-user, mas ainda não está vinculado a nenhuma função. Para fazer isso, precisaremos voltar ao nosso usuário administrador padrão. 

Execute o seguinte para desmarcar as variáveis ​​ambientais que nos definem como rbac-user: 

```bash
unset AWS_SECRET_ACCESS_KEY
unset AWS_ACCESS_KEY_ID
```

Para verificar se somos o usuário administrador novamente e não mais o usuário rbac, emita o seguinte comando: 

```bash
aws sts get-caller-identity
```

A saída deve mostrar que o usuário não é mais rbac-user: 

<sub>
{
    "Account": <AWS Account ID>,
    "UserId": <AWS User ID>,
    "Arn": "arn:aws:iam::<your AWS account ID>:assumed-role/eksworkshop-admin/i-123456789"
}
</sub>

Agora que somos o usuário administrador novamente, criaremos uma função chamada pod-reader que fornece acesso de lista, obtenção e observação para pods e implantações, mas apenas para o namespace rbac-test. Execute o seguinte para criar esta role: 

<sub>
cat << EoF > rbacuser-role.yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: rbac-test
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["list","get","watch"]
- apiGroups: ["extensions","apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "watch"]
EoF
</sub>

Temos o usuário, temos a Role e agora os vinculamos com um recurso RoleBinding. Execute o seguinte para criar este RoleBinding: 

<sub>
cat << EoF > rbacuser-role-binding.yaml
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: read-pods
  namespace: rbac-test
subjects:
- kind: User
  name: rbac-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
EoF
</sub>

Em seguida, aplicamos o Role e RoleBindings que criamos: 

```bash
kubectl apply -f rbacuser-role.yaml
kubectl apply -f rbacuser-role-binding.yaml
```

## Verificando a Role e Binding

Agora que o usuário, Role e RoleBinding estão definidos, vamos voltar para rbac-user e testar. 

Para voltar ao rbac-user, emita o seguinte comando que origina os rbac-user env vars e verifica se eles foram usados: 

```bash
. rbacuser_creds.sh; aws sts get-caller-identity
```

Você deve ver a saída refletindo que você está logado como rbac-user. 

Como usuário rbac, emita o seguinte para obter pods no namespace rbac: 

```bash
kubectl get pods -n rbac-test
``` 

A saída deve ser semelhante a: 


<sub>
NAME                    READY     STATUS    RESTARTS   AGE
nginx-55bd7c9fd-kmbkf   1/1       Running   0          23h
</sub>

Tente executar o mesmo comando novamente, mas fora do namespace rbac-test: 

```bash
kubectl get pods -n kube-system
```

Você deve receber um erro semelhante a: 

<sub>
No resources found.
Error from server (Forbidden): pods is forbidden: User "rbac-user" cannot list resource "pods" in API group "" in the namespace "kube-system"
</sub>

Porque a Role à qual você está vinculado não fornece acesso a nenhum namespace diferente de rbac-test. 

## Limpando as configurações

<sub>
unset AWS_SECRET_ACCESS_KEY
unset AWS_ACCESS_KEY_ID
kubectl delete namespace rbac-test
rm rbacuser_creds.sh
rm rbacuser-role.yaml
rm rbacuser-role-binding.yaml
aws iam delete-access-key --user-name=rbac-user --access-key-id=$(jq -r .AccessKey.AccessKeyId /tmp/create_output.json)
aws iam delete-user --user-name rbac-user
rm /tmp/create_output.json
</sub>

Em seguida, remova o mapeamento rbac-user do configMap existente editando o arquivo aws-auth.yaml existente:

<sub>
data:
  mapUsers: |
    []
</sub>

E aplique o ConfigMap e exclua o arquivo aws-auth.yaml

```bash
kubectl apply -f aws-auth.yaml
rm aws-auth.yaml
```


[**Próximo >**](./4-Lab2.md)