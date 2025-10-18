# Desafio-Bootcamp-Santander---Ciberseguran-a-2025
Simulando em VM ataques de Brute Force, Automação de Tentativas e Password Spraying

Primeiramente no ambiente Kali Linux, precisamos identificar as portas abertas na nossa VM Metasploitable2, para isso utilizamos o utilitário nmap, segue uma breve explicação do lab
referente aos passos realizados até atingir o objetivo do desafio.

Com o comando < arp -na >, consigo ver todos os hosts da minha rede.


<img width="588" height="122" alt="image" src="https://github.com/user-attachments/assets/92ffa4f8-53e3-4d34-8384-2a46b4c87543" />

Endereço IP do Kali: 192.162.56.102/24

<img width="681" height="82" alt="image" src="https://github.com/user-attachments/assets/0780bfb7-4625-4ec0-ae38-dce532a7eb23" />

Endereço IP do Metaspoitable2: 192.162.56.101/24

<img width="684" height="74" alt="image" src="https://github.com/user-attachments/assets/6622f76c-1ef2-4f99-a65d-d86b2877be61" />

 
Através do Kali Linux com o utilitário nmap consegui identificar o sistema operacional da VM mataspoitable e as portas abertas desse dessa máquina.
SO:

<img width="385" height="98" alt="image" src="https://github.com/user-attachments/assets/737bcf95-fab1-4fd9-835a-0689284ae01d" />

Portas Abertas:

<img width="676" height="479" alt="image" src="https://github.com/user-attachments/assets/17f0ff02-baa6-4da4-9331-04c08c26afa4" />


EXPLORANDO O SERVIÇO FTP
Como podemos ver a porta 21 do FTP está aberta e com essa informação podemos realizar ataque de força bruta para conseguir o login e senha do acesso ao servidor FTP.


<img width="303" height="123" alt="image" src="https://github.com/user-attachments/assets/1e668129-bd8a-46d3-b664-ca56521c14c5" />


Primeira etapa criei uma lista de usuário e senha para estar realizando o brute force através do utilitário medusa, essa ferramenta é capaz de realizar testes em muitos usuários e senhas
em paralelo trabalhando com múltiplas threads de processamento ao mesmo tempo, para alcançar a identificação de login e senha com um maior desempenho.


Listas criadas:


<img width="280" height="272" alt="image" src="https://github.com/user-attachments/assets/f7d7c527-6497-4345-88cb-600f46b27eed" />


Comando medusa utilizando para testar as combinações de usuários e senhas:
Medusa -h 192.168.56.101 -U users.txt -P pass.txt -M ftp -T 6


<img width="897" height="247" alt="image" src="https://github.com/user-attachments/assets/0e404d57-51db-4479-a5ae-c054b2c15d4c" />


Perceba que retornou sucesso para:
login: msfadmin
Senha: msfadmin

Após isso basta acessar o servidor FTP:

<img width="561" height="437" alt="image" src="https://github.com/user-attachments/assets/3ef1bbf6-05b5-415a-9313-5cf40fb554cb" />


Como se proteger na prática?
1.	Desativar serviços obsoletos e sem ter criptografia em seu protocolo.
2.	Usar outros serviços com criptografia como o SFTP ou SCP.
3.	Utilizar senhas fortes e imprevisíveis com políticas de troca de senha.
4.	Bloqueios de quantidade de tentativas de acesso ao serviço.
5.	Usar autenticação de multi-fator (MFA).
6.	Auditorias periódicas do time de segurança da informação
7.	Monitorar e alertar as tentativas de conexão.


========================================================================================================================================================================================



EXPLORANDO FORMULÁRIOS WEB
Como vimos com o nmap a porta 80 está aberta para nós realizarmos identificação e exploração do serviço que está rodando naquela porta.
Para isso usei o comando:
nmap -A -p 80 192.168.56.101

<img width="548" height="169" alt="image" src="https://github.com/user-attachments/assets/e6474d50-f790-4e65-8b85-0d69c68f7662" />


Ao verificar que é um Web Server Apache, podemos acessar através de um navegador:

<img width="536" height="483" alt="image" src="https://github.com/user-attachments/assets/3de7a2ff-c4af-4d89-843d-a491b004e9db" />


Temos 4 links para acessar os index desse servidor web e para o desafio acessei o DVWA:

<img width="566" height="276" alt="image" src="https://github.com/user-attachments/assets/ff401055-2a80-41ae-96c2-e949f6654b84" />

Agora temos um formulário de login e senha que podemos explorar com a ferramenta medusa com listas de usuários e senhas novamente, através do comando:

medusa -h 192.168.56.101 -U users.txt -P pass.txt -M http \
-m PAGE:’/dvwa/login.php’ \
-m FORM: ‘username=^USER^&password=^PASS^&login=login’ \
-m ‘FAIL=login failed’ -t 6 | grep -i success

Esse comando vai testar os usuários e senhas no formulário web, informando o caminho do formulário, o corpo da requisição e a resposta de falha que o servidor retorna para a ferramenta medusa.

<img width="722" height="248" alt="image" src="https://github.com/user-attachments/assets/6efcb5ef-8ad7-437c-91e1-6deabe78eb09" />

Agora com alguns logins e senhas que deram sucesso basta tentar logar no site com essas contas, após testar uma por uma a que realizou o login no site foi:
Login: admin
Senha: password

<img width="738" height="546" alt="image" src="https://github.com/user-attachments/assets/1c279cb7-7215-4e90-9c4e-9a32d281fd4c" />


Como se proteger?
1.	Implementar token CSRF para tratar esses ataques de CSRF.
2.	Cookies válidos e obrigatórios.
3.	Captcha de login.
4.	Senhas mais complexas.
5.	Implementação de MFA.


========================================================================================================================================================================================



EXPLORANDO O SERVIÇO SMB
Como vimos com o nmap a porta 445 está aberta para nós realizarmos identificação e exploração do serviço que está rodando naquela porta.
Para isso usei o comando:
nmap -A -p 445 192.168.56.101
<img width="740" height="412" alt="image" src="https://github.com/user-attachments/assets/1596bfd1-b30b-4ed9-ba8a-797d1fda5c67" />


A partir daqui podemos realizar ataque em cadeia, enumeração SMB com password spraying. O objetivo do spraying é testar senhas comuns em diversas contas de usuários a fim de
não bloquear a tentativa de login em contas individuais, ou seja, burlar o bloqueio quando o mesmo usuário tenta muitos logins.

Primeiro vamos realizar a enumeração de usuários, para encontrar contas existentes e com isso evitar de enviar alertas de tentativas de login com contas inexistentes para os sistemas
de monitoramento:

Enumerando os usuários com o comando:
enum4linux -a 192.168.56.101 | tee enum4-output.txt
 
<img width="735" height="624" alt="image" src="https://github.com/user-attachments/assets/c8d6f04f-3b37-42a1-b6d2-f8b9952ab4ba" />


Agora basta apenas criar uma lista que contêm esses usuários que inicia no usuário “games” e vai até o “uucp”. Depois criar uma lista de senhas comuns e realizar o ataque de spraying de senhas,
com a ferramenta medusa através desse comando:
medusa -h 192.168.56.101 -U smb_users.txt -P senhas-spray.txt -M smbnt -t 2 -T 50 | grep -i success
<img width="742" height="73" alt="image" src="https://github.com/user-attachments/assets/48fecd68-b722-4051-b538-1fdca35365cf" />


Conseguiu identificar a conta:
login: msfadmin
Senha: msfadmin

Agora basta logar no serviço SMB para ver os documentos salvos nesse servidor.
smbclient -L //192.168.56.101 -U msfadmin
<img width="730" height="322" alt="image" src="https://github.com/user-attachments/assets/40f3526b-7d63-4e0a-b4c5-e6472922a4d7" />


Como se proteger?
1.	Autenticação com MFA.
2.	Senhas fortes e com políticas de expiração de senha.
3.	Bloqueio de endereço IP após múltiplas tentativas de login.
4.	Monitoramento e correlacionamento de LOGs.
5.	Serviços internos não devem ser expostos na internet.
6.	Auditorias regulares.

