# Trabalho Prático DevSecOps com GitHub Actions

A branch principal de trabalho é a `tp-main` e o desenvolvimento está centralizado na `tp-dev`.

---

## 1. Ferramentas Integradas e Workflows

Arquivo criado: [`.github/workflows/devsecops.yml`](file:///home/jhuan/Documentos/P8/devops/ufla-devops-shop/.github/workflows/devsecops.yml)

### Requisito 1 — Gitleaks: detecção de segredos
- **Ação**: Análise de histórico e commits utilizando `gitleaks/gitleaks-action@v2`.
- **Comportamento**: Bloqueia imediatamente o pipeline caso tokens, senhas ou chaves sejam identificados.
- **Resultado**: ✅ Execução validada e aprovada nos commits e Pull Requests.

### Requisito 2 — SonarCloud: análise estática (SAST)
- **Ação**: Integração do SonarCloud utilizando `SonarSource/sonarqube-scan-action@v6.0.0` e validação com Quality Gate (`sonar-project.properties`).
- **Comportamento**: Falha a verificação se a qualidade e cobertura no código novo não atenderem aos critérios definidos.
- **Resultado**: ✅ Análise executada com sucesso e Quality Gate consultado e aprovado.

### Requisito 3 — Trivy: análise de dependências e imagens
- **Ação**: Scan de dependências do código (`fs`) e da imagem Docker construída (`image`) via `aquasecurity/trivy-action`.
- **Comportamento**:
  - `CRITICAL` e `HIGH` com correção disponível (`ignore-unfixed: true` e `exit-code: 1`): **Bloqueante**.
  - `LOW` e `MEDIUM` (`exit-code: 0`): **Não-bloqueante** (geração de log e auditoria).
- **Resultado**: ✅ Ambos os níveis de severidade devidamente categorizados e funcionando.

### Requisito 4 — Syft: geração de SBOM
- **Ação**: Geração do Software Bill of Materials (SBOM) da imagem Docker utilizando `anchore/sbom-action@v0` no formato padrão `spdx-json`.
- **Publicação**: Artefato `sbom-docker-image` publicado nos artifacts da execução via `actions/upload-artifact@v4`.
- **Resultado**: ✅ Artefato `sbom-docker-image` gerado e disponível para download.

### Publicação da Imagem Docker
- **Ação**: Build e push para o Docker Hub (`djhuan/ufla-devops-shop`) utilizando `docker/build-push-action@v6`.
- **Regra**: Somente dispara em push na branch `tp-main` e condicionada ao sucesso de **todos** os jobs anteriores (`gitleaks`, `sonarcloud`, `trivy-fs`, `docker-security-sbom`).
- **Resultado**: ✅ Execução bem-sucedida e tags enviadas para o Docker Hub.

---

## 2. Requisito 5 — Demonstração do Bloqueio e da Correção de CVE

Para comprovar que a segurança não é passiva e bloqueia efetivamente o merge:

1. **Introdução da vulnerabilidade**:
   - Criada a branch `test/cve-block` a partir de `tp-main`.
   - Adicionada a dependência vulnerável `urllib3==1.26.4` com CVE conhecida e correção disponível ([`CVE-2021-33503`](https://avd.aquasec.com/nvd/cve-2021-33503)).
   - Aberto o Pull Request [#1](https://github.com/DJhuan/ufla-devops-shop/pull/1).

2. **Comprovação do Bloqueio**:
   - O pipeline falhou no job `Trivy (Dependency Scan)` e no job `Docker Build, Trivy Image Scan & Syft SBOM`.
   - **Trecho do relatório Trivy**:
     ```text
     requirements.txt (pip)
     Total: 8 (HIGH: 8, CRITICAL: 0)

     ┌─────────┬────────────────┬──────────┬────────┬───────────────────┬────────────────┬─────────────────────────────────────────────────────────────┐
     │ Library │ Vulnerability  │ Severity │ Status │ Installed Version │ Fixed Version  │                            Title                            │
     ├─────────┼────────────────┼──────────┼────────┼───────────────────┼────────────────┼─────────────────────────────────────────────────────────────┤
     │ urllib3 │ CVE-2021-33503 │ HIGH     │ fixed  │ 1.26.4            │ 1.26.5         │ python-urllib3: ReDoS in the parsing of authority part of   │
     │         │                │          │        │                   │                │ URL                                                         │
     │         │                │          │        │                   │                │ https://avd.aquasec.com/nvd/cve-2021-33503                  │
     └─────────┴────────────────┴──────────┴────────┴───────────────────┴────────────────┴─────────────────────────────────────────────────────────────┘
     Process completed with exit code 1.
     ```
   - O Pull Request foi bloqueado, impedindo a publicação em produção.

3. **Correção e Comprovação do Sucesso**:
   - A dependência vulnerável foi removida no commit `954d7bf`.
   - O Pull Request [#1](https://github.com/DJhuan/ufla-devops-shop/pull/1) foi atualizado automaticamente.
   - Todos os checks do pipeline de segurança passaram com status verde:
     - `Gitleaks (Secret Detection)`: ✅ SUCCESS
     - `Tests & Coverage`: ✅ SUCCESS
     - `Trivy (Dependency Scan)`: ✅ SUCCESS
     - `SonarCloud SAST & Quality Gate`: ✅ SUCCESS
     - `Docker Build, Trivy Image Scan & Syft SBOM`: ✅ SUCCESS (SBOM publicado)

---

## 3. Links e Entregáveis

- **Imagens de evidências**: [Evidências](./docs/evidences.md)
- **Branch Principal do TP**: [`tp-main`](https://github.com/DJhuan/ufla-devops-shop/tree/tp-main)
- **Branch de Desenvolvimento**: [`tp-dev`](https://github.com/DJhuan/ufla-devops-shop/tree/tp-dev)
- **Pull Request de Demonstração (Bloqueio e Correção)**: [PR #1](https://github.com/DJhuan/ufla-devops-shop/pull/1)
- **Execução Aprovada (Push em `tp-main` com Docker Hub)**: [Run 40428016907](https://github.com/DJhuan/ufla-devops-shop/actions/runs/40428016907)
- **Execução Bloqueada por CVE**: [Run 40428682616](https://github.com/DJhuan/ufla-devops-shop/actions/runs/40428682616)
- **Execução Aprovada após correção da CVE**: [Run 40429028895](https://github.com/DJhuan/ufla-devops-shop/actions/runs/40429028895)
