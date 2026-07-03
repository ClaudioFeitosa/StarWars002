# Manual Stepwise Framework

## Visão Geral

O Stepwise é um framework de orquestração de capacidades que gerencia o ciclo de vida completo de desenvolvimento de software, desde a pesquisa inicial até a implementação e entrega. Ele opera como um orchestrador que coordena múltiplas habilidades (skills) em uma sequência lógica e estruturada.

## Finalidade e Propósito

O Stepwise foi projetado para:

- **Orquestrar workflows complexos** de desenvolvimento de software
- **Gerenciar contexto e estado** entre diferentes fases do ciclo de vida
- **Prover rastreabilidade** e auditoria de todas as atividades
- **Facilitar recuperação** de falhas e continuidade de sessões
- **Escalar projetos** de qualquer tamanho mantendo consistência

## Features do Stepwise

### 1. Sistema de Session Management
**Finalidade:** Identificar e rastrear sessões únicas de trabalho
**Momento de uso:** Início de qualquer skill/capacidade

- Gera SESSION_ID únicos no formato: `{TYPE}-{PROJECT_NAME}-{FEATURE/SESSION}-{YYYYMMDD}`
- Reutiliza SESSION_ID em modo REPAIR
- Propaga ID para todos os artefatos gerados

### 2. Progress Tracking com `_progress.json`
**Finalidade:** Monitorar progresso e prevenir interrupções
**Momento de uso:** Primeira e última ação de qualquer sessão

- Primeiro arquivo escrito na sessão
- Atualiza status: RUNNING → COMPLETED/FAILED
- Contém métricas de progresso e artefatos

### 3. Memory Bank System
**Finalidade:** Manter contexto contínuo entre sessões
**Momento de uso:** Durante e após cada sessão

- `active-context.md`: Estado atual da sessão
- `progress.md`: Ledger acumulado de progresso
- Compartilhado entre todas as capacidades

### 4. Modos de Operação
**Finalidade:** Diferenciar entre criação nova e correção
**Momento de uso:** Início da skill based na presença de feedback

- **BUILD MODE:** Criação nova de artefatos
- **REPAIR MODE:** Correção de artefatos existentes
- Automaticamente detectado pela presença de bloco `## ⚠️ Repair feedback`

### 5. Skill Routing System
**Finalidade:** Determinar quais seções do protocolo cada skill deve seguir
**Momento de uso:** Carregamento da skill

- Baseado no tipo de output (section-shape, list-shape, hybrid)
- Rotas específicas para diferentes famílias de skills
- Garante uso consistente do protocolo

## Ciclo de Vida do Stepwise

### Fase 1: Research & Planning
**Quando usar:** Início do projeto ou nova feature

**Skills disponíveis:**
- `researching-prd` - Sintetiza requisitos do projeto
- `researching-adrs` - Gera Architecture Decision Records
- `researching-bounded-contexts` - Descobre domínios e contextos
- `planning-epics` - Transforma PRD em backlog de epics
- `implementing-user-stories` - Decompõe epics em user stories

**Sequência recomendada:**
1. `researching-prd` → define o que vamos construir
2. `researching-bounded-contexts` → define os domínios
3. `researching-adrs` → define decisões técnicas
4. `planning-epics` → planeja entregas em épicos
5. `implementing-user-stories` -> detalha como implementar

### Fase 2: Architecture Foundation
**Quando usar:** Após planning, antes do código

**Skills disponíveis:**
- `establishing-architecture-foundation` - Constrói fundação arquitetural
- `specifying-architecture` - Completa especificação técnica

**Sequência recomendada:**
1. `establishing-architecture-foundation` → cria manifesto arquitetural
2. `specifying-architecture` → define unidades arquiteturais

### Fase 3: Test Strategy
**Quando usar:** Após arquitetura, antes da implementação

**Skills disponíveis:**
- `creating-qe-master-plan` - Plano mestre de teste
- `defining-qe-strategy` - Define estratégia de QE
- `generating-test-cases` - Gera casos de teste funcionais
- `generating-e2e-test-cases` - Gera testes E2E

**Sequência recomendada:**
1. `creating-qe-master-plan` → escopo e estratégia geral
2. `defining-qe-strategy` → abordagem técnica
3. `generating-test-cases` → testes unitários/integração
4. `generating-e2e-test-cases` → testes de ponta a ponta

### Fase 4: Code Implementation
**Quando usar:** Após arquitetura e testes planejados

**Skills disponíveis:**
- `researching-code-design` - Pesquisa design de código
- `planning-code-tasks` - Planeja tarefas de implementação
- `implementing-code` - Implementa o código

**Sequência recomendada:**
1. `researching-code-design` → pesquisa e análise
2. `planning-code-tasks` → planejamento detalhado
3. `implementing-code` → implementação efetiva

### Fase 5: Quality & Review
**Quando usar:** Durante e após implementação

**Skills disponíveis:**
- `reviewing-code` - Revisa código implementado
- `validating-architecture-compliance` - Validam arquitetura
- `human-quality-gate` - Porta de qualidade humana

**Sequência recomendada:**
1. `reviewing-code` → revisão técnica
2. `validating-architecture-compliance` → validação arquitetural
3. `human-quality-gate` → aprovação final

### Fase 6: Automation
**Quando usar:** Após aprovação do código

**Skills disponíveis:**
- `automation-planning` - Planeja automação
- `automation-execution` - Executa automação
- `executing-tests` - Executa testes

**Sequência recomendada:**
1. `automation-planning` → planejamento da automação
2. `automation-execution` → geração de código de automação
3. `executing-tests` → execução dos testes

### Fase 7: Delivery & Export
**Quando usar:** Após validação completa

**Skills disponíveis:**
- `formatting-platform-export` - Exporta para plataformas
- `assembling-final-package` - Monta pacote final
- `delivering-documentation` - Entrega documentação

**Sequência recomendada:**
1. `formatting-platform-export` → formata para destino
2. `assembling-final-package` → empacota entregáveis
3. `delivering-documentation` → entrega documentação

## Sequência Lógica Recomendada

### Para Novo Projeto (Greenfield)
```
1. researching-prd
2. researching-bounded-contexts  
3. researching-adrs
4. planning-epics
5. implementing-user-stories
6. establishing-architecture-foundation
7. specifying-architecture
8. creating-qe-master-plan
9. defining-qe-strategy
10. generating-test-cases
11. generating-e2e-test-cases
12. researching-code-design
13. planning-code-tasks
14. implementing-code
15. reviewing-code
16. validating-architecture-compliance
17. human-quality-gate
18. automation-planning
19. automation-execution
20. executing-tests
21. formatting-platform-export
22. assembling-final-package
23. delivering-documentation
```

### Para Bug Fix Maintenance
```
1. researching-bug-fixing
2. planning-code-tasks (TASK mode)
3. implementing-code
4. reviewing-code
5. executing-tests
6. formatting-platform-export (se necessário)
```

### Para Feature Incremental
```
1. researching-feature-impl
2. planning-code-tasks (TASK mode)
3. generating-test-cases (específico da feature)
4. implementing-code
5. reviewing-code
6. executing-tests
7. formatting-platform-export
```

## Padrões e Convenções

### Nomenclatura de Arquivos
- Use sempre o SESSION_ID: `{TYPE}-{PROJECT}-{FEATURE}-{DATE}.md`
- Nunca omita o SESSION_ID para evitar colisões
- Use nomes descritivos e consistentes

### Gestão de Estado
- Sempre escreva `_progress.json` primeiro
- Atualize Memory Bank após cada fase
- Mantenha rastreabilidade completa

### Recuperação de Erros
- Use REPAIR mode para correções
- Mantenha o mesmo SESSION_ID
- Registe mudanças no AUDIT

## Melhores Práticas

1. **Planejamento**: Não pule fases de research e planning
2. **Validação**: Use quality gates em cada ponto criticogito
3. **Rastreabilidade**: Mantenha connexion entre todos os artefatos
4. **Recuperação**: Use Memory Bank para continuidade
5. **Escalabilidade**: Use o framework completo para projetos grandes

## Conclusão

O Stepwise fornece uma estrutura robusta para gestão de projetos de software complexos. Seguindo a sequência lógica recomendada e usando as features no momento adequado, você garante:

- **Consistência** entre projetos
- **Qualidade** em todas as fases
- **Rastreabilidade** completa
- **Escalabilidade** para qualquer tamanho
- **Recuperação** rápida de falhas

Use este manual como guia para navegar pelo framework e adaptar a sequência às suas necessidades específicas, mantendo sempre os princípios fundamentais do Stepwise.