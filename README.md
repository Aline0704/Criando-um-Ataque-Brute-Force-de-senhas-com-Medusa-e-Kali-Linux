# Criando-um-Ataque-Brute-Force-de-senhas-com-Medusa-e-Kali-Linux
Criando um Ataque Brute Force de senhas com Medusa e Kali Linux


# Simulação de Ataques de Força Bruta com Medusa no Kali Linux

## Descrição do Projeto
Este projeto implementa e documenta cenários simulados de ataques de força bruta usando Kali Linux como plataforma atacante e Metasploitable 2/DVWA como alvos vulneráveis. Utilizei a ferramenta Medusa para testar serviços como FTP, formulários web e SMB, em um ambiente isolado (VMs no VirtualBox com rede host-only). O foco é educacional: compreender vulnerabilidades, aplicar ferramentas e propor mitigações.

**Objetivos Alcançados:**
- Configurar ambiente controlado.
- Executar ataques simulados (força bruta, password spraying).
- Documentar processos, comandos e resultados.
- Propor medidas de prevenção.
- Compartilhar via GitHub como portfólio.

**Ferramentas Utilizadas:**
- Kali Linux (versão 2023.x).
- Medusa (instalado via `sudo apt install medusa`).
- Metasploitable 2 (VM alvo).
- DVWA (aplicação web vulnerável instalada no Metasploitable).
- VirtualBox para VMs.
- Nmap para enumeração.

**Ética e Segurança:** Todos os testes foram realizados em VMs locais, sem conexão externa. Usei snapshots para reverter mudanças. Este projeto reforça a importância da segurança defensiva.

## Configuração do Ambiente
### Passos para Replicar
1. **Instalar VirtualBox:** Baixe e instale do site oficial (gratuito).
2. **Criar VMs:**
   - VM1: Kali Linux (ISO baixada de kali.org; aloque 2GB RAM, 20GB disco, 1 CPU).
   - VM2: Metasploitable 2 (ISO de sourceforge.net; mesma alocação).
3. **Configurar Rede:** Defina como "Host-only" ou "Internal Network" para isolamento (ex.: rede "intnet").
4. **Instalar DVWA no Metasploitable:**
   - No Metasploitable, instale Apache/PHP se necessário (`sudo apt update && sudo apt install apache2 php`).
   - Baixe DVWA do GitHub (digininja/DVWA) e configure em `/var/www/html/dvwa`.
   - Acesse via navegador: `http://[IP_METASPLOITABLE]/dvwa` (usuário: admin, senha: password).
5. **Testar Conectividade:** No Kali, use `ping [IP_METASPLOITABLE]` ou `nmap -sn [IP_METASPLOITABLE]` para verificar rede.
6. **Snapshots:** Crie snapshots iniciais para reverter após testes.

**Captura de Tela:** Veja `/images/config_vm.png` (exemplo de VMs no VirtualBox).

## Ataques Simulados Executados
Todos os ataques foram feitos com wordlists simples (10-20 palavras) para demonstração. Usei flags no Medusa para limitar tentativas (ex.: `-t 1` para 1 thread, evitando sobrecarga).

### Cenário 1: Força Bruta em FTP (Metasploitable 2)
- **Descrição:** O Metasploitable roda vsftpd, vulnerável a logins fracos.
- **Enumeração Inicial:** Usei Nmap: `nmap -p 21 -sV [IP_METASPLOITABLE]` (porta 21 aberta, serviço FTP).
- **Wordlist:** Criada manualmente (ver `wordlist_ftp.txt`: admin, password, 123456, etc.).
- **Comando Executado:** `medusa -h [IP_METASPLOITABLE] -u msfadmin -P wordlist_ftp.txt -M ftp -t 1`
- **Resultado:** Acesso concedido após 3 tentativas (senha: "password"). Validação: Login via `ftp [IP_METASPLOITABLE]` e listagem de arquivos.
- **Captura de Tela:** `/images/ataque_ftp.png` (terminal mostrando sucesso).

### Cenário 2: Automação em Formulário Web (DVWA)
- **Descrição:** DVWA tem um formulário de login vulnerável (nível de segurança baixo).
- **Enumeração Inicial:** Acesse DVWA e configure para "Low" security.
- **Wordlist:** Ver `wordlist_web.txt` (admin, password, letmein, etc.).
- **Comando Executado:** `medusa -h [IP_METASPLOITABLE] -u admin -P wordlist_web.txt -M http -m FORM:"/dvwa/login.php" -m FORM-DATA:"post?username=&password=&Login=Login" -t 1`
- **Resultado:** Acesso após 5 tentativas (senha: "password"). Validação: Página de dashboard carregada.
- **Captura de Tela:** `/images/ataque_dvwa.png` (formulário e sucesso).

### Cenário 3: Password Spraying em SMB (Metasploitable 2)
- **Descrição:** SMB permite compartilhamento; enumerei usuários e testei senha comum contra múltiplos.
- **Enumeração Inicial:** Usei script `script_enum_users.sh` (baseado em Nmap: `nmap --script smb-enum-users [IP_METASPLOITABLE]`). Usuários encontrados: msfadmin, user, etc.
- **Wordlist:** Ver `wordlist_smb.txt` (password, 123456, etc.).
- **Comando Executado:** `medusa -h [IP_METASPLOITABLE] -U users_list.txt -p password -M smbnt -t 1` (password spraying com senha "password").
- **Resultado:** Acesso para usuário "msfadmin" após 2 tentativas. Validação: Conexão via `smbclient //[IP_METASPLOITABLE]/share -U msfadmin`.
- **Captura de Tela:** `/images/ataque_smb.png` (enumeração e login).

## Recomendações de Mitigação
- **Geral:** Use senhas fortes (mín. 12 caracteres, com símbolos). Implemente MFA (autenticação multifator).
- **FTP:** Desabilite FTP anônimo; use SFTP ou FTPS. Configure firewalls para bloquear portas desnecessárias.
- **Web (DVWA):** Adicione CAPTCHAs, rate limiting (ex.: fail2ban) e logs de tentativas. Atualize aplicações regularmente.
- **SMB:** Restrinja compartilhamentos; use Kerberos para autenticação. Monitore logs com ferramentas como Splunk.
- **Ferramentas Defensivas:** Instale IDS/IPS (ex.: Snort), use VPNs e eduque usuários sobre phishing.

## Reflexões e Aprendizado
Este projeto me ensinou a importância da defesa em profundidade: ataques de força bruta são simples, mas mitigáveis. Desafios: Configurar VMs sem erros de rede; adaptar comandos do Medusa para módulos específicos. Ética: Segurança é sobre proteger, não explorar. Recomendo explorar Hydra como alternativa ao Medusa para comparações.

**Recursos Utilizados:**
- Kali Linux: [kali.org](https://www.kali.org/)
- Medusa: [github.com/jmk-foofus/medusa](https://github.com/jmk-foofus/medusa)
- DVWA: [github.com/digininja/DVWA](https://github.com/digininja/DVWA)
- Nmap: [nmap.org](https://nmap.org/)

Exemplos de Arquivos Adicionais

admin
password
123456
letmein
qwerty


wordlist_web.txt: (similar, com variações como "admin123").

wordlist_smb.txt: (foco em senhas comuns como "password").

script_enum_users.sh:




#!/bin/bash
nmap --script smb-enum-users -p 445 $1  # $1 é o IP do alvo
