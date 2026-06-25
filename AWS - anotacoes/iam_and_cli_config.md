# Configurando o IAM na prática — Usuários, Grupos e AWS CLI

> Anotações de estudo (curso Rocketseat) — desenvolvidas para revisão

---

## 1. Por que não usar o usuário Root no dia a dia

O usuário **root** é criado automaticamente quando você abre a conta AWS, e tem acesso **total e irrestrito** a todos os serviços e recursos — sem possibilidade de restrição via política.

Por isso, a prática recomendada (inclusive pela própria AWS) é:

- ❌ **Não usar o root** para tarefas do cotidiano (criar EC2, S3, Lambda, etc.)
- ✅ Usar o root **apenas** para tarefas administrativas da conta em si (trocar plano de suporte, alterar pagamento, fechar a conta, recuperar acesso em caso de bloqueio)
- ✅ Criar um **usuário no IAM** para usar normalmente, mesmo que seja você o único usuário da conta

Isso reduz drasticamente o risco em caso de credenciais expostas, vazadas ou comprometidas — um usuário IAM comprometido pode ter o dano limitado pelas políticas; o root, não.

---

## 2. Criando um User Group (Grupo de Usuários)

Antes de criar o usuário, é uma boa prática criar primeiro o **grupo** ao qual ele vai pertencer — assim a permissão já fica organizada desde o início.

**No exemplo da aula:**

- Foi criado um **User Group** (ex: `Administradores`)
- Foi anexada a esse grupo a política gerenciada **`AdministratorAccess`** — uma política da própria AWS que concede acesso total a praticamente todos os serviços

```
User Group: "Administradores"
      │
      └── Policy: AdministratorAccess (acesso total aos serviços)
```

📌 **Por que usar grupo em vez de aplicar a política direto no usuário?**
Mesmo com um único usuário, usar grupo facilita a escalabilidade futura — se você criar um segundo usuário (ex: para um colega, ou um usuário de teste com menos permissões), basta adicioná-lo ao grupo certo, sem reconfigurar políticas do zero.

⚠️ **Atenção:** `AdministratorAccess` dá acesso a **quase tudo** (exceto algumas ações exclusivas do root, como fechar a conta). Para estudos isso é prático, mas em ambientes reais de trabalho o ideal é seguir o princípio do **menor privilégio**, concedendo apenas as permissões que a função realmente precisa.

---

## 3. Criando o Usuário (User)

Ao criar o novo usuário no IAM, alguns pontos foram configurados:

| Configuração                                | O que significa                                                                                                                    |
| ------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| **Associar ao User Group criado**           | O usuário herda automaticamente todas as permissões definidas na política do grupo (no caso, acesso admin)                         |
| **Permitir acesso via CLI**                 | Habilita a geração de credenciais programáticas (Access Key + Secret Key) para uso fora do console, como terminal/linha de comando |
| **Exigir troca de senha no primeiro login** | Camada extra de segurança — a senha temporária criada na hora não fica sendo usada permanentemente                                 |

---

## 4. Autenticação Multifator (MFA) — obrigatório para ambos

Tanto o usuário **root** quanto o usuário **IAM criado** devem ter o **MFA (Multi-Factor Authentication)** ativado.

- MFA exige um segundo fator de verificação (ex: código gerado por app autenticador, como Google Authenticator ou Authy) além da senha.
- Mesmo que a senha seja comprometida, o acesso ainda fica bloqueado sem o segundo fator.
- É uma das proteções mais importantes e mais simples de configurar — deveria ser o primeiro passo de segurança em qualquer conta AWS.

```
Login = Senha + Código MFA (2 fatores)
```

---

## 5. Access Keys — acesso programático (CLI/SDK)

Para usar a AWS fora do console (via terminal, scripts, SDKs), é necessário gerar credenciais específicas para isso: as **Access Keys**.

Elas são geradas **dentro do usuário IAM** desejado, na seção de credenciais de segurança, e vêm em **par**:

| Credencial            | Função                                                                         |
| --------------------- | ------------------------------------------------------------------------------ |
| **Access Key ID**     | Identifica _quem_ está fazendo a requisição (como um "nome de usuário" da API) |
| **Secret Access Key** | A "senha" dessa chave — usada para autenticar/assinar as requisições           |

⚠️ **Pontos importantes sobre Access Keys:**

- A **Secret Access Key só é exibida uma única vez**, no momento da criação. Se perder, não tem como recuperá-la — só gerar uma nova.
- Nunca devem ser commitadas em repositórios de código (Git), nem compartilhadas.
- Podem (e devem) ser **desativadas ou rotacionadas** periodicamente por segurança.
- Se vazarem, qualquer pessoa pode usá-las para acessar a conta via CLI/API com as permissões daquele usuário.

---

## 6. Configurando a AWS CLI

Com a AWS CLI já instalada na máquina, o fluxo de configuração é:

### 6.1 Verificar se a CLI está instalada

```bash
aws --version
```

Retorna a versão instalada (ex: `aws-cli/2.x.x`). Se der erro de "comando não encontrado", a CLI precisa ser instalada antes de continuar.

### 6.2 Configurar as credenciais

```bash
aws configure
```

Esse comando solicita interativamente:

1. **AWS Access Key ID** → a chave gerada no passo anterior
2. **AWS Secret Access Key** → a chave secreta gerada (exibida só uma vez)
3. **Default region name** → região padrão para os comandos (ex: `sa-east-1` para São Paulo)
4. **Default output format** → formato de saída das respostas (ex: `json`, `text`, `table`)

Essas informações ficam salvas localmente em `~/.aws/credentials` e `~/.aws/config`.

---

## 7. Comandos de teste — validando a configuração

Depois de configurado, é possível testar se a CLI está autenticando corretamente com comandos simples e de baixo risco:

```bash
aws s3 ls
```

Lista todos os **buckets S3** ativos na conta. Se retornar a lista (ou vazio, caso não haja buckets) sem erro de credenciais, a configuração está correta.

```bash
aws ec2 describe-security-groups
```

Lista os **Security Groups** (regras de firewall virtual) configurados na conta, mostrando as permissões de entrada/saída de tráfego definidas para instâncias EC2.

💡 Ambos são comandos de **leitura** (somente consulta) — bons para testar acesso sem risco de criar ou alterar recursos por engano.

---

## 8. Resumo do fluxo completo

```
1. Logar como root (apenas para configuração inicial)
        │
        ▼
2. Ativar MFA no root
        │
        ▼
3. Criar User Group com policy de admin (ex: AdministratorAccess)
        │
        ▼
4. Criar User → associar ao grupo → permitir acesso via CLI → exigir troca de senha
        │
        ▼
5. Ativar MFA no novo usuário
        │
        ▼
6. Gerar Access Key + Secret Key do usuário
        │
        ▼
7. Configurar localmente: aws configure
        │
        ▼
8. Testar: aws s3 ls / aws ec2 describe-security-groups
        │
        ▼
9. Logout do root → usar somente o usuário IAM no dia a dia
```

---

## 9. Pontos-chave para fixar

- 🚫 Root é só para emergências/administração da conta — nunca para uso diário.
- 👥 Criar o **Group** antes do **User** facilita a gestão de permissões desde o início.
- 🔐 MFA ativado **nos dois** (root e usuário IAM) — não é opcional, é básico de segurança.
- 🔑 Access Keys = credenciais para CLI/API; a Secret Key só aparece **uma vez**, anote/guarde com segurança (idealmente num cofre de senhas).
- 💻 `aws configure` salva as credenciais localmente para a CLI usar nas próximas chamadas, sem precisar repetir o login.
- ✅ Testar com comandos de leitura (`ls`, `describe-*`) é a forma mais segura de validar se tudo está funcionando.

---

_Próximos passos sugeridos de estudo: uso de **IAM Identity Center** (substituto mais moderno para múltiplos usuários), criação de políticas customizadas (menor privilégio na prática), e diferença entre Access Keys de longa duração x credenciais temporárias via STS (`AssumeRole`)._
