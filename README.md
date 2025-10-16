# Dio-Santander-Ciberseguranca

# Desafio de Segurança: Simulando um Ataque de Brute Force de Senhas com Medusa e Kali Linux

Este repositório documenta a execução de um desafio prático de cibersegurança, focado na utilização da ferramenta **Medusa** em um ambiente de laboratório controlado para simular ataques de força bruta contra diferentes serviços.

**Autor:** [mferratto]
**Data:** [Data da Realização]

---

### ⚠️ Aviso Ético

Todas as atividades documentadas neste projeto foram realizadas em um ambiente de laboratório isolado e controlado (VMs em rede Host-Only), utilizando máquinas deliberadamente vulneráveis (`Metasploitable 2` e `DVWA`). O propósito deste estudo é estritamente educacional, visando a compreensão de vetores de ataque para o desenvolvimento de estratégias de defesa e mitigação. **Nunca realize estes testes em sistemas ou redes sem autorização explícita e legal.**

---

## 1. Objetivo do Projeto

Implementar, documentar e compartilhar um projeto prático utilizando **Kali Linux** e a ferramenta **Medusa** para simular cenários de ataque de força bruta contra os ambientes vulneráveis **Metasploitable 2** e **DVWA**, e propor medidas de prevenção eficazes.

---

## 2. Ferramentas e Ambiente

| Componente | Descrição |
| :--- | :--- |
| **Virtualizador** | Oracle VM VirtualBox |
| **Máquina Atacante** | Kali Linux (VM) |
| **Máquinas Alvo** | Metasploitable 2 (VM), DVWA (via Docker ou XAMPP) |
| **Configuração de Rede** | Rede Interna (Host-Only) para isolamento |
| **Ferramenta Principal** | Medusa (ferramenta de brute force) |

### Configuração do Ambiente

1.  **Instalação:** As VMs do Kali Linux e Metasploitable 2 foram importadas para o VirtualBox.
2.  **Rede:** Ambas as VMs foram configuradas para operar na mesma rede "Host-Only", garantindo comunicação entre elas e isolamento da rede externa.
3.  **Identificação de IPs:**
    * IP do Kali Linux: `ip a`
    * IP do Metasploitable 2: `ifconfig`

---

## 3. Execução dos Ataques Simulados

Para os testes, foram utilizadas as wordlists `users.txt` e `passwords.txt` disponíveis na pasta `/wordlists` deste repositório.

### Cenário 1: Ataque de Força Bruta ao Serviço FTP (Metasploitable 2)

O serviço FTP (porta 21) do Metasploitable 2 é notoriamente inseguro e um alvo clássico para este tipo de ataque.

* **Alvo:** Serviço VSFTPD no Metasploitable 2.
* **Comando Executado:**

```bash
medusa -h <IP_DO_METASPLOITABLE_2> -U wordlists/users.txt -P wordlists/passwords.txt -M ftp -v 6
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
    ftp <IP_DO_METASPLOITABLE_2>
    Name: msfadmin
    Password: msfadmin
    ftp> ls
    ```
    ![Exemplo de Sucesso FTP](images/1-ftp-success.png)

### Cenário 2: Automação de Tentativas em Formulário Web (DVWA)

O DVWA (Damn Vulnerable Web Application) possui uma página de brute force que simula um formulário de login vulnerável.

* **Configuração Prévia:**
    1.  Acessar o DVWA pelo navegador no Kali Linux.
    2.  Fazer login com as credenciais padrão (`admin:password`).
    3.  Acessar a aba "DVWA Security" e definir o nível de segurança como **`low`**.
    4.  Navegar para a página "Brute Force".

* **Análise do Formulário:** Utilizando as ferramentas de desenvolvedor do navegador (F12), inspecionamos o formulário para identificar os nomes dos campos (`username`, `password`) e o método (`GET`). Também identificamos um campo oculto `user_token` que precisa ser incluído na requisição.

* **Comando Executado:**

```bash
medusa -h <IP_DO_DVWA> -U wordlists/users.txt -P wordlists/passwords.txt -M web-form -m FORM:"/vulnerabilities/brute/index.php" -m GET_FORM_VARS:"username=%USER%&password=%PASS%&Login=Login&user_token=<?php echo $_SESSION['user_token']; ?>" -v 6
```
*Atualização: Em níveis baixos do DVWA, o `user_token` pode ser estático ou ausente. Se o comando acima falhar, uma versão simplificada pode ser usada, capturando o token da sessão ou omitindo-o se a lógica do servidor permitir.*

* **Resultado:** Medusa encontrou as credenciais `admin:password`.

### Cenário 3: Password Spraying em SMB (Metasploitable 2)

Diferente do brute force, o *password spraying* utiliza uma ou poucas senhas comuns contra uma lista extensa de usuários, evitando o bloqueio de contas.

* **Enumeração de Usuários (Opcional):** Antes do ataque, ferramentas como `enum4linux` podem ser usadas para obter uma lista de usuários válidos do serviço SMB.
    ```bash
    enum4linux -a <IP_DO_METASPLOITABLE_2>
    ```

* **Alvo:** Serviço Samba (SMB) no Metasploitable 2.
* **Comando Executado:**

```bash
medusa -h <IP_DO_METASPLOITABLE_2> -U wordlists/users.txt -p 'msfadmin' -M smbnt
```

* **Explicação dos Parâmetros:**
    * `-U`: Lista de usuários a serem testados.
    * `-p`: Uma **única senha** (`-p` minúsculo) a ser testada contra todos os usuários.
    * `-M smbnt`: Módulo para o protocolo SMB.

* **Resultado:** O comando validou que a senha `msfadmin` funciona para o usuário `msfadmin`, simulando um cenário onde uma senha padrão foi reutilizada.

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
