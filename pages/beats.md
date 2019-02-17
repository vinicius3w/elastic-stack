## Beats

Ainda falando de Elastick Stack, temos mais um componente que até o momento não foi comentado durante o treinamento... o __Beats__.

Existem 6 módulos do Beats para diferentes propósitos:

__Filebeat:__ Faz a leitura do arquivo da log desejada, enviando-a para o Logstash que fará a inserção no Elasticsearch.

__Metricbeat:__ Coleta as métricas do seu servidor (CPU, memória, IO, File system e etc) e os envia para o Elasticsearch.

__Packetbeat:__ Faz a coleta de métricas de tráfego de rede e você já deve imaginar para onde ele envia.

__Winlogbeat:__ Faz a coleta das logs de eventos da sua infraestrutura Windows (blé).

__Auditbeat:__ Realiza um serviço de auditoria, monitorando a atividade dos usuários e processos dos seus servidores Linux (like an _auditd_).

__Heartbeat:__ Verifica a disponibilidade dos seus serviços, através de pings ICPM, TCP, HTTP e etc.

Neste tópico, vamos focar no Filebeat...

Quando queremos monitorar servidores/aplicações externas ao nosso cluster de Elasticsearch, faz mais sentido utilizar o Filebeat do que ter o Logstash instalado em cada máquina que queremos monitorar, já que ele é muito mais leve, rápido e consome muito menos recursos computacionais.

Para este exemplo, vamos subir uma máquina virtual através do __Vagrant__ para monitorarmos uma instância de Apache em outro servidor.

__OBS:__ Se você já conhece um pouco mais da vida e está pensando em utilizar um container Docker de Apache, fique tranquilo que em breve iremos comentar sobre monitoração de containers.

__OBS²:__ Para dar sequência à este passo você não precisa necessariamente instalar o Vagrant, embora seja _extremamente recomendado_ pela facilidade que ele te trará na criação e configuração da sua máquina virtual. Mas, se quiser criar manualmente uma máquina virtual pelo Virtual Box ou VMWare e instalar o Apache nela, fique a vontade.

O Vagrant é um software que facilita a criação de máquinas virtuais através de arquivos de definição. De forma bem simples, você escreve como o seu servidor será configurado e o Vagrant faz a criação conforme está descrito no seu _Vagrantfile_. Instale o Vagrant conforme a indicação para sua distribuição no [site oficial](https://www.vagrantup.com/downloads.html). É importante que você tenha o [Virtual Box](https://www.virtualbox.org/wiki/Downloads) instalado no seu host para o Vagrant utilizá-lo como _provider_ da sua máquina virtual. Após realizar o download, faça conforme abaixo para criar o seu Vagrantfile:

```
mkdir elastic_stack/
cd elastic_stack/
vagrant init .
```

O comando `vagrant init .` fará a criação de um Vagrantfile para você editar com as configurações que você deseja para a sua máquina virtual. Mas, para facilitar ainda mais a sua vida, deixei um [Vagrantfile](/vagrant/Vagrantfile) pronto para você utilizar ! Use-o no lugar do Vagrantfile default que foi gerado e faça conforme abaixo para subir a sua _VM_ (lembrando que os comandos abaixo só irão funcionar no diretório que você utilizou o `vagrant init`):

```
vagrant up
```

Pronto, sua máquina virtual está no ar. Sim, é só isso mesmo.

Na primeira vez que subimos uma VM utilizando o Vagrant, ele realiza o download da _box_ (imagem do sistema operacional que sua VM utilizará como base), o que pode ser um processo meio demorado. Nas próximas vezes que você subir sua VM, você irá notar que o start será bem mais rápido. Verifique o status da sua VM com o comando abaixo:

```
vagrant status

Current machine states:

default                   running (virtualbox)

The VM is running. To stop this VM, you can run `vagrant halt` to
shut it down forcefully, or you can run `vagrant suspend` to simply
suspend the virtual machine. In either case, to restart it again,
simply run `vagrant up`.
```

Se a saída do seu comando foi conforme o exemplo acima, quer dizer que está tudo ok com a sua VM. Agora, logue na sua VM com o comando abaixo e realize a instalação do Apache:

```
vagrant ssh
[vagrant@elastic ~]$ sudo yum install httpd -y
```

Após a instalação, faça a subida do processo do Apache na sua VM:

```
[vagrant@elastic ~]$ sudo systemctl start httpd
```

Ótimo, vamos ver se esse Apache está funcionando ?

No nosso Vagrantfile, correlacionamos a porta __8080__ do nosso host com a porta __80__ da nossa VM (porta padrão do Apache). Sendo assim, acesse o endereço http://localhost:8080 no seu browser e verifique se foi apresentada a mensagem "It Works !". Se sim, isso significa que a nossa correlação de portas está funcionando como deveria. Se não, provavelmente a culpa é sua.

Agora vamos instalar o Filebeat para começarmos a enviar as logs da nossa nova instância de Apache para o nosso Elasticsearch:

```
[vagrant@elastic ~]$ curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-5.6.5-x86_64.rpm ; sudo rpm -vi filebeat-5.6.5-x86_64.rpm
```

O comando acima fará o download do pacote do Filebeat e fará a instalação do mesmo na sua VM. Vamos editar alguns campos do arquivo de configuração do Filebeat para que ele possa se conectar ao nosso Elasticsearch. Procure no arquivo /etc/filebeat/filebeat.yml os parâmetros abaixo e os edite para que contenham o seguinte conteúdo:

```
- input_type: log                         
  - /var/log/httpd/*_log

output.elasticsearch:
  # Array of hosts to connect to.
  hosts: ["<IP_HOST_ELASTICSEARCH>:9200"]
```

Com os parâmetros acima, estamos informando os arquivos que o Filebeat fará a leitura. Como não faremos nenhum processamento extra em nossos arquivos de log, podemos fazer o envio direto para o Elasticsearch, sem necessariamente  passar pelo Logstash. Faça a subida do processo do Filebeat com o comando abaixo:

```
[vagrant@elastic ~]$ sudo systemctl start filebeat
```

Agora, acesse o Kibana novamente e faça a criação do index gerado pelo Filebeat:

![](/gifs/filebeat.gif)

Para cada servidor que desejarmos monitorar, simplesmente repetimos os passos de instalação do Filebeat e configuração do _filebeat.yml_ com os parâmetros necessários e pronto... simples não é mesmo ?

Para mais informações à respeito de configuração e parametrização, acesse este __[link](https://www.elastic.co/guide/en/beats/filebeat/current/configuring-howto-filebeat.html)__ e veja as opções disponíveis que temos para configurarmos o nosso Filebeat.

Anterior: [Desenvolvimento vs Produção](/pages/dev_vs_prod.md) | Posterior: [Monitorando Containers](/pages/containers.md)
