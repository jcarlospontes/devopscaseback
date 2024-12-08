# Case de Avaliação para Estágio em DevOps

Este repositório tem o objetivo de apresentar o resultado da avaliação para o estágio em DevOps, seguindo as instruções disponíveis em https://github.com/goca-se/devopscaseback
O projeto consiste em uma aplicação backend em ruby que, está sendo integrada com a aplicação frontend através de uma imagem hospedada no Docker Hub disponível no repositório https://github.com/jcarlospontes/devopscasefront

A aplicação no Vercel está disponível no link https://devops-gocase-front-868zz8ry9-joao-carlos-projects-1952645c.vercel.app/

Obs: apenas o frontend está disponível, para executar a aplicação completa será necessário executar a aplicação frontend presente neste repositório.

## Objetivo

Criar um Dockerfile para executar a aplicação em um container Docker.

Criar um compose file para orquestrar tanto o frontend quanto o backend, incluindo o banco de dados que conversa com o backend.

## Requisitos

1. Ter o Docker instalado corretamente: https://docs.docker.com/get-started/get-docker/
   
3. Ter o git instalado corretamente: [https://git-scm.com/downloads](https://git-scm.com/book/pt-br/v2/Come%C3%A7ando-Instalando-o-Git)

4. Ter o docker compose instalado corretamente: https://docs.docker.com/compose/install/

## Instruções de uso

Clone o repositório atual utilizando o comando: 
```console
git clone https://github.com/jcarlospontes/devopscaseback.git
```
Acesse a pasta do projeto **devopscaseback**:
```console
cd devopscaseback
```
Execute o compose pois ele criará automaticamente a imagem do Dockerfile:
```console
sudo docker-compose up -d
```

Obs: O comando sudo é necessário pois ele criará um volume mapeado para garantir que o banco de dados persista localmente.

A aplicação estará disponível:

Frontend: http://localhost/

Backend: http://localhost:3000/api/v2/{nome do pokemon}

Documentação: http://localhost/api-docs

Também é possível utilizar a criação do frontend por Dockerfile (Sem o Docker Hub), você terá que modificar a linha 36 do compose

```console
image: joaocarlosp/front-app-gocase:latest
```

E substituirá por:

```console
    build: {Caminho para dockerfile do frontend}
```


## O que foi feito?

Para a solução, foram realizadas as seguintes etapas:
1. Criação do **Dockerfile**:
   
   - **Imagem ruby:2.7.7**: A imagem do ruby na versão 2.7.7 foi escolhida pela compatibilidade da versão da aplicação que está nessa mesma versão.

   Foi necessário instalar alguma dependências como o postgres-client para ter certeza sobre a compatibilidade da aplicação com o banco de dados.

   Foi colocado a variável de ambiente RAILS_ENV para production, indicando que se trata de uma aplicação configurada para produção. Necessário para indicar o banco de dados utilizado.

   Foi indicado a porta que será exposta a aplicação no EXPOSE 3000

   Foi indicado os comando executados ao fim da criação da imagem com o comando CMD.

2. Deleção de arquivos de chave:
   Foram deletados os arquivos credentials.yml.enc e .en.bkp pois estavam configurados para uma conexão anterior que necessita de uma secret não incluida no repositório.

3. Criação do docker-compse.yml:
   O docker compose é composto de 3 serviços, uma network e um volume.

   Foi utilizada a network em todos os serviços da aplicação no modo bridge, indicando que a rede utilizada é a mesma local e compartilhada entre todos os serviços.

   Foi utilizada a tag restart: unless-stopped para indicar que todas as aplicações só podem parar caso seja enviado um comando de parada, caso contrário haverá o restart da aplicação.

   - Serviço postgres: 
      Se trata do banco de dados escolhido para interagir com o backend.
      - Foi usada a imagem mais recente do postgres.
      - A porta foi escolhida pois é a porta padrão do postgres.
      - Foi mapeado um volume para ter a persistência de dados( note que a configuração :rw foi utilizada pois o containter precisa ter a capacidade de lida e escrita de dados).
      - Temos três variáveis de ambiente onde indicam o nome do banco de dados que deverá ser criado, o usuário e senha do banco.

   - Serviço backend: 
      Se trata do backend em ruby do repositório.
      - Foi criada a imagem a partir do Dockerfile criado.
      - A porta escolhida é a padrão para aplicações ruby.
      - O volume escolhido foi mapeado com o local de execução do container indicado no Dockerfile, para ter acesso a todos os arquivos do repositório.
      - Foi utilizado três variáveis de ambiente onde a primeira indica que o ambiente em questão é de produção, a segunda indica a string de conexão com o banco de dados e a terceira uma secret criada para adicionar segurança a aplicação.( Note que essa secret só foi utilizada pois não há uma chave de segurança configurada para a aplicação)
      - Foi utilizada a dependência do serviço de banco de dados pois a aplicação só deve iniciar quando o banco de dados estiver criado.

   - Serviço frontend: 
      Se trata do frontend em Node criado no outro repositório.
      - A imagem utilizada está hospedada no Docker Hub. A escolha foi feita pois o Docker Hub possibilita a utilização facil de imagens docker sem a necessidade de transferir o arquivo Dockerfile para outros ambientes.
      - A porta escolhida é a 80 para indicar que é uma interface de aplicação, sem a necessidade de indicar a porta no navegador.
      - O volume escolhido foi mapeado com o local de execução do container indicado no Dockerfile, para ter acesso a todos os arquivos do repositório.
      - Foi utilizado três variáveis de ambiente onde a primeira indica que o ambiente em questão é de produção, a segunda indica a string de conexão com o banco de dados e a terceira uma secret criada para adicionar segurança a aplicação.( Note que essa secret só foi utilizada pois não há uma chave de segurança configurada para a aplicação)
      - Foi utilizado um mapeamento de arquivo de configuração do nginx para o container pois foi feito um proxy reverso utilizando o front e o backend, o frontend recebe na rota /api-docs uma requisição e transfere para o backend tambem no endpoint /api-docs. Isso foi escolhido pois a aplicação possui uma documentação feita utilizando Swagger no caminho indicado.
      - Foi utilizada a dependência do serviço de banco de dados pois a aplicação só deve iniciar quando o banco de dados estiver criado.
