### Modelo padrao de uso de secret em Ambiente GCP e ambiente On-Premises


Este documento serve de exemplo para uso de um kubernetes em um servidor K8S on loco e no servidor GKE.

Nos exemplos abaixo, voce tera uma breve demonstracao de uso no GKE e suas configuracoes e a ultima configuracao de uso no servidor K8S on loco.



### Configuração de Conta de Serviço para Autenticação com Workload Identity no Google Cloud

Este guia fornece instruções sobre como criar uma conta de serviço e configurar a autenticação usando Workload Identity em um cluster Kubernetes.
Pré-requisitos

    gcloud SDK: Certifique-se de que o SDK do Google Cloud está instalado e autenticado no projeto.

    Permissões adequadas: Você deve ter permissões para criar contas de serviço e vincular políticas de IAM.

    Acesso ao Cluster Kubernetes: Verifique se você tem acesso ao cluster Kubernetes.

Passos para Configuração
1. Criação da Conta de Serviço

Execute o comando abaixo para criar uma conta de serviço que será usada para autenticação no Kubernetes:

```bash
gcloud iam service-accounts create my-service-account \
  --display-name="Service Account for Kubernetes Workload Identity"
```
2. Vinculação de Funções (IAM)

Adicione a conta de serviço ao projeto e atribua as permissões necessárias para acessar os segredos no Secret Manager:

```bash

gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="serviceAccount:my-service-account@PROJECT_ID.iam.gserviceaccount.com" \
  --role="roles/secretmanager.secretAccessor"
```
    Nota: Substitua PROJECT_ID pelo ID do seu projeto.

3. Configuração do Workload Identity

Permita que a conta de serviço do Kubernetes use a conta de serviço criada, atribuindo a função iam.workloadIdentityUser:

```bash

gcloud iam service-accounts add-iam-policy-binding my-service-account@PROJECT_ID.iam.gserviceaccount.com \
  --member="serviceAccount:PROJECT_ID.svc.id.goog[NAMESPACE/kubernetes-service-account]" \
  --role="roles/iam.workloadIdentityUser"
```
 Nota: Substitua:

PROJECT_ID pelo ID do seu projeto.
NAMESPACE pelo namespace Kubernetes em que a conta de serviço está localizada.
kubernetes-service-account pelo nome da conta de serviço Kubernetes que deseja utilizar.


(O proximo passo só tem necessidade quando voce esta usando um servidor K8S on loco)

4. Criação de Segredo Kubernetes

Crie um segredo no Kubernetes contendo a chave da conta de serviço. Isso permitirá que os serviços que exigem autenticação utilizem essa chave.

```bash

kubectl create secret generic gcp-key-secret --from-file=./service_account.json -n NAMESPACE
```
Nota:

Substitua NAMESPACE pelo namespace Kubernetes onde o segredo será criado.

Essa configuração assegura que a conta de serviço está adequadamente configurada para utilizar Workload Identity, permitindo que os pods no Kubernetes acessem recursos do Google Cloud de forma segura e gerenciada.