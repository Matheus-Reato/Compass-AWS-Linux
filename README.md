# Compass-AWS-Linux

# Sobre o projeto

A aplicação consiste na criação e configuração de requisitos especificados na atividade de Linux Univesp e URI

## Nota

Alguns comandos escritos terão de ser trocados por algo pessoal da sua instância/EFS/caminho que o arquivo foi criado, eles estarão entre colchetes []
Para que os comando funcionem corretamente troque os colchetes seguindo o exemplo:
```
ssh -i [caminho em que a chave SSH se encontra] ec2-user@[endereço IPv4 público]
```
```
ssh -i /root/.ssh/id_rsa.pub ec2-user@19.44.265.95
```
Os colchetes servem somente para separar o que é comando padrão e o que o usuário deve inserir de acordo com suas próprias informações!
Não se deve permanecer os colchetes [] em nenhum dos comandos abaixo quando forem executados. 

## Pré-requisitos
 * Conta na AWS
 * Distribuição Linux

# Instalação do Oracle Linux 8.7



## Passo a passo
Os comandos utilizados estarão descritos mais detalhadamente no decorrer desse arquivo e em arquivos separados para cada tópico

 * Primeiramente criei uma chave SSH pública no terminal do Linux e importei ela para dentro da AWS
 
 * Criei uma instância ec2 com Security Group abrindo as portas 22/TCP, 111/TCP/UDP, 2049/TCP/UDP, 80/TCP, 443/TCP e elastic IP conforme detalhado na atividade
 
 * Entrei dentro da instância EC2
 
 * Instalei o Apache, porém quando usava o comando curl -iv localhost ele retornva um erro 403 FORBIDDEN
 
 * Para resolver isso foi necessário criar um arquivo .html e movê-lo para dentro da pasta /var/www/html
 
 * Dentro da console AWS, entrei no serviço de EFS e criei um sistema de arquivos
 
 * Mudei o Security Group do EFS criado para o mesmo que a instância EC2 está utilizando. Para mudar essa configuração, basta entrar em detalhes de rede da EFS
 
 * Pelo terminal do Linux, montei o Amazon EFS dentro da instância EC2. A primeira vez foi necessário montar manualmente, mas depois foi possível montá-lo automaticamente colocando um comando dentro do /etc/fstab
 
 * Usei o comando df -h para verificar se o arquivo tinha sido montado com sucesso.
 
 * Precisei dar permissão ao EFS para poder criar um diretório dentro dele
 
 * Criei o diretório matheusReato
 
 * Começei a criação do script para validar se o Apache estava online e para mandar o resultado para dentro do meu diretório no EFS
 
 * Dei permissão de execução para o script
 
 * Automatizei o script para ser executado a cada 5 minutos utilizando o crontab
 


## Criando chave SSH e a importando para dentro da AWS

Em um terminal Linux digite:
```
ssh-keygen -t rsa -b 2048
```
Dê enter até a chave ser criada

Entre no diretório da chave
```
cd .ssh
```

Mostre na tela o código da chave pública
```
cat id_rsa.pub
```
Copie o texto mostrado na tela

Pelo navegador entrar na AWS no serviço de EC2

No lado esquerdo da tela em "Rede e Segurança", clicar em "Pares de chaves"

Clicar em "Ações" e em seguida em "Importar par de chaves"

Atribuir um nome para a chave 

Na caixa de texto, colar o código mostrado na tela quando foi utilizado o comando "cat id_rsa.pub"

Importar o par de chaves

Novamente dentro do terminal do Linux dar permissão para a chave SSH com o seguinte comando:
```
chmod 400 id_rsa.pub
```

## Criando um instância EC2 e entrando dentro dela

Pelo navegar, entre na AWS no serviço de EC2

Clique em "Executar instância"

Crie a instância conforme sua necessidade

Depois de criada a instância, copie o endereço IPv4 público, disponível nos detalhes da instância

Dentro do terminal do linux digite o seguinte comando:

```
ssh -i [caminho em que a chave SSH se encontra] ec2-user@[endereço IPv4 público]
```

Caso apareça alguma mensagem pedindo para digitar yes/no

Digite yes


## Subindo um Apache dentro da instância EC2

Dentro da instância EC2 digite os seguintes comandos:

```
sudo yum install httpd
```

Espere o programa terminar de baixar

```
sudo systemctl enable --now httpd.service
```

## Resolvendo erro Forbidden 403 do Apache
Caso os comandos curl abaixo retornem um HTTP 403
```
curl -iv localhost
```
ou
```
curl -Is [ip quer o usuário deseja consultar] | head -n 1
```
Digite os seguintes comandos:

```
cd /var/www
```
```
vim index.html
```
Escreva uma página de index usando html ou cole dentro do seu arquivo o seguinte html: https://github.com/Matheus-Reato/Compass-AWS-Linux/blob/main/index.html

Salve e saia do arquivo

Use o comando:
```
mv index.html html
```
Tente usar os comando de curl novamente

## Montando Amazon EFS manualmente

Primeiramente crie um EFS dentro da console da AWS

Dentro da instância EC2 digite os seguintes comandos:

```
mkdir [~/caminho que o usuário deseja criar o diretório]
```
```
sudo mount -t nfs -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport [nome de DNS do EFS]:/ [~/caminho do diretório que o usuário criou]
```
![Demonstração](https://github.com/Matheus-Reato/assets/blob/main/dns_efs.png)
```
cd [~/caminho do diretório que o usuário criou]
```
Para ter permissão de criar arquivos dentro do EFS montado use o seguinte comando:
```
sudo chmod go + rw .
```
Tente criar um diretório para teste
```
mkdir teste
```
## Montando Amazon EFS automaticamente toda vez que reiniciar a instância EC2
```
vim /etc/fstab
```
Dentro do arquivo, vá até o final dele e digite o seguinte comando: 

```
[fs-XXXXXXXX].efs.[aws-region].amazonaws.com:/ [caminho onde será montado] nfs4 nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport,_netdev 0 0
```

IMPORTANTE: Escrever exatamente dessa forma, trocando apenas as informações entre colchetes pela suas. Caso algo esteja escrito errado é possível que você não consiga mais entrar na sua instância EC2, sendo necessário a criação de uma nova.

Para informações mais detalhadas, é possível encontrar na documentação da própria AWS: https://docs.aws.amazon.com/efs/latest/ug/nfs-automount-efs.html

## Dando permissão de execução ao script

Digite o seguinte comando estando na mesma pasta que o script:

```
sudo chmod +x [nome do script]
```

## Automatizando o script para executar a cada 5 minutos
Digite o seguinte comando: 
```
crontab -e
```

Dentro do arquivo digite:
```
*/5 * * * * [caminho onde está o script] 
```
Salve e saia do arquivo

Reinicie o serviço
```
sudo systemctl restart crond.service
```
ou
```
sudo systemctl restart crond
```

# Autor

Matheus Gotardo Reato



