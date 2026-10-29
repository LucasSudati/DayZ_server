# DayZ Server Monitor

Aplicativo desktop para Windows, em planejamento, destinado a acompanhar a população de servidores **oficiais do DayZ** sem interagir com o processo ou a memória do jogo.

> **Estado em 23/10/2026:** especificação inicial. Nenhuma consulta direta a servidores foi validada, e o aplicativo ainda não foi implementado.

## Objetivo

Monitorar simultaneamente servidores oficiais selecionados e alertar sobre alterações de população **somente no servidor marcado como "Estou jogando"**, evitando consultar o launcher durante a partida.

## Decisões de arquitetura

| Área | Definição |
| --- | --- |
| Plataforma | Windows; Python; interface PySide6 |
| Catálogo | Busca de servidores oficiais por nome, número, mapa e região |
| Coleta | Steam A2S por UDP como primeira hipótese; descobrir e validar endereço/porta de consulta |
| Frequência | Meta inicial de 2 segundos por servidor, sujeita a testes e limites de uso |
| Falhas | Após consulta sem resposta, aguardar 5 segundos antes de tentar novamente; jamais interpretar falha como zero jogadores |
| Monitoramento | Vários servidores salvos e consultados em paralelo enquanto o aplicativo estiver aberto |
| Inicialização | Retomar automaticamente consultas dos servidores salvos ao abrir o aplicativo |
| Servidor ativo | Toggle manual "Estou jogando", exclusivo entre servidores |
| Alertas | Som e/ou voz apenas para o servidor ativo **quando o DayZ estiver aberto** |
| Jogo fechado | Continuar coletando e registrando dados, mas silenciar os alertas |
| Áudio | Efeitos personalizados, importação de arquivos e síntese de voz local; fila para anúncios simultâneos |
| Histórico | SQLite local, retenção de 15 dias e limpeza automática |
| Estatísticas | Gráficos por hora/dia/semana, médias, picos e comparação entre servidores |
| Execução | Bandeja do Windows; monitoramento em segundo plano; opção de pausa manual |

## Fluxo dos dados

1. **Descoberta:** localizar servidores oficiais e obter endereço de jogo e de consulta, que podem ser diferentes.
2. **Coleta:** consultar cada servidor salvo de forma assíncrona, sem bloquear a interface.
3. **Validação:** registrar quantidade atual, capacidade, horário da resposta, latência e estado de comunicação.
4. **Comparação:** calcular a **variação líquida** de jogadores entre duas respostas válidas; não é possível identificar pessoas nem detectar entradas e saídas simultâneas que se anulem.
5. **Persistência:** salvar alterações e amostras periódicas (meta: uma por minuto), com retenção de 15 dias.
6. **Notificação:** emitir uma única mensagem por variação detectada, apenas para o servidor ativo e quando o processo do DayZ estiver em execução.
7. **Visualização:** apresentar população, última atualização, estado da coleta, histórico e estatísticas.

## Regras de segurança e confiabilidade

- Não injetar código, ler memória do DayZ, modificar arquivos do jogo ou interagir com o BattlEye.
- Não afirmar que o protocolo A2S ou uma porta específica funciona em todos os servidores antes dos testes.
- Identificar dados desatualizados e falhas de consulta claramente; não confundir servidor vazio com servidor indisponível.
- Não afirmar que o número de jogadores revela localização, identidade ou atividade de raid.
- Não garantir sobreposição visual em tela cheia exclusiva; testar notificações e priorizar áudio quando necessário.
- Não armazenar credenciais de jogadores.

## Etapas planejadas

- [ ] Validar descoberta de servidores oficiais e endereço/porta A2S.
- [ ] Medir latência, frequência segura de consultas e comportamento diante de falhas.
- [ ] Implementar coleta assíncrona e comparação de população.
- [ ] Criar catálogo e interface PySide6.
- [ ] Implementar toggle exclusivo do servidor ativo e detecção do processo DayZ.
- [ ] Adicionar sons personalizados, síntese de voz e notificações.
- [ ] Criar banco SQLite, limpeza automática e gráficos.
- [ ] Testar execução em segundo plano, minimização e tela cheia.

## Organização proposta

```text
DayZ_server/
├── README.md
├── docs/
│   ├── architecture.md
│   └── data-sources.md
├── src/
│   ├── discovery/
│   ├── query/
│   ├── monitoring/
│   ├── notifications/
│   ├── storage/
│   └── ui/
└── tests/
```

Esta árvore é **planejada**; não representa arquivos já implementados.

## Projeto anterior

O projeto antigo **DayZ Admin Control** foi preservado na branch [backup-dayz-admin-control](https://github.com/LucasSudati/DayZ_server/tree/backup-dayz-admin-control). O objetivo da branch `main` passa a ser o novo monitor.

---

### Status e atualização do projeto

- **Situação:** Em desenvolvimento — especificação; implementação não validada.
- **Última revisão desta nota:** 29/10/2026.
- **Última alteração do código:** não verificada nesta revisão; consulte o histórico de commits para a data exata.

> Esta nota registra a revisão da documentação, não certifica testes nem uma versão estável.
