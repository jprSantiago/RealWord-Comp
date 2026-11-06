# RealWorld API: CI/CD

API do Conduit (clone do Medium) em Node.js + Express + TypeScript + Prisma + PostgreSQL, baseada em gothinkster/node-express-realworld-example-app, com pipeline no GitHub Actions e deploy automático no Render.

Produção: https://realworld-comp-api.onrender.com

![CI/CD](https://github.com/jprSantiago/RealWord-Comp/actions/workflows/ci.yml/badge.svg)
![Quality gate](https://sonarcloud.io/api/project_badges/quality_gate?project=jprSantiago_RealWord-Comp)
![Coverage](https://sonarcloud.io/api/project_badges/measure?project=jprSantiago_RealWord-Comp&metric=coverage)

## Como rodar

    docker run -d --name realworld-db -p 5432:5432 -e POSTGRES_USER=realworld \
    -e POSTGRES_PASSWORD=realworld -e POSTGRES_DB=realworld postgres:16-alpine

    docker exec realworld-db createdb -U realworld realworld_test

    cp .env.example .env

    npm ci && npx prisma migrate deploy

    npm run test:coverage

## O que mudei no projeto original (e por quê)-

-Ordem do mock do Prisma (7 testes quebrados no original): testes isolados.

- app.ts separado do main.ts: app testável com Supertest.- JWT_SECRET obrigatório (fail fast), sem valor padrão no código.
- npm audit fix e Express 4.22 (11 pacotes vulneráveis → 0 em produção).
- ESLint 8 via Nx (33 erros) → ESLint 9 flat config + Prettier (0 erros).
- .npmrc com legacy-peer-deps, para o conflito de versão do Nx.
- Cobertura de 25,8% → 92% de linhas, com testes de integração num banco separado.

## Arquitetura da pipeline

(diagrama Mermaid + tabela com as 6 etapas, ferramenta e motivo de cada uma)

## Decisões de segurança

- Segredos só em GitHub Secrets/Environments e no Render; Gitleaks varre a história inteira.
- Imagem multi-stage em Alpine, sem npm, rodando como usuário não-root.
- Trivy bloqueia HIGH/CRITICAL antes do push; a imagem publicada é a escaneada.
- Permissões mínimas; packages: write só no job de push.
- Actions de segurança fixadas por hash de commit.
- Deploy pela tag sha-<commit>: rastreável e com rollback.

## Quality Gate

"Sonar way" (80% de cobertura no código novo, mais rígido que os 70% do desafio),
sonar.qualitygate.wait=true, ![PR Reprovado](/docs/PR-Reprovado.jpeg)
![Outra Falha](/docs/Complemento1.jpeg)
![Sql injection](/docs/SQL%20Injection.jpeg)

## Fluxo de branches

GitHub Flow: branch → PR (etapas 1 a 4) → merge na main (1 a 6) → tag vX.Y.Z.
