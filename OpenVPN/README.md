# Configuração de VPN com autenticação por chave pública

## Cenário

> **Maquina filial:**  
>> Placas de rede:
>>> * enp0s3 - 192.168.1.10/24 - Modo bridge
>>> * enp0s8 - 192.168.10.10/24 - Rede Interna

> **Maquina Matriz:**
>> Placas de rede:
>>> * enp0s3 - 192.168.1.20/24 - Modo bridge
>>> * enp0s8 - 192.168.10.20/24 - Rede Interna

---

# MÁQUINA MATRIZ

* Na máquina matriz, instalar o pacote OpenVPN

```
# aptitude install openvpn 
```

* Acessar diretório ```/etc/openvpn``` e criar a chave de encriptação de 2048 bits, que será usada para criar a conexão

```
# openvpn --genkey --secret /etc/openvpn/chave 
```

* Criar o arquivo de configuração do servidor.

```
# > /etc/openvpn/server.conf 
```

* Editar o arquivo ```/etc/openvpn/server.conf``` com os seguinte conteúdo:

>> OBS: Deve-se utilizar para o tunelamento, uma rede distinta da suas demais placas de rede, no caso, temos uma rede Classe C no adaptador de rede enp0s3, então para o tunelamento, utilizamos uma rede Classe A.

```
dev tun 
ifconfig 10.0.0.1 10.0.0.2
secret /etc/openvpn/chave
port 5000
comp-lzo
verb 4
keepalive 10 120
persist-key
persist-tun
float
```

PARÂMETRO   |   DESCRIÇÃO
------------|------------
dev tun     |   Habilita suporte ao drive TUN/TAP
ifconfig    |   Cria o IP do servidor matriz (10.0.0.1) com suporte ao IP do servidor filial (10.0.0.2)
secret      |   Comando para chamar nossa chave criptografada e o local dela
port        |   Define a porta que a OpenVPN vai rodar
comp-lzo    |   Ativa suporte a compressão
verb        |   Nível para depuração de erros
keepalive   |   Envia um ping a cada 10 segundos sem atividade e a VPN é reiniciada depois de 120 segundos sem respostas
persist-key |   Assegura que o daemon mantenha as chaves carregadas, quando a VPN é restabelecida depois de uma queda de conexão 
verb        |   Nível para depuração de erros
keepalive   |   Envia um ping a cada 10 segundos sem atividade e a VPN é reiniciada depois de 120 segundos sem respostas
persist-key |   Assegura que o daemon mantenha as chaves carregadas, quando a VPN é restabelecida depois de uma queda de conexão
persist-tun |   Assegura que o daemon mantenha a interface tun aberta, quando a VPN é restabelecida depois de uma queda de conexão
float       |   Permite que o túnel continue aberto mesmo que o endereço IP da outra máquina mude

--- 

# MÁQUINA FILIAL

* Na máquina filial, instalar o pacote OpenVPN

```
# aptitude install openvpn 
```

* Copiar a chave do servidor matriz de forma segura (ssh sftp)

```
# scp <USER-SERVER>@<IP-SERVER>:/etc/openvpn/chave /etc/openvpn/
```

* Criar o arquivo de configuração do cliente.

```
# > /etc/openvpn/client.conf
```

* Editar ```/etc/openvpn/client.conf``` de tal maneira.

>> **Obsevação:** A única linha que muda em relação ao servidor é a terceira linha no
exmplo abaixo (**remote**)

```
dev tun 
ifconfig 10.0.0.2 10.0.0.1
remote 192.168.10.10
secret /etc/openvpn/chave
port 5000
comp-lzo
verb 4
keepalive 10 120
persist-key
persist-tun
float
```

* Detalhamento da configuração

PARÂMETRO   |   DESCRIÇÃO
------------|------------
dev tun     |   Habilita suporte ao drive TUN/TAP
ifconfig    |   Cria o IP do servidor filial (10.0.0.2) com suporte ao IP do servidor matriz (10.0.0.1)
**remote**  |   Refere-se ao IP da maquina matriz. Em nosso exemplo está sendo usado um IP local. Caso o acesso for feito pela internet, é necessário trocar por um IP externo pertencente ao link do servidor no momento 
secret      |   Comando para chamar nossa chave criptografada e o local dela
port        |   Define a porta que a OpenVPN vai rodar
comp-lzo    |   Ativa suporte a compressão
verb        |   Nível para depuração de erros
keepalive   |   Envia um ping a cada 10 segundos sem atividade e a VPN é reiniciada depois de 120 segundos sem respostas
persist-key |   Assegura que o daemon mantenha as chaves carregadas, quando a VPN é restabelecida depois de uma queda de conexão 
verb        |   Nível para depuração de erros
keepalive   |   Envia um ping a cada 10 segundos sem atividade e a VPN é reiniciada depois de 120 segundos sem respostas
persist-key |   Assegura que o daemon mantenha as chaves carregadas, quando a VPN é restabelecida depois de uma queda de conexão
persist-tun |   Assegura que o daemon mantenha a interface tun aberta, quando a VPN é restabelecida depois de uma queda de conexão
float       |   Permite que o túnel continue aberto mesmo que o endereço IP da outra máquina mude

--- 

# LEVANTANDO A VPN

## MATRIZ:

* **Na máquina matriz**, reiniciar a máquina para que possa ser reconhecido a interface de rede virtual **tun**, reponsável pelo tunelamento da rede virtual privada:

```
# init 6
```

* Com a máquina reiniciada, verificar êxito da configuração da inteface **tun**:

```
# ip a
```

>> Nas interfaces listadas, deve ter a presença de uma interface denominada **tun**, com o primeiro IP indicado no arquivo ```/etc/openvpn/server.conf``` na linha ```ipconfig```, neste caso hipotético, o IP que deve estar na interface de rede **tun** é o **10.0.0.1**

* Devemos matar o tunelamento que possivelmente foi iniciado no ato do *reboot*, e inciar novamente:

```
# killall openvpn
# openvpn --config /etc/openvpn/server.conf
```

>> Em algumas distribuições Linux, o **killall** não vem instalado por padrão, então deve instalar o seguinte pacote:

```
aptitude install psmisc 
```

* Feito isso, já podemos dizer que temos uma conexão funcional. 

## FILIAL:

* **Na máquina filial**, devemos reiniciar a máquina para que possa ser reconhecido a interface de rede virtual *tun*, reponsável pelo tunelamento da rede virtual privada:

```
# init 6
```

* Com a máquina reiniciada, verificar êxito da configuração da inteface **tun**:

```
# ip a
```

>> Nas interfaces listadas, deve ter a presença de uma interface denominada **tun**, com o primeiro IP indicado no arquivo ```/etc/openvpn/client.conf``` na linha ```ipconfig```, neste caso hipotético, o IP que deve estar na interface de rede **tun** é o **10.0.0.2**

* Devemos matar o tunelamento que possivelmente foi iniciado no ato do *reboot*, e inciar novamente:

```
# killall openvpn
# openvpn --config /etc/openvpn/client.conf
```

* Feito isso, já podemos dizer que temos uma conexão ativa. 

---

# CRIANDO AUTENTICAÇÃO POR CHAVE PÚBLICA E TESTANDO CONEXÃO

Para se autenticar por chave, a máquina **cliente** deve exportar uma chave **pública** para o **servidor**. Podemos criar uma *passprhasse* que é uma senha de aunteticação para chave, ou deixar em branco, para se autenticar se necessidade insersão de senha:

* Para criar uma chave pública, utilizamos o comando:

```
$ ssh-keygen -t rsa
```

>> O utilitário irá perguntar o caminho de hospedagem da chave, e a *passphrasse*

* Para enviar ao servidor, podemos copiar para o diretório padrão de chaves ssh (~/.ssh/), ou podemos utilizar o utlilitário que o própio ssh quem vem instalado por padrão no Linux nos proporciona, este utilitário faz a mesma coisa, porém de uma forma mais legível.

```
$ ssh-copy-id -i ~/.ssh/id_rsa.pub <USER-SERVER>@<IP-SERVER>
```

>> Como estamos enviando um arquivo ao servidor, será necessário a inserção da senha quando requirido.

* Feito isso, devemos restartar o serviço de Matriz e Filial

```
service sshd restart
```

## Teste de conexão

Para efetuar o teste, basta estabelecer uma conexão SSH pelo IP de tunelametno

* Da filiar, vamos conectar ao Matriz

```
# ssh root@10.0.0.1
```

Se tudo ocorrer bem, a conexão será estabelecida com sucesso
