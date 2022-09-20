# EKS Security Workshop

[**< Voltar**](./2-Infra.md)

# Identity and Access Management

Nesta série de laboratórios, consideramos vários tópicos relacionados ao gerenciamento de identidade e acesso.

## 1.a - Introduction to Role Based Access Control (RBAC)

Este lab é reproduzido a partir do oficial [EKSWorkshop.com](https://www.eksworkshop.com/beginner/090_rbac/intro/) 

O que é RBAC?

De acordo com os documentos oficiais do kubernetes:

    O Role-based access control (controle de acesso baseado em função) (RBAC) é um método de regular o acesso a recursos de computador ou rede com base nas funções de usuários individuais dentro de uma empresa.

Os principais componentes lógicos do RBAC são:

Entity (Entidade)
Um grupo, usuário ou conta de serviço (uma identidade que representa um aplicativo que deseja executar determinadas operações (ações) e requer permissões para isso).

Resource (Recurso)
Um pod, serviço ou secret (segredo) que a entity (entidade) deseja acessar usando determinadas operações.

Role (Função)
Usado para definir regras para as ações que a entity (entidade) pode realizar em vários recursos.

Role binding (Vinculação de papéis)
Isso anexa (vincula) uma role (função) a uma entity (entidade), informando que o conjunto de regras define as ações permitidas pela entidade anexada nos recursos especificados.

Existem dois tipos de roles (funções): Role e ClusterRole e suas respectivas associações (RoleBinding, ClusterRoleBinding). Eles diferenciam entre autorização em um namespace ou em todo o cluster.

Namespace

Os namespaces são uma excelente maneira de criar limites de segurança, eles também fornecem um escopo exclusivo para nomes de objetos, como o nome 'namespace' indica. Eles devem ser usados ​​em ambientes multilocatários para criar clusters kubernetes virtuais no mesmo cluster físico.

## Objetivos para este módulo

Neste módulo, vamos explorar o k8s RBAC criando um usuário do IAM chamado rbac-user que é autenticado para acessar o cluster EKS, mas está autorizado apenas (via RBAC) a listar, obter e observar pods e implantações no ' rbac-test' namespace.

Para conseguir isso, vamos criar um usuário do IAM, mapear esse usuário para uma função do kubernetes e, em seguida, realizar ações do kubernetes no contexto desse usuário.

[**Próximo >**](./4-Lab2.md)