# AGENTS.md — Engenharia de Agentes (Projeto #29: Kudos Board)

Chave Única: `P26-2C-9700-D26E-48A3-4886`

## Resumo e transparência

Este projeto foi desenvolvido com **um único agente de IA**: o **Claude (Anthropic), em conversa no claude.ai** (modelo Claude Sonnet 5.5). Esse agente atuou em **três papéis, em sequência**, cada um guiado por um skill da pasta `skills/`.

Os papéis **não foram executados por três agentes independentes**. A execução de comandos (terminal, deploy, testes) foi feita pelo estudante, que copiou o código gerado pelo agente e reportou os resultados de volta na conversa. O agente não teve acesso direto à máquina do estudante.

## Harness (ambiente)

| Item | Detalhe |
|---|---|
| Agente | Claude (Anthropic), chat do claude.ai |
| Interface | Conversa com texto e imagens (prints do terminal e do navegador) |
| Execução | Pelo estudante: Git Bash, Node.js (Scoop), Wrangler 4.x |
| Infraestrutura | Cloudflare Pages + Pages Functions + D1; Discord OAuth2 |

## Papéis e skills

| Papel | Skill utilizado | O que foi feito |
|---|---|---|
| Backend | `skills/serverless-d1-discord/SKILL.md` | Migração `0002_kudos.sql` (tabela `kudos` com `CHECK (from_id <> to_id)`); endpoints em `functions/api/` (`kudos`, `users`, `badges`, `me`, `logout`, `status`); sessão em cookie assinado com HMAC-SHA256 (`functions/_lib/session.js`), no lugar do cookie sem assinatura do esqueleto |
| Interface | `skills/vanilla-dom-ui/SKILL.md` | `public/index.html`, `styles.css` e `app.js` em Vanilla JS: formulário, selos, grade (CSS Grid), filtros Recebidos/Enviados por perfil, rodapé com a chave. DOM montado com `textContent` (sem `innerHTML` com dados de usuário) |
| Teste e validação | `skills/teste-validacao/SKILL.md` | Script `testes.mjs` executado contra o site publicado; teste manual da regra de auto-elogio; saída em `relatorios/saida-testes.txt` |

## Regras de negócio validadas no servidor

- O remetente vem do cookie de sessão assinado, nunca do corpo da requisição.
- Auto-elogio bloqueado (`self_kudos`), além da restrição `CHECK` no banco.
- Selo validado contra lista fixa; mensagem entre 3 e 280 caracteres; destinatário precisa existir.
- Verificação de `Origin` no POST; limite de 5 envios por minuto por usuário (implementado, **não coberto** pelos testes automatizados).
- Filtros `received`/`sent` validados por expressão regular e consultas parametrizadas.

## Resultados dos testes (executados em 20/11/2026)

- `node testes.mjs` contra `https://kudos-board-2c.pages.dev`: **8/8 testes passaram** (selos, `/api/me` sem login, POST sem login = 401, cookie forjado = 401, Origin externo = 403, filtro inválido = 400, tentativa de SQL injection = 400, listagem = 200).
- Teste manual, logado, enviando kudos ao próprio ID: resposta `self_kudos` (rejeitado pelo servidor).

## Limitações

- O requisito de **três agentes distintos** não foi cumprido: houve um agente em três papéis.
- Não houve revisão cruzada por outro agente; os problemas corrigidos foram os identificados pelo próprio agente durante o desenvolvimento (cookie de sessão sem assinatura no esqueleto original).
- Não foram testados automaticamente: limite de envios, mensagem fora do tamanho, selo inválido e destinatário inexistente.
