
# 🕷️ Web Scraper com n8n + Browserless + Puppeteer

[![n8n](https://img.shields.io/badge/n8n-automated-orange?style=flat&logo=n8n)](https://n8n.io)
[![Browserless](https://img.shields.io/badge/browserless-puppeteer-blue?logo=google-chrome)](https://www.browserless.io)
[![Docker Compose](https://img.shields.io/badge/docker--compose-ready-2496ED?logo=docker)](https://docs.docker.com/compose/)
[![Made with ❤️ by Jonathan da Cruz](https://img.shields.io/badge/feito%20por-Jonathan%20da%20Cruz-red)](https://jonathandacruz.com.br)

> Automatize a extração de links e preços com Puppeteer sem instalar nada localmente, usando n8n + Browserless.

🎥 **Canal no YouTube:** [youtube.com/@jonathandacruz](https://www.youtube.com/@jonathandacruz)

---

## 📦 Requisitos

- Docker + Docker Compose
- n8n rodando localmente (ou em container)
- Browserless self-hosted com token de acesso
- O arquivo `scrape.json` (workflow n8n exportado)

---

## ⚙️ Setup com Docker Compose

```yaml
version: '3.8'

services:
  n8n:
    image: n8nio/n8n
    ports:
      - "5678:5678"
    environment:
      - GENERIC_TIMEZONE=America/Sao_Paulo
    volumes:
      - ./n8n:/home/node/.n8n

  browserless:
    image: ghcr.io/browserless/chromium
    ports:
      - "3000:3000"
    environment:
      - TOKEN=01b27f141e9119d83cb3dde31ff4385c
      - DEFAULT_BLOCK_ADS=true
      - DEFAULT_LAUNCH_ARGS=["--no-sandbox"]
      - CONNECTION_TIMEOUT=60000
    restart: unless-stopped
```

---

## 🔹 Visão geral do processo

1. Inicia com **gatilho manual**
2. Usa **Browserless** para acessar: `https://jonathandacruz.com.br/`
3. Extrai todos os elementos `<a>`
4. Remove duplicatas e filtra apenas links que **contêm “ebook”**
5. Raspa os links de cada página encontrada
6. Verifica se há links que levam ao **Hotmart**
7. Se houver, chama o endpoint `/function` da Browserless para **extrair o preço**
8. Exemplo extra: faz **login automático** em uma página de teste (Herokuapp)

---

## 🗂️ Estrutura dos principais nós

| Etapa                 | Função                                                                 |
|-----------------------|------------------------------------------------------------------------|
| `raspando`            | Raspa os links da página inicial usando o endpoint `/scrape`           |
| `Code + Remove Duplicates` | Filtra e extrai os atributos úteis dos links `<a>`                      |
| `If`                  | Verifica se os links contêm a palavra `ebook`                          |
| `raspandoEbook`       | Roda novo scraping nos links encontrados                               |
| `getHref`             | Extrai `href` dos elementos `<a>`                                      |
| `existeHotmart`       | Verifica se o link aponta para um domínio `hotmart.com`                |
| `buscaValor`          | Usa Puppeteer via `/function` para extrair o `data-testid="product-price"` |
| `fazendoLogin`        | Demonstra como automatizar login com Puppeteer                         |

---

## 🔍 Exemplo de uso do endpoint `/function`

```js
export default async function ({ page }) {
  await page.setExtraHTTPHeaders({ "Accept-Language": "pt-BR" });
  await page.goto("https://pay.hotmart.com/S11941831Y", { waitUntil: "networkidle2" });
  await page.waitForSelector("[data-testid='product-price']", { timeout: 10000 });
  const price = await page.$eval("[data-testid='product-price']", el => el.textContent.trim());
  return {
    data: price,
    type: "text/plain"
  };
}
```

---

## 🧪 Como importar o fluxo no n8n

1. Acesse seu n8n: [http://localhost:5678](http://localhost:5678)
2. Vá em **“Workflows” > “Import”**
3. Selecione o arquivo `scrape.json`
4. Atualize os tokens e URLs se necessário
5. Clique em **"Execute workflow"**

---

## 🔐 Segurança e Boas práticas

- **NUNCA** exponha o token da Browserless em repositórios públicos
- Respeite o `robots.txt` e os termos de uso das páginas que estiver raspando
- Use delays e limite o número de requisições em produção
- Configure variáveis de ambiente no n8n em vez de colocar valores diretamente nos nós

---

## 👨‍💻 Autor

Feito com ❤️ por [Jonathan da Cruz]([https://jonathandacruz.com.br](https://www.youtube.com/@jonathandacruz))

---

## 📃 Licença

Distribuído sob a licença **MIT**. Veja o arquivo `LICENSE` para mais detalhes.
