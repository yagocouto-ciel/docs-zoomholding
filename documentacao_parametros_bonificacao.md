# Documentação de Parâmetros

## Objetivo
Esta documentação descreve os critérios que uma fatura deve atender para:

1. Gerar linha no relatório de bonificação.
2. Abater valores quando houver nota de crédito.

Também traz um checklist prático de validação e diagnóstico.

## Escopo funcional
O processamento ocorre no método de criação de bonificação da fatura.

Referência principal:

1. [zoom_bonificacao_comissao/models/account_move.py](zoom_bonificacao_comissao/models/account_move.py)

## Regras para gerar linha no relatório de bonificação

### 1. Tipo de documento permitido
A fatura precisa ser de cliente ou nota de crédito de cliente:

1. out_invoice
2. out_refund

Se não for, o processo é interrompido para esse documento.

### 2. Regra de tipo de pedido

#### Para out_invoice
O tipo de pedido deve estar entre os permitidos:

1. venda-locacao
2. venda
3. servico
4. venda_futura
5. venda-conta-ordem
6. venda-conta-ordem-vendedor

#### Para out_refund
O tipo de pedido de entrada deve estar entre:

1. credito-imposto
2. devolucao
3. devolucao_compra

### 3. Data de emissão obrigatória
invoice_date deve estar preenchida.

### 4. Diário permitido
O diário da fatura deve estar na lista fixa:

1. 41
2. 33
3. 81
4. 57
5. 65
6. 73
7. 25
8. 17
9. 332

### 5. Vínculo comercial obrigatório
Precisa existir vínculo com pedido de venda ou oportunidade.

Ordem de busca:

1. Linhas da fatura para pedido de venda.
2. Linhas contábeis como fallback para pedido de venda.
3. Oportunidade via pedido.
4. Oportunidade na própria fatura como fallback.

### 6. Equipe de vendas obrigatória
O sistema busca equipe nesta ordem:

1. equipe na própria fatura
2. equipe no pedido
3. equipe na oportunidade

Sem equipe, não há geração de linha.

### 7. Regras por vendedor da equipe
Para cada membro da equipe:

1. Deve existir user_id na linha da equipe.
2. Deve existir funcionário vinculado ao usuário na mesma empresa da fatura.
3. Deve existir parâmetro de bonificação para esse funcionário na mesma empresa.

Se qualquer item falhar, aquele vendedor específico não gera linha.

### 8. Linha de parâmetro por período
O sistema busca a linha de parâmetro cujo período cubra a data da fatura:

1. data_inicio menor ou igual à data da fatura
2. data_fim maior ou igual à data da fatura

Observação: mesmo sem linha de período encontrada, a linha de bonificação pode ser criada com percentuais zerados.

### 9. Marcação de processamento
Se ao menos uma linha for criada, a fatura recebe bonificacao_ok igual a verdadeiro.

## Regras para abatimento de nota de crédito

### 1. A nota de crédito precisa ser elegível
Para abater, a nota de crédito passa pelas mesmas validações gerais acima.

### 2. Tipo de entrada precisa ser permitido
No caso de out_refund, o tipo de pedido de entrada deve ser:

1. credito-imposto
2. devolucao
3. devolucao_compra

### 3. Abatimento por valor negativo
Quando elegível, a participação é multiplicada por menos um.

Resultado:

1. valor_participacao fica negativo
2. bonus derivado fica negativo
3. o relatório passa a refletir abatimento

## Impacto de voltar fatura para provisório
Se a fatura já possui bonificação paga:

1. o sistema não apaga a linha paga
2. cria linha de ajuste negativa

Se não está paga:

1. remove linhas de bonificação da fatura
2. ao confirmar novamente, recalcula e recria

## Checklist operacional rápido

### Para fatura normal não aparecer no relatório
1. Validar tipo do documento.
2. Validar tipo de pedido.
3. Validar diário.
4. Validar data de emissão.
5. Validar vínculo com pedido ou oportunidade.
6. Validar equipe encontrada na ordem de busca.
7. Validar funcionário do vendedor na empresa da fatura.
8. Validar parâmetro de bonificação ativo para funcionário e empresa.
9. Validar se há filtro de ativo ocultando linhas no relatório.

### Para nota de crédito não abater
1. Confirmar que é out_refund.
2. Confirmar tipo de pedido de entrada elegível.
3. Confirmar diário elegível.
4. Confirmar equipe e parametrização do vendedor.
5. Confirmar que a linha foi gerada com valor negativo.

## Mensagens de alerta comuns

### Sem parâmetros de bonificação para funcionário na empresa
Significa que o vendedor foi identificado na equipe, mas não foi encontrado bonificacao.param para:

1. employee_id do vendedor
2. company_id da fatura

Efeito:

1. esse vendedor não gera linha naquela fatura.

## Referências técnicas

1. Regras de geração: [zoom_bonificacao_comissao/models/account_move.py](zoom_bonificacao_comissao/models/account_move.py)
2. Modelo de parâmetros: [zoom_bonificacao_comissao/models/bonificacao_param.py](zoom_bonificacao_comissao/models/bonificacao_param.py)
3. Tela de parâmetros: [zoom_bonificacao_comissao/views/bonificacao_param_views.xml](zoom_bonificacao_comissao/views/bonificacao_param_views.xml)
4. Tela de relatório: [zoom_bonificacao_comissao/views/bonificacao_report_view.xml](zoom_bonificacao_comissao/views/bonificacao_report_view.xml)
