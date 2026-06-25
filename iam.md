# IAM — Identity and Access Management

> Anotações de estudo (curso Rocketseat) — desenvolvidas para revisão

---

## 1. O que é o IAM?

**IAM (Identity and Access Management)** é o serviço da AWS responsável por gerenciar **identidades** (quem pode acessar) e **permissões** (o que cada identidade pode fazer) dentro de uma conta AWS.

Em resumo, o IAM responde a duas perguntas o tempo todo:

1. **Quem é você?** → Autenticação (identidade)
2. **O que você pode fazer?** → Autorização (permissões)

É a base de toda a segurança na AWS — praticamente todo serviço da AWS verifica o IAM antes de permitir qualquer ação.

---

## 2. Componentes principais

### 2.1 Usuários (Users)

Representam uma pessoa ou aplicação que precisa interagir com a AWS.

- Possuem credenciais próprias (senha para console e/ou _access keys_ para CLI/API).
- Cada usuário deve ter **apenas as permissões necessárias** para sua função — esse é o princípio do **menor privilégio** (_least privilege_).
- Podem ter atividades **monitoradas e auditadas** (ex: via **CloudTrail**), permitindo rastrear quem fez o quê e quando.

### 2.2 Grupos (Groups)

Conjuntos de usuários que compartilham as mesmas necessidades de acesso.

- Facilitam a gestão: em vez de definir permissões usuário por usuário, você define a política **uma vez no grupo**.
- Um usuário pode pertencer a **vários grupos**.
- Boa prática: organizar grupos por função (ex: `Desenvolvedores`, `Administradores`, `Financeiro`).

### 2.3 Políticas (Policies)

Documentos (em formato **JSON**) que definem **regras de permissão** — o que é permitido (`Allow`) ou negado (`Deny`) sobre quais recursos.

Exemplo simplificado de uma política:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::meu-bucket/*"
    }
  ]
}
```

Políticas podem ser anexadas a:

- Usuários
- Grupos
- Roles (funções)

### 2.4 Roles (Funções)

Parecidas com usuários, mas **não têm credenciais fixas** — são "assumidas" temporariamente por quem precisa (um usuário, um serviço da AWS, ou até uma conta externa).

- Muito usadas para dar permissão a **serviços da AWS** (ex: uma instância EC2 que precisa acessar um bucket S3).
- Evitam a necessidade de armazenar credenciais fixas dentro de aplicações — mais seguro.

---

## 3. Gerenciamento de Usuários — boas práticas

| Prática                                                                | Por quê                                                               |
| ---------------------------------------------------------------------- | --------------------------------------------------------------------- |
| Criar um usuário por pessoa/aplicação (nunca compartilhar credenciais) | Permite rastreabilidade individual                                    |
| Aplicar o princípio do menor privilégio                                | Reduz a superfície de risco em caso de comprometimento                |
| Ativar MFA (autenticação multifator)                                   | Camada extra de segurança no login                                    |
| Monitorar atividades e histórico de acessos                            | Permite auditoria e detecção de comportamento anômalo                 |
| Evitar usar o usuário **root** da conta no dia a dia                   | O root tem acesso irrestrito; deve ser usado só em casos excepcionais |

---

## 4. Grupos e Políticas — como se conectam

```
Política (regras de acesso em JSON)
      │
      ▼
   Grupo  ──────► Usuário A
                  Usuário B
                  Usuário C
```

- A política é escrita **uma vez** e aplicada ao grupo.
- Todo usuário que entra no grupo **herda automaticamente** essas permissões.
- Se a regra de acesso muda, basta atualizar a política do grupo — não é necessário alterar usuário por usuário.

Isso simplifica bastante a administração à medida que o número de usuários cresce.

---

## 5. IAM é Global, não Regional ⚠️ (ponto-chave!)

Esse é um dos detalhes mais importantes — e que pega muita gente de surpresa:

> **O IAM não é um serviço regional.** Usuários, grupos, políticas e roles são criados **uma única vez** e existem **globalmente**, em todas as regiões da AWS, dentro da mesma conta.

Na prática, isso significa:

- Uma única conta AWS → as identidades (usuários, grupos, roles) são **as mesmas em qualquer região**.
- Você **não precisa recriar usuários** quando muda de região no console.
- As **políticas de permissão**, porém, podem **restringir o acesso a recursos de regiões específicas** — ou seja, a identidade é global, mas o que ela pode acessar pode ser limitado por região através da própria política.

📌 **Resumindo a diferença:**

|         | IAM (identidades)                                              | Recursos (EC2, S3, RDS...)                            |
| ------- | -------------------------------------------------------------- | ----------------------------------------------------- |
| Escopo  | Global                                                         | Regional (a maioria)                                  |
| Exemplo | Um usuário `joao@empresa.com` existe igual em todas as regiões | Uma instância EC2 só existe na região onde foi criada |

---

## 6. Pontos-chave para fixar

- 🔑 IAM = identidades (quem) + políticas (o que pode fazer).
- 👤 Usuários representam pessoas/aplicações; **Grupos** organizam usuários; **Roles** são identidades temporárias/assumíveis.
- 📜 Políticas em JSON definem permissões com `Allow`/`Deny`.
- 🌍 **IAM é global** — não existe "IAM da região tal". Usuários, grupos e roles valem para a conta inteira, em todas as regiões.
- 🛡️ Princípio do menor privilégio + MFA + evitar uso do root = boas práticas essenciais de segurança.

---

_Próximos passos sugeridos de estudo: diferença entre Policies gerenciadas pela AWS x Policies personalizadas (inline), IAM Roles para serviços (ex: EC2 Instance Profile), e o conceito de "Trust Policy" em roles assumidas entre contas._
