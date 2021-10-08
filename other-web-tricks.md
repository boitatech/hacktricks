# Outros Truques da Web

### Cabeçalho Host

Várias vezes o back-end confia no **cabeçalho Host** para realizar algumas ações. Por exemplo, ele pode usar seu próprio valor como o **domínio para enviar uma redefinição de senha**. Então quando você recebe um email com um link para redefinir sua senha, o domínio usado é aquele que você coloca no cabeçalho Host. Então, você pode solicitar a redefinição de senha de outros usuário e alterar o domínio para um controlado por você para roubar os códigos de redefinição deles. [WriteUp](https://medium.com/nassec-cybersecurity-writeups/how-i-was-able-to-take-over-any-users-account-with-host-header-injection-546fff6d0f2).

### Booleanos de sessão

Algumas vezes quando você completa alguma verificação corretamente o back-end irá **adicionar um booleano com o valor "Verdadeiro" para um atributo de segurança da sua sessão**. Então, um endpoint diferente saberá se você passou com sucesso aquela verificação.
Entretanto, se você **passar a verificação** e as suas sessões forem concedidas aquele valor "Verdadeiro" no atributo de segurança, você pode tentar **acessar outros recursos** que **dependem do mesmo atributo** mas que você **não deveria ter permissões** para acessar. [WriteUp](https://medium.com/@ozguralp/a-less-known-attack-vector-second-order-idor-attacks-14468009781a).

### Funcionalidade de registro

Tente registrar um usuário existente. Tente também usando caracteres equivalentes \(pontos, vários espaçoes e Unicode\).

### Assumir o controlde de emails

Cadastre um email, antes de confirmar altere o email, então, se o novo email de confirmação for enviado para a primeiro email cadastrado, você pode assumir o controle de qualquer email. Ou se você puder habilitar o segundo email confirmando o primeiro, você também pode assumir qualquer conta.

### Acesso interno ao servicedesk de empresas que usam atlassian

{% embed url="https://nomedasuaempresa.atlassian.net/servicedesk/customer/user/login" %}

### Método TRACE

Desenvolvedores tendem a esquecer de desabilitar várias opções de debugging em ambientes de produção. Por exemplo, o método HTTP `TRACE` é projetado para fins de diagnóstico. Se habilitado, o servidor web irá responder a requisições que usam o método `TRACE` ecoando na resposta a mesma requisição que foi recebida. Este comportamento muitas vezes é inofensivo, mas ocasionalmente leva a divulgação de informação, como o nome de cabeçalhos de autenticação internos que podem ser anexados a requisições por proxies reversos.
![Image for post](https://miro.medium.com/max/60/1*wDFRADTOd9Tj63xucenvAA.png?q=20)

![Image for post](https://miro.medium.com/max/1330/1*wDFRADTOd9Tj63xucenvAA.png)