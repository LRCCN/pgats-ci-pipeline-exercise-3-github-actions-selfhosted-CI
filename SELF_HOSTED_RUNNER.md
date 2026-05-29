# Exercício 3 — Self-Hosted Runner no GitHub Actions

## O que é um Self-Hosted Runner?

Um **self-hosted runner** é uma máquina (física, virtual ou container) que você mesmo provisiona e registra no GitHub/GitLab/etc. para executar os jobs da pipeline CI/CD, em vez de usar os agentes efêmeros fornecidos pela plataforma (GitHub-hosted runners).

---

## Quando faz sentido usar?

| Cenário | Por quê Self-Hosted? |
|---|---|
| **Recursos de hardware específicos** | GPU para ML/AI, CPU de alta performance, hardware proprietário |
| **Rede privada / VPN** | Acesso a banco de dados interno, serviços on-premise, ambiente sem saída pública |
| **Custo** | Pipelines longas ou com alto volume de execuções — runners hospedados são cobrados por minuto |
| **Compliance / segurança** | Código-fonte não pode sair da infraestrutura da empresa (LGPD, ISO 27001, PCI-DSS) |
| **Cache persistente** | Reutilizar `node_modules`, camadas Docker, artefatos de build entre execuções sem re-download |
| **SO ou arquitetura customizados** | ARM64, Windows específico, distribuições Linux legadas |
| **Builds que demoram mais de 6h** | GitHub-hosted runners têm timeout de 6h por job |

---

## Outras plataformas oferecem recursos similares?

Sim. O conceito é universal em plataformas de CI/CD:

| Plataforma | Nome do Recurso |
|---|---|
| **GitHub Actions** | Self-hosted runners |
| **GitLab CI/CD** | GitLab Runner (self-managed) |
| **Azure DevOps** | Self-hosted agents / agent pools |
| **Jenkins** | Agents / Nodes |
| **CircleCI** | Self-hosted runners |
| **Bitbucket Pipelines** | Runners |
| **TeamCity** | Build Agents |

---

## Como configurar o Self-Hosted Runner neste projeto

### 1. Acessar as configurações do repositório

```
https://github.com/LRCCN/pgats-ci-pipeline-exercise-3-github-actions-selfhosted-CI
→ Settings → Actions → Runners → New self-hosted runner
```

### 2. Selecionar o SO e seguir os comandos gerados

O GitHub gera os comandos específicos para seu SO. Exemplo para **Windows (PowerShell)**:

```powershell
# Criar pasta e entrar nela
mkdir actions-runner; cd actions-runner

# Baixar o runner (versão atual — use a gerada pelo GitHub)
Invoke-WebRequest -Uri https://github.com/actions/runner/releases/download/v2.x.x/actions-runner-win-x64-2.x.x.zip -OutFile actions-runner-win-x64.zip

# Extrair
Add-Type -AssemblyName System.IO.Compression.FileSystem
[System.IO.Compression.ZipFile]::ExtractToDirectory("$PWD/actions-runner-win-x64.zip", "$PWD")

# Configurar (token gerado pelo GitHub)
./config.cmd --url https://github.com/LRCCN/pgats-ci-pipeline-exercise-3-github-actions-selfhosted-CI --token <TOKEN_GERADO>

# Iniciar o runner
./run.cmd
```

Para **Linux/macOS**:

```bash
mkdir actions-runner && cd actions-runner
curl -o actions-runner-linux-x64.tar.gz -L https://github.com/actions/runner/releases/download/v2.x.x/actions-runner-linux-x64-2.x.x.tar.gz
tar xzf ./actions-runner-linux-x64.tar.gz
./config.sh --url https://github.com/LRCCN/pgats-ci-pipeline-exercise-3-github-actions-selfhosted-CI --token <TOKEN_GERADO>
./run.sh
```

### 3. Instalar como serviço (execução persistente)

**Windows:**
```powershell
./svc.cmd install
./svc.cmd start
```

**Linux:**
```bash
sudo ./svc.sh install
sudo ./svc.sh start
```

---

## Como o pipeline usa o self-hosted runner

Todos os jobs no [.github/workflows/ci.yml](.github/workflows/ci.yml) usam:

```yaml
runs-on: self-hosted
```

Isso instrui o GitHub Actions a despachar o job para qualquer runner registrado com a label `self-hosted` neste repositório, em vez de usar infraestrutura gerenciada pelo GitHub.

### Fluxo de execução

```
Push/PR → GitHub Actions agenda o job
         → Runner registrado recebe o job via polling HTTPS
         → Runner executa os steps localmente
         → Resultados/artefatos são enviados de volta ao GitHub
```

---

## Pré-requisitos na máquina do runner

Para este projeto, a máquina precisa ter:

- **Node.js 20+** (o step `actions/setup-node` gerencia isso, mas requer acesso à internet ou cache local)
- **Java 17+** (usado pelo job `allure_report` via `actions/setup-java`)
- **Playwright dependencies** (o step `npx playwright install --with-deps` instala navegadores Chromium/Firefox/WebKit)
- **Git**
- Acesso à internet para baixar actions e dependências npm

---

## Considerações de segurança

- **Nunca** registre um self-hosted runner em repositório público sem entender os riscos — qualquer pessoa pode abrir um PR e executar código arbitrário na sua máquina.
- Use runners em repositórios privados ou configure **Required Reviewers** para aprovar workflows de forks.
- Considere usar runners em containers Docker isolados para cada job (`--ephemeral` flag).
- Rotacione os tokens de registro periodicamente.
