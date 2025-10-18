# Desafio-Bootcamp-Santander---Ciberseguran-a-2025
Simulando em VM ataques de Brute Force, Automação de Tentativas e Password Spraying

Primeiramente no ambiente Kali Linux, precisamos identificar as portas abertas na nossa VM Metasploitable2, para isso utilizamos o utilitário nmap, segue uma breve explicação do lab
referente aos passos realizados até atingir o objetivo do desafio.

Com o comando < arp -na >, consigo ver todos os hosts da minha rede.

Endereço IP do Kali: 192.162.56.102/24
Endereço IP do Metaspoitable2: 192.162.56.101/24
 
Através do Kali Linux com o utilitário nmap consegui identificar o sistema operacional da VM mataspoitable e as portas abertas desse dessa máquina.
SO: Linux 2.6.x
Portas Abertas: 21, 22, 80, 25, 5432, 445, 139, 3306

EXPLORANDO O SERVIÇO FTP
Como podemos ver a porta 21 do FTP está aberta e com essa informação podemos realizar ataque de força bruta para conseguir o login e senha do acesso ao servidor FTP.

Primeira etapa criei uma lista de usuário e senha para estar realizando o brute force através do utilitário medusa, essa ferramenta é capaz de realizar testes em muitos usuários e senhas
em paralelo trabalhando com múltiplas threads de processamento ao mesmo tempo, para alcançar a identificação de login e senha com um maior desempenho.


Listas criadas:
 

Comando medusa utilizando para testar as combinações de usuários e senhas:
Medusa -h 192.168.56.101 -U users.txt -P pass.txt -M ftp -T 6
 
Perceba que retornou sucesso para:
login: msfadmin
Senha: msfadmin

Após isso basta acessar o servidor FTP:
 

Como se proteger na prática?
1.	Desativar serviços obsoletos e sem ter criptografia em seu protocolo.
2.	Usar outros serviços com criptografia como o SFTP ou SCP.
3.	Utilizar senhas fortes e imprevisíveis com políticas de troca de senha.
4.	Bloqueios de quantidade de tentativas de acesso ao serviço.
5.	Usar autenticação de multi-fator (MFA).
6.	Auditorias periódicas do time de segurança da informação
7.	Monitorar e alertar as tentativas de conexão.


============================================================================================================================================================================================================



EXPLORANDO FORMULÁRIOS WEB
Como vimos com o nmap a porta 80 está aberta para nós realizarmos identificação e exploração do serviço que está rodando naquela porta.
Para isso usei o comando:
nmap -A -p 80 192.168.56.101
 
Ao verificar que é um Web Server Apache, podemos acessar através de um navegador:
 
Temos 4 links para acessar os index desse servidor web e para o desafio acessei o DVWA:
 
Agora temos um formulário de login e senha que podemos explorar com a ferramenta medusa com listas de usuários e senhas novamente, através do comando:
medusa -h 192.168.56.101 -U users.txt -P pass.txt -M http \
-m PAGE:’/dvwa/login.php’ \
-m FORM: ‘username=^USER^&password=^PASS^&login=login’ \
-m ‘FAIL=login failed’ -t 6 | grep -i success

Esse comando vai testar os usuários e senhas no formulário web, informando o caminho do formulário, o corpo da requisição e a resposta de falha que o servidor retorna para a ferramenta medusa.
 
Agora com alguns logins e senhas que deram sucesso basta tentar logar no site com essas contas, após testar uma por uma a que realizou o login no site foi:
Login: admin
Senha: password
 

Como se proteger?
1.	Implementar token CSRF para tratar esses ataques de CSRF.
2.	Cookies válidos e obrigatórios.
3.	Captcha de login.
4.	Senhas mais complexas.
5.	Implementação de MFA.


============================================================================================================================================================================================================



EXPLORANDO O SERVIÇO SMB
Como vimos com o nmap a porta 445 está aberta para nós realizarmos identificação e exploração do serviço que está rodando naquela porta.
Para isso usei o comando:
nmap -A -p 445 192.168.56.101
 
 

A partir daqui podemos realizar ataque em cadeia, enumeração SMB com password spraying. O objetivo do spraying é testar senhas comuns em diversas contas de usuários a fim de não bloquear a tentativa de login em contas individuais, ou seja, burlar o bloqueio quando o mesmo usuário tenta muitos logins.








Primeiro vamos realizar a enumeração de usuários, para encontrar contas existentes e com isso evitar de enviar alertas de tentativas de login com contas inexistentes para os sistemas de monitoramento:

Enumerando os usuários com o comando:
enum4linux -a 192.168.56.101 | tee enum4-output.txt
 





Agora basta apenas criar uma lista que contêm esses usuários que inicia no usuário “games” e vai até o “uucp”. Depois criar uma lista de senhas comuns e realizar o ataque de spraying de senhas, com a ferramenta medusa através desse comando:
medusa -h 192.168.56.101 -U smb_users.txt -P senhas-spray.txt -M smbnt -t 2 -T 50 | grep -i success
 

Conseguiu identificar a conta:
login: msfadmin
Senha: msfadmin

Agora basta logar no serviço SMB para ver os documentos salvos nesse servidor.
smbclient -L //192.168.56.101 -U msfadmin
 

Como se proteger?
1.	Autenticação com MFA.
2.	Senhas fortes e com políticas de expiração de senha.
3.	Bloqueio de endereço IP após múltiplas tentativas de login.
4.	Monitoramento e correlacionamento de LOGs.
5.	Serviços internos não devem ser expostos na internet.
6.	Auditorias regulares.

