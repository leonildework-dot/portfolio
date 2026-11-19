# Relatório Laboratório 02

### 1. Porque é que uma palavra-passe no SSH é mais arriscada do que um par de chaves?
A palavra-passe pode ser facilmente descoberta, reutilizada ou descoberta por insistentes tentativas num ataque de força bruta, diferente da autenticação que exige a chave encriptada, assim sendo de mais difícil acesso.

### 2. Qual das duas chaves pode ser partilhada, e qual nunca pode sair do portátil? Porquê?
A chave pública (`id_ed25519.pub`, que é a fechadura no servidor) pode ser partilhada e a chave privada (`id_xxxxx`) a qual guardamos conosco no portátil, com acesso restrito e sem nunca ir para o GitHub, pois quem a possuir pode conseguir entrar onde a chave pública estiver instalada.

### 3. Porque é que testámos a entrada com a chave (B4) antes de desligar a palavra-passe (C1)?
Porque se desligássemos a palavra-passe antes de confirmar a sua entrada no analista podíamos ficar trancados fora sem acesso ao servidor.

### 4. O que acontece se um atacante roubar o portátil do analista? O que deveria o administrador fazer no servidor?
Ele poderá conseguir entrar onde a respetiva chave pública estiver instalada. Para evitar isso, o administrador deve remover a chave pública do ficheiro `authorized_keys` do utilizador analista no servidor ou também bloquear a conta.
