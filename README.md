# cinevault

> Seu próximo filme começa aqui.

Catálogo editorial de filmes construído com Vue 3. O Cinevault combina descoberta visual, busca rápida e uma lista pessoal em uma experiência inspirada em revistas de cinema.

**Versão atual:** `1.0.1`

![Cinevault](https://img.shields.io/badge/Cinevault-Vue%203-42b883?style=for-the-badge&logo=vue.js&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-6-3178C6?style=for-the-badge&logo=typescript&logoColor=white)
![TMDB](https://img.shields.io/badge/dados-TMDB-01b4e4?style=for-the-badge&logo=tmdb&logoColor=white)

## Visão geral

O Cinevault é uma página de catálogo com foco em descoberta. A home apresenta um filme em destaque e uma grade de títulos com cartazes reais, avaliações, gêneros e informações rápidas.

O projeto funciona em dois modos:

- **TMDB conectado:** busca os filmes populares diretamente na API externa.
- **Modo editorial:** usa um catálogo local com imagens públicas do TMDB quando nenhuma chave foi configurada.

## Funcionalidades

- Hero com filme em destaque e backdrop cinematográfico
- Cartazes reais dos filmes
- Busca por título em tempo real
- Filtros por gênero
- Ordenação por relevância, avaliação e data
- Favoritos com a seção **Minha lista**
- Favoritos persistentes no navegador com `localStorage`
- Modal com sinopse e detalhes do filme
- Trailers oficiais pesquisados no YouTube
- Fallback automático para imagens indisponíveis
- Layout responsivo para desktop, tablet e mobile
- Interface em português com tema escuro editorial

## Stack

| Tecnologia | Uso |
| --- | --- |
| Vue 3 | Interface e estado reativo |
| TypeScript | Tipagem do projeto |
| Vite | Desenvolvimento e build |
| TMDB API | Filmes, imagens e dados do catálogo |
| YouTube | Busca dos trailers oficiais |
| CSS | Layout responsivo e identidade visual |

## Começando

### Pré-requisitos

- Node.js `22.18+` ou `24.12+`
- npm

### Instalação

```bash
npm install
```

### Desenvolvimento

```bash
npm run dev
```

A aplicação ficará disponível no endereço exibido pelo Vite, normalmente `http://localhost:5173`.

## Configurando a TMDB

1. Crie uma conta no [TMDB](https://www.themoviedb.org/).
2. Gere uma chave na área de [API](https://www.themoviedb.org/settings/api).
3. Copie o arquivo de exemplo:

```bash
copy .env.example .env.local
```

4. Abra `.env.local` e substitua o valor:

```env
VITE_TMDB_API_KEY=sua_chave_tmdb_aqui
```

5. Reinicie o servidor de desenvolvimento.

> A chave é usada no frontend porque este é um projeto demonstrativo. Em produção, prefira um backend ou proxy para proteger credenciais e aplicar limites de requisição.

## Scripts

```bash
npm run dev          # inicia o servidor de desenvolvimento
npm run type-check   # valida Vue + TypeScript
npm run build-only   # gera o build de produção
npm run build        # valida tipos e gera o build
npm run preview      # serve o build localmente
```

## Estrutura principal

```text
src/
├── App.vue              # catálogo, filtros, favoritos e modal
├── main.ts              # inicialização do Vue
└── assets/
    ├── base.css         # tokens globais e reset
    └── main.css         # layout e componentes visuais
```

## Qualidade

Antes de publicar uma alteração, execute:

```bash
npm run type-check
npm run build-only
```

## Changelog

### `1.0.1`

- Mantém os filmes favoritos após recarregar ou reabrir o site.
- Restaura automaticamente a seção **Minha lista** usando o armazenamento local do navegador.

# Créditos

Os dados e imagens do catálogo são fornecidos pelo [The Movie Database (TMDB)](https://www.themoviedb.org/). Este projeto não é endossado nem certificado pelo TMDB.

Os trailers são encontrados no YouTube por meio de buscas com o título do filme e a indicação de trailer oficial.
