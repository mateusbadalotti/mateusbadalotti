# Publicação do badalotti.dev

Migração de 28 de outubro de 2026 para Cloudflare Workers Static Assets, no plano gratuito. A Cloudflare serve somente o conteúdo de `public/`, sem dependências, build, backend ou código de Worker por requisição. A preview foi validada antes do corte dos domínios.

| Recurso | Valor |
| --- | --- |
| Repositório | `ssh://git@github.com/mateusbadalotti/mateusbadalotti`. |
| Branch | `master`. |
| Commit de preparação enviado | `6c3515a`. |
| Worker | `badalotti-dev`. |
| Primeira versão estática | `1af80387`. |
| Conta Cloudflare | `e30c36ae3295685e80c11ff2472888a9`. |
| URL para conferência | https://badalotti-dev.mateus-badalotti.workers.dev/. |
| Domínio principal | https://badalotti.dev/. |
| Redirecionamento | https://www.badalotti.dev/ para o domínio principal. |

## Conteúdo e rotas

A homepage pública anterior diferia do `index.html` da raiz, mesmo após `git pull`. Por decisão explícita, `public/index.html` preserva a homepage que estava no site público, com layout compacto e referência à Insi. O `index.html` da raiz preserva o layout anterior e a alteração local do Tab Volume. Esse arquivo foi versionado, mas fica fora da pasta publicada. Edite `public/index.html` para atualizar o site.

`robots.txt`, `sitemap.xml` e `llms.txt` foram movidos para `public/` sem alterar seus bytes. A foto `profile.jpeg`, com 388.772 bytes, e o currículo `mateus_badalotti_resume.pdf`, com 41.290 bytes, estavam ausentes do checkout e foram recuperados do site público.

`public/audio-priority/index.html` restaura a página encontrada no checkout local `audio-priority-bar/site/index.html`. Antes da migração, `/audio-priority` redirecionava para uma URL HTTP e `/audio-priority/` retornava 403. As fontes do Google e as imagens externas do GitHub existentes nas páginas foram preservadas.

`public/_redirects` declara os redirecionamentos nativos:

```text
/raycast https://raycast.com/?via=mateus-petkovicz 301
/raycast/* https://raycast.com/?via=mateus-petkovicz 301
```

O servidor anterior descartava o caminho adicional e a query recebidos em `/raycast`. A preview confirmou esse comportamento com status 301 para `/raycast`, a variante com barra, caminhos filhos e queries. O destino permanece exatamente `https://raycast.com/?via=mateus-petkovicz`. A Cloudflare interpreta `_redirects` como configuração e não expõe seu conteúdo como asset.

Por decisão explícita, URLs inexistentes passam a retornar HTTP 404. Antes, o Nginx devolvia a homepage com 200, inclusive para PDFs e CSS ausentes. Essa correção evita manter o fallback de SPA para um site de arquivos estáticos.

## Configuração

`wrangler.json` registra o Worker `badalotti-dev`, a pasta `public/` e os Custom Domains `badalotti.dev` e `www.badalotti.dev`. Não há propriedade `main` nem script de Worker. Os assets usam `html_handling: auto-trailing-slash` e `not_found_handling: none`.

O processo usa o navegador, sem instalar Wrangler. O upload pelo painel não lê o `wrangler.json` local. Mantenha os parâmetros do painel iguais aos desse arquivo.

Na zona `badalotti.dev`, o Single Redirect nativo `Redirect www to apex` usa:

| Campo | Valor |
| --- | --- |
| Request URL | `*://www.badalotti.dev/*`. |
| Target URL | `https://badalotti.dev/${2}`. |
| Status | `301`. |
| Preserve query string | Ativado. |

Os dois Custom Domains estão ligados à publicação `badalotti-dev`. As respostas públicas confirmaram `www` com 301 preservando caminho e query, e HTTP no domínio principal com 301 para HTTPS. `SSL/TLS > Edge Certificates > Always Use HTTPS` já estava ativado e foi mantido.

## Atualização pelo repositório

Execute `git pull --ff-only` no checkout da branch `master`. Edite os arquivos em `public/`. Este repositório não tem `package.json` ou comandos próprios de build, lint ou teste.

1. Confira a sintaxe do JavaScript inline das páginas com Node.js e verifique os links e assets alterados.
2. No painel Cloudflare da conta acima, abra o Worker existente `badalotti-dev` e selecione `New deployment`.
3. Envie somente `public/`. Não envie a raiz, o `index.html` antigo, `.git`, arquivos de ambiente ou arquivos operacionais.
4. Em `Advanced settings`, use a raiz dos arquivos enviados como `/`, `auto-trailing-slash` e `none` para arquivos ausentes.
5. Confira e publique. Uma nova implantação no Worker já conectado aos domínios atualiza a produção. Valide localmente antes desse passo.
6. Confira a URL `workers.dev`, os domínios finais, a foto, o PDF, `robots.txt`, `sitemap.xml`, `llms.txt`, `/audio-priority`, `/raycast`, os redirecionamentos e um caminho inexistente.
7. Versione e envie as alterações na branch `master`, seguindo o fluxo do repositório.

Reutilize o Worker existente. Não crie um projeto por atualização. Não há integração de CI criada por esta migração nem necessidade de novos tokens ou GitHub Secrets para o processo pelo navegador.

## Estado e validação

Consulte `Workers & Pages > badalotti-dev > Deployments` e `Domains` no painel da conta acima. O conteúdo está no commit `6c3515a`, enviado a `origin/master`, e na versão estática `1af80387`. A sintaxe do JavaScript inline da homepage pública, da homepage preservada na raiz e da página Audio Priority passou em `node --check`. `git diff --check` passou para essa preparação.

A preview passou pelas seguintes verificações antes do corte:

- Os sete arquivos de conteúdo retornaram 200, com bytes iguais ao checkout e tipos MIME corretos. Homepage, arquivos SEO, foto e PDF também conferiram com a captura anterior do site público.
- Os redirecionamentos de `/raycast` preservaram o destino fixo e não expuseram `_redirects` como arquivo.
- `/audio-priority` retornou 307 para o caminho relativo com barra, preservando a query. `/audio-priority/` retornou a página com 200.
- Caminhos inexistentes e tentativas de acessar arquivos operacionais retornaram 404.
- Os screenshots confirmaram a homepage preservada e a página Audio Priority restaurada, sem erros de JavaScript. As fontes do Google e imagens do GitHub carregaram no navegador.

A validação pública final confirmou HTTPS válido no domínio principal e em `www`, HTTP com redirecionamento para HTTPS e `www` com 301 para o domínio principal, preservando caminho e query, inclusive caracteres codificados. Os sete arquivos de conteúdo retornaram 200, com bytes e tipos MIME corretos e iguais a `public/`. Homepage, arquivos SEO, foto e PDF também conferiram com a captura anterior do site público.

As variantes de `/raycast` retornaram 301 para o destino fixo. `/audio-priority` retornou 307 para o caminho com barra, preservando a query, e `/audio-priority/` retornou a página com 200. Caminhos ausentes, `/.env`, `/.git/HEAD` e `/.git/config` retornaram 404. Os screenshots de desktop e celular foram inspecionados e comparados com a baseline. A homepage permaneceu preservada, a página Audio Priority funcionou e não houve erros de JavaScript.

O painel confirmou que esta publicação contém somente assets e não oferece métricas de execução de Worker. Não há logs de um script por requisição. Use estados de implantação e domínio, respostas HTTP e o Analytics da zona para investigar o atendimento público. Não habilite recursos pagos para obter logs.

As evidências anteriores estão fora do repositório, em `/Users/mateus/Dev/Personal/.cloudflare-migration-2026-10-28/baseline/badalotti.dev`. Elas incluem conteúdo, assets recuperados, respostas HTTP, diferenças do checkout e screenshot anterior.

Os resultados finais estão em `/Users/mateus/Dev/Personal/.cloudflare-migration-2026-10-28/badalotti-final/validation.txt`, com os screenshots no mesmo diretório. Não restaram checks pendentes desta migração.

## DNS e reversão

Os exports DNS estão fora do repositório, em `/Users/mateus/Dev/Personal/.cloudflare-migration-2026-10-28/dns-before/badalotti.dev.txt` e no arquivo correspondente em `dns-after/`. Os dois registros web anteriores eram CNAMEs com proxy ativado e TTL `Auto`, apontando para `5fb848d9-1c7b-4524-880c-36f1579b1703.cfargotunnel.com`.

- `badalotti.dev`.
- `www.badalotti.dev`.

O corte substituiu somente esses CNAMEs web pelos vínculos ao Worker. O painel DNS confirma dois registros do tipo Worker associados a `badalotti-dev`. O export BIND omite esses registros gerenciados. Confira também os vínculos em `Domains` e as respostas públicas.

A comparação dos exports anteriores e posteriores confirmou os três MX, o SPF e o DKIM sem diferenças. Os MX são `route3.mx.cloudflare.net` com prioridade 60, `route2.mx.cloudflare.net` com prioridade 42 e `route1.mx.cloudflare.net` com prioridade 85. Não existia registro DMARC no inventário anterior, e esse estado foi mantido. Após o corte, o painel de Email Routing confirmou `Routing status: Enabled` e `DNS records: Locked`. Nenhuma regra, destino ou configuração de e-mail foi alterada.

Para voltar ao túnel após o corte:

1. Remova somente os dois Custom Domains do Worker `badalotti-dev`.
2. Recrie os dois CNAMEs acima com o destino anterior, proxy ativado e TTL `Auto`.
3. Desative ou remova somente a regra nativa de `www` criada pela migração para restaurar o comportamento anterior, que servia a homepage em `www` sem redirecionar.
4. Confira HTTPS, conteúdo, rotas e registros de e-mail. Considere o cache DNS ao comparar respostas.

Não reimporte a zona inteira. Nenhuma configuração do homelab ou do túnel foi alterada. A configuração antiga permanece disponível para reversão. O rollback não exige reinstalar ou reconfigurar o túnel.

## Limites e custo

Os [assets estáticos têm requisições gratuitas e ilimitadas, sem custo adicional de armazenamento](https://developers.cloudflare.com/workers/static-assets/billing-and-limitations/). No plano gratuito, os [limites atuais](https://developers.cloudflare.com/workers/platform/limits/) incluem 20.000 arquivos por versão e 25 MiB por arquivo. O atendimento direto dos assets não consome a franquia diária de execução de scripts Workers.

O Single Redirect de `www` ocupa uma das dez [regras gratuitas disponíveis](https://developers.cloudflare.com/rules/url-forwarding/single-redirects/) na zona. O arquivo `_redirects` suporta até 2.000 regras estáticas e 100 dinâmicas, totalizando 2.100. Nenhum plano ou recurso pago foi habilitado, e nenhum software foi instalado. Verifique novamente os limites antes de ampliar a arquitetura.

## Limpeza futura no homelab

Esta limpeza fica para uma sessão com acesso ao servidor.

- Remover somente `badalotti.dev` e `www.badalotti.dev` da configuração do túnel `homelab`, preservando outros hostnames e a regra final de fallback.
- Remover somente os blocos Nginx exclusivos desse site.
- Revisar os scripts antigos de deploy e rollback antes de removê-los ou alterá-los.
