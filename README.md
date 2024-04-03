# Implementando metallb para acesso externo ao cluster K8S em HomeLab (on premises)

> Disclaimer:
> O conteúdo deste tutorial é estritamente educacional e não recomendo o uso em ambientes de produção.

Após concluir meus laboratórios chegou a hora de praticar CI/CD.

E nada melhor que o Jenkins e o ArgoCD para este estudo.

Mas no cluster Kubernetes em uma ambiente local é necessário configurar o metallb, ou outra solução como NGINX e HAProxy, para que o acesso externo funcione.

Exemplo:

- A rede da sua casa é 192.168.0.1/24 (na visão de redes de computadores ela é a Internet do ponto de vista do cluster K8S).
- A rede do POD do seu cluster Kubernetes é 10.244.0.0/16 ( required Flannel: https://github.com/flannel-io/flannel/blob/master/Documentation/kubernetes.md).
- Após implantar uma aplicação você não terá acesso, pois na rede local não existe atribuição automática de IP's para o serviço (services) LoadBalancer.

A partir deste ponto a solução é usar o metallb.

Segue abaixo:

Nota: a instalação está documentada aqui https://metallb.universe.tf/installation/

1. Criar um arquivo YAML para implantar o primeiro conjunto de endereços IP que serão usado pelo metallb no processo de atribuição:
```
# Abre um novo arquivo usando o editor "VIM"!

$ vim first-pool-metallb.yaml

# Exemplo de configuração do metallb:

apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: first-pool
  namespace: metallb-system
spec:
  addresses:
  - 192.168.10.0/24
  - 192.168.9.1-192.168.9.5
  - fc00:f853:0ccd:e799::/124

# Comando no Kubernetes que vai implantar a configuração no ambiente
$ kubectl apply -f first-pool-metallb.yaml

# Configuração do Layer 2 apontando para o recurso: "first-pool" configurado anteriormente:

$ vim layer2-metallb.yaml

# Exemplo de configuração do metallb:

apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: example
  namespace: metallb-system

# Comando no Kubernetes que vai implantar a configuração no ambiente
$ kubectl apply -f layer2-metallb.yaml
```
2. Seguindo a documentação do link abaixo iremos atribuir ao ArgoCD o services LoadBalancer usando o comando:

> Link: "https://devops.cisel.ch/deploy-argocd-and-a-first-app-on-kubernetes"

```
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```
E pronto: ArgoCD acessível reproduzindo o ambiente real onde acessamos através de um endereço IP público:

![image](https://github.com/zecaoliveira/k8s-metallb-homelab/assets/42525959/adaf2eb4-52c2-4a00-a2f0-d10c582969e7)

![image](https://github.com/zecaoliveira/k8s-metallb-homelab/assets/42525959/a3caa425-581c-4d61-8ffc-994df2e5adf3)

Agora é praticar GitOps:

> Imagem do site do Rocardo Sanchez: https://picluster.ricsanfre.com/docs/argocd/

![image](https://github.com/zecaoliveira/k8s-metallb-homelab/assets/42525959/b7bdcc5b-5e7a-4826-9061-78a4e346c676)


Fontes:

- https://metallb.universe.tf/configuration/
- https://argo-cd.readthedocs.io/en/stable/
- https://devops.cisel.ch/deploy-argocd-and-a-first-app-on-kubernetes
- https://blog.4linux.com.br/instalando-e-configurando-o-metallb-em-um-ambiente-on-premises/
- https://blog.marcolancini.it/2021/blog-kubernetes-lab-baremetal/
- https://picluster.ricsanfre.com/docs/argocd/
