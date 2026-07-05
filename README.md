# 📚 Minha Estante

Ferramenta pessoal para catalogar livros — busca automática de dados, capas, estatísticas de leitura e login por conta própria. Tudo em um único arquivo HTML, sem instalação.

## Índice

- [O que essa ferramenta faz](#o-que-essa-ferramenta-faz)
- [Como usar](#como-usar)
- [Tecnologia por trás](#tecnologia-por-trás)
- [Configuração do backend (Supabase)](#configuração-do-backend-supabase)
- [Hospedagem](#hospedagem)
- [Estrutura de dados](#estrutura-de-dados)
- [Limitações conhecidas](#limitações-conhecidas)
- [Ideias para o futuro](#ideias-para-o-futuro)

---

## O que essa ferramenta faz

- **Busca automática**: digite o título de um livro e a ferramenta preenche sozinha título, autor, editora, idioma, gênero e capa (via Open Library).
- **Catálogo completo**: título, autor, série/coleção, volume, gênero, subgênero, editora, idioma, formato, observações, status de leitura, ano de aquisição e origem.
- **Estante visual**: cada livro vira uma "lombada" colorida no topo, com a capa de verdade quando disponível.
- **Estatísticas**: progresso de leitura, top gêneros e top autores da sua coleção.
- **Login próprio**: email/senha ou Google, com os dados salvos na nuvem (Supabase) — acesse de qualquer computador.
- **Exportar/Importar**: backup em `.json` (para reimportar depois) ou `.csv` (para abrir no Excel/Sheets).
- **Tabela ordenável e paginada**: clique nos cabeçalhos para ordenar; pagina automaticamente acima de 20 livros.

## Como usar

1. Abra o arquivo `minha-estante.html` no navegador (ou acesse a versão hospedada, se houver).
2. Crie uma conta ou entre com Google.
3. Clique no botão **+** (canto inferior direito) para adicionar um livro.
4. Digite o título na busca e clique em "usar dados" no resultado desejado, ou preencha manualmente.
5. Ajuste os campos que a busca não preencheu (série, volume, status, observações etc.) e salve.
6. Use os filtros e os cabeçalhos da tabela para encontrar e organizar seus livros.

## Tecnologia por trás

| Camada | Tecnologia |
|---|---|
| Interface | HTML + CSS + JavaScript puro (sem frameworks) |
| Autenticação e banco de dados | [Supabase](https://supabase.com) (Postgres + Auth) |
| Busca de metadados de livros | [Open Library API](https://openlibrary.org/developers/api) |
| Login social | Google OAuth (via Supabase) |

Não há processo de build — é um único arquivo `.html` que pode ser aberto diretamente ou hospedado em qualquer lugar (Netlify, Vercel, GitHub Pages etc.).

## Configuração do backend (Supabase)

O projeto já vem conectado a um projeto Supabase configurado com:

- **Project URL**: `https://nqdmxegdbmlahmcigudk.supabase.co`
- **Chave pública (publishable key)**: já embutida no arquivo — é segura de expor no front-end, pois a proteção real vem das políticas de RLS (Row Level Security).

### Tabela `livros`

```sql
create table public.livros (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references auth.users(id) on delete cascade not null,
  title text not null,
  author text,
  series text,
  volume text,
  genre text,
  subgenre text,
  publisher text,
  language text,
  format text,
  notes text,
  status text,
  year text,
  origin text,
  cover text,
  created_at timestamptz default now()
);

alter table public.livros enable row level security;

create policy "Usuários veem só os próprios livros"
  on public.livros for select using (auth.uid() = user_id);
create policy "Usuários inserem só pra si"
  on public.livros for insert with check (auth.uid() = user_id);
create policy "Usuários atualizam só os próprios"
  on public.livros for update using (auth.uid() = user_id);
create policy "Usuários deletam só os próprios"
  on public.livros for delete using (auth.uid() = user_id);
```

Cada pessoa só enxerga e edita os próprios livros — mesmo com a chave pública exposta no código.

### Login com Google

Configurado em **Authentication → Providers → Google** no painel do Supabase, com Client ID e Client Secret gerados no [Google Cloud Console](https://console.cloud.google.com/). A URL de callback usada foi:

```
https://nqdmxegdbmlahmcigudk.supabase.co/auth/v1/callback
```

## Hospedagem

O login com Google e a confirmação de email **só funcionam de verdade quando o arquivo está hospedado** em um endereço `http://` ou `https://` real (não funciona abrindo o arquivo direto do computador, via `file://`).

Opção mais simples: [Netlify Drop](https://app.netlify.com/drop) — arraste o `.html` e ele já gera uma URL pública. Depois disso, no painel do Supabase:

1. **Authentication → URL Configuration**
2. Preencha **Site URL** e **Redirect URLs** com a URL gerada pelo Netlify.

O código já está preparado para redirecionar dinamicamente para onde quer que o arquivo esteja hospedado — não precisa mexer em mais nada além dessa configuração.

## Estrutura de dados

Cada livro salvo tem os seguintes campos:

| Campo | Descrição |
|---|---|
| `title` | Título do livro |
| `author` | Autor(es) |
| `series` | Série ou coleção |
| `volume` | Número do volume na série |
| `genre` | Gênero |
| `subgenre` | Subgênero |
| `publisher` | Editora |
| `language` | Idioma |
| `format` | Físico, Kindle, Audiobook, PDF ou Outro |
| `status` | Quero ler, Lendo, Lido, Relendo ou Abandonado |
| `year` | Ano de aquisição (não é o ano de publicação) |
| `origin` | Comprei, Ganhei, Herdei, Emprestado ou Outro |
| `notes` | Observações livres (ex: capa dura, edição especial) |
| `cover` | URL da capa (preenchida automaticamente pela busca) |

## Limitações conhecidas

- **Livros nacionais/independentes**: a Open Library tem cobertura fraca de lançamentos brasileiros e editoras pequenas — a busca pode não encontrar nada ou vir sem gênero/capa, exigindo preenchimento manual.
- **`file://` local**: login com Google e confirmação de email por link exigem hospedagem real; não funcionam abrindo o arquivo direto do computador.
- **Estantes muito grandes**: a paginação e ordenação foram implementadas pensando em coleções grandes, mas ainda não testadas em uso real com centenas de livros.

## Ideias para o futuro

- Ícones ou badges customizados para dar um toque visual sem usar emojis.
- Estatísticas adicionais (livros lidos por mês/ano, páginas totais).
- Busca alternativa (Google Books API) como fallback quando a Open Library não encontrar o livro.
- Modo escuro.