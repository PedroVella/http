# Relatório — Laboratório de Inspeção HTTP/HTTPS — Fluxo B (sem privilégio administrativo)

> **Como usar este template:** substitua os campos `[...]` pelas suas respostas,
> anexe as capturas de tela na pasta `evidencias/` e referencie-as onde indicado.
> Preserve a formatação markdown (tabelas, blocos de código) para facilitar a correção.
>
> **Observação:** toda a análise prática deste relatório é feita sobre tráfego **HTTP em texto claro**. A análise de HTTPS é teórica, baseada na fundamentação do `readme.md` do repositório.

---

## Identificação

| Campo                                       | Valor                                         |
|---------------------------------------------|-----------------------------------------------|
| Nome                                        | Pedro Azevedo Vella                           |
| RA                                          | 0050482411009                                 |
| Disciplina                                  | Redes de Computadores                         |
| Turma                                       | Noturno                                       |
| Data                                        | 09/05/2026                                    |
| Fluxo                                       | **B — Aluno sem privilégio de administrador** |
| SO utilizado                                | [Windows 11]                                  |
| Ferramenta de proxy                         | Fiddler Classic per-user                      |
| Navegador(es)                               | Chrome 124                                    |
| HTTPS-First Mode / HTTPS-Only desabilitado? | sim                                           |

---

## Atividade 1 — Primeira captura (`http://example.com`)

**Captura de tela:** <img width="1235" height="774" alt="atv1_sessao" src="https://github.com/user-attachments/assets/8d83b80b-424a-47b3-b54c-4554583d3bb7" />


**Request-line enviada:**

```http
GET http://example.com/ HTTP/1.1
```

**Status-line recebida:**

```http
HTTP/1.1 200 OK
```

### Pergunta 1.1
> Quantos cabeçalhos o navegador enviou no request? Liste-os.

**Resposta:**
7

Cabeçalhos:
- Host: example.com
- Connection: keep-alive
- Upgrade-Insecure-Requests: 1
- User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/148.0.0.0 Safari/537.36
- Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,/;q=0.8,application/signed-exchange;v=b3;q=0.7
- Accept-Encoding: gzip, deflate
- Accept-Language: pt-BR,pt;q=0.9,en-US;q=0.8,en;q=0.7

### Pergunta 1.2
> Qual foi o `Content-Length` da resposta? Se ele não apareceu, registre `Transfer-Encoding`, versão do protocolo ou outro indício observado. O corpo retornado é HTML, texto puro, JSON ou binário? Como você descobriu?

**Resposta:** Transfer-Encoding: chunked

---

## Atividade 2 — Anatomia de um GET (`http://httpbin.org/get?...`)

**Captura de tela:** <img width="1009" height="475" alt="atv2_raw" src="https://github.com/user-attachments/assets/eda02fc6-b04b-45e2-9925-89e14016602a" />

**Request-line completa:**

```http
GET http://httpbin.org/get?... HTTP/1.1
```

**Cabeçalhos-chave capturados:**

| Cabeçalho    | Valor                    |
|--------------|--------------------------|
| `Host`       | httpbin.org              |
| `User-Agent` | Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/148.0.0.0 Safari/537.36                    |
| `Accept`     | text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,/;q=0.8,application/signed-exchange;v=b3;q=0.7                   |

**Campos do JSON de resposta:**

```json
{
  "args":
    "aluno": "PEDRO_VELLA", 
    "curso": "redes"
  "headers": [Accept=text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7]
  "origin":  [187.58.19.46]
}
```

### Pergunta 2.1
> O valor do campo `origin` corresponde a qual elemento da rede? Por que normalmente não é o IP local?

**Resposta:**  Ao IP público visto pelo servidor. Normalmente não é o IP local por conta de quem entra em contato com a internet ser o roteador.

### Pergunta 2.2
> Compare o `User-Agent` enviado com o que aparece no JSON da resposta. Coincidem?

**Resposta:** Sim

### Pergunta 2.3
> Em `http://httpbin.org/headers`, liste até três cabeçalhos que o servidor vê mas **não aparecem** no Raw do request. De onde vêm? Se não encontrar três, explique por que o resultado pode variar.

**Resposta:**

| Cabeçalho visto pelo servidor | Origem provável | Observação |
|-------------------------------|-----------------|------------|
| host                          | Servidor        | [...]      |
| X-Amzn-Trace-Id               | Servidor        | [...]      |
| Upgrade-Insecure-Requests     | Servidor        | [...]      |

---

## Atividade 3 — POST e envio de formulário (`http://httpbin.org/forms/post` → `/post`)

**Captura de tela:** <img width="1010" height="523" alt="atv3_post_raw" src="https://github.com/user-attachments/assets/49100217-c36a-466e-aea8-0127ac0860b5" />


**Request-line do POST:**

```http
POST http://httpbin.org/post HTTP/1.1
```

**Cabeçalhos do request:**

| Cabeçalho        | Valor |
|------------------|-------|
| `Content-Type`   | application/json |
| `Content-Length` | 1189 |

**Corpo completo do request:**

```custname=Pedro&custtel=13+99999+9999&custemail=vellaaurudo%40gmail.com&size=large&topping=bacon&topping=cheese&topping=mushroom&delivery=12%3A45&comments=vem+rapido
```

**Trecho do JSON de resposta (campo `form`):**

```json
"form": {
    "comments": "vem rapido", 
    "custemail": "vellaaurudo@gmail.com", 
    "custname": "Pedro", 
    "custtel": "13 99999 9999", 
    "delivery": "17:33", 
    "size": "large", 
    "topping": [
      "bacon", 
      "cheese", 
      "mushroom"
    ]
}
```

### Pergunta 3.1
> Qual o formato do corpo? Como esse formato codifica caracteres especiais (espaço, acentos)?

**Resposta:** application/x-www-form-urlencoded, caracteres fora do ASCII são convertidos para bytes UTF-8 e depois codificados em hexadecimal com %.

### Pergunta 3.2
> Comparando **Request → WebForms** e **Request → Raw**: qual das duas corresponde literalmente aos bytes enviados no socket TCP?

**Resposta:** RAW

### Pergunta 3.3 — Composer
> Envie manualmente via Composer um `POST` para `http://httpbin.org/post` com JSON. Registre a resposta. Qual campo do JSON confirma que o servidor interpretou o JSON?

**Captura de tela:** <img width="1010" height="523" alt="atv3_post_raw" src="https://github.com/user-attachments/assets/f22138fe-c001-4ce4-8d99-130d4428c098" />


**Response JSON (trecho relevante):**

```json
{
  "args": {}, 
  "data": "", 
  "files": {}, 
  "form": {}, 
  "headers": {
    "Content-Length": "0", 
    "Host": "httpbin.org", 
    "User-Agent": "Fiddler", 
    "X-Amzn-Trace-Id": "Root=1-69ff3f5b-2d2a075d3cbb0a7d397fcd5f"
  }, 
  "json": null, 
  "origin": "187.58.19.47", 
  "url": "http://httpbin.org/post"
}
```

**Resposta:** no content-type ele afirma ser um json

---

## Atividade 4 — Catálogo de status codes (`http://httpbin.org/...`)

**Captura de tela (lista do Fiddler com as 7 sessões):** <img width="563" height="351" alt="atv4_lista" src="https://github.com/user-attachments/assets/f29f134f-fe8f-471b-864c-1e47b4eebbfe" />


| # | Método | URL | Status-line | `Content-Length` / `Transfer-Encoding` | Body presente? |
|---|--------|-----|-------------|-----------------------------------------|----------------|
| 1 | GET    | `http://httpbin.org/status/200` | 200 | 0 | não |
| 2 | GET    | `http://httpbin.org/redirect-to?status_code=301&url=/get` | 301 | 0 | não |
| 3 | GET    | `http://httpbin.org/status/404` | 404 | 0 | não |
| 4 | GET    | `http://httpbin.org/status/418` | 418 | 138 | sim |
| 5 | GET    | `http://httpbin.org/status/500` | 500 | 0 | não |
| 6 | GET    | `http://httpbin.org/status/503` | 503 | 0 | não |
| 7 | GET    | `http://httpbin.org/cache` com `If-Modified-Since` | Gzip. | 630 | não |

### Pergunta 4.1
> Em qual dos status o corpo está ausente/tamanho zero? Isso é obrigatório pela especificação ou depende do servidor?

**Resposta:** 200, 301, 500, 503 e 304. depende do servidor

### Pergunta 4.2
> No `301`, qual cabeçalho da resposta informa para onde ir? O que aconteceria se estivesse ausente?

**Resposta:** o cabeçalho informa o destino do redirecionamento sem ele o navegador nao saberia para onde redirecionar o user

### Pergunta 4.3
> Diferença semântica entre `200`, `304` e `404` do ponto de vista do cache do navegador.

**Resposta:** Não sei responder essa

---

## Atividade 5 — Identificação de cabeçalhos (`http://httpbin.org/response-headers?...` + `/gzip`)

**Captura de tela (Inspectors → Headers):** <img width="671" height="307" alt="atv5_headers" src="https://github.com/user-attachments/assets/617a09a0-23e2-4d1f-8396-9338939c27e8" />


| Cabeçalho                    | Req/Resp | Valor capturado | Função em uma frase |
|------------------------------|----------|------------------|----------------------|
| `Host`                       | REQ    | httpbin.org            | [...]                |
| `User-Agent`                 | REQ    | User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/148.0.0.0 Safari/537.36            | [...]                |
| `Accept`                     | REQ    | :text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,/;q=0.8,application/signed-exchange;v=b3;q=0.7            | [...]                |
| `Accept-Encoding`            | REQ    | gzip, deflate            | [...]                |
| `Cookie`                     | REQ    | Cookie: teste=1            | [...]                |
| `Server`                     | RES    | gunicorn/19.9.0            | [...]                |
| `Content-Type`               | RES    | application/json            | [...]                |
| `Content-Encoding`           | RES    | gzip            | [...]                |
| `Set-Cookie`                 | RES    | teste=1            | [...]                |
| `Cache-Control`              | RES    | max-age=3600            | [...]                |
| `Strict-Transport-Security`  | Não esperado em HTTP — ver Pergunta 5.3 | — | — |

### Pergunta 5.1
> `Content-Encoding: gzip`/`br` apareceu? Compare `Content-Length`, quando presente, com o conteúdo visível. O que explica a diferença?

**Resposta:**  Sim, geralmente gzip. O Content-Length (compactado) será muito menor que o tamanho no TextView (descompactado), pois o Fiddler descompacta o arquivo para leitura humana.

### Pergunta 5.2
> Cliente envia `Accept: application/json` mas o recurso só existe em `text/html`. Qual status code esperar?

**Resposta:** O código é 406 Not Acceptable. Significa que o servidor tem o recurso, mas não no formato que o cliente exigiu.

### Pergunta 5.3
> `Strict-Transport-Security` apareceu nas respostas HTTP? Por que esse cabeçalho está ausente neste fluxo? (Consulte a RFC 6797.) Qual é seu papel contra downgrades para HTTP puro?

**Resposta:** Ausente. O HSTS (Strict-Transport-Security) só é válido em conexões HTTPS (RFC 6797). Ele serve para impedir ataques de SSL Stripping, forçando o navegador a nunca usar HTTP puro naquele domínio.

---

## Atividade 6 — HTTP vs HTTPS (análise sem decriptação)

**Captura de tela HTTP (`neverssl.com`):** <img width="474" height="660" alt="atv6_http" src="https://github.com/user-attachments/assets/473dee04-e72d-434c-a63e-c470f7f5d445" />

**Captura de tela HTTPS (`https://httpbin.org/get`, apenas CONNECT):** <img width="476" height="654" alt="atv6_https" src="https://github.com/user-attachments/assets/74c8a4be-1b50-4371-8fb6-b6fa27a03ea6" />


### Pergunta 6.1
> Que método HTTP aparece na sessão do `https://httpbin.org/get`? O que ele faz e por que existe?

**Resposta:** Método CONNECT. Ele serve para criar um "túnel" TCP através de um proxy. O proxy apenas repassa os bytes criptografados entre o navegador e o servidor sem conseguir ler o conteúdo.

### Pergunta 6.2
> Tabela comparativa dos campos visíveis ao Fiddler em cada caso:

| Campo                          | Visível em HTTP? | Visível em HTTPS (sem decriptação)? |
|--------------------------------|------------------|-------------------------------------|
| Método                         | Sim            | Sim                               |
| URL completa (path + query)    | Sim            | Não                               |
| Cabeçalhos de request          | Sim            | Não                               |
| Corpo de request               | Não            | Não                               |
| Status code                    | Sim            | Sim                               |
| Cabeçalhos de response         | Sim            | Sim                               |
| Corpo de response              | Sim            | Sim                               |
| Host (via SNI, no `CONNECT`)   | Sim            | Sim                               |
| IP e porta de destino          | Não            | Sim                               |

### Pergunta 6.3 (teórica)
> O que você **veria** no Fiddler se tivesse privilégio de administrador e pudesse habilitar *Decrypt HTTPS traffic*? Indique telas/abas e justifique por que essa inspeção exige a instalação de um certificado raiz.

**Resposta:** Com decriptação, você veria tudo na aba Inspectors -> Raw (texto claro). Isso exige um certificado raiz porque o Fiddler faz um ataque de Man-in-the-Middle (MitM) consentido, gerando certificados falsos para os sites e assinando-os com sua própria CA que o sistema passou a confiar

### Pergunta 6.4
> Por que a técnica de decriptação dos *debugging proxies* **não** funcionaria contra um usuário se um atacante a tentasse sem instalar o certificado?

**Resposta:**  Sem a instalação do certificado, o navegador exibiria um erro de segurança gravíssimo ("Sua conexão não é particular"), bloqueando o acesso. A técnica depende da cooperação do usuário para confiar na autoridade certificadora do atacante/proxy.

---

## Atividade 7 — Cookies e sessão (`http://httpbin.org/cookies/...`)

**Captura de tela da sequência:** <img width="907" height="375" alt="atv7_cookies" src="https://github.com/user-attachments/assets/9efe59f9-92d7-42ca-809d-a16c04b0b2db" />


| # | URL | `Set-Cookie` recebido | `Cookie` enviado |
|---|-----|-----------------------|-------------------|
| 1 | `/cookies/set?...`       | disciplina=redes; Path=/ e professor=claudio.; Path=/ | [nenhum / ...] |
| 2 | `/cookies` (1ª visita)   | [...] | disciplina=redes; professor=claudio.          |
| 3 | `/cookies` (reload 1)    | [...] | disciplina=redes; professor=claudio.          |
| 4 | `/cookies` (reload 2)    | [...] | disciplina=redes; professor=claudio.          |

### Pergunta 7.1
> `Set-Cookie` aparece uma vez ou em toda requisição? Justifique.

**Resposta:**  O Set-Cookie aparece apenas na resposta do servidor para criar o cookie. Nas requisições seguintes, o navegador envia o cabeçalho Cookie (sem o "Set-").

### Pergunta 7.2
> Que atributos o `Set-Cookie` trouxe? Explique cada um presente. Para atributos não observados, registre `não observado`.

> **Nota:** o httpbin define cookies mínimos — apenas o atributo `Path=/` estará presente. Para cada atributo ausente, registre **não observado** e explique o comportamento padrão do navegador na sua ausência (ex.: sem `Expires`/`Max-Age` → cookie de sessão; sem `Secure` → pode ser enviado por HTTP; sem `SameSite` → o navegador aplica a política padrão da versão em uso).

**Resposta:** Path=/: O cookie vale para todo o site.Expires/Max-Age: (Não observado) Padrão: cookie de sessão (apaga ao fechar o browser).Secure: (Não observado) Padrão: cookie pode ser enviado via HTTP inseguro.HttpOnly: (Não observado) Padrão: JavaScript pode ler o cookie.SameSite: (Não observado) Padrão: varia por browser (Lax no Chrome moderno).

| Atributo  | Valor | Função | Observado? |
|-----------|-------|--------|------------|
| `Path`    | `/`   | Define em quais caminhos do site o cookie será enviad  | Sim        |
| `Domain`  | —     | Limita quais domínios podem receber o cookie; sem ele, usa o domínio atual  | não observado |
| `Expires` | —     | Define data de expiração; sem ele, o cookie é de sessão (some ao fechar o navegador)  | não observado |
| `Max-Age` | —     | Define tempo de vida em segundos; sem ele, comportamento de sessão  | não observado |
| `Secure`  | —     | Só envia o cookie em HTTPS; sem ele, pode ser enviado também em HTTP  | não observado |
| `HttpOnly`| —     | Impede acesso via JavaScript (proteção contra XSS); sem ele, JS pode ler o cooki  | não observado |
| `SameSite`| —     | Controla envio em requisições cross-site (proteção contra CSRF); sem ele, navegador usa política padrão (geralmente Lax)  | não observado |

### Pergunta 7.3
> O atributo `Secure` pode aparecer num cookie recebido por HTTP puro? Qual seria o comportamento esperado? Relacione com o fato de que todo o tráfego desta atividade é visível em texto claro.

**Resposta:** O servidor pode enviar, mas é inútil. (b) O navegador ignora um cookie Secure se recebido via HTTP puro (RFC 6265bis), pois o canal é inseguro e o segredo já pode ter sido interceptado.

### Pergunta 7.4
> Na aba **Inspectors → Cookies**, o cookie armazenado coincide com o campo `cookies` do JSON?

**Resposta:** Sim, devem coincidir, pois o JSON do httpbin reflete exatamente o que ele recebeu no cabeçalho Cookie.



---

## Atividade 8 — Manipulação com breakpoints

> Esta atividade usa os breakpoints interativos do Fiddler Classic.

**Captura de tela da edição do User-Agent:** <img width="907" height="428" alt="atv8_ua_edit" src="https://github.com/user-attachments/assets/48b05a1a-0204-4223-b36a-b52e491b8e25" />


**JSON de resposta após edição:**

```json
{
  "user-agent": "[valor forjado]"
}
```

### Pergunta 8.1
> O servidor pode detectar que o `User-Agent` foi forjado? Discuta.

**Resposta:**  Não de forma infalível. O servidor confia no que o cliente envia, mas pode usar técnicas de fingerprinting (analisando a ordem dos headers ou comportamento TCP) para suspeitar de forja.

### Pergunta 8.2
> Após editar a status-line de `200 OK` para `404 Not Found`, o que o navegador exibe? Comente o papel do proxy como MITM.

**Captura de tela:** <img width="907" height="428" alt="atv8_ua_edit" src="https://github.com/user-attachments/assets/f1dac39f-b904-4f80-a6e8-aabe055f58d0" />


**Resposta:** O servidor envia 200, você altera para 404 no Fiddler. O navegador exibirá a página de erro "404 Not Found", provando que o proxy tem controle total sobre a percepção do cliente (Man-in-the-Middle).

### Pergunta 8.3
> Confirme que todos os breakpoints foram desabilitados.

- [ ] Breakpoints desabilitados ao final (Shift+F11)

---

## Atividade 9 — Redirecionamento HTTP → HTTPS

**Captura de tela:** <img width="903" height="440" alt="atv9_redir" src="https://github.com/user-attachments/assets/3ee74874-a6e6-417a-90b1-2db8ad870d9d" />


**Status-line da resposta a `http://httpbin.org/redirect-to?status_code=301&url=https%3A%2F%2Fhttpbin.org%2Fget`:**

```http
[[HTTP/1.1 301 MOVED PERMANENTLY]]
```

**Cabeçalho `Location` da resposta:**

```
Location: [https://httpbin.org/get]
```

### Pergunta 9.1
> Código de status e cabeçalho que direcionaram o navegador para `https://`.

**Resposta:** Status 302 Found ou 301 Moved Permanently. O cabeçalho é o Location: https://....

### Pergunta 9.2
> Além do redirecionamento 3xx, qual outro mecanismo/cabeçalho faz o navegador passar a forçar HTTPS em visitas futuras? Cite a RFC.

**Resposta:** O mecanismo é o HSTS. Cabeçalho Strict-Transport-Security, definido pela RFC 6797.

### Pergunta 9.3
> Se esse cabeçalho fosse enviado por uma resposta servida via HTTP puro, o navegador deveria obedecer? Justifique com base na RFC.

**Resposta:**  Não. O navegador deve ignorar o HSTS se recebido via HTTP (RFC 6797, Seção 8.1). Motivo: um atacante na rede poderia injetar um cabeçalho HSTS falso com tempo zero para desativar a segurança ou causar negação de serviço. Confiança só existe após o túnel TLS ser estabelecido.



### 7. Impacto prático de `Cache-Control: no-store`.

[Proíbe qualquer armazenamento de dados, forçando o download integral do servidor a cada acesso para garantir privacidade máxima.]

[resposta]

### 8. Como um debugging proxy decifra HTTPS sem violar a criptografia, e por que isso exige cooperação do usuário (e por que, justamente, você não pôde executar essa etapa)?

Funciona como MITM com certificado raiz instalado pelo usuário; sem isso, o navegador bloqueia por falta de confiança, por isso exige cooperação e não foi possível executar sem privilégios.

[resposta]

### 9. Exemplo de cabeçalho de request que o navegador envia automaticamente, sem a página pedir.

User-Agent, enviado automaticamente pelo navegador.

[resposta]

### 10. Se fosse automatizar parte da inspeção mantendo o Fiddler como proxy, que abordagem usaria? Por quê?

Usaria filtros, regras e exportação/logs (ou FiddlerScript) para capturar e analisar tráfego em massa

[resposta]

### 11. (Exclusiva do Fluxo B) Três cabeçalhos de segurança que não aparecem ou não fazem sentido em respostas HTTP puro. Para cada um, o que aconteceria se enviado por um servidor HTTP? (Cite RFC 6797 para HSTS.)

**Resposta:**

| Cabeçalho | Comportamento esperado sobre HTTP | Referência |
|-----------|-----------------------------------|-----------|
| Strict-Transport-Security (HSTS)     | É ignorado pelo navegador. Conforme a RFC 6797, o HSTS só é válido se enviado via conexão segura (HTTPS).                             | RFC 6797     |
| Upgrade-Insecure-Requests     | Embora enviado pelo cliente, um servidor HTTP puro não conseguiria forçar o upgrade para uma conexão segura inexistente.                             | W3C CSP     |
| Expect-CT     | Não teria efeito prático, pois serve para monitorar a conformidade de Certificados Digitais (Certificate Transparency), inexistentes em HTTP puro.                             | RFC 9163     |

---

## Reflexão final (opcional, até 10 linhas)

> O que você aprendeu que não conhecia antes deste laboratório? Há algum
> cabeçalho, código de status ou comportamento que passou a olhar com
> mais atenção? Alguma dificuldade que recomendaria evitar para a próxima turma?

[reflexão]

---

## Encerramento — justificativa de segurança (Fluxo B)

**Parágrafo: por que a remoção de certificado é dispensável neste fluxo e por que seria obrigatória para o aluno administrador:**

[redigir, em até 5 linhas, com base na seção 4.6 do readme.md]

- [ ] HTTPS-First Mode / HTTPS-Only Mode reabilitado no navegador
- [ ] Fiddler fechado (porta de proxy liberada)
- [ ] Configuração de proxy removida do navegador (se aplicável)

---

## Checklist de entrega

- [ ] Todos os campos `[...]` substituídos
- [ ] Pasta `evidencias/` com capturas nomeadas por atividade (incluindo Atv. 9)
- [ ] 11 questões de verificação respondidas
- [ ] Atividade 9 (redirecionamento HTTP→HTTPS) documentada
- [ ] Justificativa de encerramento redigida
- [ ] Arquivo compactado como `NOME_RA_LAB_HTTP_FLUXOB.zip`
- [ ] Submetido no Microsoft Teams dentro do prazo
