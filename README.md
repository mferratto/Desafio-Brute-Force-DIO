# Dio-Santander-Ciberseguranca

# Desafio de Segurança: Simulando um Ataque de Brute Force de Senhas com Medusa e Kali Linux

Este repositório documenta a execução de um desafio prático de cibersegurança, focado na utilização da ferramenta **Medusa** em um ambiente de laboratório controlado para simular ataques de força bruta contra diferentes serviços.

**Autor:** [mferratto]
**Data:** [05/11/2025]

---

### ⚠️ Aviso Ético

Todas as atividades documentadas neste projeto foram realizadas em um ambiente de laboratório isolado e controlado (VMs em rede host-only), utilizando máquinas deliberadamente vulneráveis (`Metasploitable 2` e `DVWA`). O propósito deste estudo é estritamente educacional, visando a compreensão de vetores de ataque para o desenvolvimento de estratégias de defesa e mitigação. **Nunca realize estes testes em sistemas ou redes sem autorização explícita e legal.**

---

## 1. Objetivo do Projeto

•	Compreender ataques de força bruta em diferentes serviços (FTP, Web, SMB);
•	Utilizar o Kali Linux e o Medusa para auditoria de segurança em ambiente controlado;
•	Documentar processos técnicos de forma clara e estruturada;
•	Reconhecer vulnerabilidades comuns e propor medidas de mitigação;
•	Utilizar o GitHub como portfólio técnico para compartilhar documentação e evidências.

---

## 2. Ferramentas e Ambiente

| Componente | Descrição |
| :--- | :--- |
| **Virtualizador** | Oracle VM VirtualBox |
| **Máquina Atacante** | Kali Linux (VM) |
| **Máquinas Alvo** | Metasploitable 2 (VM) e DVWA |
| **Configuração de Rede** | Rede Interna (Host-only) para isolamento |
| **Ferramenta Principal** | Medusa (ferramenta de brute force) |

### Configuração do Ambiente

1.  **Instalação:** As VMs do Kali Linux e Metasploitable 2 foram importadas para o VirtualBox.
2.  **Rede:** Ambas as VMs foram configuradas para operar na mesma rede "host-only", garantindo comunicação entre elas e isolamento da rede externa.
![Configuração Rede Kali Linux](Desafio-Brute-Force-DIO/images/config_rede_kali.png)
![Configuração Rede Metasploitable](Desafio-Brute-Force-DIO/images/config_rede_meta.png)
4.  **Identificação de IPs:**
    * IP do Kali Linux: `ip addr`
    * IP do Metasploitable 2: `ifconfig`

---

## 3. Execução dos Ataques Simulados

Para os testes, foram utilizadas as wordlists `users.txt`, `passwords.txt` e `pass_spray.txt` disponíveis na pasta `/wordlists` deste repositório.

### Cenário 1: Ataque de Força Bruta ao Serviço FTP (Metasploitable 2)

O serviço FTP (porta 21) do Metasploitable 2 é notoriamente inseguro e um alvo clássico para este tipo de ataque.

* **Alvo:** Serviço VSFTPD no Metasploitable 2.
* **Comando Executado:**

```bash
nmap -p 21,80,445 192.168.56.102
```
Verificou-se que as portas 21 (FTP), 80 (HTTP/DVWA) e 445 (SMB) estão abertas, então prosseguimos.

```bash
medusa -h <192.168.56.102> -U users.txt -P passwords.txt -M ftp -v 6
```

* **Explicação dos Parâmetros:**
    * `-h`: Host/IP do alvo.
    * `-U`: Caminho para a lista de usuários.
    * `-P`: Caminho para a lista de senhas.
    * `-M`: Módulo do serviço a ser atacado (neste caso, `ftp`).
    * `-v 6`: Nível de verbosidade (mostra tentativas em tempo real).

* **Resultado:** O Medusa identificou com sucesso as credenciais `msfadmin:msfadmin`.

* **Validação:** O acesso foi validado utilizando o cliente FTP padrão do Kali:
    ```bash
    ftp 192.168.56.102
    Name: msfadmin
    Password: msfadmin
    ftp> ls
    ```
    ![Mensagem de sucesso na identificação das credenciais para acessp ao FTP](Desafio-Brute-Force-DIO/images/ftp_success.png)

### Cenário 2: Automação de Tentativas em Formulário Web (DVWA)

O DVWA (Damn Vulnerable Web Application) possui uma página de brute force que simula um formulário de login vulnerável.

* **Configuração Prévia:**
    1.  Acessar o DVWA pelo navegador no Kali Linux.
    2.  Fazer login com as credenciais padrão (`admin:password`).
    3.  Acessar a aba "DVWA Security" e definir o nível de segurança como **`low`**.
    4.  Fazer logout.

* **Análise do Formulário:** Utilizando as ferramentas de desenvolvedor do navegador (F12), inspecionamos o formulário para identificar os nomes dos campos (`username`, `password`) e o método (`POST`).

* **Comando Executado:**

```bash
hydra -L users.txt -P passwords.txt 192.168.56.102 http-post-form "/dvwa/login.php:username=^USER^&password=^PASS^&Login=Login:Login failed" -v 6
```

* **Resultado:** Medusa encontrou as credenciais `admin:password`.
  ![Mensagem de sucesso na identificação das credenciais para acesso em formulário WEB](Desafio-Brute-Force-DIO/images/dvwa_success.png)

### Cenário 3: Password Spraying em SMB (Metasploitable 2)

Diferente do brute force, o *password spraying* utiliza uma ou poucas senhas comuns contra uma lista extensa de usuários, evitando o bloqueio de contas.

* **Enumeração de Usuários:** Antes do ataque, a ferramenta `enum4linux` foi utilizada para obter uma lista de usuários válidos do serviço SMB. Nota: optou-se por utilizar uma lista exemplificativa mais simples.
    ```bash
    enum4linux -a 192.168.56.102
    ```

* **Alvo:** Serviço Samba (SMB) no Metasploitable 2.
* **Comando Executado:**

```bash
medusa -h 192.168.56.102 -U users.txt -P pass_spray.txt -M smbnt
```

* **Explicação dos Parâmetros:**
    * `-U`: Lista de usuários a serem testados.
    * `-P`: Uma **única senha** salva no arquivo a ser testada contra todos os usuários.
    * `-M smbnt`: Módulo para o protocolo SMB.

* **Resultado:** O comando validou que a senha `msfadmin` funciona para o usuário `msfadmin`, simulando um cenário onde uma senha padrão foi reutilizada.
  ![Mensagem de sucesso na identificação das credenciais para acesso em formulário WEB](Desafio-Brute-Force-DIO/images/pass_spray_success.png)

---

## 4. Recomendações de Mitigação

Com base nos testes realizados, as seguintes medidas de segurança são recomendadas para prevenir ataques de força bruta:

1.  **Políticas de Senhas Fortes:** Exigir senhas com maior complexidade (comprimento, caracteres especiais, maiúsculas/minúsculas) e proibir o uso de senhas comuns ou que já vazaram.

2.  **Account Lockout (Bloqueio de Contas):** Implementar uma política que bloqueie temporariamente uma conta após um número específico de tentativas de login malsucedidas (ex: 5 tentativas em 10 minutos).

3.  **Autenticação Multifator (MFA):** Adicionar uma segunda camada de verificação (como um código de aplicativo ou token físico). O MFA é uma das defesas mais eficazes contra o uso de credenciais roubadas.

4.  **CAPTCHA:** Para formulários web, implementar um CAPTCHA após algumas tentativas de login para diferenciar usuários humanos de scripts automatizados.

5.  **Rate Limiting:** Limitar o número de tentativas de login que podem ser feitas a partir de um único endereço IP em um determinado período.

6.  **Monitoramento e Alertas:** Manter logs detalhados de tentativas de login (sucesso e falha) e configurar alertas para atividades suspeitas, como um grande volume de falhas de login de um mesmo IP ou para um mesmo usuário.

---

## 5. Conclusão

Este projeto demonstrou a simplicidade e eficácia de ataques de força bruta automatizados contra serviços mal configurados. A ferramenta Medusa provou ser versátil para atacar diferentes protocolos. A principal lição é que a segurança fundamental, como políticas de senhas robustas e o monitoramento de acessos, continua sendo a linha de frente crucial na defesa contra a grande maioria das ameaças de acesso não autorizado.
