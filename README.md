# POC AWS - Website Estático com API Gateway e Lambda

Este projeto demonstra a implementação de uma aplicação web estática hospedada no AWS S3, com uma API REST implementada usando API Gateway e Lambda, distribuída através do CloudFront.

## 📋 Arquitetura

A solução é composta por:

- **S3**: Hospedagem do website estático
- **API Gateway**: Endpoint REST para a API
- **Lambda**: Função que processa as requisições da API
- **CloudFront**: CDN para distribuição do conteúdo estático
- **IAM**: Gerenciamento de permissões

## 🚀 Deploy

### Pré-requisitos

- AWS CLI configurado com credenciais apropriadas
- Acesso à AWS com permissões para criar recursos (S3, Lambda, API Gateway, CloudFront, IAM)

### Passos para Deploy

1. **Deploy do CloudFormation**

```bash
aws cloudformation deploy \
  --template-file poc-template.yaml \
  --stack-name poc-aws-free-tier \  
  --capabilities CAPABILITY_NAMED_IAM
```

2. **Obtenha os outputs do stack**

```bash
aws cloudformation describe-stacks \
  --stack-name poc-aws-free-tier \
  --query 'Stacks[0].Outputs'
```

4. **Deploy do website**

```bash
# Substitua os valores no arquivo index.html
API_URL=$(aws cloudformation describe-stacks \
  --stack-name poc-aws-free-tier \
  --query 'Stacks[0].Outputs[?OutputKey==`ApiUrl`].OutputValue' \
  --output text)

sed -i '' "s|YOUR_API_ID.execute-api.YOUR_REGION.amazonaws.com/prod/poc|$API_URL|g" index.html

# Upload do arquivo para o S3
BUCKET_NAME=$(aws cloudformation describe-stacks \
  --stack-name poc-aws-free-tier \
  --query 'Stacks[0].Outputs[?OutputKey==`BucketName`].OutputValue' \
  --output text)

aws s3 cp index.html s3://$BUCKET_NAME/
```

## 📄 Arquivos do Projeto

### poc-template.yaml

Template CloudFormation que define toda a infraestrutura necessária. Principais recursos:

- Bucket S3 para website estático
- Função Lambda com resposta simples
- API Gateway configurado com CORS
- Distribuição CloudFront
- Roles e políticas IAM necessárias

### index.html

Página web estática com:

- Botão para acionar a API
- JavaScript para fazer a requisição fetch
- Área para exibição da resposta
- Tratamento básico de erros

## 🔧 Configuração do Website

1. Após o deploy, atualize o arquivo `index.html` com a URL correta da API (disponível nos outputs do CloudFormation)
2. Faça upload do arquivo para o bucket S3 criado
3. Acesse o site através da URL do CloudFront (disponível nos outputs)

## 🌐 URLs Importantes

- **Website**: https://{CloudFront-Domain-Name}
- **API**: https://{API-Gateway-ID}.execute-api.{region}.amazonaws.com/{environment}/poc

## ⚙️ Parâmetros de Deploy

- **Environment**: Ambiente de deploy (dev/prod)

## 🔒 Segurança

- Bucket S3 configurado com políticas apropriadas
- CORS habilitado na API
- HTTPS forçado no CloudFront
- Versionamento do bucket ativado
- Política de retenção para evitar deleção acidental

## 📝 Notas Importantes

1. Aguarde alguns minutos após o deploy para que o CloudFront propague as configurações
2. Certifique-se de atualizar a URL da API no arquivo index.html
3. O ambiente pode ser alterado usando o parâmetro Environment no deploy
4. Todas as requisições são redirecionadas para HTTPS
5. O cache do CloudFront está configurado para 1 hora por padrão

## 🧹 Limpeza

Para remover todos os recursos:

```bash
aws cloudformation delete-stack \
  --stack-name poc-aws-free-tier
```

## 📊 Monitoramento

- Logs do Lambda disponíveis no CloudWatch Logs
- Métricas do API Gateway no CloudWatch
- Estatísticas de acesso do CloudFront disponíveis
