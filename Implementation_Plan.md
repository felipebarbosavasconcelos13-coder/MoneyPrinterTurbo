# Plano de Implementacao - Atualizacao para o Upstream Mais Recente

Este documento detalha o planejamento para atualizar o fork do **MoneyPrinterTurbo** para o commit mais recente do repositório original upstream, preservando todas as customizacoes e a infraestrutura de deploy no Coolify.

---

## 1. Contexto e Diagnostico da Auditoria

### 1.1 Repositorios e Commits
- **Nosso Repositorio**: `https://github.com/felipebarbosavasconcelos13-coder/MoneyPrinterTurbo`
  - **Branch**: `main`
  - **SHA Atual**: `f47bd3b3fc624f81da00702c481ba21066d54f5c` (*feat: add mandatory CTA to generated scripts*)
- **Repositorio Original (Upstream)**: `https://github.com/harry0703/MoneyPrinterTurbo`
  - **Branch**: `main`
  - **SHA Alvo do Upstream**: `f420c092ee56e92035db0115d900f8a9d4c1abfe` (*feat(video): report clip processing and final render progress* - 30/09/2026)
- **Ponto de Divergencia (`git merge-base`)**:
  - `058f18a450781e7ced72df23b4ea210138d71fea` (*fix subtitle background text clipping*)
- **Estatisticas de Divergencia**:
  - Nosso fork: **13 commits a frente** do ponto de divergencia.
  - Upstream: **429 commits a frente** (tags de `v1.1.0` a `v1.3.7`).

### 1.2 Customizacoes Exclusivas do Nosso Fork (Total: 4 arquivos alterados)
1. **Permitir ate 25 Paragrafos**:
   - `app/models/schema.py`: `paragraph_number` com `ge=1, le=25` em `VideoParams` e `VideoScriptParams`.
   - `app/services/llm.py`: `MAX_SCRIPT_PARAGRAPH_NUMBER = 25`.
   - `webui/Main.py`: O slider ja le dinamicamente `llm.MAX_SCRIPT_PARAGRAPH_NUMBER`.
2. **CTA Obrigatorio nos Roteiros**:
   - `app/services/llm.py`: Constante `MANDATORY_SCRIPT_CTA_PROMPT` e anexo automatico em `build_script_prompt(...)`.
3. **Deploy e Roteamento no Coolify / Traefik**:
   - `docker-compose.yml`: Servicos `webui` (8501) e `api` (8080), rede externa `coolify` (`traefik.docker.network=coolify`), labels explicitos de roteamento HTTPS para os dominios configurados.
4. **Testes Unitarios**:
   - `test/services/test_llm.py`: Validacao de 25 paragrafos e presenca do CTA em prompt padrao e customizado.

### 1.3 Auditoria da Producao no Coolify
- **Aplicacao**: `money-printer-turbo:completo` (UUID: `pn25p4iuncprsq6g5bnj3m84`)
- **Projeto**: `My first project` (UUID: `bs6ayu7xv18oatjgtc92sv70`)
- **Ambiente**: `production` (UUID: `lga254b7nnun7d8exbylv6vm`)
- **Build Pack**: `dockercompose`
- **Dominios Ativos**:
  - WebUI: [https://moneyprinterturbo.genialsolucoesdigitais.com.br/](https://moneyprinterturbo.genialsolucoesdigitais.com.br/) (HTTP 200)
  - API: [https://api-moneyprinterturbo.genialsolucoesdigitais.com.br/docs](https://api-moneyprinterturbo.genialsolucoesdigitais.com.br/docs) (HTTP 200)
- **Persistencia**:
  - Storage montado em `/MoneyPrinterTurbo` (UUID: `ypo7k7jzl3ndm4o0s2oqca9y`).

---

## 2. Estrategia de Execucao e Status Atual

1. **Backup Remoto**:
   - [x] Criada e enviada branch `backup-before-upstream-update` a partir do commit `f47bd3b3fc624f81da00702c481ba21066d54f5c`.
2. **Branch de Integracao**:
   - [x] Criada branch `upgrade-moneyprinterturbo` baseada no commit `f420c092ee56e92035db0115d900f8a9d4c1abfe` (upstream/main).
3. **Reaplicacao das Customizacoes**:
   - [x] Reaplicado o limite de 25 paragrafos em `app/models/schema.py` e `app/services/llm.py` (commit `a27badc`).
   - [x] Reaplicado `MANDATORY_SCRIPT_CTA_PROMPT` em `app/services/llm.py` (commit `2683e0f`).
   - [x] Configurado `docker-compose.yml` para integracao com Traefik e rede `coolify` (commit `1752bab`).
   - [x] Ajustado `test/services/test_video.py` para compatibilidade com consoles Windows CP1252 (commit `1135bb0`).
   - [x] Adicionada documentacao e diretrizes (commit `55ee8bc`).
4. **Validacao Local**:
   - [x] Executados testes automatizados: `test_llm` (115 OK), `test_controller_llm` (4 OK), `test_video` (72 OK).
5. **Pull Request e Integracao no Main**:
   - [x] Criado PR #1 no GitHub e executado merge com integracao no `main` (commit `210607b`).
   - [x] Push realizado com sucesso para `origin/main`.
6. **Deploy no Coolify**:
   - [x] Enfileirado deploy da versao atualizada via MCP Coolify (Deployment inicial `iosv0zcuen5dgfqp7nsfxsbe` -> correcao de imagem base Debian Bookworm em commit `d779fe1` -> deploy final `qhubcekipanedmvc3xnon74f` concluido com status `finished`).
   - [x] Validacao dos smoke checks pos-deploy (WebUI e API retornando HTTP 200 OK, validacao visual no browser confirmada).
7. **Finalizacao e Documentacao**:
   - [x] `LOG_DESENVOLVIMENTO.md` atualizado e sincronizado.
   - [x] `Implementation_Plan.md` atualizado e sincronizado.
   - [x] `DOCUMENTACAO.md` atualizado e sincronizado.

