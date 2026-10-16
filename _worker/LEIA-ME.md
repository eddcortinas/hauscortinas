# Worker `haus-acessos` (Cloudflare)

Servidor de login e de "Meus orçamentos" do gerador (`orcamento.html`) da Haus.
É **separado** de qualquer outro site: worker próprio (`haus-acessos`) e KV próprio
(`haus-acessos-usuarios`). Assim a Haus não vê usuários nem orçamentos de outra empresa, e vice-versa.

Esta pasta começa com `_`, então o GitHub Pages **não** a publica no site.
O repositório é público: por isso as chaves **não** estão aqui (`__DEFINIR_NO_WORKER__`).
**Nunca** faça commit do arquivo com senha, salt, hash ou chave.

## Status
Ainda **não publicado**. Enquanto isso, o gerador abre a tela de login, mas não consegue entrar.

## Como publicar
1. **Gerar as chaves**: abra `gerar-senha.html` no computador (duplo clique), digite a senha do
   administrador e copie os 3 valores (`salt`, `hash` e `CHAVE`).
2. **Criar o KV**: no painel da Cloudflare, *Storage & Databases → KV → Create* com o nome
   `haus-acessos-usuarios`.
3. **Criar o worker**: *Workers & Pages → Create → Worker*, nome `haus-acessos`.
   Em *Edit code*, cole o conteúdo de `haus-acessos.js` (em UTF-8) e troque os três
   `__DEFINIR_NO_WORKER__` pelos valores do passo 1. Confira também:
   - `ADMIN.usuario` (login do administrador, hoje `"haus"`);
   - `ORIGENS`: precisa ter o endereço onde o site está no ar. Enquanto o domínio
     `hauscortinas.com.br` não apontar para o GitHub Pages, inclua `https://<conta>.github.io`
     (só a origem, sem `/hauscortinas`).
   Clique em *Deploy*.
4. **Ligar o KV**: em *Settings → Bindings → Add → KV namespace*, nome da variável `USUARIOS`,
   namespace `haus-acessos-usuarios`. Faça o deploy de novo.
   (Pela API: `PUT /accounts/<conta>/workers/scripts/haus-acessos`, multipart com `metadata`
   `{"main_module":"worker.js","bindings":[{"type":"kv_namespace","name":"USUARIOS","namespace_id":"<id do KV>"}],"compatibility_date":"2024-01-01"}`.)
5. **Apontar o gerador para o worker**: em `js/orcamento.js`, troque
   `https://haus-acessos.CONTA.workers.dev` pelo endereço que a Cloudflare mostrar e faça commit.
6. **Testar**: abrir `orcamento.html`, entrar com o administrador, criar um orçamento
   (o número deve sair `ORC MM/AA-001`), salvar, ver em "Meus orçamentos" e adicionar
   um acesso em "Gerenciar acessos".

## Rotas
- `POST /login`, `GET /eu`
- `GET /orcamentos`, `GET /orcamentos/<id>`, `POST /orcamentos`,
  `POST /orcamentos/status`, `POST /orcamentos/arquivar`
- `GET /proximo-numero?tipo=Pedido&data=AAAA-MM-DD&ignorar=<id>` (numeração por mês e tipo:
  `ORC MM/AA-NNN` e `PED MM/AA-NNN`)
- `POST /orcamentos/renumerar` (só admin): `{aplicar:false}` mostra a prévia, `{aplicar:true}` grava
- `GET /usuarios`, `POST /usuarios`, `POST /usuarios/status`, `POST /usuarios/remover` (só admin)
