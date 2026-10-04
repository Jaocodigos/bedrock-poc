# PoC: Revisor de READMEs usando AWS Bedrock

Tudo roda na sua máquina. Não há URL pública: o servidor escuta só em `127.0.0.1` e usa as suas credenciais AWS para chamar o Bedrock.

```
Navegador (index.html) → server.py (localhost:8000) → Bedrock
```

Arquivos desta pasta:

- `server.py`: serve a página e chama o Bedrock. O prompt fixo e o checklist ficam no topo dele.
- `src/index.html`: interface estática.

## 1. Pré-requisitos

- Conta AWS com acesso ao console.
- Python 3.10 ou superior.
- AWS CLI v2 instalado(`aws --version`).

## 2. Controle de custos (OPCIONAL MAS RECOMENDADO)

1. No console, abra **Billing and Cost Management → Budgets**.
2. Crie um orçamento mensal baixo (por exemplo, US$ 10) com alerta por e-mail em 80%.

## 3. Credenciais com permissão mínima

Crie um usuário IAM (ou, se preferir, use o IAM Identity Center com login SSO), com esta política:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["bedrock:InvokeModel", "bedrock:InvokeModelWithResponseStream"],
      "Resource": "*"
    }
  ]
}
```

Depois configure o CLI:

```bash
aws configure          # chaves de acesso do usuário IAM
# ou, com SSO:
aws configure sso
```

Se o modelo usar um perfil de inferência, a política pode precisar incluir o ARN desse perfil. Se a chamada falhar com erro de permissão, a mensagem indica o recurso que está faltando.

## 4. Escolher e liberar o modelo no Bedrock

1. Escolha a região (por exemplo `us-east-1`). A disponibilidade de modelos varia por região.
2. No console do Bedrock, abra o catálogo de modelos e escolha um modelo de texto que atenda português.
3. Se o console pedir para solicitar ou aceitar os termos de acesso ao modelo, faça isso.
4. Copie o **ID do modelo** (ou o **ID do perfil de inferência**, quando o console exigir) exatamente como aparece.

## 5. Instalar e configurar

```bash
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
uv sync
```

Crie um arquivo **.env** dentro da pasta **src** e defina as variáveis de ambiente .env:

```dotenv
BEDROCK_MODEL_ID="cole-o-id-aqui"
AWS_REGION="regiao-de-preferencia"
```

## 6. Ajustar o checklist

Abra `server.py` e edite a lista `CHECKLIST` com os critérios do que um README deve cobrir (já vem preenchida com um checklist inicial: descrição, requisitos de sistema, instalação, variáveis de ambiente, como rodar etc.). Quanto mais específico e verificável for cada item, melhor o resultado. O texto de `PROMPT` também fica ali: é o prompt fixo, e você só envia o documento.

## 7. Rodar

```bash
python server.py
```

Abra `http://127.0.0.1:8000`, cole um README e clique em **Analisar README**.

## 8. Avaliar o resultado

Escolha os READMEs (de projetos próprios ou de teste) com lacunas que você já conhece e confira.
Ajuste o prompt e o checklist conforme a necessidade e repita.

## Boas práticas de segurança

- Use só dados fictícios ou anonimizados.
- Não troque `127.0.0.1` por `0.0.0.0`: isso exporia o servidor à rede local.
- Quando terminar os testes, desative ou apague as chaves do usuário IAM.

## Limitações do PoC

- Aceita só texto (`.md`, `.txt` ou colado). O scan vê apenas o README, não o código do projeto.
- Limite de 200.000 caracteres por documento, definido em `MAX_CHARS`.
- Sem login, histórico ou fila de processamento.

## Plano de integração total com a AWS

1. Migrar a lógica do `server.py` para uma Lambda, com API Gateway e autenticação por Cognito, e a interface em S3 + CloudFront.
2. Usar o padrão assíncrono (resposta imediata com identificador de tarefa e consulta de status) para documentos longos.

## Conclusão

A intenção era entender o comportamento injetando o prompt por API usando ao usar um FM, e devo dizer que pra grande maioria dos casos envolvendo documentações que não sejam extensos ou numerosos,
validações de regras sem grandes rodeios de negócio, essa forma já cumpre bem e consegue ser até "barato".
