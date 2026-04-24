# Roteiro Prático — Fluxo A (Aluno COM privilégio de administrador)

> **Pré-requisito deste roteiro:** você possui privilégio de administrador na máquina do laboratório (ou conhece a senha de administrador para o prompt do UAC).
>
> **Escopo:** executa a atividade **na íntegra**, incluindo captura e inspeção de tráfego **HTTP e HTTPS** (com decriptação TLS via certificado raiz do Fiddler).
>
> **Fundamentação teórica:** a teoria (estrutura de mensagens, métodos, status codes, cabeçalhos, papel do proxy) está no arquivo [`readme.md`](../readme.md) na raiz do repositório. Este roteiro é auto-contido nos aspectos práticos.

---

## Sumário

- [1. Validação do privilégio administrativo](#1-validação-do-privilégio-administrativo)
- [2. Preparação do Ambiente](#2-preparação-do-ambiente)
  - [2.1. Instalação do Fiddler Classic](#21-instalação-do-fiddler-classic)
  - [2.2. Ativação da decriptação HTTPS](#22-ativação-da-decriptação-https)
  - [2.3. Validação do ambiente](#23-validação-do-ambiente)
- [3. Atividades Práticas](#3-atividades-práticas)
- [4. Questões de Verificação](#4-questões-de-verificação)
- [5. Entrega](#5-entrega)
- [6. Encerramento obrigatório](#6-encerramento-obrigatório)

---

## 1. Validação do privilégio administrativo

Antes de iniciar, confirme que este é o fluxo correto. Abra um prompt do PowerShell e execute:

```powershell
(New-Object System.Security.Principal.WindowsPrincipal([System.Security.Principal.WindowsIdentity]::GetCurrent())).IsInRole([System.Security.Principal.WindowsBuiltInRole]::Administrator)
```

- Retornou `True` **ou** você consegue executar o Fiddler "Como administrador" sem que o UAC exija senha de outra conta → **prossiga com este roteiro**.
- Caso contrário, utilize o roteiro do Fluxo B (na pasta irmã `fluxo-b-sem-administrador/`).

---

## 2. Preparação do Ambiente

### 2.1. Instalação do Fiddler Classic

**Download:** https://www.telerik.com/fiddler/fiddler-classic

**Passos (Windows):**

1. Baixar o instalador (`FiddlerSetup.exe`).
2. Clicar com botão direito → **Executar como administrador**.
3. Concluir a instalação com as opções padrão.
4. Abrir o Fiddler Classic (também **como administrador** no primeiro uso).
5. Verificar que a captura está ativa: canto inferior esquerdo mostra **"Capturing"**. Caso contrário, pressionar **F12** ou usar *File → Capture Traffic*.

**Interface principal:**

- **Painel da esquerda:** lista de sessões (cada linha = um par request/response).
- **Painel da direita:** abas `Inspectors`, `AutoResponder`, `Composer`, `Filters`.
- Dentro de **Inspectors**, abas horizontais separam **Request** (cima) e **Response** (baixo) com visualizações `Raw`, `Headers`, `WebForms`, `JSON`, `Cookies`, `TextView`.

**Alternativas cross-platform** (caso prefira Linux/macOS):

| Ferramenta        | SO                  | Observação                                          |
|-------------------|---------------------|-----------------------------------------------------|
| **Fiddler Everywhere** | Win/Mac/Linux  | Interface moderna; requer conta (tier gratuito limitado) |
| **mitmproxy**     | Win/Mac/Linux       | CLI/TUI; excelente para scripting em Python         |
| **HTTP Toolkit**  | Win/Mac/Linux       | GUI moderna, configuração assistida do navegador    |
| **Burp Suite Community** | Win/Mac/Linux | Foco em segurança                                 |

Os termos abaixo usam o **Fiddler Classic**, mas os conceitos se aplicam a todas.

### 2.2. Ativação da decriptação HTTPS

Por padrão, o Fiddler mostra para HTTPS apenas um túnel `CONNECT` opaco. Para inspecionar o conteúdo:

1. Menu **Tools → Options → HTTPS**.
2. Marcar:
   - ☑ **Capture HTTPS CONNECTs**
   - ☑ **Decrypt HTTPS traffic**
3. Aceitar a instalação do certificado raiz (`DO_NOT_TRUST_FiddlerRoot`) quando solicitado. Confirmar na caixa de diálogo do Windows (clicar **Sim** no prompt de confiança).
4. **Firefox** (se usar): o Firefox tem armazenamento de certificados próprio. Exportar o certificado em *Actions → Export Root Certificate to Desktop* e importá-lo em `about:preferences#privacy → Ver Certificados → Importar → FiddlerRoot.cer`, marcando "Confiar nesta CA para identificar sites".

### 2.3. Validação do ambiente

Acesse `https://httpbin.org/get` no navegador. No Fiddler, a sessão deve aparecer com:

- Ícone de **cadeado aberto** (indicando HTTPS decifrado).
- Em **Inspectors → Response → JSON**, um JSON legível contendo campos `args`, `headers`, `origin`, `url`.

Se o Fiddler ainda mostrar apenas `CONNECT` com conteúdo cifrado, repita a etapa 2.2 e confirme a instalação do certificado.

> ⚠️ **Aviso ético.** O certificado raiz do Fiddler permite a decriptografia de **qualquer** conexão HTTPS feita por esta máquina enquanto estiver instalado. **Não esqueça de removê-lo** ao final do laboratório (seção 6).

---

## 3. Atividades Práticas

> **Como registrar.** Para cada atividade, preencha a seção correspondente no arquivo [`relatorio.md`](relatorio.md) desta mesma pasta com:
> - Captura de tela da sessão no Fiddler.
> - Trecho da mensagem *raw* (request ou response) conforme solicitado.
> - Respostas às questões embutidas.

### Atividade 1 — Primeira captura

**Objetivo:** validar o ambiente e familiarizar-se com a interface.

1. Com o Fiddler capturando, abrir o navegador e acessar `http://example.com`.
2. Identificar a sessão correspondente na lista do Fiddler (coluna `Host` = `example.com`).
3. Selecionar a sessão e, no painel `Inspectors`, abrir a aba **Raw** (tanto em Request quanto em Response).

**Registrar no relatório:**
- Captura de tela da sessão selecionada.
- A linha inicial (*request-line*) do pedido enviado.
- A linha inicial (*status-line*) da resposta recebida.

**Pergunta 1.1:** Quantos cabeçalhos o navegador enviou no request? Liste-os.

**Pergunta 1.2:** Qual foi o `Content-Length` da resposta? O corpo retornado é HTML, texto puro, JSON ou binário? Como você descobriu?

---

### Atividade 2 — Anatomia de um GET

**Objetivo:** dissecar uma requisição GET com *query string* e correlacioná-la à resposta.

1. Acessar no navegador: `https://httpbin.org/get?aluno=SEU_NOME&curso=redes`.
2. Localizar a sessão no Fiddler.
3. Inspecionar em **Request → Raw** e **Response → JSON**.

**Registrar no relatório:**
- *Request-line* completa, destacando método, *request-target* e versão.
- Valor exato dos cabeçalhos `Host`, `User-Agent` e `Accept`.
- Conteúdo dos campos `args`, `headers` e `origin` no JSON de resposta.

**Pergunta 2.1:** O valor do campo `origin` no JSON retornado corresponde a qual elemento da rede? Por que não é o IP local da sua máquina (em muitos casos)?

**Pergunta 2.2:** Compare o cabeçalho `User-Agent` enviado pelo navegador com o que aparece no JSON da resposta. Eles coincidem? Justifique.

**Pergunta 2.3:** Modifique a URL para `https://httpbin.org/headers`. Liste três cabeçalhos que o servidor vê mas que **não aparecem explicitamente** na aba Raw do request, e explique de onde eles vêm (dica: Fiddler insere alguns, o servidor de proxy reverso do httpbin pode inserir outros).

---

### Atividade 3 — POST e envio de formulário

**Objetivo:** observar o envio de dados no corpo da mensagem e o tipo MIME associado.

1. Acessar `https://httpbin.org/forms/post`.
2. Preencher os campos do formulário (pizza, tamanho, toppings, etc.).
3. Clicar em **Submit order**.
4. Localizar a nova sessão no Fiddler (método **POST**).

**Registrar no relatório:**
- *Request-line* do POST.
- Valor de `Content-Type` e `Content-Length` no request.
- Corpo completo do request (aba **Request → Raw**, abaixo da linha em branco).
- Trecho do JSON de resposta mostrando o campo `form`.

**Pergunta 3.1:** Qual o formato do corpo enviado? (`application/x-www-form-urlencoded`, `multipart/form-data` ou outro?) Como esse formato codifica caracteres especiais como espaço e acentos?

**Pergunta 3.2:** Na aba **Request → WebForms** do Fiddler, o conteúdo aparece tabulado. Compare com a aba **Raw**: qual das duas visões corresponde literalmente aos bytes enviados no socket TCP?

**Pergunta 3.3:** Usando a aba **Composer** do Fiddler, envie manualmente um `POST` para `https://httpbin.org/post` com `Content-Type: application/json` e o corpo `{"protocolo":"HTTP","versao":"1.1"}`. Registre a *response* completa. Que campo do JSON de resposta confirma que o servidor recebeu e interpretou corretamente o JSON?

---

### Atividade 4 — Catálogo de status codes

**Objetivo:** provocar e identificar códigos de status representativos.

Use o serviço `httpstat.us`, que retorna o código especificado na URL.

Para cada URL abaixo, acesse no navegador e localize a sessão no Fiddler:

| # | URL                                   | Classe esperada |
|---|---------------------------------------|-----------------|
| 1 | `https://httpstat.us/200`             | 2xx             |
| 2 | `https://httpstat.us/301`             | 3xx             |
| 3 | `https://httpstat.us/404`             | 4xx             |
| 4 | `https://httpstat.us/418`             | 4xx (humorado)  |
| 5 | `https://httpstat.us/500`             | 5xx             |
| 6 | `https://httpstat.us/503`             | 5xx             |

Adicionalmente, para observar um **304 Not Modified**:
7. Acesse `https://www.example.com`, aguarde carregar.
8. Pressione **F5** (recarregar normal). O Fiddler deve mostrar requests condicionais com `If-Modified-Since` ou `If-None-Match` recebendo `304`.

**Registrar no relatório:**
- Tabela com as 7 sessões: método, URL, *status-line* completa, `Content-Length`, presença ou ausência de body.

**Pergunta 4.1:** Em qual dos status acima o corpo da resposta está **ausente** ou tem tamanho zero? Isso é obrigatório pela especificação ou depende do servidor?

**Pergunta 4.2:** No caso do `301`, qual cabeçalho da resposta informa ao navegador para onde ir? O que aconteceria se esse cabeçalho não estivesse presente?

**Pergunta 4.3:** Explique a diferença semântica entre `200`, `304` e `404` do ponto de vista do cache do navegador.

---

### Atividade 5 — Identificação de cabeçalhos

**Objetivo:** reconhecer o propósito funcional de cada cabeçalho em tráfego real.

1. Acessar `https://www.google.com` com o cache limpo (janela anônima / privativa).
2. Selecionar a **primeira** sessão para `www.google.com` (documento HTML principal).
3. Na aba **Inspectors → Headers**, analisar request e response.

**Preencher a tabela abaixo no relatório** com os valores reais que você observou:

| Cabeçalho            | Presente em (Req/Resp)? | Valor capturado | Função em uma frase |
|----------------------|-------------------------|-----------------|---------------------|
| `Host`               |                         |                 |                     |
| `User-Agent`         |                         |                 |                     |
| `Accept`             |                         |                 |                     |
| `Accept-Encoding`    |                         |                 |                     |
| `Cookie`             |                         |                 |                     |
| `Server`             |                         |                 |                     |
| `Content-Type`       |                         |                 |                     |
| `Content-Encoding`   |                         |                 |                     |
| `Set-Cookie`         |                         |                 |                     |
| `Cache-Control`      |                         |                 |                     |
| `Strict-Transport-Security` |                  |                 |                     |

**Pergunta 5.1:** O servidor retornou `Content-Encoding: gzip` (ou `br`)? Se sim, compare o valor de `Content-Length` com o tamanho do HTML visível na aba **Response → TextView**. O que explica a diferença?

**Pergunta 5.2:** O que acontece com um request onde o cliente envia `Accept: application/json` mas o recurso só existe em `text/html`? (Pesquisar o código de status correto.)

**Pergunta 5.3:** O cabeçalho `Strict-Transport-Security` apareceu? Qual seu papel em relação a downgrades para HTTP puro?

---

### Atividade 6 — HTTP vs HTTPS

**Objetivo:** observar, lado a lado, a diferença entre tráfego claro e cifrado.

1. **Desabilitar** temporariamente a decriptação HTTPS: *Tools → Options → HTTPS → desmarcar Decrypt HTTPS traffic*. Clicar OK.
2. Acessar `http://neverssl.com` (força HTTP puro).
3. Acessar `https://www.google.com`.
4. Comparar ambas as sessões no Fiddler.

**Registrar no relatório:**
- Captura de tela de cada sessão com a aba **Raw** aberta.
- O método, o host e o que é (ou não é) visível em cada caso.

**Pergunta 6.1:** No caso do `https://www.google.com`, que método HTTP aparece na sessão? Explique o que esse método faz e por que ele existe.

**Pergunta 6.2:** Com a decriptação desabilitada, quais informações sobre o tráfego HTTPS **ainda** são visíveis para o Fiddler e quais estão ocultas?

5. **Reabilitar** a decriptação e repetir o acesso a `https://www.google.com`.

**Pergunta 6.3:** O que muda na interface do Fiddler quando a decriptação está ativa? Quais dados passam a ser inspecionáveis?

**Pergunta 6.4:** Do ponto de vista de segurança, explique por que a técnica usada pelo Fiddler para decifrar HTTPS **não** funcionaria contra você se um atacante a tentasse sem instalar o certificado na sua máquina.

---

### Atividade 7 — Cookies e sessão

**Objetivo:** rastrear o ciclo de vida de um cookie ao longo de múltiplas requisições.

1. Em uma janela anônima, acessar `https://httpbin.org/cookies/set?disciplina=redes&professor=claudio`.
2. Observar o **`Set-Cookie`** na resposta. Anotar os valores.
3. Seguir o redirecionamento automático para `https://httpbin.org/cookies` (pode ser necessário um clique).
4. Observar o cabeçalho **`Cookie`** enviado nesta segunda requisição.
5. Recarregar mais duas vezes a página `https://httpbin.org/cookies`.

**Registrar no relatório:**
- Sequência (numerada) das sessões capturadas.
- Valores dos cabeçalhos `Set-Cookie` e `Cookie` em cada uma.

**Pergunta 7.1:** O cabeçalho `Set-Cookie` só aparece uma vez ou em toda requisição? Justifique.

**Pergunta 7.2:** Que atributos o `Set-Cookie` trouxe (`Path`, `Domain`, `Expires`, `Max-Age`, `Secure`, `HttpOnly`, `SameSite`)? Para cada um presente, explique brevemente sua função.

**Pergunta 7.3:** Se o cookie não tivesse o atributo `Secure`, em que cenário ele poderia vazar? (Relacione com a Atividade 6.)

**Pergunta 7.4:** Na aba **Inspectors → Cookies** do Fiddler, compare o cookie armazenado com o que o servidor devolve no campo `cookies` do JSON da resposta. Eles coincidem?

---

### Atividade 8 — Manipulação com *breakpoints*

**Objetivo:** compreender o proxy como agente ativo, capaz de modificar tráfego em trânsito.

1. No Fiddler, habilitar **breakpoint de request**: menu **Rules → Automatic Breakpoints → Before Requests** (atalho **F11**).
2. No navegador, acessar `https://httpbin.org/user-agent`.
3. A sessão pausará no Fiddler com um ícone vermelho.
4. Na aba **Inspectors → Request → Raw**, editar o cabeçalho `User-Agent` para um valor inventado, por exemplo:
   `User-Agent: LaboratorioRedes/1.0 (Aluno NOME)`
5. Clicar em **Run to Completion** (botão verde no topo do inspector).
6. Na resposta, observar o JSON retornado.

**Registrar no relatório:**
- Captura de tela da edição do request.
- Trecho do JSON de resposta mostrando o campo `user-agent`.

**Pergunta 8.1:** O servidor tem como detectar que o `User-Agent` foi forjado? Discuta.

**Pergunta 8.2:** Repita o exercício, mas desta vez habilite **breakpoint de response** (**Rules → Automatic Breakpoints → After Responses**). Acesse `https://httpstat.us/200` e, quando o Fiddler pausar, edite a *status-line* para `HTTP/1.1 404 Not Found`. Libere. O que o navegador exibe? A manipulação afetou apenas a visualização local — comente sobre o papel do proxy como *man-in-the-middle*.

**Pergunta 8.3:** Desabilite todos os breakpoints (**Rules → Automatic Breakpoints → Disabled**, atalho **Shift+F11**) ao terminar.

---

## 4. Questões de Verificação

Responda no relatório. Respostas concisas, com base no observado, são preferíveis a explicações genéricas.

1. Qual é a **ordem dos elementos** em uma mensagem HTTP/1.1? O que separa os cabeçalhos do corpo?
2. Por que o cabeçalho `Host` é **obrigatório** em HTTP/1.1 mas era opcional em HTTP/1.0? (Dica: virtual hosting.)
3. Explique a diferença entre os códigos `401 Unauthorized` e `403 Forbidden`.
4. Um `POST` enviado duas vezes produz o mesmo efeito que um único envio? E um `PUT`? Justifique em termos de **idempotência**.
5. Por que HTTPS, mesmo cifrando todo o tráfego de aplicação, ainda permite que um observador saiba **qual site** o usuário está visitando? (Cite SNI e DNS.)
6. O que `Content-Encoding: gzip` muda no fluxo? Em que ponto os dados são compactados e em que ponto são descompactados?
7. Um servidor envia `Cache-Control: no-store` em uma resposta. Qual o impacto prático no comportamento do navegador?
8. Descreva, em até 5 linhas, como o Fiddler consegue decifrar HTTPS sem violar a criptografia — e por que isso só é possível com **cooperação do usuário**.
9. Dê um exemplo concreto, observado nas atividades, de um cabeçalho de **request** que o navegador envia automaticamente, sem a página pedir.
10. Se você quisesse automatizar a inspeção (script), qual das ferramentas alternativas da seção 2.1 seria mais adequada? Por quê?

---

## 5. Entrega

### O que entregar
- Arquivo **`relatorio.md`** preenchido (template nesta mesma pasta).
- Pasta **`evidencias/`** com as capturas de tela nomeadas por atividade (`atv1_sessao.png`, `atv3_post_raw.png`, etc.), incluindo obrigatoriamente a captura do `certmgr.msc` após a remoção do certificado (ver seção 6).
- Arquivo **`httpbin_composer.saz`** (opcional): exportação da sessão do Composer da Atividade 3 via *File → Save → Selected Sessions*.

### Como entregar
- Compactar a pasta do aluno em `NOME_RA_LAB_HTTP_FLUXOA.zip`.
- Submeter no **Microsoft Teams**, atividade correspondente, até a data definida em aula.

### Critérios de avaliação

| Critério                                              | Peso |
|-------------------------------------------------------|------|
| Exatidão técnica das respostas                        | 40%  |
| Evidências coerentes (capturas legíveis e pertinentes) | 25%  |
| Profundidade da análise das questões de verificação   | 25%  |
| Organização e clareza do relatório                    | 10%  |

---

## 6. Encerramento obrigatório

Estes passos são parte da avaliação do Fluxo A. A remoção do certificado raiz é **obrigatória** por ser um risco de segurança mantê-lo instalado.

1. **Desabilitar a decriptação HTTPS:** *Tools → Options → HTTPS → desmarcar Decrypt HTTPS traffic* → OK.
2. **Remover o certificado raiz:**
   - **Windows:** `Win+R` → `certmgr.msc` → *Autoridades de Certificação Raiz Confiáveis → Certificados* → localizar **`DO_NOT_TRUST_FiddlerRoot`** → clique direito → *Excluir*.
   - **Firefox:** `about:preferences#privacy` → *Ver Certificados* → aba *Autoridades* → selecionar `DO_NOT_TRUST_FiddlerRoot` → *Excluir ou desconfiar*.
3. **Fechar o Fiddler** para liberar a porta 8888.
4. **Anexar ao relatório** uma captura de tela do `certmgr.msc` mostrando que o certificado **foi removido** (evidência de higiene de segurança).

> ⚠️ **Por que remover.** Um certificado raiz instalado permite que qualquer processo com a chave privada correspondente emita certificados válidos para **qualquer domínio** (bancos, e-mails, redes sociais) que o navegador da máquina aceitará sem alerta. Mantê-lo após o laboratório configura uma porta de entrada para ataques locais de interceptação.
