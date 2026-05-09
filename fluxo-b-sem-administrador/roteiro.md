# Roteiro Prático — Fluxo B (Aluno SEM privilégio de administrador)

> **Pré-requisito deste roteiro:** você **não** possui privilégio de administrador na máquina do laboratório. Toda a análise prática será feita sobre tráfego **HTTP em texto claro** — a inspeção de HTTPS é substituída por análise teórica baseada na fundamentação do [`readme.md`](../readme.md).
>
> **Escopo:** captura e inspeção, com Fiddler Classic, de sites que aceitam HTTP puro (sem redirecionamento forçado para HTTPS). Sem instalação de certificados. Sem alteração de configurações protegidas pelo UAC.
>
> **Fundamentação teórica:** a teoria (estrutura de mensagens, métodos, status codes, cabeçalhos, papel do proxy e decriptação TLS) está no arquivo [`readme.md`](../readme.md) na raiz do repositório. Este roteiro é auto-contido nos aspectos práticos.

---

## Sumário

- [1. Validação do perfil](#1-validação-do-perfil)
- [2. Preparação do Ambiente](#2-preparação-do-ambiente)
   - [2.1. Instalação do Fiddler sem privilégio administrativo](#21-instalação-do-fiddler-sem-privilégio-administrativo)
  - [2.2. Desabilitar promoção forçada para HTTPS no navegador](#22-desabilitar-promoção-forçada-para-https-no-navegador)
  - [2.3. Validação do ambiente](#23-validação-do-ambiente)
  - [2.4. Sites de teste que aceitam HTTP puro](#24-sites-de-teste-que-aceitam-http-puro)
- [3. Atividades Práticas](#3-atividades-práticas)
- [4. Entrega](#4-entrega)
- [5. Encerramento](#5-encerramento)

---

## 1. Validação do perfil

Confirme que este é o fluxo correto. Abra um prompt do PowerShell e execute as duas linhas abaixo (cole linha por linha):

```powershell
$id = [System.Security.Principal.WindowsIdentity]::GetCurrent()
(New-Object System.Security.Principal.WindowsPrincipal($id)).IsInRole([System.Security.Principal.WindowsBuiltInRole]::Administrator)
```

A saída esperada é exatamente `True` ou `False` na última linha.

**Teste prático complementar.** Clique com o botão direito em qualquer instalador (`.exe`) e escolha **Executar como administrador**:

- Se aparecer um prompt do UAC pedindo **usuário e senha de outra conta** (não apenas um botão "Sim"), você está no **Fluxo B** — siga este roteiro.
- Se aparecer apenas o botão "Sim" ou nada (você já é administrador), siga o **Fluxo A** (pasta irmã `fluxo-a-administrador/`).

---

## 2. Preparação do Ambiente

### 2.1. Instalação do Fiddler sem privilégio administrativo

Este fluxo usa apenas o **Fiddler Classic** em instalação por usuário (*per-user*), sem execução como administrador e sem instalação de certificado raiz.

**Download:** https://www.telerik.com/fiddler/fiddler-classic

> A página de download solicita preenchimento de um formulário (nome, e-mail, empresa). Use seu e-mail institucional ou um descartável; nenhum cadastro é validado.

1. Baixar `FiddlerSetup.exe`.
2. Executar **com duplo clique** (não use "Executar como administrador"). O instalador atual mostra apenas a tela de licença e, ao clicar **I Agree**, instala em `%LOCALAPPDATA%\Programs\Fiddler` sem solicitar UAC. Se aparecer um prompt do UAC pedindo credenciais de outra conta, **cancele** e contate o professor.
3. Abrir o Fiddler Classic.

> **Importante.** Em **nenhum caso** instale o certificado raiz do proxy no armazenamento da máquina do laboratório. Este fluxo **não** inspeciona HTTPS.

**Configuração do navegador para captura.**

- **Chrome ou Edge** (recomendado): nada a fazer — o Fiddler ajusta automaticamente o proxy do sistema (WinINET) e o tráfego destes navegadores passa pelo Fiddler.
- **Firefox** (não usa o proxy do sistema): vá em `about:preferences` → role até **Configurações de rede** → **Configurar...** → marque **Configuração manual de proxy** → HTTP Proxy `127.0.0.1`, Porta `8888`, e marque **Usar este proxy também para HTTPS**.

**Interface do Fiddler Classic:**

- **Painel da esquerda:** lista de sessões (cada linha = um par request/response).
- **Painel da direita:** abas `Inspectors`, `AutoResponder`, `Composer`, `Filters`.
- Dentro de **Inspectors**, abas horizontais separam **Request** (cima) e **Response** (baixo) com visualizações `Raw`, `Headers`, `WebForms`, `JSON`, `Cookies`, `TextView`.

> 💡 **Idioma da interface.** O Fiddler Classic existe somente em inglês. Todos os menus, abas e botões mencionados neste roteiro estão em inglês exatamente como aparecem na tela.

**Checklist antes de prosseguir:**

- [ ] **Tools → Options → HTTPS:** opção *Decrypt HTTPS traffic* **DESMARCADA**.
- [ ] Canto inferior esquerdo do Fiddler exibe **"Capturing"** (caso contrário, tecle **F12** ou clique no campo).
- [ ] Navegador escolhido configurado conforme descrito acima.

### 2.2. Desabilitar promoção forçada para HTTPS no navegador

Navegadores modernos promovem silenciosamente `http://` para `https://` via HSTS e "HTTPS-First Mode". Para inspecionar HTTP puro, desabilite esse comportamento **só para a sessão do laboratório**:

**Chrome / Edge:**

1. Abra `chrome://settings/security` (ou `edge://settings/privacy`).
2. Localize a opção **"Sempre usar conexões seguras"** / **"Always use secure connections"** e **desative**.
3. Se a opção não existir nessa versão, abra `chrome://flags` (ou `edge://flags`), pesquise por `HTTPS-First`, defina **"HTTPS-First Mode"** e **"HTTPS-Upgrades"** como **Disabled** e clique **Relaunch**.

**Firefox** (use também como fallback se as flags do Chrome/Edge estiverem bloqueadas por política de grupo do laboratório):

1. Abra `about:preferences#privacy`.
2. Role até a seção **Segurança HTTPS-Only**.
3. Selecione **"Não ativar o modo HTTPS-Only"**.

> 💡 **Limpeza preventiva de HSTS em cache.** Se você já visitou os sites de teste em sessões anteriores, o navegador pode ter armazenado uma promessa HSTS que continuará promovendo para HTTPS mesmo após os passos acima. Antes de começar a Atividade 1: no Chrome/Edge, abra `chrome://net-internals/#hsts`, vá ao campo **"Delete domain security policies"**, digite `example.com` e clique **Delete**; repita para cada domínio que for usar. No Firefox, abra `about:preferences#privacy → Limpar dados → Cookies e dados de sites`.

> ⚠️ Mesmo com tudo desabilitado, sites com HSTS **pré-carregado no navegador** (p. ex. `google.com`, `facebook.com`, grandes bancos) continuarão redirecionando para HTTPS. Use apenas os sites listados na seção 2.4, que **não** estão na lista HSTS preload.

### 2.3. Validação do ambiente

Antes de tudo, confirme novamente que o canto inferior esquerdo do Fiddler exibe **"Capturing"**. Sem isso, a lista de sessões fica vazia e parece que nada está funcionando.

Acesse `http://neverssl.com` no navegador. Na lista de sessões do Fiddler, a sessão deve aparecer com:

- **Método `GET`** (e não `CONNECT`).
- Aba **Response → Raw** mostrando HTML legível em texto claro.

**Sintomas de problema e o que fazer:**

- A lista do Fiddler permanece vazia → tecle **F12** ou clique em "Capturing"; verifique a configuração de proxy do navegador (seção 2.1).
- Aparece **apenas** uma sessão `CONNECT tunnel to neverssl.com:443` → o navegador promoveu para HTTPS; volte para a seção 2.2 e limpe o HSTS em cache.
- A barra de endereço mudou para `https://` → idem.

### 2.4. Sites de teste que aceitam HTTP puro

Todos foram verificados para aceitar conexões não cifradas:

| # | URL                                            | Uso no roteiro                                                   |
|---|------------------------------------------------|------------------------------------------------------------------|
| 1  | `http://neverssl.com`                          | Página HTML simples — primeira captura                          |
| 2  | `http://example.com`                           | Minimalista, ótimo para análise linha a linha                   |
| 3  | `http://httpbin.org/gzip`                      | Resposta compactada em HTTP puro                                |
| 4  | `http://httpbin.org/get?...`                   | Eco de request em JSON (aceita HTTP)                            |
| 5  | `http://httpbin.org/forms/post` e `/post`      | Formulários e POST                                              |
| 6  | `http://httpbin.org/cookies/set?...`           | Cookies e sessão                                                |
| 7  | `http://httpbin.org/user-agent`                | Inspeção de `User-Agent`                                        |
| 8  | `http://httpbin.org/status/{codigo}`           | Geração controlada de códigos de status (200, 404, 418, 500...) |
| 9  | `http://httpbin.org/redirect-to?...`           | Redirecionamentos 3xx com `Location` controlado                 |
| 10 | `http://httpbin.org/cache`                     | Resposta com `Last-Modified` garantido — Atividade 4 (304)      |
| 11 | `http://httpbin.org/response-headers?...`      | Cabeçalhos de resposta customizáveis — Atividade 5              |
| 12 | `http://eu.httpbin.org/...`                    | Espelho europeu do httpbin, útil se o principal estiver lento   |
| 13 | `http://www.columbia.edu/~fdc/sample.html`     | Página acadêmica estável em HTTP                                |
| 14 | `http://info.cern.ch` *(opcional)*             | Primeiro site da web (1991); curiosidade histórica              |
| 15 | `http://www.textfiles.com` *(opcional)*        | Site histórico, apenas texto/HTML                               |

---

## 3. Atividades Práticas

> **Como registrar.** Para cada atividade, preencha a seção correspondente no arquivo [`relatorio.md`](relatorio.md) desta mesma pasta com:
>
> - Captura de tela da sessão no Fiddler.
> - Trecho da mensagem *raw* (request ou response) conforme solicitado.
> - Respostas às questões embutidas.

### Antes de começar

1. **Crie a pasta `evidencias/`** ao lado do `relatorio.md`. Todas as capturas de tela ficarão lá.
2. Use a tabela abaixo para nomear os arquivos de captura **exatamente** como indicado:

   | Atividade | Arquivo(s) esperado(s) em `evidencias/`             |
   |-----------|-----------------------------------------------------|
   | 1         | `atv1_sessao.png`                                   |
   | 2         | `atv2_raw.png`                                      |
   | 3         | `atv3_post_raw.png`, `atv3_composer.png`            |
   | 4         | `atv4_lista.png`                                    |
   | 5         | `atv5_headers.png`                                  |
   | 6         | `atv6_http.png`, `atv6_https.png`                   |
   | 7         | `atv7_cookies.png`                                  |
   | 8         | `atv8_ua_edit.png`, `atv8_status_edit.png`          |
   | 9         | `atv9_redir.png`                                    |

3. **Limpe a lista de sessões do Fiddler** entre uma atividade e outra com **Edit → Remove → All Sessions** (atalho `Ctrl+X`). Isso evita confusão na hora de localizar a sessão alvo.

### Glossário rápido

- **Request-line:** primeira linha do pedido enviado pelo cliente, no formato `MÉTODO request-target VERSÃO` (ex.: `GET /index.html HTTP/1.1`).
- **Status-line:** primeira linha da resposta do servidor, no formato `VERSÃO CÓDIGO RAZÃO` (ex.: `HTTP/1.1 200 OK`).
- **Raw:** visualização que mostra a mensagem HTTP exatamente como trafegou nos bytes do socket TCP — é a fonte de verdade quando outras visualizações (Headers, JSON, WebForms) parecem divergir.
- **Body:** o corpo da mensagem, separado dos cabeçalhos por uma linha em branco.

### Atividade 1 — Primeira captura

**Objetivo:** validar o ambiente e familiarizar-se com a interface.

1. Com o Fiddler capturando, abrir o navegador e acessar `http://example.com`.
2. Identificar a sessão correspondente na lista do Fiddler (coluna `Host` = `example.com`).
3. Selecionar a sessão e, no painel `Inspectors`, abrir a aba **Raw** (tanto em Request quanto em Response).

> Se o navegador redirecionar para `https://` mesmo após as configurações da seção 2.2, volte ao bloco de **Limpeza preventiva de HSTS em cache** daquela seção.

**Registrar no relatório:**
- Captura de tela da sessão selecionada.
- A linha inicial (*request-line*) do pedido enviado.
- A linha inicial (*status-line*) da resposta recebida.

**Pergunta 1.1:** Quantos cabeçalhos o navegador enviou no request? Liste-os. Considere apenas as **linhas de cabeçalho** visíveis na aba **Request → Raw** (ou seja, exclua a *request-line* inicial e a linha em branco que separa cabeçalhos do corpo).

**Pergunta 1.2:** Qual foi o `Content-Length` da resposta? Se esse cabeçalho não apareceu, registre `Transfer-Encoding`, a versão do protocolo ou outro indício observado. O corpo retornado é HTML, texto puro, JSON ou binário? Como você descobriu?

---

### Atividade 2 — Anatomia de um GET

**Objetivo:** dissecar uma requisição GET com *query string* e correlacioná-la à resposta.

1. Acessar no navegador: `http://httpbin.org/get?aluno=SEU_NOME&curso=redes` — **substituindo `SEU_NOME` pelo seu próprio nome** (sem espaços; use underline se necessário, ex: `joao_silva`). Isso identifica sua captura no relatório.
2. Localizar a sessão no Fiddler.
3. Inspecionar em **Request → Raw** e **Response → JSON**.

**Registrar no relatório:**
- *Request-line* completa, destacando método, *request-target* e versão.
- Valor exato dos cabeçalhos `Host`, `User-Agent` e `Accept`.
- Conteúdo dos campos `args`, `headers` e `origin` no JSON de resposta.

**Pergunta 2.1:** O valor do campo `origin` no JSON retornado corresponde a qual elemento da rede? Por que não é o IP local da sua máquina (em muitos casos)?

**Pergunta 2.2:** Compare o cabeçalho `User-Agent` enviado pelo navegador com o que aparece no JSON da resposta. Eles coincidem? Se **não** coincidirem, qual elemento intermediário (proxy escolar, antivírus, extensão de navegador, o próprio Fiddler) poderia tê-lo modificado?

**Pergunta 2.3:** Modifique a URL para `http://httpbin.org/headers`. Compare a aba **Request → Raw** com o JSON de resposta e liste até três cabeçalhos que **só aparecem no JSON** (não estavam no Raw). Exemplos típicos: `X-Forwarded-For`, `X-Forwarded-Host`, `X-Amzn-Trace-Id`, `Via`. Para cada um, indique a origem provável (proxy reverso da Amazon que hospeda o httpbin, balanceador de carga, ou o próprio Fiddler). Se não encontrar três, registre os que encontrar e explique por que o resultado pode variar.

---

### Atividade 3 — POST e envio de formulário

**Objetivo:** observar o envio de dados no corpo da mensagem e o tipo MIME associado.

1. Acessar `http://httpbin.org/forms/post`.
2. Preencher os campos do formulário (pizza, tamanho, toppings, etc.).
3. Clicar em **Submit order**.
4. Localizar a nova sessão no Fiddler (método **POST** para `/post`).

> 💡 **Estrutura de uma mensagem HTTP** (a separação entre cabeçalhos e corpo é uma linha em branco — duas sequências `CRLF` consecutivas):
>
> ```text
> POST /post HTTP/1.1            ← request-line
> Host: httpbin.org              ← cabeçalhos
> Content-Type: ...
> Content-Length: ...
>                                ← LINHA EM BRANCO (separador)
> custname=...&custtel=...       ← corpo (body)
> ```

**Registrar no relatório:**
- *Request-line* do POST.
- Valor de `Content-Type` e `Content-Length` no request.
- Corpo completo do request (aba **Request → Raw**, abaixo da linha em branco).
- Trecho do JSON de resposta mostrando o campo `form`.

**Pergunta 3.1:** Qual o formato do corpo enviado? (`application/x-www-form-urlencoded`, `multipart/form-data` ou outro?) Como esse formato codifica caracteres especiais como espaço e acentos?

**Pergunta 3.2:** Na aba **Request → WebForms** do Fiddler, o conteúdo aparece tabulado. Compare com a aba **Raw**: qual das duas visões corresponde literalmente aos bytes enviados no socket TCP?

**Pergunta 3.3:** Envie manualmente um `POST` para `http://httpbin.org/post` com `Content-Type: application/json` e o corpo `{"protocolo":"HTTP","versao":"1.1"}`. Registre a *response* completa. Que campo do JSON de resposta confirma que o servidor recebeu e interpretou corretamente o JSON?

> 💡 **Dica de inspeção.** Verifique especificamente o campo `json` da resposta. Se ele estiver `null` e o conteúdo aparecer apenas em `data` como string, isso indica que o `Content-Type` foi enviado errado (ou ficou em branco) — o servidor recebeu os bytes mas não os interpretou como JSON.

> **Como enviar o POST manual no Composer do Fiddler Classic:**
>
> 1. Abrir a aba **Composer** (no painel direito).
> 2. No menu suspenso de método, selecionar **POST**.
> 3. No campo de URL, digitar `http://httpbin.org/post`.
> 4. Na **caixa de texto de cabeçalhos** (acima da caixa do body), em uma linha própria, digitar exatamente: `Content-Type: application/json` (já existirão um `User-Agent`/`Host` — não os apague).
> 5. Na **caixa do Request Body** (abaixo dos cabeçalhos), colar: `{"protocolo":"HTTP","versao":"1.1"}` — sem aspas extras, sem quebras de linha.
> 6. Clicar em **Execute** (botão à direita).

---

### Atividade 4 — Catálogo de status codes

**Objetivo:** provocar e identificar códigos de status representativos.

Use o serviço `httpbin.org`, que possui endpoints próprios para provocar códigos de status controlados.

#### 4a — Status codes diretos

Para cada URL abaixo, acesse no navegador e localize a sessão no Fiddler.

| # | URL                                                         | Classe esperada       |
|---|-------------------------------------------------------------|-----------------------|
| 1 | `http://httpbin.org/status/200`                             | 2xx                   |
| 2 | `http://httpbin.org/redirect-to?status_code=301&url=/get`   | 3xx                   |
| 3 | `http://httpbin.org/status/404`                             | 4xx                   |
| 4 | `http://httpbin.org/status/418`                             | 4xx (ver nota abaixo) |
| 5 | `http://httpbin.org/status/500`                             | 5xx                   |
| 6 | `http://httpbin.org/status/503`                             | 5xx                   |

> **Nota sobre o `418`.** Não é um erro do servidor: é um status humorado definido pela RFC 2324 ("*I'm a teapot*"). Está aqui propositalmente para demonstrar que clientes precisam tolerar status codes não previstos.

> **Nota sobre o `301`.** O navegador seguirá o redirecionamento automaticamente e criará uma segunda sessão para o destino (`/get`). Para esta atividade, **registre a sessão original** que retornou `301 Moved Permanently`, não a do destino.

#### 4b — Resposta condicional `304 Not Modified`

1. Acesse `http://httpbin.org/cache` e anote o valor do cabeçalho `Last-Modified` da resposta (visível em **Inspectors → Response → Headers**).
2. Abra a aba **Composer** do Fiddler:
   - Método: `GET`
   - URL: `http://httpbin.org/cache`
   - Adicione, na caixa de cabeçalhos, uma linha no formato:

     ```text
     If-Modified-Since: Wed, 21 Oct 2015 07:28:00 GMT
     ```

     **substituindo** o valor pelo `Last-Modified` que você observou no passo 1 (formato literal, com vírgula após o dia da semana e `GMT` ao final).
3. Clique em **Execute** e localize a nova sessão.
4. Confirme que a resposta retornou `304 Not Modified`. Use essa sessão como a **linha 7** da tabela do relatório. Se vier `200 OK`, registre os cabeçalhos de cache observados e explique a diferença.

**Registrar no relatório:**
- Tabela com as 7 sessões: método, URL, *status-line* completa, `Content-Length` ou `Transfer-Encoding` quando presentes, presença ou ausência de body.

**Pergunta 4.1:** Em qual dos status acima o corpo da resposta está **ausente** ou tem tamanho zero? (Olhe o número de bytes abaixo da linha em branco na aba **Response → Raw**.) Isso é obrigatório pela especificação RFC 9110 — que define explicitamente os casos canônicos `204 No Content` e `304 Not Modified` — ou depende do servidor?

**Pergunta 4.2:** No caso do `301`, qual cabeçalho da resposta informa ao navegador para onde ir? O que aconteceria se esse cabeçalho não estivesse presente?

**Pergunta 4.3:** Explique a diferença semântica entre `200`, `304` e `404` do ponto de vista do cache do navegador.

---

### Atividade 5 — Identificação de cabeçalhos

**Objetivo:** reconhecer o propósito funcional de cada cabeçalho em tráfego real.

1. Acessar `http://httpbin.org/response-headers?Cache-Control=max-age%3D3600&Set-Cookie=teste%3D1` para provocar cabeçalhos variados em uma resposta controlada.

   > 💡 A URL acima está percent-encoded. Decodificada, equivale a pedir os parâmetros `Cache-Control=max-age=3600` e `Set-Cookie=teste=1`. Cole-a **exatamente como está** na barra de endereço — copiar a versão decodificada quebra a query string.

2. Acessar `http://httpbin.org/gzip` para observar uma resposta compactada em HTTP puro.
3. Recarregar a primeira URL uma vez, para verificar se o cookie `teste=1` passa a aparecer no cabeçalho `Cookie` do request.
4. Selecionar cada sessão e, na aba **Inspectors → Headers**, analisar request e response.

**Preencher a tabela abaixo no relatório** com os valores reais observados, agregando dados das sessões acima conforme o cabeçalho apareça:

| Cabeçalho                   | Onde observar                           | Req/Resp | Valor capturado | Função em uma frase |
|-----------------------------|-----------------------------------------|----------|-----------------|---------------------|
| `Host`                      | qualquer sessão                         |          |                 |                     |
| `User-Agent`                | qualquer sessão                         |          |                 |                     |
| `Accept`                    | qualquer sessão                         |          |                 |                     |
| `Accept-Encoding`           | qualquer sessão                         |          |                 |                     |
| `Cookie`                    | recarga de `/response-headers`          |          |                 |                     |
| `Server`                    | qualquer resposta httpbin               |          |                 |                     |
| `Content-Type`              | qualquer resposta                       |          |                 |                     |
| `Content-Encoding`          | resposta de `/gzip`                     |          |                 |                     |
| `Set-Cookie`                | resposta de `/response-headers`         |          |                 |                     |
| `Cache-Control`             | resposta de `/response-headers`         |          |                 |                     |
| `Strict-Transport-Security` | Não esperado em HTTP — ver Pergunta 5.3 | —        | —               | —                   |

**Pergunta 5.1:** O servidor retornou `Content-Encoding: gzip` (ou `br`)? Se sim, compare o valor de `Content-Length`, quando presente, com o tamanho do conteúdo visível na aba **Response → TextView**. O que explica a diferença?

**Pergunta 5.2:** O que acontece com um request onde o cliente envia `Accept: application/json` mas o recurso só existe em `text/html`? Pesquise na RFC 9110 o **código de status 4xx específico de negociação de conteúdo** (não confunda com `400 Bad Request` ou `415 Unsupported Media Type`).

**Pergunta 5.3:** O cabeçalho `Strict-Transport-Security` apareceu nas respostas HTTP observadas? **Por que esse cabeçalho está ausente neste fluxo?** (Dica: consulte a RFC 6797 sobre em qual protocolo o HSTS é válido.) Explique o papel do HSTS contra downgrades para HTTP puro, ainda que você não o tenha observado em tráfego real.

---

### Atividade 6 — HTTP vs HTTPS (análise comparativa sem decriptação)

**Objetivo:** observar a diferença entre tráfego claro e cifrado, mesmo sem poder decifrar o HTTPS.

1. Acessar `http://neverssl.com` (HTTP puro).
2. Acessar `https://httpbin.org/get` no mesmo navegador.
3. Comparar as duas sessões no Fiddler.

**Registrar no relatório:**
- Captura de tela de cada sessão com a aba **Raw** aberta.
- Para o HTTP, transcrever a *request-line* + os 3 primeiros cabeçalhos do request, a *status-line* + os 3 primeiros cabeçalhos da response e os primeiros ~200 caracteres do corpo.
- Para o HTTPS, descrever **o que aparece** na sessão: método, host (visível no `CONNECT` mesmo sem decifrar, via SNI), bytes cifrados ilegíveis na aba `TextView` e os totais da aba `Inspectors → Statistics` (tamanho e duração).

**Pergunta 6.1:** No caso do `https://httpbin.org/get`, que método HTTP aparece na sessão do Fiddler? Explique o que esse método faz e por que ele existe. (Dica: o método `CONNECT` é definido na **RFC 9110, Seção 9.3.6** — anteriormente RFC 7231 §4.3.6.)

**Pergunta 6.2:** Faça uma **tabela comparativa** dos campos visíveis ao Fiddler em cada caso:

| Campo                          | Visível em HTTP? | Visível em HTTPS (sem decriptação)? |
|--------------------------------|------------------|-------------------------------------|
| Método                         |                  |                                     |
| URL completa (path + query)    |                  |                                     |
| Cabeçalhos de request          |                  |                                     |
| Corpo de request               |                  |                                     |
| Status code                    |                  |                                     |
| Cabeçalhos de response         |                  |                                     |
| Corpo de response              |                  |                                     |
| Host (via SNI, no `CONNECT`)   |                  |                                     |
| IP e porta de destino          |                  |                                     |

**Pergunta 6.3 (teórica):** Descreva em um parágrafo, com base na fundamentação teórica do [`readme.md`](../readme.md) (seção 4.6), o que você **veria** no Fiddler se tivesse privilégio de administrador e pudesse habilitar *Decrypt HTTPS traffic*. Indique as telas e abas (Inspectors → Raw, Response → JSON/TextView, etc.) onde cada informação apareceria, e **por que** essa inspeção exige a instalação de um certificado raiz no sistema.

**Pergunta 6.4:** Do ponto de vista de segurança, explique por que a técnica de decriptação usada pelos *debugging proxies* **não** funcionaria contra um usuário se um atacante a tentasse sem instalar o certificado na máquina-alvo. Relacione com o conceito de "cooperação do usuário" da seção 4.6 do [`readme.md`](../readme.md).

---

### Atividade 7 — Cookies e sessão

**Objetivo:** rastrear o ciclo de vida de um cookie ao longo de múltiplas requisições.

1. Abra uma **janela anônima/privativa** no navegador (para garantir que não há cookies de sessões anteriores influenciando o resultado) e acesse `http://httpbin.org/cookies/set?disciplina=redes&professor=claudio`.
2. Observar o **`Set-Cookie`** na resposta. Anotar os valores.
3. O httpbin retorna um `302` que o navegador segue automaticamente para `http://httpbin.org/cookies`. Se o navegador não seguir, acesse `http://httpbin.org/cookies` manualmente.
4. Observar o cabeçalho **`Cookie`** enviado na segunda requisição (sessão com status `200`).
5. Recarregar mais duas vezes a página `http://httpbin.org/cookies`.

> 💡 **Sessões esperadas no Fiddler ao final desta atividade:**
>
> | # | Método/URL                            | Status | Cabeçalho-chave esperado    |
> |---|---------------------------------------|--------|-----------------------------|
> | 1 | `GET /cookies/set?...`                | `302`  | **`Set-Cookie`** na response|
> | 2 | `GET /cookies` (redirecionamento)     | `200`  | **`Cookie`** no request     |
> | 3 | `GET /cookies` (1º reload)            | `200`  | **`Cookie`** no request     |
> | 4 | `GET /cookies` (2º reload)            | `200`  | **`Cookie`** no request     |

**Registrar no relatório:**
- Sequência (numerada) das sessões capturadas.
- Valores dos cabeçalhos `Set-Cookie` e `Cookie` em cada uma.

**Pergunta 7.1:** O cabeçalho `Set-Cookie` só aparece uma vez ou em toda requisição? Justifique.

**Pergunta 7.2:** Que atributos o `Set-Cookie` trouxe (`Path`, `Domain`, `Expires`, `Max-Age`, `Secure`, `HttpOnly`, `SameSite`)? Para cada um presente, explique brevemente sua função. Se algum atributo da lista não apareceu, registre como **não observado**.

> **Nota:** o httpbin define cookies mínimos — apenas o atributo `Path=/` estará presente nas respostas deste exercício. Registre todos os demais atributos listados como **não observado** e, para cada um, explique o comportamento padrão adotado pelo navegador na ausência desse atributo (ex.: sem `Expires`/`Max-Age`, o cookie é de sessão; sem `Secure`, pode ser enviado por HTTP; sem `SameSite`, o navegador aplica a política padrão da versão em uso).

**Pergunta 7.3:** Responda em duas partes: **(a) do ponto de vista do servidor:** ele *pode* enviar um cookie com atributo `Secure` numa resposta HTTP puro (a especificação proíbe? só desencoraja?). **(b) do ponto de vista do navegador:** o que ele faz com esse cookie? (Consulte a RFC 6265bis, seção sobre o atributo `Secure`.) Relacione com o fato de que todo o tráfego desta atividade é **visível em texto claro** ao Fiddler — e, portanto, a qualquer observador na rede.

**Pergunta 7.4:** Na aba **Inspectors → Cookies** do Fiddler, compare o cookie armazenado com o que o servidor devolve no campo `cookies` do JSON da resposta. Eles coincidem?

---

### Atividade 8 — Manipulação com *breakpoints*

**Objetivo:** compreender o proxy como agente ativo, capaz de modificar tráfego em trânsito.

> As etapas de breakpoint abaixo dependem do menu **Rules → Automatic Breakpoints** do Fiddler Classic.

1. No Fiddler, habilitar **breakpoint de request**: menu **Rules → Automatic Breakpoints → Before Requests**.

   > ⚠️ **Atenção ao atalho F11.** Existe um atalho `F11` documentado, **mas só funciona com o foco na janela do Fiddler**. Se você acionar F11 com o foco no navegador, o navegador entra em modo tela cheia. Para evitar confusão, **prefira sempre o caminho de menu** nesta atividade.

2. No navegador, acessar `http://httpbin.org/user-agent`.
3. A sessão pausará no Fiddler com um ícone vermelho.
4. Na aba **Inspectors → Request → Raw** (clique dentro da área de texto para habilitar a edição), localize a linha `User-Agent: ...` e substitua o valor por algo inventado, por exemplo:
   `User-Agent: LaboratorioRedes/1.0 (Aluno NOME)`
5. Clicar em **Run to Completion** (botão verde no topo do inspector).
6. Na resposta, observar o JSON retornado.

**Registrar no relatório:**
- Captura de tela da edição do request.
- Trecho do JSON de resposta mostrando o campo `user-agent`.

**Pergunta 8.1:** O servidor tem como detectar que o `User-Agent` foi forjado? Discuta.

**Pergunta 8.2:** Repita o exercício, mas desta vez habilite **breakpoint de response** (**Rules → Automatic Breakpoints → After Responses**). Acesse `http://httpbin.org/status/200` e, quando o Fiddler pausar, vá em **Inspectors → Response → Raw** e edite a primeira linha (a *status-line*) para `HTTP/1.1 404 Not Found`. Clique em **Run to Completion**. Descreva **o que VOCÊ observou** no navegador (o comportamento varia entre Chrome, Edge e Firefox — qualquer descrição honesta da tela renderizada é válida). Comente sobre o papel do proxy como *man-in-the-middle*: o servidor original retornou `200`, mas o navegador acreditou em `404`.

**Pergunta 8.3:** Desabilite todos os breakpoints (**Rules → Automatic Breakpoints → Disabled**, atalho **Shift+F11**) ao terminar.

---

### Atividade 9 — Redirecionamento HTTP → HTTPS (exclusiva do Fluxo B)

**Objetivo:** observar, no próprio Fiddler, um redirecionamento controlado de HTTP para HTTPS e relacioná-lo aos mecanismos de segurança discutidos na teoria.

1. Com os breakpoints desabilitados, acessar no navegador a URL abaixo (cole exatamente como está — a parte `https%3A%2F%2F` é a versão percent-encoded de `https://` e é necessária para que o httpbin aceite o destino como parâmetro):

   ```text
   http://httpbin.org/redirect-to?status_code=301&url=https%3A%2F%2Fhttpbin.org%2Fget
   ```

2. Localizar no Fiddler a sessão original para `httpbin.org` que retornou `301 Moved Permanently`.
3. Observar que o navegador pode criar uma segunda sessão para o destino `https://httpbin.org/get`; para esta atividade, registre a sessão original do redirecionamento.

**Registrar no relatório:**
- Captura de tela da sessão.
- *Status-line* completa da resposta.
- Cabeçalho `Location` da resposta (obrigatório em qualquer resposta `3xx` — se estiver ausente, registre isso como anomalia e verifique se capturou a sessão correta).

**Pergunta 9.1:** Qual foi o **código de status** retornado e qual **cabeçalho** direcionou o navegador para `https://`?

**Pergunta 9.2:** Com base na fundamentação teórica do [`readme.md`](../readme.md) (seções 4.5 e Anexo A), além do redirecionamento 3xx, **qual outro mecanismo** faz o navegador passar a forçar HTTPS em visitas futuras, mesmo que o usuário digite `http://`? Responda citando o cabeçalho de response envolvido e a RFC que o define.

**Pergunta 9.3:** Considerando especificamente o cabeçalho que você citou na resposta da Pergunta 9.2: se o servidor o enviasse numa resposta servida via **HTTP puro** (porta 80, sem TLS), o navegador deveria obedecer? Justifique com base na RFC — o ponto-chave é por que confiar em uma promessa de segurança recebida por um canal inseguro seria contraproducente.


---

## 4. Entrega

### O que entregar
- Arquivo **`relatorio.pdf`** gerado a partir do `relatorio.md` preenchido (ver instruções abaixo).
- Pasta **`evidencias/`** com as capturas de tela nomeadas por atividade (`atv1_sessao.png`, `atv3_post_raw.png`, etc.), incluindo a captura da Atividade 9 (redirecionamento controlado de `http://httpbin.org/...` para `https://httpbin.org/get`).

### Como gerar o PDF a partir do `relatorio.md`

O relatório é escrito em Markdown. Use uma das duas opções abaixo:

| Ferramenta | Quando usar | Como usar |
|---|---|---|
| **Markdown PDF** — extensão VS Code (**recomendado**) | Você já está com o repositório aberto no VS Code; funciona offline. | Instalar a extensão *"Markdown PDF"* (autor: yzane). Com o `relatorio.md` aberto, pressionar `Ctrl+Shift+P` → *"Markdown PDF: Export (pdf)"*. O arquivo é gerado ao lado do `.md`. |
| **Markdown to PDF** (md2pdf) — site público gratuito | Fallback se a extensão não puder ser instalada (laboratório com restrição). Não exige cadastro. | Abrir https://md2pdf.netlify.app, arrastar o arquivo `relatorio.md` para a área indicada (ou colar o conteúdo) e clicar em **Convert**. Salvar o PDF gerado. |

> **Dica:** antes de gerar o PDF, revise o preview do Markdown (VS Code: `Ctrl+Shift+V`) para confirmar que tabelas e blocos de código estão formatados corretamente. Imagens referenciadas em `evidencias/` devem estar **na mesma pasta** do `relatorio.md` para aparecerem no PDF.

### Como entregar
- Compactar a pasta do aluno em `SOBRENOME_NOME_RA_LAB_HTTP_FLUXOB.zip` contendo o `relatorio.pdf` e a pasta `evidencias/`.
  - Exemplo: `silva_joao_2023123_LAB_HTTP_FLUXOB.zip` (sem espaços, sem acentos, tudo minúsculo).
- Submeter no **Microsoft Teams**, atividade correspondente, até a data definida em aula.

---

## 5. Encerramento


1. **Reabilitar** o HTTPS-First Mode / HTTPS-Only Mode no navegador (passos inversos de 2.2), para que seu dia-a-dia volte a priorizar conexões seguras.
2. **Fechar o Fiddler** para liberar a porta de proxy e remover qualquer configuração de proxy do navegador.
3. **Entrega adicional obrigatória:** anexar ao relatório (na seção *Encerramento — justificativa de segurança* do `relatorio.md`) um parágrafo explicando **por que**, neste fluxo, a etapa de remoção de certificado é **dispensável** — e **por que**, no fluxo do aluno administrador, ela seria **obrigatória**. Use a seção 4.6 do [`readme.md`](../readme.md) como referência.
