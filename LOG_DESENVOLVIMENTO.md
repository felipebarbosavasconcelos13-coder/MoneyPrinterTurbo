# Log de Desenvolvimento

Este arquivo registra alteracoes relevantes feitas no projeto **MoneyPrinterTurbo**, com foco em manutencao, deploy e decisoes operacionais.

## [30/09/2026] - Auditoria Geral e Plano de Atualizacao para o Upstream Mais Recente

### Auditoria do GitHub
- Identificado o commit atual do `origin/main`: `f47bd3b3fc624f81da00702c481ba21066d54f5c`.
- Ponto de divergencia (`git merge-base`): `058f18a450781e7ced72df23b4ea210138d71fea`.
- Commit alvo do upstream (`upstream/main`): `f420c092ee56e92035db0115d900f8a9d4c1abfe` (30/09/2026).
- Nosso fork esta 13 commits a frente do ponto de divergencia, e o upstream avancou 429 commits.
- Mapeadas todas as customizacoes do nosso fork (isoladas em apenas 4 arquivos: `app/models/schema.py`, `app/services/llm.py`, `docker-compose.yml`, `test/services/test_llm.py`).

### Auditoria do Coolify
- Aplicacao localizada no Coolify: `money-printer-turbo:completo` (UUID: `pn25p4iuncprsq6g5bnj3m84`).
- Projeto: `My first project`, Servidor: `localhost`.
- Confirmado que os servicos em producao estao ativos e saudaveis (WebUI e API retornando HTTP 200).
- File storage persistente identificado em `/MoneyPrinterTurbo` (UUID: `ypo7k7jzl3ndm4o0s2oqca9y`).
- Elaborado plano detalhado de execucao, testes e rollback em `Implementation_Plan.md` e no artefato `upgrade_moneyprinterturbo_plan.md`.

## [29/06/2026] - CTA obrigatorio nos roteiros e deploy Coolify

### Geracao de Roteiros

- Adicionado `MANDATORY_SCRIPT_CTA_PROMPT` em `app/services/llm.py`.
- A regra fixa determina que todo roteiro gerado pelo LLM deve incluir naturalmente uma chamada para a pessoa se inscrever no canal, deixar like, curtir o video, compartilhar com outra pessoa e seguir o perfil/canal quando fizer sentido para a plataforma.
- A orientacao e anexada em `build_script_prompt()` depois da selecao entre `custom_system_prompt` e `DEFAULT_SCRIPT_SYSTEM_PROMPT`, garantindo que a regra tambem se aplica quando o usuario usa prompt customizado.

### Testes

- Atualizado `test/services/test_llm.py` para validar que o CTA obrigatorio aparece no prompt padrao.
- Atualizado `test/services/test_llm.py` para validar que o CTA obrigatorio tambem aparece com `custom_system_prompt`.
- Validacao executada:

```shell
uv run python -m unittest test.services.test_llm
```

- Resultado: `OK`, 50 testes executados e 1 skipped.
- Durante a execucao no Windows, houve mensagens conhecidas de logging Unicode no console `cp1252`, mas os testes passaram.

### Git

- Commit criado e enviado para `origin/main`:
  - `f47bd3b feat: add mandatory CTA to generated scripts`
- Arquivos incluidos no commit:
  - `app/services/llm.py`
  - `test/services/test_llm.py`

### Coolify CLI e Deploy

- Instalada temporariamente a Coolify CLI `v1.6.2` em:
  - `C:\Users\felip\AppData\Local\Temp\opencode\coolify-cli\coolify.exe`
- Contexto CLI configurado e validado:
  - Nome: `prod`
  - Host: `https://coolify.genialsolucoesdigitais.com.br`
  - Versao Coolify: `4.1.2`
- Aplicacao localizada:
  - Nome: `money-printer-turbo:completo`
  - UUID: `pn25p4iuncprsq6g5bnj3m84`
  - Repositorio: `felipebarbosavasconcelos13-coder/MoneyPrinterTurbo`
  - Branch: `main`
  - Build pack: `dockercompose`
- Deploy concluido com sucesso:
  - Deployment UUID: `rhzn16dfuckirti2ux264byv`
  - Commit: `f47bd3b3fc624f81da00702c481ba21066d54f5c`
  - Status final: `finished`
- Um deployment anterior do mesmo commit ficou com status `failed`, mas o deployment posterior `rhzn16dfuckirti2ux264byv` concluiu e publicou a aplicacao.

### Smoke Checks Pos-Deploy

- WebUI:
  - `https://moneyprinterturbo.genialsolucoesdigitais.com.br/`
  - Resultado: `HTTP 200`
- API docs:
  - `https://api-moneyprinterturbo.genialsolucoesdigitais.com.br/docs`
  - Resultado: `HTTP 200`
- OpenAPI:
  - `https://api-moneyprinterturbo.genialsolucoesdigitais.com.br/openapi.json`
  - Resultado: `HTTP 200`
- Ping:
  - `https://api-moneyprinterturbo.genialsolucoesdigitais.com.br/ping`
  - Resultado: `HTTP 404`, esperado porque a rota existe no codigo, mas nao esta registrada em `app/router.py`.

### Observacao Operacional

- Logs recentes da aplicacao indicaram falha de geracao de roteiro quando o provider esta configurado como `openai` sem `openai_api_key` no `config.toml` do container.
- Isso e uma pendencia de configuracao de runtime/secrets, nao um erro do deploy nem da alteracao do CTA.

## [16/06/2026] - Configuracao do MCP do Coolify

### Configuracoes de Sistema/IDE

- Ajustado `mcp_config.json` para configurar o servidor MCP remoto do Coolify.
- Endpoint: `https://coolify.genialsolucoesdigitais.com.br/mcp`
- Cabecalho de autenticacao parametrizado e validado com o token inserido pelo usuario.
- Executado teste do protocolo JSON-RPC via cliente StdIO conectado ao proxy do `mcp-remote`.
- O servidor remoto autenticou o token e retornou a lista de ferramentas expostas com sucesso, incluindo `get_infrastructure_overview` e `list_projects`.

## [16/06/2026] - Configuracao Inicial e Correcao de i18n/Testes

### Ambiente e Dependencias

- Instalado o gerenciador de pacotes `uv` no interpretador Python local.
- Executada sincronizacao do ambiente virtual local com `uv sync --frozen`, criando `.venv` e instalando as dependencias oficiais do projeto.

### Internacionalizacao (i18n)

- Adicionadas chaves ausentes no arquivo `webui/i18n/ru.json`:
  - `Match Materials to Script Order`
  - `Match Materials to Script Order Help`
- A ausencia dessas chaves causava falhas nos testes unitarios de traducao.

### Correcoes de Testes

- Removidos comandos `print` de depuracao em `test/services/test_video.py` que imprimiam texto chines no console Windows.
- Esses prints podiam causar `UnicodeEncodeError` em ambiente Windows com encoding `cp1252`.

### Documentacao

- Criado `Implementation_Plan.md` com planejamento inicial.
- Criado `DOCUMENTACAO.md` com explicacao da arquitetura e fluxo do projeto.
