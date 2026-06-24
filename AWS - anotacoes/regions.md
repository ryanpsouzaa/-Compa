# Regiões e Arquitetura Global — AWS

> Anotações de estudo (curso Rocketseat) — desenvolvidas para revisão

---

## 1. O que é uma Região na AWS?

Uma **Região** é uma localização geográfica física no mundo onde a AWS mantém um conjunto de **data centers**. Cada região é totalmente independente das outras — isso é proposital: se algo der errado numa região, isso não deve afetar as demais.

Cada região é composta por múltiplas **Zonas de Disponibilidade (Availability Zones — AZs)**, que são, na prática, um ou mais data centers separados fisicamente, mas interligados por redes de baixa latência e alta velocidade dentro da mesma região.

```
Região (ex: us-east-1, sa-east-1)
 ├── AZ-1 (data center isolado)
 ├── AZ-2 (data center isolado)
 └── AZ-3 (data center isolado)
```

📌 **Correção importante:** a AWS possui hoje **mais de 30 regiões** ao redor do mundo (não "centenas"). O número de regiões é mais contido — o que existe em grande quantidade são os **serviços** e os **data centers individuais** dentro de cada região. Vale sempre confirmar o número atual no site oficial da AWS, pois ele cresce com o tempo.

---

## 2. Por que a estrutura de Regiões + AZs existe?

### 2.1 Aumenta a disponibilidade dos serviços

Distribuir a infraestrutura entre várias AZs significa que, se uma AZ tiver problema (falha de energia, hardware, etc.), as outras continuam operando normalmente.

### 2.2 Melhora a resiliência da infraestrutura

Aplicações bem arquitetadas podem ser projetadas para resistir a falhas — distribuindo recursos entre AZs, evitando um único ponto de falha (_single point of failure_).

### 2.3 Permite redundância e recuperação de desastres (Disaster Recovery)

Ter cópias dos dados e da infraestrutura em **regiões diferentes** permite que, em caso de um desastre regional (ex: catástrofe natural), a aplicação continue rodando a partir de outra região.

---

## 3. Considerações importantes ao escolher uma Região

| Fator                                    | Por que importa                                                                                                   |
| ---------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| **Escolha da região certa**              | Impacta custo, latência, conformidade legal e quais serviços estarão disponíveis                                  |
| **Zonas de Disponibilidade disponíveis** | Regiões diferentes têm números diferentes de AZs — mais AZs geralmente significa mais opções de redundância       |
| **Requisitos de latência**               | Quanto mais perto do usuário final, menor o tempo de resposta                                                     |
| **Conformidade (compliance)**            | Algumas leis exigem que dados fiquem armazenados dentro do próprio país (ex: LGPD no Brasil, GDPR na Europa)      |
| **Serviços disponíveis na região**       | Nem todo serviço da AWS está disponível em todas as regiões — é preciso verificar antes de planejar a arquitetura |
| **Custo**                                | Os preços dos serviços podem variar de região para região                                                         |

---

## 4. Latência e Performance

A **latência** é o tempo que um dado leva para ir do usuário até o servidor e voltar.

- Quanto **mais próxima** a região estiver do usuário final, **menor a latência** e melhor a experiência.
- Por isso, é comum escolher a região mais próxima da maior base de usuários da aplicação (ex: `sa-east-1`, em São Paulo, para usuários no Brasil).
- Aplicações globais costumam usar múltiplas regiões + serviços como **CloudFront (CDN)** para entregar conteúdo com baixa latência em qualquer lugar do mundo.

---

## 5. Disponibilidade e Confiabilidade dos Serviços

Antes de escolher uma região, é necessário verificar:

- ✅ Se os **serviços que a aplicação precisa** estão disponíveis ali (nem todo serviço é lançado simultaneamente em todas as regiões)
- ✅ O **SLA (Service Level Agreement)** de disponibilidade oferecido
- ✅ Se a região possui **AZs suficientes** para a estratégia de redundância desejada

---

## 6. Resumo visual da hierarquia AWS

```
AWS (Infraestrutura Global)
 └── Região (ex: sa-east-1 — São Paulo)
      ├── Zona de Disponibilidade A
      ├── Zona de Disponibilidade B
      └── Zona de Disponibilidade C
```

- **Região** → isolamento geográfico e legal
- **AZ** → isolamento físico (energia, refrigeração, rede) dentro da mesma região
- **Data center** → a unidade física real dentro de uma AZ (uma AZ pode ter mais de um)

---

## 7. Pontos-chave para fixar

- 🌍 Regiões garantem **escala global** com **isolamento de falhas**.
- 🛡️ AZs dentro de uma região garantem **alta disponibilidade local**.
- 🔄 Múltiplas regiões garantem **recuperação de desastres** em larga escala.
- ⚡ Região mais próxima = **menor latência**.
- 📋 Sempre checar **serviços disponíveis** e **conformidade legal** antes de escolher a região.

---

_Próximos passos sugeridos de estudo: Edge Locations / CloudFront, como funciona o failover entre regiões, e diferença entre "Multi-AZ" e "Multi-Region" em serviços como RDS._
