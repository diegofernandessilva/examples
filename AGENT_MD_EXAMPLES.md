# Exemplos de Projetos com agent.md para Desenvolvimento Backend Node.js

Este documento lista todos os exemplos de projetos que agora possuem arquivos `agent.md` para auxiliar no desenvolvimento de aplicações backend usando Node.js.

## Projetos com agent.md

### 1. NestJS com TypeScript - AWS Lambda
**Diretório:** `aws-node-typescript-nest/`

**Tecnologias:**
- NestJS 5.x
- TypeScript 3.x
- AWS Lambda
- Serverless Framework
- Jest para testes

**Características:**
- Arquitetura modular do NestJS
- Dependency Injection
- Integração com AWS Lambda via aws-serverless-express
- Servidor cacheado para melhorar cold starts

**agent.md:** `/aws-node-typescript-nest/agent.md`

---

### 2. Express.js - AWS Lambda
**Diretório:** `aws-node-express-api/`

**Tecnologias:**
- Express.js 4.x
- JavaScript (Node.js)
- AWS Lambda
- Serverless Framework
- serverless-http adapter

**Características:**
- Express tradicional adaptado para Lambda
- Função Lambda única gerencia todas as rotas
- Simples e direto para APIs RESTful básicas
- Fácil de estender com middleware Express

**agent.md:** `/aws-node-express-api/agent.md`

---

### 3. TypeScript REST API com MongoDB
**Diretório:** `aws-node-rest-api-typescript/`

**Tecnologias:**
- TypeScript 3.x
- MongoDB Atlas
- Mongoose 5.x
- AWS Lambda
- Mocha/Chai para testes
- TSLint

**Características:**
- Arquitetura em camadas (Controller-Service-Model)
- Integração com MongoDB Atlas
- DTOs (Data Transfer Objects)
- Gestão multi-ambiente (.env.dev, .env.stg, .env.pro)
- Cobertura de testes

**agent.md:** `/aws-node-rest-api-typescript/agent.md`

---

### 4. REST API com DynamoDB
**Diretório:** `aws-node-rest-api-with-dynamodb/`

**Tecnologias:**
- JavaScript (Node.js)
- AWS DynamoDB
- AWS Lambda
- Serverless Framework
- UUID para geração de IDs

**Características:**
- CRUD completo (Create, Read, Update, Delete)
- Uma função Lambda por operação
- Integração nativa com DynamoDB
- CORS habilitado
- Tabela DynamoDB gerenciada pelo Serverless Framework

**agent.md:** `/aws-node-rest-api-with-dynamodb/agent.md`

---

## Como Usar os Arquivos agent.md

Os arquivos `agent.md` servem como guias completos para agentes de IA que trabalham com desenvolvimento backend. Cada arquivo contém:

### Conteúdo Típico

1. **Visão Geral do Projeto**
   - Descrição da arquitetura
   - Stack de tecnologias
   - Estrutura de diretórios

2. **Padrões de Arquitetura**
   - Padrões de design utilizados
   - Organização de código
   - Melhores práticas

3. **Diretrizes de Desenvolvimento**
   - Como adicionar novos recursos
   - Exemplos de código
   - Padrões de nomenclatura

4. **Comandos**
   - Desenvolvimento local
   - Testes
   - Deploy
   - Logs

5. **Integração com Banco de Dados**
   - Operações CRUD
   - Otimizações
   - Boas práticas

6. **Testes**
   - Estratégias de teste
   - Exemplos de testes unitários
   - Testes de integração

7. **Segurança**
   - Validação de entrada
   - Gestão de secrets
   - IAM e permissões
   - Boas práticas

8. **Performance**
   - Otimizações
   - Cold starts
   - Escalabilidade

9. **Debugging e Monitoring**
   - CloudWatch Logs
   - Métricas
   - Troubleshooting

10. **Recursos**
    - Links para documentação oficial
    - Guias e tutoriais

## Exemplos de Uso

### Para Desenvolvedores

1. Clone o repositório
2. Navegue até o diretório do exemplo desejado
3. Leia o `agent.md` para entender a arquitetura
4. Siga as instruções de setup e desenvolvimento

### Para Agentes de IA

Os agentes de IA podem usar esses arquivos para:
- Entender a estrutura do projeto rapidamente
- Seguir as convenções estabelecidas
- Implementar novos recursos de forma consistente
- Aplicar as melhores práticas documentadas

## Comparação Rápida dos Exemplos

| Projeto | Framework | Linguagem | Banco de Dados | Complexidade |
|---------|-----------|-----------|----------------|--------------|
| NestJS | NestJS | TypeScript | Nenhum (exemplo básico) | Média |
| Express API | Express | JavaScript | Nenhum (exemplo básico) | Baixa |
| TypeScript REST | Custom | TypeScript | MongoDB Atlas | Alta |
| REST DynamoDB | Custom | JavaScript | DynamoDB | Média |

## Casos de Uso

### Use NestJS quando:
- Precisa de uma arquitetura escalável e bem estruturada
- Quer dependency injection out-of-the-box
- Equipe familiarizada com Angular
- Projeto de médio a grande porte

### Use Express quando:
- Precisa de algo simples e rápido
- Equipe familiarizada com Express
- Projeto pequeno ou proof-of-concept
- Quer máxima flexibilidade

### Use TypeScript REST API quando:
- Precisa de type safety
- Vai usar MongoDB
- Quer arquitetura em camadas clara
- Projeto enterprise com múltiplos ambientes

### Use REST com DynamoDB quando:
- Precisa de alta escalabilidade
- Quer pay-per-request pricing
- Dados não relacionais
- Integração nativa com AWS

## Próximos Passos

Para expandir esses exemplos, considere adicionar:

1. **Autenticação e Autorização**
   - JWT
   - AWS Cognito
   - Custom authorizers

2. **Validação Avançada**
   - Schema validation (Joi, Yup)
   - DTOs com decorators (class-validator)

3. **Documentação de API**
   - Swagger/OpenAPI
   - Postman collections

4. **CI/CD**
   - GitHub Actions
   - AWS CodePipeline
   - Testes automatizados

5. **Monitoring Avançado**
   - AWS X-Ray
   - CloudWatch Dashboards
   - Alertas personalizados

## Contribuindo

Para adicionar novos exemplos com agent.md:

1. Crie o exemplo de projeto
2. Adicione um arquivo `agent.md` completo
3. Siga a estrutura dos exemplos existentes
4. Documente todas as particularidades do projeto
5. Inclua exemplos de código práticos
6. Adicione ao README principal

## Recursos Adicionais

- [Serverless Framework Documentation](https://www.serverless.com/framework/docs/)
- [AWS Lambda Best Practices](https://docs.aws.amazon.com/lambda/latest/dg/best-practices.html)
- [NestJS Documentation](https://docs.nestjs.com/)
- [Express.js Guide](https://expressjs.com/en/guide/routing.html)
- [MongoDB Atlas](https://www.mongodb.com/cloud/atlas)
- [DynamoDB Developer Guide](https://docs.aws.amazon.com/dynamodb/)

---

**Última Atualização:** 2026-01-18

**Autor:** GitHub Copilot Agent

**Exemplos Documentados:** 4 projetos backend Node.js com agent.md completos
