# ufla-devops-shop

Repositório da turma de **DevOps na Prática** — DCC / UFLA.

Prof. Dr. Rafael Serapilha Durelli · [rafael.durelli@ufla.br](mailto:rafael.durelli@ufla.br)

Este repositório é o **projeto fio condutor** da disciplina. Você faz um *fork*
dele na Atividade 0 e trabalha no seu fork durante as 15 semanas: a mesma
aplicação vai sendo containerizada, testada, empacotada, implantada, monitorada
e provisionada, semana após semana.

---

## Comece por aqui

1. Instale as ferramentas: **[docs/instalacao.md](docs/instalacao.md)**
   (Ubuntu/Debian, Fedora, macOS e Windows com WSL2).
2. Faça o *fork* deste repositório (botão **Fork**, canto superior direito).
3. Clone o **seu** fork e verifique a máquina:

   ```bash
   git clone git@github.com:<seu-usuario>/ufla-devops-shop.git
   cd ufla-devops-shop
   ./scripts/check-ambiente.sh
   ```

4. Siga o enunciado da **Atividade 0**, publicado no campus virtual.

---

## Estrutura

```
ufla-devops-shop/
├── app/                       # a API (FastAPI): uvicorn app:api
├── static/                    # o front estático, servido pela API em /
├── tests/                     # pytest — roda sem banco nem cache
├── requirements.txt           # dependências de execução (vão para a imagem)
├── requirements-dev.txt       # + testes, cobertura e lint (CI e desenvolvimento)
├── pyproject.toml             # configuração de ruff, pytest e cobertura
├── docs/
│   ├── instalacao.md          # roteiro de instalação por sistema operacional
│   └── aplicacao.md           # como a aplicação funciona, rotas e variáveis
├── scripts/
│   ├── check-ambiente.sh      # verificador de ambiente (Atividade 0)
│   └── baixar-dados.sh        # traz o access.log da Atividade 2
├── alunos/                    # apresentação de cada aluno (Atividade 0)
├── analises/                  # análises da Atividade 1 e (em logs/) da Atividade 2
└── .github/
    ├── pull_request_template.md
    └── workflows/             # verificação automática de cada atividade
```

## A aplicação

`ufla-devops-shop` é uma loja mínima: API em Python (FastAPI), PostgreSQL,
Redis e um front estático. Ela é o **fio condutor** das 15 semanas — cada
atividade acrescenta uma camada sobre a mesma base, e nenhuma começa do zero.

```bash
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements-dev.txt
uvicorn app:api --reload        # http://localhost:8000  (API + front + /docs)
pytest --cov=app                # os testes não precisam de banco nem de cache
```

Sem variáveis de ambiente ela sobe **sozinha** (SQLite + cache em memória).
Com `DATABASE_URL` e `REDIS_URL` ela usa PostgreSQL e Redis. Rotas, variáveis,
modos e os tamanhos reais de imagem estão em
**[docs/aplicacao.md](docs/aplicacao.md)** — leia antes da Atividade 4.

### Seu fork está desatualizado?

Quem fez o *fork* antes de a aplicação entrar aqui precisa **trazê-la**:
`git pull` puxa do seu fork, não deste repositório.

```bash
git remote add upstream https://github.com/rdurelli/ufla-devops-shop.git   # uma vez só
git fetch upstream && git switch main && git merge upstream/main && git push origin main
```

Ou **Sync fork → Update branch** na página do seu fork. Faça isso no início de
**toda** atividade.

## `scripts/check-ambiente.sh`

Verifica ferramentas, versões mínimas, o daemon do Docker, a configuração do Git
e os recursos da máquina. Quando algo falta, imprime o comando de instalação
para o sistema detectado.

```bash
./scripts/check-ambiente.sh                      # relatório colorido
./scripts/check-ambiente.sh | tee ambiente.txt   # salva para anexar ao PR
```

Sai com código `0` se o ambiente estiver pronto e `1` se faltar algo
obrigatório. Versões mínimas: `git` 2.30, `docker` 24.0, `kubectl` 1.28,
`kind` 0.20, `helm` 3.12, `terraform` 1.5, Docker Compose v2.

O script também é material didático: na Semana 4 (shell script) abrimos este
arquivo em aula e discutimos as construções usadas nele.

---

## Como entregar

Toda atividade tem **duas etapas obrigatórias**. Faltando qualquer uma delas, a
atividade é considerada não entregue.

1. **Pull Request neste repositório**, com o trabalho feito na *branch*
   `atividade-NN` do seu fork.
2. **Link do Pull Request postado no campus virtual**, na tarefa correspondente.
   O horário desta postagem é o registro oficial da entrega.

O fluxo é sempre o mesmo:

```bash
git switch -c atividade-00                       # branch da atividade
git add alunos/<seu-usuario>.md ambiente.txt     # seus arquivos
git commit -m "feat: apresentacao e ambiente de <seu-usuario>"
git push -u origin atividade-00
```

Depois abra o Pull Request pelo GitHub, com o título no padrão
`Atividade 0 -- Seu Nome`.

### Verificação automática

Assim que o PR é aberto, o GitHub Actions roda sozinho e comenta o resultado em
cerca de um minuto. Se aparecer ✗ vermelho, leia o comentário, corrija, faça um
novo commit e dê `push` na mesma branch — o PR se atualiza sozinho. Você pode
repetir isso quantas vezes quiser antes do prazo.

A verificação automática **não é a nota**. Ela cobre o que a máquina consegue
julgar; a avaliação do professor complementa nos aspectos que ela não julga —
clareza, justificativa das decisões e qualidade da documentação.

### Atrasos e descarte

Atraso: desconto de 1,0 ponto por dia corrido, contado pelo horário da postagem
no campus virtual. Descarte: a menor nota entre as atividades do semestre é
descartada no cálculo final, sem necessidade de justificativa.

---

## Trabalho Prático — Pipeline DevSecOps com GitHub Actions

Este repositório implementa um pipeline de integração contínua com foco em segurança (DevSecOps e *shift-left security*), automatizando a identificação de vulnerabilidades e vazamentos antes da integração no branch principal (`tp-main`).

### 1. Ferramentas Integradas

- **Gitleaks**: Análise do histórico e commits em busca de chaves de API, senhas, tokens ou outros segredos vazados.
- **SonarCloud**: Análise estática de segurança e qualidade de código (SAST) integrada ao Quality Gate voltado a código novo.
- **Trivy**: Varredura de vulnerabilidades tanto nas dependências do projeto (filesystem) quanto na imagem Docker construída.
- **Syft**: Geração automatizada do Software Bill of Materials (SBOM) no formato padrão SPDX JSON, publicado como artefato da execução.
- **Docker Hub**: Publicação da imagem de produção (`djhuan/ufla-devops-shop`), condicionada à aprovação em todos os testes e gates de segurança.

### 2. Regras de Bloqueio do Pipeline

- **Detecção de Segredos**: Qualquer padrão identificado pelo Gitleaks falha a verificação imediatamente.
- **SonarCloud Quality Gate**: Se a cobertura mínima ou limites de confiabilidade/segurança não forem atingidos em código novo, o gate é reprovado.
- **Vulnerabilidades de Dependências e Imagem (Trivy)**:
  - Vulnerabilidades **CRITICAL** e **HIGH** com correção disponível (`ignore-unfixed: true`) bloqueiam o pipeline (exit-code 1).
  - Vulnerabilidades **LOW** e **MEDIUM** são registradas em relatório detalhado de auditoria sem interromper a execução.
- **Proteção e Publicação**: A imagem Docker só é gerada e publicada para a branch principal (`tp-main`) se todos os jobs de segurança e testes forem bem-sucedidos.

---

## Onde pedir ajuda

No **fórum da atividade**, no campus virtual. Pergunta pública gera resposta que
serve para a turma toda — e quase sempre alguém já passou pelo mesmo problema.

Ao relatar um erro, inclua:

1. o que você estava tentando fazer;
2. o comando exato que executou;
3. a mensagem de erro **completa**, copiada como texto (não como foto da tela);
4. o que você já tentou.

Esta é a mesma estrutura de um bom *issue* e de um bom *postmortem*.

---

## Uso de IA generativa

Permitido e incentivado. Em contrapartida, você deve ser capaz de explicar
qualquer linha entregue: a arguição final pergunta o *porquê* das decisões, não
a sintaxe.
