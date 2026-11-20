# 📸 Galeria

Aplicação de galeria de fotos com **álbuns**, feita como projeto de estudo da [RocketSeat](https://www.rocketseat.com.br/).
O repositório tem o **frontend** em React e uma **API** em Fastify, que guarda os dados num arquivo JSON local.

> 🚧 **Projeto em desenvolvimento.** A API já funciona. O frontend ainda está no começo: hoje só exibe a tela inicial.

---

## 🧱 Tecnologias

### Frontend

| Tecnologia | Uso |
| --- | --- |
| [React 19](https://react.dev/) + [TypeScript](https://www.typescriptlang.org/) | Construção da interface com tipagem forte |
| [Vite](https://vite.dev/) | Servidor de desenvolvimento e build |
| [Tailwind CSS v4](https://tailwindcss.com/) (`@tailwindcss/vite`) | Estilização |
| [React Router DOM](https://reactrouter.com/) | Rotas da aplicação |
| [Oxlint](https://oxc.rs/) | Lint |

Estas bibliotecas já estão instaladas e serão usadas nas próximas etapas:

- [TanStack Query](https://tanstack.com/query) + [Axios](https://axios-http.com/): consumo da API e cache dos dados
- [React Hook Form resolvers](https://github.com/react-hook-form/resolvers) + [Zod](https://zod.dev/): validação de formulários
- [Radix UI Dialog](https://www.radix-ui.com/): modais acessíveis
- [nuqs](https://nuqs.47ng.com/): estado sincronizado com a URL (filtros e busca)
- [Sonner](https://sonner.emilkowal.ski/): notificações (toasts)
- [classnames](https://github.com/JedWatson/classnames) e [tw-animate-css](https://github.com/Wombosvideo/tw-animate-css): classes condicionais e animações

### Backend

| Tecnologia | Uso |
| --- | --- |
| [Fastify 4](https://fastify.dev/) | Servidor HTTP |
| `@fastify/cors` | Libera o acesso do frontend à API |
| `@fastify/multipart` | Upload de imagens (limite de 50 MB) |
| `@fastify/static` | Serve as imagens em `/images/` |
| [Zod](https://zod.dev/) | Validação de `params`, `query` e `body` |
| [tsup](https://tsup.egoist.dev/) | Build do servidor (ESM) |

**Banco de dados:** arquivo `data/db.json`, criado sozinho na primeira execução. As imagens enviadas ficam em `data/images/`.

---

## 📂 Estrutura do projeto

```
galeria/
├── data/
│   ├── db.json                # "Banco de dados" em JSON
│   └── images/                # Imagens enviadas (criada no primeiro upload)
├── server/                    # API Fastify
│   ├── main.ts                # Inicia o servidor, registra plugins, serviços e rotas
│   ├── models.ts              # Tipos das entidades (Photo, Album, PhotoOnAlbum, Database)
│   ├── albums/
│   │   ├── albums-interfaces.ts   # Schemas Zod e tipos derivados
│   │   ├── albums-routes.ts       # Rotas /albums
│   │   └── albums-service.ts      # Regras de negócio dos álbuns
│   ├── photos/
│   │   ├── photos-interfaces.ts   # Schemas Zod e tipos derivados
│   │   ├── photos-routes.ts       # Rotas /photos
│   │   └── photos-service.ts      # Regras de negócio das fotos
│   └── services/
│       ├── database-service.ts    # Leitura e escrita do db.json
│       └── image-service.ts       # Salvamento e remoção dos arquivos de imagem
├── src/                       # Frontend React
│   ├── main.tsx               # Ponto de entrada (StrictMode)
│   ├── App.tsx                # Componente raiz
│   └── index.css              # Import do Tailwind
├── index.html
├── vite.config.ts             # Vite + React + Tailwind
├── tsup.config.ts             # Build do servidor → server/dist
├── tsconfig.app.json          # TS do frontend
├── tsconfig.node.json         # TS dos arquivos de configuração
└── tsconfig.server.json       # TS do servidor
```

O backend é dividido em camadas: **rotas** (validam a entrada com Zod e respondem no HTTP), **services** (regras de negócio) e **DatabaseService/ImageService** (acesso ao disco). Os serviços são criados em `main.ts` e passados para as rotas por injeção de dependência.

---

## 🗃️ Modelo de dados

```ts
interface Photo {
  id: string;          // UUID
  title: string;
  imageId?: string;    // Nome do arquivo em data/images (ex.: "<uuid>.png")
  albumsIds?: string[];
}

interface Album {
  id: string;          // UUID
  title: string;
}

interface PhotoOnAlbum {  // Relação N:N entre fotos e álbuns
  photoId: string;
  albumId: string;
}
```

### Regras de negócio

- Uma foto pode estar em **vários álbuns**, e um álbum pode ter **várias fotos**. A ligação fica em `photosOnAlbums`.
- A foto é criada **primeiro só com o título**. A imagem é enviada depois, numa segunda chamada.
- Formatos de imagem aceitos: **`.jpg`, `.jpeg` e `.png`**, com no máximo **50 MB**.
- Enviar uma nova imagem para uma foto **substitui** a anterior.
- Apagar uma foto apaga também o arquivo de imagem e as ligações dela com álbuns.
- Apagar um álbum remove só as ligações. **As fotos continuam existindo.**
- Título do álbum: de 1 a 100 caracteres. Título da foto: de 1 a 255 caracteres.
- Todos os IDs são UUIDs e são validados nas rotas.

---

## 🔌 API

URL base: `http://localhost:5799`

### Álbuns

| Método | Rota | Descrição | Body |
| --- | --- | --- | --- |
| `GET` | `/albums` | Lista todos os álbuns | — |
| `GET` | `/albums/:id` | Busca um álbum | — |
| `POST` | `/albums` | Cria um álbum | `{ "title": string }` |
| `DELETE` | `/albums/:id` | Remove um álbum | — |

### Fotos

| Método | Rota | Descrição | Body |
| --- | --- | --- | --- |
| `GET` | `/photos?albumId=&q=` | Lista as fotos, com filtro opcional por álbum (`albumId`) e busca no título (`q`) | — |
| `GET` | `/photos/:id` | Busca uma foto com seus álbuns, `nextPhotoId` e `previousPhotoId` (para navegar entre fotos) | — |
| `POST` | `/photos` | Cria uma foto | `{ "title": string, "albumsIds"?: string[] }` |
| `POST` | `/photos/:id/image` | Envia a imagem da foto | `multipart/form-data` (arquivo) |
| `PATCH` | `/photos/:id` | Altera o título | `{ "title": string }` |
| `PUT` | `/photos/:id/albums` | Define em quais álbuns a foto está (adiciona e remove para bater com a lista enviada) | `{ "albumsIds": string[] }` |
| `DELETE` | `/photos/:id` | Remove a foto e a imagem dela | — |

### Outros

| Método | Rota | Descrição |
| --- | --- | --- |
| `GET` | `/health` | Health check |
| `GET` | `/images/:arquivo` | Devolve a imagem salva |

Códigos de resposta: `200`, `201` (criado), `204` (removido), `400` (validação, com `details` do Zod), `404` (não encontrado) e `500`.

---

## 🚀 Como rodar

**Pré-requisito:** [Node.js](https://nodejs.org/) instalado.

```bash
# 1. Clone o repositório e instale as dependências
git clone <url-do-repositorio>
cd galeria
npm install

# 2. Suba a API (recompila e reinicia a cada alteração)
npm run dev-server
# → http://localhost:5799

# 3. Em outro terminal, suba o frontend
npm run dev
# → http://localhost:5173
```

A porta da API pode ser trocada com a variável de ambiente `PORT`.

### Scripts

| Script | O que faz |
| --- | --- |
| `npm run dev` | Frontend em modo de desenvolvimento (Vite) |
| `npm run build` | Checa os tipos e gera o build do frontend em `dist/` |
| `npm run preview` | Serve localmente o build do frontend |
| `npm run lint` | Roda o Oxlint |
| `npm run dev-server` | API em modo watch (tsup + node) |
| `npm run build-server` | Checa os tipos do servidor e gera o build em `server/dist/` |
| `npm run run-server` | Roda a API já compilada |

---

## 🗺️ Roadmap

- [x] Estrutura do projeto com Vite, React, TypeScript e Tailwind
- [x] React Router DOM instalado
- [x] API Fastify com CRUD de fotos e álbuns, upload de imagens e configuração de build
- [ ] Rotas e layout do frontend
- [ ] Listagem de fotos com filtro por álbum e busca
- [ ] Página de detalhe da foto com navegação anterior/próxima
- [ ] Criação de foto e álbum com formulários validados
- [ ] Gestão dos álbuns de cada foto
- [ ] Feedback com toasts e modais

---

Feito com 💜 durante os estudos na RocketSeat.
