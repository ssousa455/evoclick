Claro! Vou preparar um **README.md** atualizado, mantendo os créditos ao repositório original, adicionando um disclaimer sobre os desafios e incluindo um guia passo a passo para instalação em VPS com **Dokploy**.

---

```markdown
# 🚀 EvoClick – Self‑Hosted Click Tracker for Affiliates

> **Créditos:** Este projeto é baseado no [EvoClick](https://github.com/EricFrancis12/evoclick) criado por [EricFrancis12](https://github.com/EricFrancis12). Agradecemos imensamente pelo trabalho incrível!  
> Este fork/adaptação foi feito para facilitar a implantação em **VPS (Virtual Private Server)** com Docker e Dokploy, incluindo correções de compatibilidade e um guia detalhado para evitar dores de cabeça.

---

## 📌 Disclaimer – Desafios enfrentados

Durante a instalação em VPS, enfrentamos alguns desafios comuns:

- **Prisma e OpenSSL:** A imagem `node:18-alpine` não possui a biblioteca `libssl.so.1.1`, necessária para o motor do Prisma. A solução foi trocar a imagem base para `node:18-bullseye` (Debian) que já inclui OpenSSL 1.1.
- **Migrações do Banco:** O banco PostgreSQL inicia vazio. É preciso rodar `npx prisma db push` para criar as tabelas. Incluímos esse comando no `command` do container para que ele execute automaticamente a cada inicialização.
- **Proxy da Cloudflare:** Se você usar o proxy (nuvem laranja), pode haver interferência com o SSL do Traefik. Recomendamos manter os subdomínios em **DNS only** (nuvem cinza) durante os testes.
- **Domínios e Roteamento:** O Dokploy pode gerenciar domínios via interface (UI) ou via labels no `docker-compose.yml`. Escolha **um método apenas** para evitar conflitos.

Este guia foi escrito com base nessas soluções, para que você possa ter uma instalação tranquila.

---

## 🧰 Pré‑requisitos

- **VPS** com Ubuntu 20.04/22.04 (ou similar) – recomendamos pelo menos 2 vCPUs e 4 GB de RAM.
- **Docker** e **Docker Compose** instalados.
- **Dokploy** instalado na VPS (opcional, mas facilita o gerenciamento).
- Um **domínio** apontando para o IP da VPS (com registros A para os subdomínios que você vai usar).
- Conta no **GitHub** para armazenar o código (recomendado para integração com Dokploy).

---

## 📝 Instalação em VPS com Dokploy (Recomendado)

### 1. Clone este repositório (ou faça fork do original)

```bash
git clone https://github.com/seu-usuario/evoclick.git
cd evoclick
```

### 2. Configure as variáveis de ambiente

Crie um arquivo `.env` na raiz do projeto com o seguinte conteúdo (substitua os valores pelos seus):

```env
# Domínios (use os SEUS subdomínios)
APP_DOMAIN=evoclick.seudominio.com
API_DOMAIN=evoclick-api.seudominio.com

# Credenciais de login (você vai usar pra entrar no painel)
ROOT_USERNAME=admin
ROOT_PASSWORD=UmaSenhaBemForte123!

# Senha do banco (qualquer string forte)
POSTGRES_PASSWORD=OutraSenhaForte456!

# JWT Secret (gere uma string aleatória de 64+ caracteres)
JWT_SECRET=cole-aqui-uma-string-aleatoria-bem-longa

# (opcional) Portas e outras configurações
PORT=3000
API_PORT=3001
```

### 3. Ajuste os Dockerfiles para usar a imagem base correta

No `Dockerfile` (do `next-app`) e no `Dockerfile.api`, altere a primeira linha de:

```dockerfile
FROM node:18-alpine
```

para:

```dockerfile
FROM node:18-bullseye
```

Isso resolve o problema do Prisma com OpenSSL.

### 4. Ajuste o `docker-compose.yml`

Adicione o comando de migração no serviço `next-app` (não no `api`):

```yaml
services:
  next-app:
    # ... outras configurações
    command: sh -c "npx prisma db push && npm start"
    # ⬆️ Isso garante que as tabelas sejam criadas ao iniciar

  api:
    # ... outras configurações
    # ⬇️ NÃO adicione command aqui, deixe o CMD do Dockerfile.api padrão
```

### 5. Faça o deploy no Dokploy

- Crie um novo **Projeto** no Dokploy.
- Conecte seu repositório GitHub.
- Selecione o serviço e configure as variáveis de ambiente (as mesmas do `.env`).
- Clique em **Deploy**.

> 🔁 Se você preferir não usar o Dokploy, pode simplesmente executar `docker compose up -d` na VPS, mas precisará configurar um proxy reverso (como Traefik) manualmente para HTTPS.

### 6. Configure os domínios no Dokploy

No painel do Dokploy, vá até o serviço `evoclick` → aba **Domains** e adicione:

- **next-app** → `evoclick.seudominio.com` (porta 3000, HTTPS ativado)
- **api** → `evoclick-api.seudominio.com` (porta 3001, HTTPS ativado)

> ⚠️ **Importante:** Se você usou labels no `docker-compose.yml`, **não** adicione domínios pela UI (ou vice‑versa) para evitar conflitos. Escolha um método só.

### 7. Verifique os logs e acesse

- Acompanhe os logs do `next-app` para confirmar que as migrações rodaram e o servidor iniciou.
- Acesse `https://evoclick.seudominio.com` e faça login com as credenciais definidas no `.env`.

---

## 🧪 Teste rápido (sem Dokploy)

Se quiser testar localmente ou em uma VPS sem Dokploy:

1. Configure o `.env` e os Dockerfiles como descrito.
2. Execute:

```bash
docker compose up -d
```

3. O sistema estará disponível em `http://localhost:3000` (ou no IP da VPS, se você configurar o firewall e um proxy).

---

## 📂 Estrutura do Projeto (resumida)

- `next-app/` – Frontend e API em Next.js (Node.js)
- `api/` – API em Go (para alto desempenho em cliques)
- `docker-compose.yml` – Orquestração dos containers
- `Dockerfile` e `Dockerfile.api` – Definições das imagens

---

## 🤝 Agradecimentos e Contribuições

- **EricFrancis12** – criador do projeto original.
- Comunidade open‑source – por compartilhar conhecimento sobre Docker, Prisma e Traefik.

Se você encontrar problemas ou tiver sugestões de melhoria, sinta‑se à vontade para abrir uma issue ou enviar um PR.

---

## 📄 Licença

Este projeto é disponibilizado sob a licença MIT (consulte o arquivo `LICENSE` do repositório original).

---

**Boa sorte com suas campanhas!** 🎯
```

---

### ✨ Como usar este README

1. Substitua o conteúdo do seu `README.md` atual por este.
2. Ajuste os links (URL do seu repositório, exemplo de domínio etc.).
3. Se desejar, traduza para o inglês (posso ajudar se precisar).
4. Faça commit e push.

Se quiser, posso ajustar algum trecho ou adicionar mais detalhes sobre algum passo específico. É só avisar! 🚀
