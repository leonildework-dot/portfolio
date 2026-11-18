# Relatório de Forense - Laboratório 1

## 1. Resumo
Através da análise do ficheiro access.log do Apache, foi possível identificar os User-Agents utilizados pelo atacante, destacando-se ferramentas de escaneamento de portas (Nmap) e enumeração de diretórios (DirBuster/gobuster). O IP de origem iniciou a varredura e gerou vários pedidos de teste no servidor.

## 2. Provas
* **Ponto de entrada:** A inspeção dos pedidos HTTP do tipo POST revelou que o atacante utilizou uma página de carregamento de ficheiros (`upload.php`) para injetar scripts ou ficheiros não autorizados no servidor web. Adicionalmente, o uso de diretórios ocultos permitiu encontrar avisos deixados por ferramentas de força bruta.
* **Persistência:** Para garantir persistência no sistema (ou seja, assegurar que conseguia voltar a entrar), o atacante inseriu uma tarefa no agendador do sistema (`/etc/crontab`). Esta tarefa configura uma *reverse shell* executada com privilégios de `root` a cada minuto, ligando-se de volta ao endereço IP externo do atacante numa porta específica.
* **Conta suspeita:** Verificou-se a existência de uma segunda conta associada ao UID 0 (a "chave mestra" do sistema), o que concede privilégios totais de administrador. A conta secundária tinha um hash de palavra-passe configurado diretamente, servindo como uma porta de entrada secundária invisível para utilizadores legítimos. 

## 3. Linha temporal

| Hora (UTC) | IP de Origem | O que aconteceu | Prova (Ficheiro e Comando) |
| :--- | :--- | :--- | :--- |
| 08:01:12 | 198.51.100.50 | Reconhecimento e varredura de portas | `access.log` (grep -a "Nmap") |
| 09:14:02 | 198.51.100.50 | Enumeração de diretórios e pastas | `access.log` (grep -ai "dirbuster") |
| 09:31:10 | 198.51.100.50 | Envio de ficheiros maliciosos via POST | `access.log` (grep -a "POST") |
| 09:45:05 | Local/Remoto | Criação de persistência e alteração de contas | `/etc/crontab` e `/etc/passwd` |

## 4. Recomendações
1. **Validação rigorosa de uploads:** Restringir e validar estritamente os tipos de ficheiros permitidos na página de submissão de uploads para evitar a execução de scripts arbitrários.
2. **Auditoria de integridade de ficheiros:** Monitorizar regularmente ficheiros críticos do sistema, como o `/etc/crontab` e o `/etc/passwd`, para detetar alterações não autorizadas ou a criação de contas com UID 0.
3. **Implementação de regras de Firewall e Fail2ban:** Bloquear endereços IP que apresentem comportamentos suspeitos de escaneamento automatizado ou múltiplas tentativas falhadas de autenticação.
