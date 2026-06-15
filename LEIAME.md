# Termo — guia de instalação

O jogo agora está dividido em arquivos, hospeda no GitHub (grátis) e guarda um
**placar compartilhado** no Supabase. Esta pasta tem tudo que você precisa:

```
termo/
├─ index.html      → o jogo (não precisa editar)
├─ respostas.txt   → palavras que podem ser sorteadas (513)
├─ palavras.txt    → palavras aceitas como chute (9.571)
├─ config.js       → onde você cola a URL e a chave do Supabase
└─ LEIAME.md       → este guia
```

> **Importante:** o jogo precisa ser aberto por um servidor (GitHub Pages, ou um
> servidor local), **não** por duplo-clique no arquivo. Abrir como `file://` faz
> o navegador bloquear a leitura de `respostas.txt` e `palavras.txt`.

A pontuação, as cores e a mecânica continuam exatamente como estavam.

---

## Parte 1 — Placar compartilhado (Supabase)

Sem isso o jogo já roda, mas cada aparelho guarda o próprio placar. Para ter um
placar único que todo mundo vê, faça uma vez:

1. Crie uma conta grátis em **https://supabase.com** e clique em **New project**.
   Dê um nome, escolha uma senha qualquer e a região mais próxima. Espere ~1 min.

2. No menu lateral, abra **SQL Editor**, cole o bloco abaixo e clique em **Run**.
   Isso cria a tabela do placar e libera leitura/gravação pública:

   ```sql
   create table ranking (
     id         bigint generated always as identity primary key,
     nome       text  not null,
     pontos     int   not null,
     seq        int   not null,
     created_at timestamptz default now()
   );

   alter table ranking enable row level security;

   create policy "leitura publica"  on ranking for select using (true);
   create policy "insercao publica" on ranking for insert with check (true);
   ```

3. No menu, vá em **Project Settings → API** (ou **Data API**) e copie dois valores:
   - **Project URL** — algo como `https://abcdwxyz.supabase.co`
   - **anon public** (chave pública / publishable)

4. Abra o arquivo **config.js** e cole os dois valores:

   ```js
   window.SUPABASE_URL = "https://abcdwxyz.supabase.co";
   window.SUPABASE_KEY = "eyJhbGciOiJI...";   // a chave anon public
   ```

   Pode salvar a chave aqui sem medo: a `anon` é pública por natureza, e quem
   controla o acesso são as políticas (RLS) que você criou no passo 2.

Pronto — a partir daí todo placar salvo vai para o mesmo lugar e aparece em
qualquer aparelho.

---

## Parte 2 — Publicar no GitHub (jogar pelo celular)

1. Crie uma conta em **https://github.com** (se ainda não tiver).

2. Clique em **New repository**. Dê um nome (ex.: `termo`), deixe **Public** e
   clique em **Create repository**.

3. Na página do repositório vazio, clique em **uploading an existing file** e
   arraste os **5 arquivos** desta pasta (`index.html`, `respostas.txt`,
   `palavras.txt`, `config.js`, `LEIAME.md`). Clique em **Commit changes**.

4. Vá em **Settings → Pages**. Em **Build and deployment → Source**, escolha
   **Deploy from a branch**, selecione a branch **main** e a pasta **/ (root)**.
   Clique em **Save**.

5. Espere ~1 minuto e atualize a página. O GitHub mostra o link, algo como:

   ```
   https://SEU-USUARIO.github.io/termo/
   ```

   Esse é o link do jogo. Abra no celular, no PC, mande no grupo da família.
   Para jogar no celular como se fosse um app, abra o link e use
   **"Adicionar à tela de início"**.

### Atualizar palavras depois

Se um dia quiser mexer nas palavras, é só editar `respostas.txt` ou
`palavras.txt` (uma palavra por linha, 5 letras, MAIÚSCULAS e sem acento) e
subir o arquivo de novo no GitHub. O jogo recarrega sozinho.

---

## Dúvidas comuns

**O placar não aparece entre aparelhos.** Confira se a URL e a chave em
`config.js` estão corretas e se você rodou o SQL do passo 2 (incluindo as duas
políticas). Sem isso, o jogo cai no modo "placar só neste aparelho".

**Quero zerar o placar.** No Supabase, **Table Editor → ranking**, selecione as
linhas e apague; ou rode `delete from ranking;` no SQL Editor.

**Deu "Palavra não reconhecida" num palavrão/palavra real.** Os palavrões
continuam sendo aceitos como chute (estão em `palavras.txt`), só não são
sorteados como resposta. Se faltar alguma palavra, adicione em `palavras.txt`.
