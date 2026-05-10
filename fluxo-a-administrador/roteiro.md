# Roteiro Prático — Fluxo A (Aluno COM privilégio de administrador)

> **Pré-requisito deste roteiro:** você possui privilégio de administrador na máquina do laboratório (ou conhece a senha de administrador para o prompt do UAC).
>
> **Escopo:** executa a atividade **na íntegra**, incluindo captura e inspeção de tráfego **HTTP e HTTPS** (com decriptação TLS via certificado raiz do Fiddler).
>
> **Fundamentação teórica:** a teoria (estrutura de mensagens, métodos, status codes, cabeçalhos, papel do proxy e decriptação TLS) está no arquivo [`readme.md`](../readme.md) na raiz do repositório. Este roteiro é auto-contido nos aspectos práticos.

---

## Sumário

- [1. Validação do privilégio administrativo](#1-validação-do-privilégio-administrativo)
- [2. Preparação do Ambiente](#2-preparação-do-ambiente)
  - [2.1. Instalação do Fiddler Classic](#21-instalação-do-fiddler-classic)
  - [2.2. Ativação da decriptação HTTPS](#22-ativação-da-decriptação-https)
  - [2.3. Validação do ambiente](#23-validação-do-ambiente)
- [3. Atividades Práticas](#3-atividades-práticas)
- [4. Entrega](#4-entrega)
- [5. Encerramento obrigatório](#5-encerramento-obrigatório)

---

## 1. Validação do privilégio administrativo

Antes de iniciar, confirme que este é o fluxo correto. Abra um prompt do PowerShell e execute as duas linhas abaixo (cole linha por linha):

```powershell
$id = [System.Security.Principal.WindowsIdentity]::GetCurrent()
(New-Object System.Security.Principal.WindowsPrincipal($id)).IsInRole([System.Security.Principal.WindowsBuiltInRole]::Administrator)
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

**Configuração do navegador para captura.**

- **Chrome ou Edge** (recomendado): nada a fazer — o Fiddler ajusta automaticamente o proxy do sistema (WinINET) e o tráfego destes navegadores passa pelo Fiddler.
- **Firefox**: vá em `about:preferences` → role até **Configurações de rede** → **Configurar...** → marque **Usar as configurações de proxy do sistema**. Para HTTPS decifrado, também será necessário importar o certificado do Fiddler na etapa 2.2.

> 💡 **Idioma da interface.** O Fiddler Classic existe somente em inglês. Todos os menus, abas e botões mencionados neste roteiro estão em inglês exatamente como aparecem na tela.

### 2.2. Ativação da decriptação HTTPS

Por padrão, o Fiddler mostra para HTTPS apenas um túnel `CONNECT` opaco. Para inspecionar o conteúdo:

1. Menu **Tools → Options → HTTPS**.
2. Marcar:
   - ☑ **Capture HTTPS CONNECTs**
   - ☑ **Decrypt HTTPS traffic**
3. Aceitar a instalação do certificado raiz (`DO_NOT_TRUST_FiddlerRoot`) quando solicitado. Confirmar na caixa de diálogo do Windows (clicar **Sim** no prompt de confiança).
4. **Firefox** (se usar): o Firefox tem armazenamento de certificados próprio. Exportar o certificado em *Actions → Export Root Certificate to Desktop* e importá-lo em `about:preferences#privacy → Ver Certificados → Importar → FiddlerRoot.cer`, marcando "Confiar nesta CA para identificar sites".

**Checklist antes de prosseguir:**

- [ ] **Tools → Options → HTTPS:** opções *Capture HTTPS CONNECTs* e *Decrypt HTTPS traffic* marcadas.
- [ ] Certificado raiz `DO_NOT_TRUST_FiddlerRoot` instalado no Windows e, se aplicável, no Firefox.
- [ ] Canto inferior esquerdo do Fiddler exibe **"Capturing"** (caso contrário, tecle **F12** ou clique no campo).
- [ ] Navegador escolhido configurado conforme descrito acima.

### 2.3. Validação do ambiente

Acesse `https://httpbin.org/get` no navegador. No Fiddler, a sessão deve aparecer com:

- Ícone de **cadeado aberto** (indicando HTTPS decifrado).
- Em **Inspectors → Response → JSON**, um JSON legível contendo campos `args`, `headers`, `origin`, `url`.

Se o Fiddler ainda mostrar apenas `CONNECT` com conteúdo cifrado, repita a etapa 2.2 e confirme a instalação do certificado.

> ⚠️ **Aviso ético.** O certificado raiz do Fiddler permite a decriptografia de **qualquer** conexão HTTPS feita por esta máquina enquanto estiver instalado. **Não esqueça de removê-lo** ao final do laboratório (seção 5).

---

## 3. Atividades Práticas

> **Como registrar.** Para cada atividade, preencha a seção correspondente no arquivo [`relatorio.md`](relatorio.md) desta mesma pasta com:
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
   | 6         | `atv6_http.png`, `atv6_https_sem.png`, `atv6_https_com.png` |
   | 7         | `atv7_cookies.png`                                  |
   | 8         | `atv8_ua_edit.png`, `atv8_status_edit.png`          |
   | Encerramento | `encerramento_certmgr.png`                       |

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

**Registrar no relatório:**
- Captura de tela da sessão selecionada.
- A linha inicial (*request-line*) do pedido enviado.
- A linha inicial (*status-line*) da resposta recebida.

**Pergunta 1.1:** Quantos cabeçalhos o navegador enviou no request? Liste-os.

**Pergunta 1.2:** Qual foi o `Content-Length` da resposta? Se esse cabeçalho não apareceu, registre `Transfer-Encoding`, a versão do protocolo ou outro indício observado. O corpo retornado é HTML, texto puro, JSON ou binário? Como você descobriu?

---

### Atividade 2 — Anatomia de um GET

**Objetivo:** dissecar uma requisição GET com *query string* e correlacioná-la à resposta.

1. Acessar no navegador: `https://httpbin.org/get?aluno=SEU_NOME&curso=redes` — **substituindo `SEU_NOME` pelo seu próprio nome** (sem espaços; use underline se necessário, ex: `joao_silva`). Isso identifica sua captura no relatório.
2. Localizar a sessão no Fiddler.
3. Inspecionar em **Request → Raw** e **Response → JSON**.

**Registrar no relatório:**
- *Request-line* completa, destacando método, *request-target* e versão.
- Valor exato dos cabeçalhos `Host`, `User-Agent` e `Accept`.
- Conteúdo dos campos `args`, `headers` e `origin` no JSON de resposta.

**Pergunta 2.1:** O valor do campo `origin` no JSON retornado corresponde a qual elemento da rede? Por que não é o IP local da sua máquina (em muitos casos)?

**Pergunta 2.2:** Compare o cabeçalho `User-Agent` enviado pelo navegador com o que aparece no JSON da resposta. Eles coincidem? Se **não** coincidirem, qual elemento intermediário (proxy escolar, antivírus, extensão de navegador, o próprio Fiddler) poderia tê-lo modificado?

**Pergunta 2.3:** Modifique a URL para `https://httpbin.org/headers`. Liste até três cabeçalhos que o servidor vê mas que **não aparecem explicitamente** na aba Raw do request, e explique de onde eles vêm (dica: Fiddler pode inserir alguns, o servidor de proxy reverso do httpbin pode inserir outros). Se não encontrar três, registre os que encontrar e explique por que o resultado pode variar.

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

> 💡 **Dica de inspeção.** Verifique especificamente o campo `json` da resposta. Se ele estiver `null` e o conteúdo aparecer apenas em `data` como string, isso indica que o `Content-Type` foi enviado errado (ou ficou em branco) — o servidor recebeu os bytes mas não os interpretou como JSON.

> **Como enviar o POST manual no Composer do Fiddler Classic:**
>
> 1. Abrir a aba **Composer** (no painel direito).
> 2. No menu suspenso de método, selecionar **POST**.
> 3. No campo de URL, digitar `https://httpbin.org/post`.
> 4. Na **caixa de texto de cabeçalhos** (acima da caixa do body), em uma linha própria, digitar exatamente: `Content-Type: application/json` (já existirão um `User-Agent`/`Host` — não os apague).
> 5. Na **caixa do Request Body** (abaixo dos cabeçalhos), colar: `{"protocolo":"HTTP","versao":"1.1"}` — sem aspas extras, sem quebras de linha.
> 6. Clicar em **Execute** (botão à direita).

---

### Atividade 4 — Catálogo de status codes

**Objetivo:** provocar e identificar códigos de status representativos.

Use o serviço `httpbin.org`, que possui endpoints próprios para provocar códigos de status controlados. Para o `301`, use o endpoint de redirecionamento indicado, pois ele retorna também o cabeçalho `Location`.

Para cada URL abaixo, acesse no navegador e localize a sessão no Fiddler:

> No caso do `301`, o navegador provavelmente seguirá o redirecionamento e criará uma segunda sessão para o destino (`/get`). Para esta atividade, registre a sessão original que retornou `301 Moved Permanently`.

| # | URL                                   | Classe esperada |
|---|---------------------------------------|-----------------|
| 1 | `https://httpbin.org/status/200`      | 2xx             |
| 2 | `https://httpbin.org/redirect-to?status_code=301&url=/get` | 3xx |
| 3 | `https://httpbin.org/status/404`      | 4xx             |
| 4 | `https://httpbin.org/status/418`      | 4xx (humorado)  |
| 5 | `https://httpbin.org/status/500`      | 5xx             |
| 6 | `https://httpbin.org/status/503`      | 5xx             |

Adicionalmente, para observar um **304 Not Modified** de forma controlada:
7. Acesse `https://httpbin.org/cache` e anote o valor do cabeçalho `Last-Modified` da resposta (visível em **Inspectors → Response → Headers**).
8. No **Composer** do Fiddler, envie um `GET` para `https://httpbin.org/cache` incluindo o cabeçalho `If-Modified-Since` com o valor exato de `Last-Modified` observado no passo anterior.
9. Localize a sessão condicional gerada no passo 8 e confirme que a resposta retornou `304 Not Modified`. Use esta sessão condicional como a linha 7 da tabela. Se o servidor retornar `200 OK`, registre os cabeçalhos de cache observados e explique a diferença.

**Registrar no relatório:**
- Tabela com as 7 sessões: método, URL, *status-line* completa, `Content-Length` ou `Transfer-Encoding` quando presentes, presença ou ausência de body.

**Pergunta 4.1:** Em qual dos status acima o corpo da resposta está **ausente** ou tem tamanho zero? Isso é obrigatório pela especificação ou depende do servidor?

**Pergunta 4.2:** No caso do `301`, qual cabeçalho da resposta informa ao navegador para onde ir? O que aconteceria se esse cabeçalho não estivesse presente?

**Pergunta 4.3:** Explique a diferença semântica entre `200`, `304` e `404` do ponto de vista do cache do navegador.

---

### Atividade 5 — Identificação de cabeçalhos

**Objetivo:** reconhecer o propósito funcional de cada cabeçalho em tráfego real.

1. Em uma janela anônima / privativa, acessar `https://httpbin.org/response-headers?Cache-Control=max-age%3D3600&Set-Cookie=teste%3D1&Strict-Transport-Security=max-age%3D31536000` para provocar cabeçalhos controlados.

   > 💡 A URL acima está percent-encoded. Decodificada, equivale a pedir os parâmetros `Cache-Control=max-age=3600`, `Set-Cookie=teste=1` e `Strict-Transport-Security=max-age=31536000`. Cole-a **exatamente como está** na barra de endereço — copiar a versão decodificada quebra a query string.
2. Acessar `https://httpbin.org/gzip` para observar uma resposta compactada.
3. Recarregar a primeira URL uma vez, para verificar se o cookie `teste=1` passa a aparecer no cabeçalho `Cookie` do request.
4. Selecionar as sessões correspondentes e, na aba **Inspectors → Headers**, analisar request e response.

**Preencher a tabela abaixo no relatório** com os valores reais observados, agregando dados das sessões acima conforme o cabeçalho apareça:

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

**Pergunta 5.1:** O servidor retornou `Content-Encoding: gzip` (ou `br`)? Se sim, compare o valor de `Content-Length`, quando presente, com o tamanho do conteúdo visível na aba **Response → TextView**. O que explica a diferença?

**Pergunta 5.2:** O que acontece com um request onde o cliente envia `Accept: application/json` mas o recurso só existe em `text/html`? (Pesquisar o código de status correto.)

**Pergunta 5.3:** O cabeçalho `Strict-Transport-Security` apareceu? Qual seu papel em relação a downgrades para HTTP puro?

---

### Atividade 6 — HTTP vs HTTPS

**Objetivo:** observar, lado a lado, a diferença entre tráfego claro e cifrado.

1. **Desabilitar** temporariamente a decriptação HTTPS: *Tools → Options → HTTPS → desmarcar Decrypt HTTPS traffic*. Clicar OK.
2. Acessar `http://neverssl.com` (força HTTP puro).
3. Acessar `https://httpbin.org/get`.
4. Comparar ambas as sessões no Fiddler.

**Registrar no relatório:**
- Captura de tela de cada sessão com a aba **Raw** aberta.
- O método, o host e o que é (ou não é) visível em cada caso.

**Pergunta 6.1:** No caso do `https://httpbin.org/get`, que método HTTP aparece na sessão? Explique o que esse método faz e por que ele existe.

**Pergunta 6.2:** Com a decriptação desabilitada, quais informações sobre o tráfego HTTPS **ainda** são visíveis para o Fiddler e quais estão ocultas?

5. **Reabilitar** a decriptação e repetir o acesso a `https://httpbin.org/get`.

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

**Pergunta 7.2:** Que atributos o `Set-Cookie` trouxe (`Path`, `Domain`, `Expires`, `Max-Age`, `Secure`, `HttpOnly`, `SameSite`)? Para cada um presente, explique brevemente sua função. Se algum atributo da lista não apareceu, registre como **não observado**.

**Pergunta 7.3:** O cookie observado trouxe o atributo `Secure`? Se não trouxe, em que cenário ele poderia vazar? (Relacione com a Atividade 6.)

**Pergunta 7.4:** Na aba **Inspectors → Cookies** do Fiddler, compare o cookie armazenado com o que o servidor devolve no campo `cookies` do JSON da resposta. Eles coincidem?

---

### Atividade 8 — Manipulação com *breakpoints*

**Objetivo:** compreender o proxy como agente ativo, capaz de modificar tráfego em trânsito.

1. No Fiddler, habilitar **breakpoint de request**: menu **Rules → Automatic Breakpoints → Before Requests**.

   > ⚠️ **Atenção ao atalho F11.** Existe um atalho `F11` documentado, **mas só funciona com o foco na janela do Fiddler**. Se você acionar F11 com o foco no navegador, o navegador entra em modo tela cheia. Para evitar confusão, **prefira sempre o caminho de menu** nesta atividade.
2. No navegador, acessar `https://httpbin.org/user-agent`.
3. A sessão pausará no Fiddler com um ícone vermelho.
4. Na aba **Inspectors → Request → Raw** (clique dentro da área de texto para habilitar a edição), localizar a linha `User-Agent: ...` e substituir o valor por algo inventado, por exemplo:
   `User-Agent: LaboratorioRedes/1.0 (Aluno NOME)`
5. Clicar em **Run to Completion** (botão verde no topo do inspector).
6. Na resposta, observar o JSON retornado.

**Registrar no relatório:**
- Captura de tela da edição do request.
- Trecho do JSON de resposta mostrando o campo `user-agent`.

**Pergunta 8.1:** O servidor tem como detectar que o `User-Agent` foi forjado? Discuta.

**Pergunta 8.2:** Repita o exercício, mas desta vez habilite **breakpoint de response** (**Rules → Automatic Breakpoints → After Responses**). Acesse `https://httpbin.org/status/200` e, quando o Fiddler pausar, vá em **Inspectors → Response → Raw** e edite a primeira linha (a *status-line*) para `HTTP/1.1 404 Not Found`. Clique em **Run to Completion**. Descreva **o que VOCÊ observou** no navegador (o comportamento varia entre Chrome, Edge e Firefox — qualquer descrição honesta da tela renderizada é válida). Comente sobre o papel do proxy como *man-in-the-middle*: o servidor original retornou `200`, mas o navegador acreditou em `404`.

**Pergunta 8.3:** Desabilite todos os breakpoints (**Rules → Automatic Breakpoints → Disabled**, atalho **Shift+F11**) ao terminar.

---

## 4. Entrega

### O que entregar
- Arquivo **`relatorio.pdf`** gerado a partir do `relatorio.md` preenchido (ver instruções abaixo).
- Pasta **`evidencias/`** com as capturas de tela nomeadas por atividade (`atv1_sessao.png`, `atv3_post_raw.png`, etc.), incluindo obrigatoriamente a captura do `certmgr.msc` após a remoção do certificado (ver seção 5).

### Como gerar o PDF a partir do `relatorio.md`

O relatório é escrito em Markdown. Use uma das duas opções abaixo:

| Ferramenta | Quando usar | Como usar |
|---|---|---|
| **Markdown PDF** — extensão VS Code (**recomendado**) | Você já está com o repositório aberto no VS Code; funciona offline. | Instalar a extensão *"Markdown PDF"* (autor: yzane). Com o `relatorio.md` aberto, pressionar `Ctrl+Shift+P` → *"Markdown PDF: Export (pdf)"*. O arquivo é gerado ao lado do `.md`. |
| **Markdown to PDF** (md2pdf) — site público gratuito | Fallback se a extensão não puder ser instalada (laboratório com restrição). Não exige cadastro. | Abrir https://md2pdf.netlify.app, arrastar o arquivo `relatorio.md` para a área indicada (ou colar o conteúdo) e clicar em **Convert**. Salvar o PDF gerado. |

> **Dica:** antes de gerar o PDF, revise o preview do Markdown (VS Code: `Ctrl+Shift+V`) para confirmar que tabelas e blocos de código estão formatados corretamente. Imagens referenciadas em `evidencias/` devem estar **na mesma pasta** do `relatorio.md` para aparecerem no PDF.

### Como entregar
- Compactar a pasta do aluno em `SOBRENOME_NOME_RA_LAB_HTTP_FLUXOA.zip` contendo o `relatorio.pdf` e a pasta `evidencias/`.
   - Exemplo: `silva_joao_2023123_LAB_HTTP_FLUXOA.zip` (sem espaços, sem acentos, tudo minúsculo).
- Submeter no **Microsoft Teams**, atividade correspondente, até a data definida em aula.

### Critérios de avaliação

| Critério                                              | Peso |
|-------------------------------------------------------|------|
| Exatidão técnica das respostas                        | 40%  |
| Evidências coerentes (capturas legíveis e pertinentes) | 25%  |
| Profundidade da análise das questões de verificação   | 25%  |
| Organização e clareza do relatório                    | 10%  |

---

## 5. Encerramento obrigatório

Estes passos são parte da avaliação do Fluxo A. A remoção do certificado raiz é **obrigatória** por ser um risco de segurança mantê-lo instalado.

1. **Desabilitar a decriptação HTTPS:** *Tools → Options → HTTPS → desmarcar Decrypt HTTPS traffic* → OK.
2. **Remover o certificado raiz:**
   - **Windows:** `Win+R` → `certmgr.msc` → *Autoridades de Certificação Raiz Confiáveis → Certificados* → localizar **`DO_NOT_TRUST_FiddlerRoot`** → clique direito → *Excluir*.
   - **Firefox:** `about:preferences#privacy` → *Ver Certificados* → aba *Autoridades* → selecionar `DO_NOT_TRUST_FiddlerRoot` → *Excluir ou desconfiar*.
3. **Fechar o Fiddler** para liberar a porta 8888.
4. **Anexar ao relatório** uma captura de tela do `certmgr.msc` mostrando que o certificado **foi removido** (evidência de higiene de segurança).

> ⚠️ **Por que remover.** Um certificado raiz instalado permite que qualquer processo com a chave privada correspondente emita certificados válidos para **qualquer domínio** (bancos, e-mails, redes sociais) que o navegador da máquina aceitará sem alerta. Mantê-lo após o laboratório configura uma porta de entrada para ataques locais de interceptação.
