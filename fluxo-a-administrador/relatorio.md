# Relatório — Laboratório de Inspeção HTTP/HTTPS — Fluxo A (Administrador)

> **Como usar este template:** substitua os campos `[...]` pelas suas respostas,
> anexe as capturas de tela na pasta `evidencias/` e referencie-as onde indicado.
> Preserve a formatação markdown (tabelas, blocos de código) para facilitar a correção.

---

## Identificação

| Campo       | Valor                  |
|-------------|------------------------|
| Nome        | [seu nome completo]    |
| RA          | [seu RA]               |
| Disciplina  | Redes de Computadores  |
| Turma       | [sua turma]            |
| Data        | [data da realização]   |
| Fluxo       | **A — Aluno com privilégio de administrador** |
| SO utilizado | [Windows 11 / Ubuntu 22.04 / macOS ...] |
| Ferramenta de proxy | [Fiddler Classic / mitmproxy / HTTP Toolkit / ...] |
| Navegador(es)       | [Chrome 124 / Firefox 125 / ...] |

---

## Atividade 1 — Primeira captura

**Captura de tela:** `evidencias/atv1_sessao.png`

**Request-line enviada:**

```http
[colar aqui a linha inicial do request, ex: GET / HTTP/1.1]
```

**Status-line recebida:**

```http
[colar aqui, ex: HTTP/1.1 200 OK]
```

### Pergunta 1.1
> Quantos cabeçalhos o navegador enviou no request? Liste-os.

**Resposta:**
[número total]

Cabeçalhos:
- [cabeçalho 1]
- [cabeçalho 2]
- ...

### Pergunta 1.2
> Qual foi o `Content-Length` da resposta? O corpo retornado é HTML, texto puro, JSON ou binário? Como você descobriu?

**Resposta:** [...]

---

## Atividade 2 — Anatomia de um GET

**Captura de tela:** `evidencias/atv2_raw.png`

**Request-line completa:**

```http
[colar aqui]
```

**Cabeçalhos-chave capturados:**

| Cabeçalho    | Valor                    |
|--------------|--------------------------|
| `Host`       | [...]                    |
| `User-Agent` | [...]                    |
| `Accept`     | [...]                    |

**Campos do JSON de resposta:**

```json
{
  "args":    [colar valor],
  "headers": [colar valor resumido],
  "origin":  [colar valor]
}
```

### Pergunta 2.1
> O valor do campo `origin` corresponde a qual elemento da rede? Por que normalmente não é o IP local?

**Resposta:** [...]

### Pergunta 2.2
> Compare o `User-Agent` enviado com o que aparece no JSON da resposta. Coincidem?

**Resposta:** [...]

### Pergunta 2.3
> Em `https://httpbin.org/headers`, liste três cabeçalhos que o servidor vê mas **não aparecem** no Raw do request. De onde vêm?

**Resposta:**

| Cabeçalho visto pelo servidor | Origem provável |
|-------------------------------|------------------|
| [...]                         | [...]            |
| [...]                         | [...]            |
| [...]                         | [...]            |

---

## Atividade 3 — POST e envio de formulário

**Captura de tela:** `evidencias/atv3_post_raw.png`

**Request-line do POST:**

```http
[colar aqui]
```

**Cabeçalhos do request:**

| Cabeçalho        | Valor |
|------------------|-------|
| `Content-Type`   | [...] |
| `Content-Length` | [...] |

**Corpo completo do request:**

```
[colar aqui o body enviado]
```

**Trecho do JSON de resposta (campo `form`):**

```json
"form": {
  [colar aqui]
}
```

### Pergunta 3.1
> Qual o formato do corpo? Como esse formato codifica caracteres especiais (espaço, acentos)?

**Resposta:** [...]

### Pergunta 3.2
> Comparando **Request → WebForms** e **Request → Raw**: qual das duas corresponde literalmente aos bytes enviados no socket TCP?

**Resposta:** [...]

### Pergunta 3.3 — Composer
> Envie manualmente via Composer um `POST` para `https://httpbin.org/post` com JSON. Registre a resposta. Qual campo do JSON confirma que o servidor interpretou o JSON?

**Captura de tela:** `evidencias/atv3_composer.png`

**Response JSON (trecho relevante):**

```json
{
  [colar aqui]
}
```

**Resposta:** [...]

---

## Atividade 4 — Catálogo de status codes

**Captura de tela (lista do Fiddler com as 7 sessões):** `evidencias/atv4_lista.png`

| # | Método | URL | Status-line | `Content-Length` | Body presente? |
|---|--------|-----|-------------|--------------------|----------------|
| 1 | GET    | `https://httpstat.us/200` | [...] | [...] | [sim/não] |
| 2 | GET    | `https://httpstat.us/301` | [...] | [...] | [sim/não] |
| 3 | GET    | `https://httpstat.us/404` | [...] | [...] | [sim/não] |
| 4 | GET    | `https://httpstat.us/418` | [...] | [...] | [sim/não] |
| 5 | GET    | `https://httpstat.us/500` | [...] | [...] | [sim/não] |
| 6 | GET    | `https://httpstat.us/503` | [...] | [...] | [sim/não] |
| 7 | GET    | `https://www.example.com` (F5) | [...] | [...] | [sim/não] |

### Pergunta 4.1
> Em qual dos status o corpo está ausente/tamanho zero? Isso é obrigatório pela especificação ou depende do servidor?

**Resposta:** [...]

### Pergunta 4.2
> No `301`, qual cabeçalho da resposta informa para onde ir? O que aconteceria se estivesse ausente?

**Resposta:** [...]

### Pergunta 4.3
> Diferença semântica entre `200`, `304` e `404` do ponto de vista do cache do navegador.

**Resposta:** [...]

---

## Atividade 5 — Identificação de cabeçalhos

**Captura de tela (Inspectors → Headers):** `evidencias/atv5_headers.png`

| Cabeçalho                    | Req/Resp | Valor capturado | Função em uma frase |
|------------------------------|----------|------------------|----------------------|
| `Host`                       | [...]    | [...]            | [...]                |
| `User-Agent`                 | [...]    | [...]            | [...]                |
| `Accept`                     | [...]    | [...]            | [...]                |
| `Accept-Encoding`            | [...]    | [...]            | [...]                |
| `Cookie`                     | [...]    | [...]            | [...]                |
| `Server`                     | [...]    | [...]            | [...]                |
| `Content-Type`               | [...]    | [...]            | [...]                |
| `Content-Encoding`           | [...]    | [...]            | [...]                |
| `Set-Cookie`                 | [...]    | [...]            | [...]                |
| `Cache-Control`              | [...]    | [...]            | [...]                |
| `Strict-Transport-Security`  | [...]    | [...]            | [...]                |

### Pergunta 5.1
> `Content-Encoding: gzip`/`br` apareceu? Compare `Content-Length` com o HTML visível. O que explica a diferença?

**Resposta:** [...]

### Pergunta 5.2
> Cliente envia `Accept: application/json` mas o recurso só existe em `text/html`. Qual status code esperar?

**Resposta:** [...]

### Pergunta 5.3
> `Strict-Transport-Security` apareceu? Qual seu papel contra downgrades para HTTP puro?

**Resposta:** [...]

---

## Atividade 6 — HTTP vs HTTPS

**Captura de tela HTTP (`neverssl.com`):** `evidencias/atv6_http.png`
**Captura de tela HTTPS sem decriptação:** `evidencias/atv6_https_sem.png`
**Captura de tela HTTPS com decriptação:** `evidencias/atv6_https_com.png`

### Pergunta 6.1
> No `https://www.google.com` sem decriptação, que método aparece? O que ele faz e por que existe?

**Resposta:** [...]

### Pergunta 6.2
> Com decriptação desabilitada, o que ainda é visível no HTTPS e o que está oculto?

**Resposta (visível):** [...]
**Resposta (oculto):** [...]

### Pergunta 6.3
> O que muda quando a decriptação é ativada? Que dados passam a ser inspecionáveis?

**Resposta:** [...]

### Pergunta 6.4
> Por que a técnica do Fiddler **não** funcionaria contra você se um atacante a tentasse sem instalar o certificado?

**Resposta:** [...]

---

## Atividade 7 — Cookies e sessão

**Captura de tela da sequência:** `evidencias/atv7_cookies.png`

| # | URL | `Set-Cookie` recebido | `Cookie` enviado |
|---|-----|-----------------------|-------------------|
| 1 | `/cookies/set?...`       | [...] | [nenhum / ...] |
| 2 | `/cookies` (1ª visita)   | [...] | [...]          |
| 3 | `/cookies` (reload 1)    | [...] | [...]          |
| 4 | `/cookies` (reload 2)    | [...] | [...]          |

### Pergunta 7.1
> `Set-Cookie` aparece uma vez ou em toda requisição? Justifique.

**Resposta:** [...]

### Pergunta 7.2
> Que atributos o `Set-Cookie` trouxe? Explique cada um presente.

**Resposta:**

| Atributo | Valor | Função |
|----------|-------|--------|
| [...]    | [...] | [...]  |

### Pergunta 7.3
> Sem o atributo `Secure`, em que cenário o cookie poderia vazar?

**Resposta:** [...]

### Pergunta 7.4
> Na aba **Inspectors → Cookies**, o cookie armazenado coincide com o campo `cookies` do JSON?

**Resposta:** [...]

---

## Atividade 8 — Manipulação com breakpoints

**Captura de tela da edição do User-Agent:** `evidencias/atv8_ua_edit.png`

**JSON de resposta após edição:**

```json
{
  "user-agent": "[valor forjado]"
}
```

### Pergunta 8.1
> O servidor pode detectar que o `User-Agent` foi forjado? Discuta.

**Resposta:** [...]

### Pergunta 8.2
> Após editar a status-line de `200 OK` para `404 Not Found`, o que o navegador exibe? Comente o papel do proxy como MITM.

**Captura de tela:** `evidencias/atv8_status_edit.png`

**Resposta:** [...]

### Pergunta 8.3
> Confirme que todos os breakpoints foram desabilitados.

- [ ] Breakpoints desabilitados ao final (Shift+F11)

---

## Questões de Verificação

### 1. Ordem dos elementos em uma mensagem HTTP/1.1. O que separa cabeçalhos do corpo?

[resposta]

### 2. Por que `Host` é obrigatório em HTTP/1.1 mas era opcional em HTTP/1.0?

[resposta]

### 3. Diferença entre `401 Unauthorized` e `403 Forbidden`.

[resposta]

### 4. Um `POST` enviado duas vezes produz o mesmo efeito? E um `PUT`? Justifique em termos de idempotência.

[resposta]

### 5. Por que HTTPS permite ainda que um observador saiba qual site está sendo visitado? (SNI, DNS)

[resposta]

### 6. O que muda com `Content-Encoding: gzip`? Onde os dados são compactados e descompactados?

[resposta]

### 7. Impacto prático de `Cache-Control: no-store`.

[resposta]

### 8. Como o Fiddler decifra HTTPS sem violar a criptografia, e por que exige cooperação do usuário?

[resposta]

### 9. Exemplo de cabeçalho de request que o navegador envia automaticamente, sem a página pedir.

[resposta]

### 10. Se fosse automatizar a inspeção via script, qual ferramenta alternativa escolheria? Por quê?

[resposta]

---

## Reflexão final (opcional, até 10 linhas)

> O que você aprendeu que não conhecia antes deste laboratório? Há algum
> cabeçalho, código de status ou comportamento que passou a olhar com
> mais atenção? Alguma dificuldade que recomendaria evitar para a próxima turma?

[reflexão]

---

## Encerramento — Higiene de segurança (obrigatório no Fluxo A)

**Captura de tela do `certmgr.msc` após remoção do certificado:** `evidencias/encerramento_certmgr.png`

- [ ] *Decrypt HTTPS traffic* desabilitado no Fiddler
- [ ] Certificado `DO_NOT_TRUST_FiddlerRoot` removido do Windows (`certmgr.msc`)
- [ ] Certificado `DO_NOT_TRUST_FiddlerRoot` removido do Firefox (se aplicável)
- [ ] Fiddler fechado (porta 8888 liberada)

**Comentário do aluno sobre a importância dessa etapa:**

[explicar, em até 5 linhas, por que manter o certificado instalado é um risco]

---

## Checklist de entrega

- [ ] Todos os campos `[...]` substituídos
- [ ] Pasta `evidencias/` com capturas nomeadas por atividade
- [ ] 10 questões de verificação respondidas
- [ ] Evidência de remoção do certificado anexada
- [ ] Arquivo compactado como `NOME_RA_LAB_HTTP_FLUXOA.zip`
- [ ] Submetido no Microsoft Teams dentro do prazo
