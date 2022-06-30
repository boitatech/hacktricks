# Outras Tecnicas para Web

### Cabeçalho Header (Host Header)

### Host header
É extremamente comum que o back-end confie no cabeçalho Header para realizar algumas ações. Por exemplo, pode usar o valor do mesmo como **domínio para enviar um reset de senha**. Então quando você recebe um email com para alterar sua senha, o domínio que está sendo usado é o que você coloca no cabeçalho Header. Então, você pode requisitar o reset de senha de outros usuários e controlar o domínio que irá receber este reset e assim roubar o código de reset de suas senhas. [WriteUp](https://medium.com/nassec-cybersecurity-writeups/how-i-was-able-to-take-over-any-users-account-with-host-header-injection-546fff6d0f2). 

### Sessão de booleanos
As vezes quando você completa alguma verificação corretamente o back-end ira apenas **adicionar um booleano com o valor "True" a um atributo de segurança a sua sessão"**. Então, um diferente serviço ira saber se você passou no teste de verificação.

Entretanto, se você **passar o teste** e sua sessão recebe esse valor "True" no atributo de segurança, você pode tentar **acessar outros recursos** que **dependem do mesmo atributo** mas que você **não deveria ter permissão** de acessar. [WriteUp](https://medium.com/@ozguralp/a-less-known-attack-vector-second-order-idor-attacks-14468009781a).

### Função de Registro
Tente registrar algum usuário já existente. Tente também usar caracteres equivalentes como \(pontos, diversos espaços e Unicode\).

### Assumir controle de emails
Registre um email, antes de confirmá-lo troque o email, então, se o email de confirmação for enviado para o primeiro email registrado, você pode assumir controle de qualquer email. Ou se você poder trocar o segundo email confirmando primeiro, você também pode assumir controle de qualquer conta.

### Acessar serviços internos de ServiceDesk usando o Atlassian

"https://yourcompanyname.atlassian.net/servicedesk/customer/user/login"

### Método TRACE
Os desenvolvedores podem acabar esquecendo de desativar diversas opções de debug em ambientes de produção. Por exemplo, o método `TRACE` é utilizado para diagnósticos. Se habiltiado, o servidor web irá responder as requisições que usam o método `TRACE` ecoando na resposta a mesma requisição que foi recebida.
Este tipo de comportamento é geralmente inofensivo para a aplicação, mas pode ocasionalmente conduzir a um vazamento de informações, como nome de cabeçalhos internos de autenticação que podem ser adicionado as requisições através de proxys reversos.
![Image for post](https://miro.medium.com/max/1330/1*wDFRADTOd9Tj63xucenvAA.png)

