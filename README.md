# DayZ Server — DayZ Admin Control

Sistema de administração para servidores **DayZ**, desenvolvido em dois componentes:

- **DayZ Admin Control In-Game** — mod para acessar uma interface administrativa dentro do jogo.
- **DayZ Admin Control Desktop** — aplicativo de administração externo, desenvolvido separadamente.

> **Estado do projeto (08/10/2026):** **v0.2.8 em testes**. O F7, a consulta RPC de permissões e a renderização de textos foram validados. Nas v0.2.6 e v0.2.7.1, o `ImageWidget` exibiu imagens, porém com formatos ovais devido ao sprite circular esticado. A v0.2.8 introduz um imageset e uma textura retangular próprios; **ainda não há confirmação de funcionamento no DayZ**. Comandos administrativos seguem desativados.

## Objetivo

Construir uma interface administrativa integrada ao DayZ, com autenticação validada no servidor, separação de níveis de acesso e, futuramente, ferramentas de gerenciamento de jogadores, eventos e operações do servidor.

O desenvolvimento atual prioriza:

1. Carregamento estável de scripts Enforce Script.
2. Comunicação cliente–servidor para consultar permissões.
3. Abertura e fechamento do menu pelo teclado.
4. Renderização e interação da interface gráfica.
5. Implementação gradual das ações administrativas, com validação **no lado do servidor**.

## Funcionalidades e estado

| Recurso | Estado |
| --- | --- |
| Mod reconhecido e carregado pelo DayZ | ✅ Validado |
| Comunicação RPC para consultar permissão | ✅ Validado |
| Nível administrativo recebido no cliente | ✅ Validado |
| Registro e detecção do atalho **F7** | ✅ Validado |
| Abertura e fechamento da rotina do menu | ✅ Validado |
| Exibição correta do painel gráfico | 🧪 Imagens renderizadas nas v0.2.6 e v0.2.7.1; retângulos em teste na v0.2.8 |
| Cursor ao abrir o menu | ✅ Observado |
| Navegação por seções e botões | 🧪 Pendente de validação |
| Comandos administrativos efetivos | ⏳ Não implementados/ativados nesta etapa |

Os resultados acima descrevem os testes do ambiente de desenvolvimento; não são uma garantia de compatibilidade com todas as instalações.

## Estrutura do mod In-Game

Estrutura de referência utilizada no desenvolvimento:

```text
DayZAdminControl/
├── config.cpp
├── inputs.xml
├── gui/
│   └── imagesets/
│       ├── dac_solid.imageset
│       └── dac_solid.edds
└── scripts/
    ├── 3_Game/
    │   └── DAC_Constants.c
    ├── 4_World/
    │   ├── DAC_Permissions.c
    │   └── DAC_PlayerRPC.c
    └── 5_Mission/
        ├── DAC_Menu.c
        ├── DAC_ClientMission.c
        └── DAC_ServerMission.c
```

A lista representa os principais arquivos conhecidos e pode evoluir ao longo das versões. O menu e seus componentes podem envolver scripts adicionais.

### Componentes

- **`config.cpp`**: registra o mod, suas dependências e diretórios de scripts.
- **`inputs.xml`**: registra a ação de teclado usada para abrir o painel.
- **`gui/imagesets/dac_solid.imageset`** e **`dac_solid.edds`**: recursos visuais introduzidos na v0.2.8 para tentar renderizar retângulos; carregamento ainda em validação.
- **`scripts/5_Mission/DAC_Menu.c`**: criação dos widgets e controle do menu no cliente.
- **`3_Game`**: constantes compartilhadas, incluindo identificadores das mensagens RPC.
- **`4_World/DAC_Permissions.c`**: consulta do nível de acesso configurado no servidor.
- **`4_World/DAC_PlayerRPC.c`**: mensagens RPC entre cliente e servidor para autenticação do menu.
- **`5_Mission`**: integração do mod às missões cliente e servidor, incluindo inicialização, entrada e ciclo de vida do menu.

## Arquitetura de permissões

O cliente solicita a permissão ao servidor por RPC. O servidor consulta sua configuração administrativa e responde com o nível autorizado. Em testes, um usuário de desenvolvimento recebeu o nível **3 — Superadministrador**.

```text
Cliente DayZ               Servidor DayZ
     |                          |
     | --- Solicita nível ----> |
     |                          | Consulta permissões
     | <--- Responde nível ---- |
     |                          |
     | F7 -> tenta abrir menu   |
```

**Segurança:** a exibição do menu no cliente **não** deve conceder autoridade para executar comandos. Qualquer ação administrativa futura precisa validar novamente, **no servidor**, a identidade do solicitante, o nível exigido e os parâmetros do comando. Não confie em níveis ou dados enviados pelo cliente.

A configuração inicial de permissões é feita em `scripts/4_World/DAC_Permissions.c`. Evite publicar SteamIDs administrativos, senhas, tokens ou outras credenciais nos arquivos do repositório.

## Preparação e compilação do PBO

### Requisitos

- **DayZ** e, para desenvolvimento/testes locais, **DayZ Server**.
- **DayZ Tools**, especialmente **Addon Builder**.
- **BankRev** (opcional, para inspeção do conteúdo do PBO).
- Código-fonte completo do mod.

### Procedimento

1. Mantenha `config.cpp` na raiz da pasta-fonte **`DayZAdminControl`**.
2. Mantenha `inputs.xml` na **raiz do mod**, conforme o caminho configurado em `config.cpp`.
3. Abra o **DayZ Tools → Addon Builder**.
4. Selecione a pasta-fonte do mod.
5. Selecione como destino o **diretório `Addons`**, não um caminho que termine em `.pbo`.
6. Em **Options → List of files to copy directly**, mantenha os padrões **`*.xml`**, **`*.imageset`** e **`*.edds`** na v0.2.8. O XML registra o atalho e os dois últimos são recursos visuais; mantenha `*.layout` apenas quando estiver compilando versões antigas que o utilizem.
7. Gere o PBO.
8. Caso necessário, abra o PBO no **BankRev** e confirme a presença de `inputs.xml`, `gui/imagesets/dac_solid.imageset`, `gui/imagesets/dac_solid.edds` e dos scripts esperados.

Padrão de distribuição:

```text
@DayZAdminControl/
└── Addons/
    └── DayZAdminControl.pbo
```

O prefixo e os caminhos internos do PBO devem ser compatíveis com as referências do `config.cpp` e dos scripts.

## Instalação para testes

### Servidor

Copie a pasta `@DayZAdminControl` para o diretório do servidor e acrescente o mod ao parâmetro de inicialização:

```bat
-mod=@DayZAdminControl
```

Se houver outros mods, preserve-os e separe os nomes com ponto e vírgula, conforme a configuração de inicialização utilizada.

### Cliente

Carregue **o mesmo mod e a versão correspondente** no cliente, usando o DayZ Launcher (ou outro método de inicialização configurado). Reinicie o cliente e o servidor depois de substituir o PBO.

> O mod In-Game exige carregamento adequado no cliente e no servidor. O aplicativo Desktop é outro componente e não substitui o PBO.

## Uso atual

1. Inicie o servidor com o mod.
2. Entre no jogo com um usuário cadastrado nas permissões administrativas.
3. Aguarde a resposta de permissão do servidor.
4. Pressione **F7** para abrir ou fechar o menu.
5. Se o cursor surgir mas o painel não aparecer, consulte o diagnóstico abaixo.

O acesso pela tecla F7 e a resposta de nível 3 foram registrados com sucesso em teste. A renderização dos textos e de imagens foi confirmada nas v0.2.6 e v0.2.7.1. A forma retangular definitiva está em teste na **v0.2.8**.

## Logs e diagnóstico

No Windows, os logs de script do **cliente** ficam normalmente em:

```text
%LOCALAPPDATA%\DayZ
```

Localize o arquivo `script_*.log` mais recente após reproduzir o problema. Os logs do servidor dependem do parâmetro de perfil configurado, como o diretório `config` usado no ambiente de desenvolvimento.

Mensagens úteis já observadas:

```text
[DAC] Solicitacao de permissao enviada
[DAC] UADACOpenMenu registrada; pressione F7
[DAC] Nivel administrativo recebido: 3
[DAC] F7 detectado; verificando menu e permissao
[DAC] Menu criado com sucesso
[DAC] Painel fechado
```

### F7 abre o cursor, mas a janela não aparece

Esse comportamento foi observado durante o teste da **v0.2.3**. O log registrou criação do menu e fechamento normal, mas o painel gráfico não era visível. Isso indica que uma mensagem de sucesso da rotina, isoladamente, **não comprova a renderização** dos widgets.

Pontos de verificação:

- Confirmar `gui/layouts/dac_menu.layout` **dentro do PBO**.
- Confirmar os caminhos de carregamento do layout.
- Revisar a criação dos widgets e seus retornos.
- Verificar dimensões, posições, visibilidade, opacidade e hierarquia dos componentes.
- Revisar os logs após pressionar **F7**.

Nas v0.2.6 e v0.2.7.1, o uso de `ImageWidget` com sprite nativo circular permitiu renderizar os fundos, porém deformados. Na v0.2.7, o uso do tipo `PanelWidget` em Enforce Script causou erro de compilação `Bad type 'PanelWidget'`. A v0.2.8 testa textura retangular própria e ainda aguarda validação. **A extensão `.edds` não garante, por si só, que os bytes DDS sejam aceitos pelo DayZ; poderá ser necessária conversão pela ferramenta oficial de texturas.**

### Atalho não funciona

Verifique se `inputs.xml` foi incluído no PBO e se **`*.xml`** está na lista de cópia direta do Addon Builder. Durante os testes, a inclusão desse padrão resolveu o problema do carregamento dos inputs.

### Erros de script

Confira os arquivos `script_*.log` do cliente e do servidor. Caso haja erros de compilação Enforce Script, corrija-os antes de investigar a interface.

## Histórico de versões — In-Game

| Versão | Marco |
| --- | --- |
| v0.1.0–v0.1.1 | Primeiras integrações e correções de compilação. |
| v0.1.2 | RPC de consulta de permissão confirmado em teste. |
| v0.2.0–v0.2.1 | Introdução do menu e depuração do atalho F7. |
| v0.2.2 | Carregamento do `inputs.xml` corrigido; F7 abre a interface inicial. |
| v0.2.3 | Novo layout visual; cursor abre, mas o painel não foi renderizado no teste. |
| v0.2.4 | Ajustes de dimensões; interface ainda não apareceu no teste. |
| v0.2.5 | Interface construída por script; textos renderizados, mas fundos ausentes. |
| v0.2.6 | `ImageWidget` renderizou fundos com sprite circular esticado; teste visual bem-sucedido, geometria inadequada. |
| v0.2.7 | Tentativa com `PanelWidget`; **falha de compilação** (`Bad type 'PanelWidget'`). |
| v0.2.7.1 | Recuperação baseada em `ImageWidget`; imagem novamente visível, ainda oval. |
| **v0.2.8** | Imageset e textura retangular próprios; **em testes, sem confirmação no jogo**. |

## Próximas etapas

- Validar o carregamento da textura retangular da v0.2.8 e corrigir eventual conversão para formato suportado.
- Garantir foco, cursor, fechamento e navegação sem prejudicar o jogo.
- Implementar ações administrativas com autorização verificada no servidor.
- Adicionar registros de auditoria de ações administrativas.
- Ampliar ferramentas de gerenciamento de jogadores e operações do servidor.
- Documentar comandos, permissões e instalação conforme forem implementados.

## Observações

Este é um **projeto em desenvolvimento**, não um pacote de administração pronto para produção. Algumas telas e seções são protótipos visuais e **não executam ações administrativas reais**.

Os códigos-fonte, os binários PBO e eventuais instaladores podem ser adicionados ao repositório em etapas futuras. Este README documenta o estado do trabalho até a v0.2.8 (ainda em testes).

---

Desenvolvido por [LucasSudati](https://github.com/LucasSudati).
