# PGATS - CI — Exercício 3: GitHub Actions com Self-Hosted Runner

## Pré-requisitos

1. Instale o [git](https://git-scm.com)
2. Instale o [Node.js 20+](https://nodejs.org/)
3. Clone este repositório e instale as dependências
   ```shell
   cd pgats-ci-pipeline-exercise-3-github-actions-selfhosted-CI
   npm install
   ```
4. Execute os testes de unidade
   ```shell
   npm test
   ```
5. Execute os testes de mutação com o Stryker
   ```shell
   npm run test:mutation
   ```
6. Instale os navegadores do Playwright
   ```shell
   npx playwright install --with-deps
   ```
7. Execute os testes end-to-end com o Playwright
   ```shell
   npm run e2e
   ```
8. Execute a aplicação localmente
   ```shell
   npm start
   ```

---

## Estratégia de CI/CD

O pipeline é executado pelo **GitHub Actions** com um **self-hosted runner** registrado na máquina local, eliminando a dependência de infraestrutura gerenciada pelo GitHub.

**Fluxo:**
1. O desenvolvedor faz push ou abre um Pull Request
2. O GitHub Actions detecta os jobs com `runs-on: self-hosted`
3. O runner local recebe e executa os jobs via polling HTTPS
4. Artefatos e resultados ficam disponíveis na aba **Actions** do repositório

---

## Pipeline atual (.github/workflows/ci.yml)

O pipeline possui 4 jobs executados em paralelo (exceto `allure_report`):

| Job | Comando | Artefatos |
|---|---|---|
| `unit_tests` | `npm test` | `allure-results/`, `reports/coverage/` |
| `mutation_tests` | `npm run test:mutation` | `reports/mutation/` |
| `e2e_tests` | `npm run e2e` | `playwright-report/`, `test-results/`, `results.xml` |
| `allure_report` | geração do relatório Allure | `allure-report/` |

O job `allure_report` depende de `unit_tests` e `e2e_tests` e sempre executa (`if: always()`), consolidando os resultados em um único relatório Allure.

---

## Configurando o Self-Hosted Runner

Consulte o guia completo em [SELF_HOSTED_RUNNER.md](SELF_HOSTED_RUNNER.md).

**Resumo rápido:**
1. Acesse **Settings → Actions → Runners → New self-hosted runner** no repositório
2. Siga os comandos gerados pelo GitHub para seu SO
3. Inicie o runner: `./run.cmd` (Windows) ou `./run.sh` (Linux/macOS)

O runner precisa estar ativo para os jobs serem despachados. Você pode acompanhar as execuções em:
```
https://github.com/LRCCN/pgats-ci-pipeline-exercise-3-github-actions-selfhosted-CI/actions
```
