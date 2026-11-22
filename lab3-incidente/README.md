Laboratório 03

Resumo executivo 
Durante a investigação forense do servidor Linux (10.128.134.49) através dos registos do sistema, foram identificadas múltiplas tentativas falhadas de autenticação remota contra várias contas a partir de um mesmo endereço IP de origem, bem como a criação de uma conta nova com privilégios administrativos (sudo) e o registo de uma flag no histórico de comandos bash. Estas evidências indicam risco de acesso não autorizado, escalonamento de privilégios e persistência no sistema. Recomenda-se restringir o acesso SSH com bloqueio de origem no firewall e autenticação por chave, auditar e rever contas com acesso sudo, e reforçar a monitorização e retenção centralizada dos registos. 

Provas — uma por pergunta das Tarefas 2, 3 e 4, no formato acima.
Tarefa 2, pergunta 1

Comando:
grep -iE "timesyncd|ntp|pool" /var/log/syslog

Resultado:
ntp.ubuntu.com

Leitura:
O serviço systemd-timesyncd tentou sincronizar o relógio da máquina com o servidor de tempo indicado no registo.

Livro:
/var/log/syslog Hora: <13:42:17>

Tarefa 2, pergunta 2

Comando:
grep -i "yama" /var/log/syslog

Resultado:becoming mindful.

Leitura:
O módulo de segurança Yama do kernel foi ativado no arranque para reforçar controlos de segurança do sistema.

Livro:
/var/log/syslog Hora: 13:41:48


Tarefa 3, pergunta 1
Comando:
grep "sshd" /var/log/auth.log | grep "Failed" | awk '{print $(NF-3)}' | sort | uniq -c | sort -nr

Resultado:
10.14.94.82

Leitura:
Um mesmo endereço IP externo registou várias tentativas falhadas de autenticação SSH contra diferentes contas no servidor.

Livro:
/var/log/auth.log Hora: <15:56:18>


Tarefa 3, pergunta 2
Comando:
grep -E "(useradd|usermod)\[" /var/log/auth.log

Resultado:
<conta oculta>

Leitura:
Foi criada uma nova conta no sistema e adicionada ao grupo sudo, conferindo-lhe privilégios administrativos.

Livro:/var/log/auth.log Hora: <18:04:10>
Tarefa 4, pergunta 1
Comando:
grep -i "unzip" /var/log/dpkg.log

Resultado:
6.0-28ubuntu4.1

Leitura:
O gestor de pacotes dpkg registou a instalação do utilitário unzip com a versão indicada no sistema.

Livro:
/var/log/dpkg.log Hora: <16:41:24>

Tarefa 4, pergunta 2
Comando:
grep -rnE "THM\{[^}]+\}|[a-zA-Z0-9_]+\{[^}]+\}" /home/*/.bash_history /root/.bash_history 2>/dev/null

Resultado:
THM{note_to_remember}

Leitura:
Foi encontrada uma flag no histórico de comandos bash de uma conta do sistema.

Livro:
.bash_history Hora: <n/a>

Mapa
Mapa da máquina: porta 22/tcp (sshd), confirmada pelos registros de falhas de autenticação SSH em /var/log/auth.log. O mapa foi recolhido no terminal com ss -tulnp. 
Porta por onde o atacante tentou entrar: 
porta 22/tcp (sshd), confirmada pelos registros de falhas de autenticação SSH em /var/log/auth.log.
Tabela de IoCs
Tipo
Valor
Onde foi encontrado
Porque é suspeito
Endereço IP
IP de origem das falhas de SSH ( 10.14.94.82)
/var/log/auth.log
Tentativas repetidas e sem sucesso de autenticação SSH em várias contas, indicando possível força bruta ou sondagem.
Conta / Privilégio
Nova conta adicionada ao grupo sudo (valor omitido no relatório por regra de segurança)
/var/log/auth.log
Criação de conta e atribuição de privilégios administrativos sudo requer validação de alteração legítima.
Artefacto / Flag
THM{note_to_remember}
.bash_history
Evidência de atividade ou anotação deixada no histórico interativo da linha de comandos.
Pacote / Versão
unzip (6.0-28ubuntu4.1)
/var/log/dpkg.log
Instalação de utilitário de extração de ficheiros que pode ser usado para extrair ferramentas ou cargas após acesso.



Linha temporal (ordenada por data e hora)
2025-08-12 16:41:24 — Instalação do pacote unzip (6.0-28ubuntu4.1) registada em /var/log/dpkg.log.
2025-08-13 13:41:48 — Inicialização do módulo de segurança Yama do kernel registada em /var/log/syslog.
2025-08-13 13:57:49 — Sincronização de relógio via systemd-timesyncd com ntp.ubuntu.com registrada em /var/log/syslog.
Data/hora das falhas SSH — Tentativas falhadas repetidas de início de sessão SSH em múltiplas contas registadas em /var/log/auth.log às 18:04:10.
Data/hora de criação / sudo — Criação de conta e adição ao grupo sudo registadas em /var/log/auth.log às 18:04:10.
Sem carimbo de hora — Registo de THM{note_to_remember} encontrado em .bash_history.


 Plano de resposta

IoC
Ação de contenção e correção
Comando de referência
Sessão do módulo
IP com falhas SSH
Bloquear o endereço IP suspeito na firewall para travar novas tentativas na porta 22.
sudo ufw deny from 10.14.94.82 to any port 22
Firewall
Acesso SSH
Desativar autenticação por palavra-passe e exigir chaves SSH, mantendo o serviço seguro.
sudo sshd -t (após PasswordAuthentication no em /etc/ssh/sshd_config e recarregar ssh)
SSH
Conta com sudo
Rever membros do grupo sudo e bloquear/remover contas não autorizadas após validação.
getent group sudo / sudo gpasswd -d <conta a rever> sudo
Contas
Histórico / Atividade (THM{note_to_remember} e unzip)
Ativar regras de auditoria para monitorizar comandos privilegiados, alterações de contas e execuções relevantes.
sudo auditctl -l (com regras configuradas em /etc/audit/rules.d/)
Auditoria


 Reflexão

Se o auth.log tivesse sido apagado: outros registos e sinais podiam ajudar a reconstruir os factos — por exemplo, /var/log/syslog (eventos do sistema e serviços), /var/log/dpkg.log ou /var/log/apt/history.log (instalação de pacotes como unzip), registos de auditoria (auditd), ligações de rede/processos ativos (ss -tulnp) e históricos de comandos (.bash_history), com a limitação de que o histórico bash não regista comandos não interativos nem tem hora por omissão. Para evitar a perda desses registos antes de um ataque, o ideal seria enviar os logs em tempo real para um servidor centralizado de logs (SIEM / rsyslog remoto) e aplicar permissões restritas e auditoria contínua.
Medida que teria impedido o ataque mais cedo: o bloqueio na firewall do IP de origem e o SSH com chaves (desativando palavras-passe) teriam travado ou dificultado logo no início as tentativas de força bruta na porta 22, antes de qualquer tentativa de escalonamento ou persistência no sistema.




