# Guia de publicação — Landing Page Cicle Dois Irmãos

Site: **https://cicledoisirmaos.com.br**
Repositório: https://github.com/rochatales/landing-cicle-dois-irmaos (branch `main`)
Hospedagem prevista: **Hostinger** (Apache/LiteSpeed, painel hPanel)

O site é **HTML/CSS/JS puro**: não tem build, banco de dados, PHP nem Node. Publicar é copiar os arquivos para a pasta pública da hospedagem.

---

## 1. Antes de começar — o que precisa estar em mãos

| Item | Quem fornece | Para quê |
|---|---|---|
| Acesso ao hPanel da Hostinger (ou usuário FTP) | Cliente | Subir os arquivos e ativar SSL |
| Acesso ao DNS do domínio (Registro.br ou onde estiver) | Cliente | Apontar o domínio para a Hostinger |
| Se o cliente usa e-mail `@cicledoisirmaos.com.br` | Cliente | **Não apagar os registros MX** ao mexer no DNS |
| URL que recebe os leads (RAFE/CRM, Make, n8n…) | Cliente / Roki Media | Sem ela o formulário não envia leads (ver seção 5) |
| IDs de GTM/GA4, Meta Pixel, TikTok Pixel | Cliente / Roki Media | Medição das campanhas (ver seção 5) |

---

## 2. Arquivos que vão para o servidor

Tudo vai para a **raiz do site** (`public_html` do domínio), mantendo as pastas:

```
public_html/
├── .htaccess                  ← obrigatório (HTTPS, cache, segurança)
├── index.html
├── guia-nova-mobilidade.pdf   ← o e-book; o nome precisa ser exatamente este
├── robots.txt
├── sitemap.xml
└── assets/
    ├── fonts/  (Klavika .woff2)
    └── img/
```

**Não precisam subir:** `ROADMAP.md`, `MOTION.md`, `DEPLOY.md`, `_headers`, `.nojekyll`, `.gitignore`.
Se forem enviados por engano (por exemplo, no deploy via Git), o `.htaccess` já bloqueia o acesso público aos `.md` e ao `_headers`.

> O `.htaccess` começa com ponto e fica **oculto** em muitos programas. Confirme que ele foi enviado (no Gerenciador de Arquivos da Hostinger, ative "mostrar arquivos ocultos").

---

## 3. Publicar na Hostinger

Antes de tudo: se já existir um site em `public_html`, **faça backup** (baixe a pasta ou use Backups no hPanel) e apague o conteúdo antigo, incluindo `default.php` / `index.php` padrão da Hostinger, que teria prioridade sobre o `index.html`.

### Opção A — Deploy via Git (recomendado; atualizações ficam fáceis)

1. hPanel → **Sites** → selecione o domínio → **Avançado → Git**.
2. Repositório: `https://github.com/rochatales/landing-cicle-dois-irmaos.git` · Branch: `main` · Diretório: deixe vazio (instala em `public_html`).
   - A Hostinger exige que `public_html` esteja **vazia** para o primeiro deploy.
3. Clique em **Criar** e depois em **Deploy**.
4. Para atualizar no futuro: novo push em `main` → hPanel → Git → **Deploy**. Opcionalmente, configure o *Auto Deployment* (a Hostinger gera uma URL de webhook para colar em GitHub → Settings → Webhooks).

### Opção B — Gerenciador de Arquivos (upload manual)

1. No GitHub: **Code → Download ZIP** (ou compacte os arquivos da seção 2).
2. hPanel → **Gerenciador de Arquivos** → `public_html` → **Upload** do .zip.
3. Clique com o botão direito no .zip → **Extrair**. Se ele criar uma subpasta (ex.: `landing-cicle-dois-irmaos-main/`), mova o **conteúdo** dela para a raiz de `public_html` e apague a subpasta e o .zip.

### Opção C — FTP

Use FileZilla com os dados de FTP do hPanel (**Arquivos → Contas FTP**) e envie o conteúdo da seção 2 para `public_html`.

---

## 4. Domínio e HTTPS

### 4.1 Apontar o domínio

No hPanel, o domínio precisa estar adicionado ao plano (**Sites → Adicionar site** ou **Domínios**). Depois, escolha **uma** das formas:

- **Nameservers (mais simples):** no Registro.br, em *DNS → Alterar servidores DNS*, coloque os nameservers que o hPanel mostrar (normalmente `ns1.dns-parking.com` e `ns2.dns-parking.com`). A partir daí o DNS é gerenciado na Hostinger.
  - Se o cliente tem e-mail no domínio hospedado em outro lugar (Google Workspace, Zoho, etc.), **recrie na Hostinger os registros MX/TXT** antes de trocar os nameservers.
- **Só registros A (mantém o DNS onde está):** crie/edite
  - `A` · `@` → IP do plano (hPanel → *Detalhes do plano*)
  - `A` ou `CNAME` · `www` → mesmo IP / `cicledoisirmaos.com.br`

A propagação leva de minutos a algumas horas (até 24–48 h na troca de nameservers). Para conferir: https://dnschecker.org

### 4.2 Certificado SSL

hPanel → **Segurança → SSL** → instale o SSL gratuito (Let's Encrypt) para `cicledoisirmaos.com.br` **e** `www.cicledoisirmaos.com.br`.

> **Importante:** o `.htaccess` força HTTPS e redireciona `www` → sem `www`. Sem o certificado ativo, o navegador mostra erro de segurança. Se precisar testar antes de o SSL sair, comente temporariamente as linhas do bloco `mod_rewrite` no `.htaccess`.

---

## 5. Configurações que faltam no `index.html`

### 5.1 Destino dos leads — **obrigatório antes de ligar as campanhas**

No fim do `index.html`, bloco `CONFIG`:

```js
var CONFIG = {
  endpoint: "",   // ← colar aqui a URL que recebe o lead (POST JSON)
  whatsapp: "5524988658380",
  pdf: "guia-nova-mobilidade.pdf"
};
```

Com `endpoint` vazio a página funciona em **modo demonstração**: entrega o PDF, mas **o lead não é enviado a lugar nenhum**.

O destino precisa:
- aceitar `POST` com `Content-Type: application/json`;
- liberar **CORS** para `https://cicledoisirmaos.com.br` (responder ao `OPTIONS` e enviar `Access-Control-Allow-Origin`).

Campos enviados: `nome`, `email`, `whatsapp`, `interesses[]`, `consentimento`, `utm_source/medium/campaign/content/term`, `gclid`, `fbclid`, `ttclid`, `timestamp`.

### 5.2 Rastreamento (Google, Meta, TikTok)

No `<head>` do `index.html` há um comentário marcando onde colar as tags (o ideal é um único container do **Google Tag Manager** carregando GA4/Google Ads, Meta Pixel e TikTok Pixel).

A página já dispara estes eventos — só falta configurar as conversões nas plataformas:
- `generate_lead` (dataLayer/gtag), `fbq('track','Lead')`, `ttq.track('SubmitForm')` — ao enviar o formulário;
- `whatsapp_click`, `fbq('track','Contact')`, `ttq.track('Contact')` — ao clicar no WhatsApp.

### 5.3 Trocar o e-book

Substitua `guia-nova-mobilidade.pdf` mantendo **exatamente o mesmo nome**. O visitante recebe o arquivo salvo como `Guia_Nova_Mobilidade_Cicle_Dois_Irmaos.pdf`. O cache do PDF é de 1 dia; para forçar a versão nova na hora, limpe o cache em hPanel → *Desempenho → Cache*.

---

## 6. Checklist de testes depois de publicar

- [ ] `http://cicledoisirmaos.com.br` e `https://www.cicledoisirmaos.com.br` redirecionam para `https://cicledoisirmaos.com.br/`
- [ ] Cadeado HTTPS sem aviso
- [ ] Imagens e fontes carregam (nada quebrado no console do navegador — F12)
- [ ] Formulário: preencher e enviar → mensagem de obrigado aparece → **botão "Baixar o PDF" baixa o e-book**
- [ ] O lead chegou no CRM/webhook com os UTMs (testar com `?utm_source=google`, `?utm_source=instagram`, `?utm_source=tiktok`)
- [ ] Validações: WhatsApp inválido, sem interesse marcado, sem consentimento
- [ ] Botões de WhatsApp abrem a conversa com a mensagem certa
- [ ] Pixels disparando (Google Tag Assistant, Meta Pixel Helper, TikTok Pixel Helper)
- [ ] `https://cicledoisirmaos.com.br/ROADMAP.md` retorna **403** (arquivos internos bloqueados)
- [ ] Testar em iPhone (Safari), Android (Chrome) e desktop
- [ ] Lighthouse mobile ≥ 90 em desempenho e acessibilidade
- [ ] Enviar o `sitemap.xml` no Google Search Console

---

## 7. Problemas comuns

| Sintoma | Causa provável | Solução |
|---|---|---|
| Aparece a página padrão da Hostinger | Sobrou `default.php`/`index.php` em `public_html` | Apagar esses arquivos |
| "Muitos redirecionamentos" | SSL forçado em dois lugares (hPanel + `.htaccess`) ou proxy Cloudflare em modo *Flexible* | Desativar "Forçar HTTPS" do hPanel (o `.htaccess` já faz) ou usar *Full (strict)* na Cloudflare |
| Erro de certificado | SSL ainda não emitido ou domínio não propagado | Aguardar a propagação e reinstalar o SSL no hPanel |
| Erro 500 | Diretiva do `.htaccess` não suportada | Comentar blocos um a um para achar a linha; avisar a Roki Media |
| Botão do PDF abre a própria página | Arquivo do PDF ausente ou com outro nome (o 404 cai no `index.html`) | Conferir `guia-nova-mobilidade.pdf` na raiz |
| Lead não chega | `endpoint` vazio ou bloqueado por CORS | Ver seção 5.1; checar o console (F12 → Console/Rede) |
| Site antigo continua aparecendo | Cache do navegador/LiteSpeed | Limpar cache no hPanel e testar em aba anônima |

---

Responsável técnico: **Roki Media** — Tales Rocha.
