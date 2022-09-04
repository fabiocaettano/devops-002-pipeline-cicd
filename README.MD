## PIPELINE CI/CD

Pipeline CICD é automação para entrega da aplicação.

CI: Integração Continua

CD: Entrega Continua



## Máquina Virtual na Digital Ocean

Criar um Droplet no site da Digital Ocean.

Na máquina local gerar uma chave ssh que será copiada na configuração do Droplet.

``` cli
cd ~/.ssh
ssh-keygen -t rsa -b 2048
ls ~/.ssh
cat ~/.ssh/id_rsa.pub
```

Para acessar a máquina virtual: 

``` cli
ssh root@numeroDoIp
 ```
 

## Instalar o java:

``` cli
apt-get update
apt-get install openjdk-8-jdk --yes
java -version
```

## Instalar o jenkins:

``` cli
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt install jenkins
``` 

Referência: https://www.digitalocean.com/community/tutorials/how-to-install-jenkins-on-ubuntu-20-04

## Instalar o Docker:

Ubuntu 20:
```
 curl -fsSL https://get.docker.com | bash
 systemctl status docker
 usermod -aG docker
```

Ubuntu 22:
```
# Instalar as dependências:
sudo apt install lsb-release ca-certificates apt-transport-https software-properties-common -y

# Adicionar o repositório Docker no Ubuntu:
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Adicionar o repositório Docker no Ubuntu:
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Atualizar o repositório:
sudo apt update

# Instalar Docker:
sudo apt install docker-ce

#Confirar com Y

# Verificar status:
sudo systemctl status docker

# 
usermod -aG docker jenkins
```


## Reinciar o Jenkins:

```
systemctl restart jenkins
systemctl status jenkins
```

## Configurar o Jekins com o Kubernetes

1. Acessar o site, que dará instrução para copiar uma chave.
http://ipDaMaquinaVirtual:8080


2. Copiar a chave e colar na campo da página:

```
cat /var/lib/jenkins/secrets/initialAdminPassword
```

3. A página vai sugerir a Instalação de Plugins. Clicar no botão e aguardar o términio.


4. Criar o usuário administrativo

Informar os dados e clicar em "Save and Continue".

A página irá retornar a URL do jenkings: http://142.93.194.185:8080/

Clicar em "Save and Finish".

Clicar em "Start Using Jenkins".


5. Adicionar Extensões

O próximo passo é adicionar extensões.

Ir na opção Gerenciar Jenkins >> Configuração de Sistema >> Gerenciar extensões >> Aba Disponiveis >> Pesquisar e Selecionar Docker, Docker Pipeline, Kubernetes, Kubernetes Continuos Deploy >> Clicar em "Baixar agora e instalar após reinício" >> Ir até o final da página e "marcar Reinicie o Jekins" , a página será reiniciada.

Aguardar a página ficar disponivel.

Informar as credenciais do item 4.

Checar se os Plugins foram instalados , ir na opção Gerenciar Jenkins >> Gerenciar extensões >> Aba Instalados.


6. Adicionar Credentias

Docker:
Gerenciar Jenkins >> Segurança >> Manage Credentials >> Clicar em Global >> Clicar em Add Credentials >> No campo Kind selecionar "Username With PassWord" >> No campo Username informar o ID do Docker Hub >>
No campo Password informar a senha do Docker Hub >> No campo Id e Description informar dockerhub >> Clicar em Botão Create.

Kubernetes (Arquivo Config):
Gerenciar Jenkins >> Segurança >> Manage Credentials >> Clicar em Global >> Clicar em Add Credentials >> No campo Kind selecionar "Secret File" >> Em File selecione o arquivo gerado pela Digital Ocean >> No campo Id e Description informar dockerhub >> Clicar em Botão Create.


kubeconfig:
copiar e colar o conteúdo do config



## Configurar o Cloud Provider

Conecta o Jekins ao Cluster Kubernetes.

Gerenciar Jenkings >> Segurança >> Configurar Seguranaça Global >> Agents >> Selecionar Randômico >> Botão Salvar.

Gerenciar Jenkings >> Configuração do Sistema >> Gerenciar Nós >> Configurar Nuvens  >> No campo "Adicionar uma Nuvem" Selecionar Kubernetes >> Clicar em "Kubernetes Cloud Details".

Em "Credentials" selecionar o arquivo yaml com as credenciais do Kubernetes e depois clicar em "Test Connection".

Em "Jekins Url" copiar o endereço  http://numeroIpDaMaquinaVirtual:8080/

Em "Pod Labels" , no Botão "Add Pod Label", selecionar "Pod Label" >> No campo "Key" informar "jenkins" e no campo "Value" informar "slave".

Em "Pod Retention" configuar como On Failure.

Em "Pod Template", clicar em "Add Pod Template" >> no campo "Name" informar "pod-template" >> Clicar em "Pod Template Details" >> No campo "Label" informar "kubepods" >> Em Container, clicar em "Add Container", e selecionar "Container Template", e colocar o nome "jnlp" >> no campo "Imagem" colocar "jenkins/jnlp-slave:latest" >> No campo "Work Directory" informar "/home/jenkins" >> Os campos Command e Arguments deixar em branco >> Botão Salvar.
    

## Subindo Aplicação para o DockerHub

Aplicação que será utilizada é a Pedelogo-Catalogo, desenvolvida pelo Fabricio Veronez.

https://github.com/KubeDev/pedelogo-catalogo


Git Clone da Imagem:

``` cli
git clone https://github.com/KubeDev/pedelogo-catalogo.git
```

Criar a imagem:

```
cd pedelogo-catalogo
docker build -t fabiocaettano74/api-produto:teste -f ./src/PedeLogo.Catalogo.Api/Dockerfile .
```

Checando a imagem:

```
docker image ls | grep api-produto
```


Docker push:

``` 
docker push fabiocaettano74/api-produto:teste
```

## Deploy no Kubernetes

Realizar o download  do arquivo "config" no site da Digital Ocean.

Mover o arquivo para pasta .kube.

``` cli
mv /mnt/c/Users/fabio/Downloads/k8s*.yaml /home/fabio/.kube/config
```

Checar se o cluster Kubernetes está acessivel.

``` cli
kubectl get nodes
```

No manifesto "k8s / api.yaml" foi configurada a imagem que subiu para o Docker Hub.

``` yaml
spec:            
      containers:
      - name: api
        image: fabiocaettano74/api-produto:teste
        ports:
        - containerPort: 80        
```

Realizar o Deploy:
``` cli 
kubectl apply -f ./k8s/ -R
```

Checar o deploy:
```
kubectl get all
```

Anotar o IP:
``` cli
kubectl get services
```

Testar aplicação no Browser

``` http
http://numeroDoIp/swagger
```

Deletar aplicação no Cluster, pois no proximo passo a Pipeline CI/CD que irá fazer este papel:

``` cli
kubectl delete -f ./k8s/ -R
```


## Configurando a Pipeline

Em GitHubProject, no campo Project URL informar o repositório GIT (https://github.com/fabiocaettano/kubedev-pedelogo-catalogo.git)

Em Pipeline, no campo "Definition" e selecionar "Pipeline Script From SCM" >> No campo "SCM" selecionar "GIT" >> No campo "Repository" colocar a url do git.

No campo Script Path informar Jekinsfile e clicar em "Salvar".
