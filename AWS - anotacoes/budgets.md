# AWS Budgets — Criando Alertas de Gasto com Templates

> Anotações de estudo (curso Rocketseat) — desenvolvidas para revisão

---

## 1. O que é o AWS Budgets?

**AWS Budgets** é o serviço de gestão de custos que permite definir limites de gasto (ou de uso) e ser **notificado automaticamente** quando esse limite é atingido ou está previsto para ser atingido.

📌 **Importante entender desde já:** o AWS Budgets **não bloqueia nem impede** gastos por padrão — ele é uma ferramenta de **monitoramento e alerta**, não de prevenção. Quem decide o que fazer com o alerta (desligar um recurso, investigar, etc.) é você. Existem ações automáticas mais avançadas (como desligar recursos automaticamente), mas isso não é o comportamento padrão dos templates simples.

---

## 2. Onde criar um Budget

1. Acessar o **Billing and Cost Management Console**
2. No menu lateral, ir em **Budgets**
3. Clicar em **Create budget**
4. Escolher entre dois fluxos:
   - **Use a template (simplified)** → fluxo rápido, em uma única página, com configurações já sugeridas
   - **Customize (Advanced)** → fluxo mais longo (5 etapas), com controle total sobre período, contas vinculadas, filtros por serviço/tag, etc.

Para o seu caso (estudo pessoal, conta única), o fluxo de **template simplificado** é suficiente e mais rápido.

---

## 3. Templates disponíveis

A AWS oferece templates prontos para os casos de uso mais comuns:

| Template                                 | Para que serve                                                                                                          |
| ---------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| **Zero spend budget**                    | Notifica quando o gasto sai de **$0** — ideal para quem quer saber assim que QUALQUER cobrança real começar a acontecer |
| **Monthly cost budget** (recomendado)    | Notifica quando o gasto **atinge ou está previsto a atingir** um valor mensal definido                                  |
| **Daily Savings Plans coverage budget**  | Acompanha a cobertura de Savings Plans (mais usado em ambientes corporativos com contratos de uso)                      |
| **Daily reservation utilization budget** | Acompanha o uso de Reserved Instances (também mais corporativo)                                                         |

Para estudo pessoal, os dois primeiros (que você configurou) são exatamente os mais relevantes — eles formam uma dupla de proteção: **"avise se eu gastar algo"** + **"avise se eu gastar muito"**.

---

## 4. Budget 1 — Zero Spend Budget (alerta em $1)

### O que esse template faz por padrão

O **Zero Spend Budget** é desenhado para notificar **assim que o gasto sai de zero** — ou seja, no momento em que você deixa de estar 100% dentro da camada gratuita (Free Tier / Always Free).

> Na configuração padrão da AWS, esse alerta dispara a partir de **qualquer valor acima de $0,00** (já a partir de centavos). Como você configurou o seu para alertar em **$1**, isso ajusta a margem de tolerância — você só recebe o e-mail quando o gasto acumulado bate $1, em vez de no primeiro centavo.

### Por que isso é útil

- Funciona como um **"alarme de incêndio"**: se algo escapou do Always Free (ex: você esqueceu uma instância EC2 ligada), você fica sabendo rapidamente, sem esperar o fechamento da fatura no fim do mês.
- Ideal justamente para quem, como você, está usando a conta **apenas para estudos** e quer ter certeza de que está dentro do que é gratuito.

### Como configurar (resumo do passo a passo)

1. Em **Create budget** → **Use a template (simplified)**
2. Selecionar o template **Zero spend budget**
3. Definir o **valor de alerta** (no seu caso, $1 em vez do padrão de centavos)
4. Informar o **e-mail** que receberá a notificação
5. Criar o budget

---

## 5. Budget 2 — Monthly Cost Budget (alerta em $5)

### O que esse template faz

O **Monthly cost budget** monitora o gasto acumulado **dentro do mês corrente** e dispara alertas com base em um valor de referência definido por você — no seu caso, **$5**.

Diferente do Zero Spend, esse budget normalmente permite configurar **múltiplos limites de alerta (thresholds)**, por exemplo:

- Alertar em **85%** do valor definido (~ $4,25 dos seus $5)
- Alertar em **100%** do valor definido ($5)
- Alertar também com base no **gasto previsto (forecasted)**, e não só no gasto já realizado — ou seja, a AWS pode te avisar antes mesmo de você gastar, com base na tendência atual de consumo

### Diferença-chave entre os dois budgets

|                    | Zero Spend Budget                                               | Monthly Cost Budget                                         |
| ------------------ | --------------------------------------------------------------- | ----------------------------------------------------------- |
| Funciona como      | Detector de "qualquer gasto fora do esperado"                   | Limite de orçamento mensal definido                         |
| Valor configurado  | $1                                                              | $5                                                          |
| Tipo de alerta     | Geralmente só "gasto real ultrapassou X"                        | Pode alertar por gasto real **e** gasto previsto (forecast) |
| Quando é mais útil | Para detectar erro/recurso esquecido rapidamente                | Para ter um teto de gasto mensal tolerável                  |
| Renovação          | Não é mensal por natureza — é cumulativo até o reset de billing | Reinicia a contagem todo mês (orçamento recorrente)         |

### Como configurar (resumo do passo a passo)

1. Em **Create budget** → **Use a template (simplified)**
2. Selecionar o template **Monthly cost budget**
3. Definir o **nome do budget**
4. Definir o **valor orçado** ($5, no seu caso)
5. Informar o **e-mail** de notificação
6. O escopo (todos os serviços) e os thresholds de alerta já vêm pré-configurados pelo template — podendo ser revisados antes de criar

---

## 6. Por que ter os dois budgets juntos faz sentido

```
$0 ────────────── $1 ─────────────────── $5
 │                  │                      │
 Always Free     🔔 Zero Spend         🔔 Monthly Cost
 (esperado)      "algo saiu do          "limite máximo
                  gratuito!"             tolerável atingido"
```

- O **Zero Spend ($1)** funciona como uma **detecção precoce** — o primeiro sinal de que algo não está mais 100% gratuito.
- O **Monthly Cost ($5)** funciona como um **teto de segurança** — mesmo que você decida tolerar um pequeno gasto, ele garante que isso nunca passe de um valor que você definiu como aceitável.

Essa combinação é uma prática sólida e recomendada especialmente para quem está estudando e usando a conta de forma exploratória, como é o seu caso.

---

## 7. Observações importantes

- ⚠️ Criar um budget **não impede gastos** — é só monitoramento. Para realmente bloquear, seria necessário configurar **AWS Budgets Actions** (ação automática, ex: aplicar uma política que restringe acesso) — algo mais avançado, fora do escopo dos templates simples.
- 📧 As notificações chegam por **e-mail** (via Amazon SNS por trás dos panos); é possível também integrar com Slack, Chime ou outros tópicos SNS customizados.
- 🔁 Vale revisar periodicamente o painel de **Budgets** no console para visualizar o gasto atual versus o limite definido, mesmo sem esperar o e-mail de alerta.
- 🧮 Para estimar custos antes de testar um serviço novo, vale usar o **AWS Pricing Calculator**, como reforço complementar aos budgets.

---

## 8. Pontos-chave para fixar

- 🎯 AWS Budgets = ferramenta de **alerta**, não de **bloqueio** automático por padrão.
- 🆓 **Zero Spend Budget** → detecta cedo quando você saiu do Always Free/Free Tier.
- 💰 **Monthly Cost Budget** → define um teto mensal tolerável, com alertas por gasto real e/ou previsto.
- 🔔 Ter os dois ativos ao mesmo tempo cobre tanto o "erro pequeno e rápido" quanto o "limite máximo aceitável".
- 📩 Sempre confirme que o e-mail de notificação está correto e acessível — de nada vale o alerta se ele cair numa caixa que você não checa.

---

_Próximos passos sugeridos de estudo: AWS Budgets Actions (bloqueio automático real), Cost Explorer para análise visual de gastos, e Billing Alarms via CloudWatch como alternativa/complemento aos Budgets._
