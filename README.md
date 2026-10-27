# Trabalho Prático — Pipeline DevSecOps com GitHub Actions

Este repositório contém a implementação do pipeline de integração e entrega contínuas com foco em segurança (DevSecOps e *shift-left security*) para a aplicação `ufla-devops-shop`.

O pipeline é executado via **GitHub Actions** em Pull Requests direcionados para a branch principal (`tp-main`) e em pushes diretos nela, garantindo que nenhum código vulnerável, segredo vazado ou imagem insegura seja integrado ou publicado.

---

## 1. Ferramentas Integradas

| Ferramenta | Tipo | Função no Pipeline |
| :--- | :--- | :--- |
| **Gitleaks** | Secret Detection | Analisa o histórico e commits em busca de segredos, tokens, senhas ou credenciais expostas. |
| **Pytest + Coverage** | Testes & Cobertura | Executa a suíte de testes unitários da aplicação e gera o relatório `coverage.xml`. |
| **SonarCloud** | SAST & Quality Gate | Executa análise estática de código e segurança, validando métricas e Quality Gate sobre o código novo. |
| **Trivy (FS)** | SCA (Dependências) | Varre arquivos de configuração e dependências do repositório (`requirements.txt`, etc.). |
| **Trivy (Image)** | Container Security | Varre a imagem Docker construída em busca de vulnerabilidades no sistema operacional base e bibliotecas empacotadas. |
| **Syft** | Geração de SBOM | Cria o inventário de componentes de software (Software Bill of Materials) da imagem Docker em formato SPDX JSON. |
| **Docker Hub** | Container Registry | Armazena e distribui a imagem validada (`djhuan/ufla-devops-shop`), condicionada à aprovação de todos os gates. |

---

## 2. Verificações do Pipeline

O workflow [`.github/workflows/devsecops.yml`](.github/workflows/devsecops.yml) orquestra as seguintes verificações:

1. **Gitleaks (Secret Detection)**:
   - Executa `gitleaks/gitleaks-action@v2` com profundidade total (`fetch-depth: 0`).
   - Verifica commits introduzidos em Pull Requests e pushes.
2. **Build, Testes e Cobertura**:
   - Configura o runtime Python 3.12 e instala dependências de desenvolvimento (`requirements-dev.txt`).
   - Roda `pytest tests/ --cov=app --cov-report=xml` e publica o artefato `coverage.xml`.
3. **SonarCloud (SAST e Quality Gate)**:
   - Executa a análise estática via `SonarSource/sonarqube-scan-action@v6.0.0` consumindo os relatórios de cobertura.
   - Aguarda o processamento no SonarCloud e consulta o status do Quality Gate para o projeto `DJhuan_ufla-devops-shop`.
4. **Trivy (Varredura de Dependências - Filesystem)**:
   - Analisa o repositório em dois níveis de severidade: auditoria e aplicação de políticas de corte.
5. **Docker Build, Trivy Image Scan e Geração de SBOM**:
   - Constrói a imagem Docker multi-stage localmente.
   - Analisa a imagem construída com o Trivy.
   - Gera o arquivo SBOM `sbom.spdx.json` utilizando o Syft (`anchore/sbom-action@v0`).
   - Publica o arquivo como artefato da execução (`actions/upload-artifact@v4`) com o nome `sbom-docker-image`.
6. **Publicação no Registry (Docker Hub)**:
   - Disparado exclusivamente em eventos de `push` na branch `tp-main`.
   - Requer como dependência explícita o sucesso de todos os jobs anteriores (`gitleaks`, `sonarcloud`, `trivy-fs`, `docker-security-sbom`).

---

## 3. Regras de Bloqueio

O pipeline aplica o conceito de *shift-left security*, interrompendo o ciclo de integração nas seguintes condições:

- **Detecção de Segredos**: Qualquer ocorrência de chave privada, token de API, senha ou credencial identificada pelo Gitleaks encerra o job imediatamente com erro (`exit code 1`).
- **Reprovação no Quality Gate (SonarCloud)**: Caso o código introduzido viole as políticas de qualidade ou segurança definidas no SonarCloud (status `ERROR`), a esteira é bloqueada.
- **Vulnerabilidades CRITICAL e HIGH com Correção (Trivy)**:
  - Dependências (`scan-type: fs`) ou pacotes da imagem (`scan-type: image`) com severidade **CRITICAL** ou **HIGH** que possuam correção disponível (`ignore-unfixed: true`) acionam bloqueio mandatário (`exit-code: 1`).
- **Auditoria de LOW e MEDIUM (Trivy)**:
  - Vulnerabilidades de severidade **LOW** e **MEDIUM** são registradas detalhadamente nos logs para fins de conformidade e auditoria, sem bloquear a execução (`exit-code: 0`).
- **Gate de Publicação da Imagem**: A publicação da imagem final para o Docker Hub é bloqueada se qualquer uma das verificações acima falhar.

---

## 4. Estrutura do Repositório

```text
ufla-devops-shop/
├── .github/
│   └── workflows/
│       └── devsecops.yml       # Definição completa do pipeline DevSecOps
├── app/                        # Código-fonte da API (FastAPI)
├── static/                     # Arquivos estáticos do frontend
├── tests/                      # Testes automatizados (pytest)
├── Dockerfile                  # Construção multi-stage da imagem da aplicação
├── sonar-project.properties    # Configurações do projeto e coverage para o SonarCloud
├── requirements.txt            # Dependências de execução
├── requirements-dev.txt        # Dependências de desenvolvimento, lint e testes
└── README.md                   # Documentação do projeto e pipeline
```
