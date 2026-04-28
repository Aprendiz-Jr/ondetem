# Test Plan — Botão Sair em /agendamentos (PR #23, já mergeado)

## O que mudou
Antes: `/agendamentos` não tinha opção de logout no header (só `Voltar` para home) nem no bottom-nav mobile (havia um item `Perfil` apontando para `#`). Usuário precisava sair manualmente apagando localStorage.

Depois (`agendamentos.html` no `main` pós-PR #23):
- Header desktop (linha 271): novo `<button id="btnSairAgendamentos">` com ícone `bi-box-arrow-right` ao lado do botão Voltar.
- Bottom-nav mobile (linha 306): item `Perfil` substituído por `<a id="btnSairAgendamentosMobile">` com mesmo ícone.
- `inicializarLogout()` (linha 347) liga ambos os botões a um handler que exibe `confirm('Deseja sair da sua conta?')` e chama `OndeTemAuth.logout()` se o usuário confirmar.

`OndeTemAuth.logout()` (`auth-guard.js` linhas 15-23) limpa `ondetem_token` + `ondetem_usuario` do localStorage e redireciona para `/login`.

## Primary flow (gravação única, ~30s)

1. Ir para `http://localhost:3000/login`, logar com `joao@email.com` / `123456`.
2. Abrir `/agendamentos`.
   - **Assertion A**: Botão com texto `Sair` e ícone `box-arrow-right` é visível no header roxo à direita (ao lado do `Voltar`).
3. Clicar no botão `Sair`.
   - **Assertion B**: Um diálogo nativo do browser aparece com o texto exato `Deseja sair da sua conta?` e botões OK/Cancelar.
4. Clicar OK no diálogo.
   - **Assertion C**: URL muda para `http://localhost:3000/login`.
   - **Assertion D**: `localStorage.getItem('ondetem_token')` retorna `null` (verificar via DevTools Application tab — mas valido indiretamente pelo redirect, já que `/agendamentos` é guardada por `auth-guard.js` e redireciona se não houver token).

## Why this distinguishes working vs broken

Se a mudança estivesse quebrada:
- Sem o botão: A falha em A (não existe elemento "Sair").
- Botão sem handler: A passa, mas C falha (clique não faz nada).
- `OndeTemAuth.logout()` não chamado: B pode aparecer mas C falha (sem redirect).
- `OndeTemAuth` não carregado em `/agendamentos`: `inicializarLogout()` daria erro no console ao tentar chamar `.logout()`; URL não muda.

Cada assertion tem um valor concreto esperado que difere visivelmente do estado quebrado.

## Out of scope
- Bottom-nav mobile: vou verificar visualmente redimensionando a janela e capturar screenshot, mas não vou reexecutar o fluxo completo — a função handler é a mesma compartilhada, então clicar em qualquer dos dois tem comportamento idêntico.
- Regressão de login, carregamento de agendamentos, filtros — não mudaram neste PR.
