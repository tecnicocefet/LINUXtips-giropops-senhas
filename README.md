# LINUXtips-giropops-senhas

## Objetivo geral
Criar e otimizar uma aplicação Kubernetes segura e eficiente, utilizando o projeto giropops-senhas como base. O foco está em práticas de segurança, eficiência de recursos, monitoramento e automação.

## Pré-requisitos
Para um correto funcionamento é preciso que as ferramentas já tenham sido instaladas
- **docker**
```
apt-get update

apt-get install curl -y

curl -fsSL https://get.docker.com/ | bash
docker version
```
- **trivy**
```
curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin v0.47.0
trivy version
```
- **kind**
```
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.14.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
kind version
```
- **kubernetes**
```
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
kubectl version --client
```

## 1 - Preparação do projeto

### 1.1 - Estruturação do repositório
TODO: Adicionar print da estrutura de arquivos ao finalizar o projeto
- **app.py**: arquivo principal da aplicação python
- **Dockerfile**: arquivo com as instruções para criação da imagen de container
- **LICENSE**: termo onde constam os detalhes de como o source code do projeto pode ser manipulado, divulgado e usado por terceiros
- **README.md**: arquivo contendo detalhes do projeto
- **requirements.txt**: dependências obrigatórias para o correto funcionamento da aplicação(ex: Flask, Redis e Prometheus) 
- **static**: pasta com os arquivos estáticos do projeto(ex: css, js e imagens)
- **templates**: pasta com os arquivos de layout/frontend(ex: html)
- **tailwind.config.js**: arquivo para exportação de módulos(ex: temas, plugins)
- **k8s**: pasta contendo os manifestos yaml(cluster, deployments e services)

### 1.2 - Construção das imagens
> [!IMPORTANT]
> **O que é uma imagem?** No mundo de containers, imagens são formadas por camadas, onde dentro dessas camadas é possível encontrar todas as dependências, blibliotecas e arquivos necessários para criação do container

> [!IMPORTANT]
> **O que é um container?** É um processo isolado do sistema operacional onde utiliza recursos como cgroups, namespaces e chroot para provisionamento de seus processos. Todo container é derivado de uma imagem, ou seja, sua origem tem como base uma imagem.

### 1.3 - Otimização de imagens
> [!TIP]
> **O que é o multistage build?** É uma técnica usada para realizar o build de imagens separados em fases dentro do Dockerfile visando criar imagens menores, otimizadas e sem dependências ou arquivos desnecessários

> [!TIP]
> **O que é distroless?** Distroless é uma filosofia ou abordagem no mundo de containers onde é possível deixar as imagens somente com uma camada(apko) otimizando assim o tempo de execução do container e deixando-o mais seguro devido possuírem somente o necessário(sem gerenciador de pacotes ou shell) o que torna diferente de uma distribuição Linux.

> [!NOTE]
> **Chainguard, já ouviu falar?** Empresas como Chainguard e Google se destacam nesse contexto por serem grandes provedores desse modelo(distroless) de imagem de container. Como exemplo, podemos citar o Wolfi OS mantido pela Chainguard que é uma undistro(sem distribuição), ou seja, usa a filosofia distroless com um modelo minimalista projetado para ambientes em containers contendo somente os recursos necessários para seu funcionamento.

Os comandos abaixo irão gerar uma imagem distroless com multistage build deixando-a mais otimizada, com um tamanho menor que imagens convencionais e sem vulnerabilidades, além de fazer o push para o dockerhub
```
docker image build -t reysonbarros/giropops-senhas:1.0 .
docker login
docker image push reysonbarros/giropops-senhas:1.0
```
![image](https://github.com/reysonbarros/LINUXtips-giropops-senhas/assets/4474192/99c65fdc-a273-4677-a0d9-504b1879ed03)



### 1.4 - Verificação de segurança
> [!WARNING]
> **O que é o trivy?** É uma ferramenta para scan de vulnerabilidades em containers.
Comando executado no trivy para realizar a análise de vulnerabilidades em uma imagem:
```
trivy image reysonbarros/giropops-senhas:1.0

```
![image](https://github.com/reysonbarros/LINUXtips-giropops-senhas/assets/4474192/783e7af7-c06d-4fd5-8996-d7263119845b)


### 1.5 - Manifestos YAML
> [!IMPORTANT]
> **O que é um cluster?** Conjunto de 1 ou mais nodes dentro de uma rede

> [!IMPORTANT]
> **O que é um Pod?** Agrupamento com 1 ou mais containers compartilhando o mesmo namespace

> [!IMPORTANT]
> **O que é um Deployment?** É um objeto no Kubernetes que permite gerenciar um conjunto de Pods identificados por uma label

> [!IMPORTANT]
> **O que é um Service?** Um objeto que permite expor uma aplicação para o mundo externo

> [!IMPORTANT]
> **O que é um StatefulSet?** São objetos que servem para que possa criar aplicações que precisam manter a identidade do Pod e persistir dados em volumes locais

Criação do Cluster giropops com 1 node control-plane e 3 nodes workers
```
kind create cluster --config k8s/cluster.yaml
```
Criação do ConfigMap para o redis
```
kubectl apply -f k8s/redis-configmap.yaml
```
Criação do StatefulSet para o redis
```
kubectl apply -f k8s/redis-statefulset.yaml
```
Criação do Service para o redis
```
kubectl apply -f k8s/redis-headless-svc.yaml
```
Criação do Deployment para a aplicação giropops-senhas
```
kubectl apply -f k8s/giropops-senhas-deployment.yaml
```
Criação do Service para a aplicação giropops-senhas
```
kubectl apply -f k8s/giropops-senhas-svc.yaml
```
Listagem dos Pods
```
kubectl get pods
```
![image](https://github.com/reysonbarros/LINUXtips-giropops-senhas/assets/4474192/e3b01f17-a0bb-45b2-94c6-2b74f5f5dd40)

Listagem dos Services
```
kubectl get services
```
![image](https://github.com/reysonbarros/LINUXtips-giropops-senhas/assets/4474192/731d8a2d-87bb-4488-ae04-1a83f869d031)

Listagem do Persistent Volume Claim
```
kubectl get pvc
```
![image](https://github.com/reysonbarros/LINUXtips-giropops-senhas/assets/4474192/8caf66b1-db5d-427e-8290-66e9d2695231)


Listagem do Persistent Volume
```
kubectl get pv
```
![image](https://github.com/reysonbarros/LINUXtips-giropops-senhas/assets/4474192/4763f172-1d33-4f2d-8e83-c387cd8f9981)


Teste interno(dentro do cluster) de comunicação entre os pods do giropops-senhas e redis
```
k exec -it giropops-senhas-954766894-cfm6v -- python
>>> import redis
>>> import os
>>> redis_host = os.environ.get('REDIS_HOST')
>>> redis_password = os.environ.get('REDIS_PASSWORD')
>>> redis_port = 6379
>>> r = redis.StrictRedis(host=redis_host, port=redis_port, password=redis_password, decode_responses=True)
>>> r.ping()
>>> print('connected to redis "{}"'.format(redis_host))
```

### 1.6 - Práticas recomendadas
A definir

### 1.7 - Linting de YAML
A definir




