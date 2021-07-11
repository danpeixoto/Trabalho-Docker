# Trablho de integração contínua (Docker e Jenkins)

Esse arquivo serve como roteiro de estudos (sequência de ações/comandos que eu realizei).

## Estrutura final do projeto

	trabalho-1
    ├── apache
    │   └── Dockerfile
    ├── jenkins_home
    │   └── Dockerfile
    └── site
        ├── Todos os arquivos do site aqui


## Configuração inicial da máquina docker

É bem simples, já que, na máquina o docker está instalado. Então, apenas crie uma pasta com nome qualquer para ser a raiz do seu projeto.
No meu caso criei a pasta trabalho-1

	mkdir trabalho-1

## Criação do site e do repositório

Bom nessa parte não tem segredo, então não vou descrever nada aqui.

**LEMBRE DE DAR PELO MENOS UM COMIT NO REPO**

## Criação do container do jenkins

Para criação do jenkins primeiro crie uma pasta na raiz do seu projeto chamado `jenkis_home`.

Dentro dela crie um Dockerfile com os seguintes comandos:

	FROM jenkins/jenkins:lts-jdk11

	USER root

	COPY "$PWD"/jenkins_home/* /var/jenkins_home

Depois, crie a imagem do jenkins executando o comando abaixo na raiz do projeto:

	docker build -t jenkis -f jenkins_home/Dockerfile .

Com a imagem criada, agora você vai criar o container do jenkins. Execute o comando abaixo:

	docker run -dit --name jenkins -p 8090:8080 --restart always -v "$PWD"/jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock -v /usr/bin/docker:/usr/bin/docker jenkins




## Configurando o jenkins
Caso você tenha seguido tudo corretamente, agora você tem o jenkins rodando e pode acessar com <ip_máquina_host>:8090 no navegador

Ao abrir o navegador, o jenkins vai ser configurado e pedirá uma senha. Para conseguir essa senha você vai ter que entrar no container no arquivo especificado na sua tela. No meu caso preciso entrar nessa pasta */var/jenkins_home/secrets/initialAdminPassword*. Utilize o comando abaixo para conseguir a senha:

	docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword

Cole a senha no campo requisitado!

Selecione a primeira opção, para instalar os plugins recomendados.

Preencha os campos com as informações do seu user.

Aceite a url que foi mostrada para você. Isso irá concluir a instalação e configuração do jenkis.




## Configurando o servidor de email

No meu caso vou utilizar um gmail para enviar as notificações. E precisa habilitar o acesso a app menos seguro na conta do google.

Também é necessario ir nas configurações do gmail-> Encaminhamento e POP/IMAP e ativar os dois.

Na página inicial do jenkins vá em *Gerenciar Jenkins* e clique em *Configurar o sitema*

Ao clicar você será redirecionado para uma tela com diversas configurações. Procure *Extended E-mail Notification*

Em *Servidor SMTP* adicione *smtp.gmail.com* e em *SMTP Port* adicione 465 depois clique em *avançado*

Coloque o email e senha que você está utilizando para enviar os emails e habilite o *Use SSL*.

Depois vá até o final dessa seção e habilite *Allow sending to unregistered users*.

Por fim clique em default Triggers e marque *Always*.


## Configurando um job que faz um pull a cada dia

Na página inicial do jenkins clique em *Novo job*.

Selecione a primeira opção e de um nome qualquer, dei o nome de site (esse nome será utilizado na criação do servidor apache, não esqueça ele).

Não precisa ter uma descrição.

Em *Gerenciamento de código fonte* marque a opção git e cole a url do seu repo:

	https://github.com/danpeixoto/Trabalho-Docker.git

Em *trigger de builds* selecione *Construir periodicamente*. No campo de texto digite *@daily*.


Em *Ações de pós-build* selecione *Editable Email Notification*.

Coloque os emails que você deseja que recebam a notificação na área *Project Recipient List*.

Salve e depois clique em *Construir build* se estiver tudo ok o email vai chegar.




## Criação do container da aplicação web

Se atá aqui deu tudo certo, essa vai ser a parte mais simples!

Crie uma pasta chamada apache dentro da raiz do projeto. Ela deve conter o Dockerfile para criação da imagem do apache.

O conteúdo do Dockerfile está abaixo:

	FROM httpd:2.4	
	COPY "$PWD"/jenkins_home/workspace/site/* /usr/local/apache2/htdocs/

Após a criação do Dockerfile vá para a raiz do seu projeto e faça o build da imagem.

	docker build -t apache -f apache/Dockerfile .

O comando acima irá criar uma imagem, agora é possível criar um conteiner a partir dela. O comando para criar o container é o seguinte:

	docker run -dit --name apache -p 80:80 --restart always -v "$PWD"/jenkins_home/workspace/site:/usr/local/apache2/htdocs apache

**Agora seu site está rodando, só acessar o ip da máquina hospedeira no navegador. Também, qualquer alteração na pasta site aparecerá automaticamente no navegador**
**OBS: caso o site não atualize sozinho (o css) apertar ctrl+shif+r**
