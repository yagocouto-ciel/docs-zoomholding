# Fluxo de Aprovação de Despesas (Zoom Custom)

## Objetivo
Este documento explica, de forma simples, como funciona o novo fluxo de aprovação de despesas.

A aprovação agora é baseada em grupos de usuários (e não em IDs fixos de usuário), deixando o processo mais seguro e fácil de manter entre ambientes.

## Alçadas de aprovação
A despesa passa pelas seguintes alçadas:

1. Controladoria
2. Diretoria Financeira
3. Financeiro

Cada etapa só pode ser aprovada por usuários que pertençam ao grupo correspondente.

## Grupos utilizados
- Aprovadores Controladoria
- Aprovadores Diretoria Financeiro
- Aprovadores Financeiro
- Grupo Prioritário (atalho para envio direto à Controladoria)

## Sequência do fluxo
1. A despesa inicia em **Provisório (draft)**.
2. A despesa é enviada para aprovação.
3. Passa por **Controladoria**.
4. Passa por **Diretoria Financeira**.
5. Passa por **Financeiro**.
6. Fica **Aprovada**.

## Regra do Grupo Prioritário
Se o colaborador estiver no Grupo Prioritário, pode enviar a despesa direto para a Controladoria, pulando a etapa de gestor.

## Regras de segurança e visibilidade
- Botões de aprovação só aparecem para quem está no grupo da etapa atual.
- Usuário fora do grupo não consegue aprovar.
- Em caso de retorno para Provisório, aprovações anteriores são resetadas.

## Parâmetros de sistema obrigatórios
Para o fluxo funcionar corretamente no ambiente, os parâmetros abaixo devem estar preenchidos:

- zoom_custom.expense_group_controladoria_id
- zoom_custom.expense_group_diretoria_id
- zoom_custom.expense_group_financeiro_id
- zoom_custom.expense_group_prioritario_id

## Valores de referência configurados em produção
- Controladoria: 206
- Diretoria Financeira: 392
- Financeiro: 393
- Prioritário: 229

## Problemas comuns e como verificar
### 1) Botão de aprovar não aparece
Verificar:
- etapa atual da despesa
- se o usuário pertence ao grupo correto

### 2) Usuário não consegue aprovar
Verificar:
- se o usuário está no grupo da alçada correspondente
- se os parâmetros de sistema estão corretos

### 3) Aprovações foram perdidas
Se a despesa voltou para Provisório, o reset de aprovações é comportamento esperado.

## Resumo final
O fluxo atual garante aprovação por alçada de grupo, com controle de visibilidade e permissões por etapa.
Com grupos e parâmetros corretos, o processo fica previsível para operação e suporte.
