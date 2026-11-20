Relatório Laboratório 02


1.Resumo 
Configurámos o servidor SSH para usar chaves EdXXXXX em vez de palavras-passe, limitámos o acesso ao utilizador analista e bloqueámos o login direto de root com controlo de firewall (ufw). Fizemos isto para prevenir ataques de força bruta e acessos não autorizados, validando primeiro a chave. As verificações e registos confirmaram que apenas o acesso autorizado funciona e tentativas excessivas são bloqueadas. 


2. Provas 01 a 06 

Prova 1 — Chave Pública e Fingerprint (SSH)
Comandos executados:
Bash
cat ~/.ssh/id_EdXXXXX.pub
ssh-keygen -lf ~/.ssh/id_EdXXXXX.pub


Descrição / Resultado:
Exibição da chave pública EdXXXXX gerada para o utilizador analista (analista-lab2).
Exibição do fingerprint (impressão digital) e do algoritmo da chave para verificação de identidade e autenticidade.


Prova 2 — Teste de Acesso por Chave e Permissões
Comandos executados:
Bash
visitante ssh -i ~/.ssh/id_EdXXXXX analista@10.10.10.1 whoami
ls -la /home/analista/.ssh


Descrição / Resultado:
Confirmação do acesso SSH sem solicitação de palavra-passe, autenticado via chave privada.
Validação das permissões estritas no servidor: diretório .ssh com permissão 700 (drwx------) e ficheiro authorized_keys com permissão 600 (-rw-------), garantindo que apenas o proprietário tem acesso.


Prova 3 — Validação e Aplicação das Regras de Hardening
Comandos executados:
Bash
sshd -t
systemctl reload ssh
sshd -T | grep -E '^(passwordauthentication|permitrootlogin|pubkeyauthentication|allowusers)'


Descrição / Resultado:
Verificação da sintaxe do ficheiro de configuração 00-hardening.conf sem erros e recarregamento do serviço SSH.
Confirmação das diretivas ativas:
PermitRootLogin no (bloqueio do login direto como root)
PasswordAuthentication no (desativação de autenticação por palavra-passe)
PubkeyAuthentication yes (autenticação exclusiva por chave pública)
AllowUsers analista (restrição de acesso apenas ao utilizador autorizado)


Prova 4 — Teste de Rejeição de Palavra-passe e Acesso Negado
Comando executado:
Bash
visitante ssh -o PubkeyAuthentication=no analista@10.10.10.1


Descrição / Resultado:
Tentativa de conexão forçando a autenticação por palavra-passe.
O servidor recusou a ligação imediatamente com a mensagem Permission denied (publickey), comprovando a eficácia do hardening.


Prova 5 — Configuração e Estado do Firewall (UFW)
Comandos executados:
Bash
ufw status verbose


Descrição / Resultado:
Confirmação do estado do firewall como active.
Regras aplicadas:
Política padrão: deny (incoming) e allow (outgoing).
Regra de taxa de limite (limit) ativada na porta 22/TCP (SSH) para mitigar ataques de força bruta.


Prova 6 — Teste de Rate Limiting e Registos de Bloqueio
Comandos executados:
Bash
Teste de múltiplas ligações rápidas/repetidas
tail -n 20 /var/log/ufw.log


Porque é que uma palavra-passe no SSH é mais arriscada do que um par de chaves? 
A palavra-passe pode ser facilmente descoberta, reutilizada ou descoberta por insistentes tentativas num ataque de força bruta, diferente da autenticação que exige a chave encriptada, assim sendo de mais difícil acesso. 


Qual das duas chaves pode ser partilhada, e qual nunca pode sair do portátil? Porquê? 
A chave pública (id_EdXXXXX.pub, que é a fechadura no servidor) pode ser partilhada e a chave privada (id_xxxxx) a qual guardamos conosco, com acesso restrito, pois quem a possuir pode conseguir entrar onde a chave pública estiver instalada. 

Porque é que testámos a entrada com a chave (B4) antes de desligar a palavra-passe (C1)? 
Porque se desligassemos a palavra-passe antes de confirmar a sua entrada no analista podíamos ficar trancados fora sem acesso ao servidor 


O que acontece se um atacante roubar o portátil do analista? O que deveria o administrador fazer no servidor? 
Ele poderá conseguir entrar onde a respectiva chave pública estiver instalada. Para evitar isso, o administrador deve remover a chave pública do ficheiro authorized_keys do utilizador analista no servidor ou também bloquear a conta. 

