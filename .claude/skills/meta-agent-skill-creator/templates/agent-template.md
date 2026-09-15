---
name: <nome-curto-em-kebab-case>
description: <O que este agente faz e quando o orquestrador/usuário deve invocá-lo. Seja específico sobre o tipo de tarefa, não genérico.>
tools: <lista explícita de ferramentas permitidas, ex.: Read, Grep, Glob — nunca "*" a menos que o agente realmente precise de acesso total, validado na Fase 2>
model: <inherit | modelo específico, apenas se houver razão explícita para fixar>
---

# <Nome legível do agente>

## Papel

<Uma ou duas frases: o que este agente é responsável por fazer e o que
está fora do seu escopo.>

## Contexto que ele recebe

<O agente roda "a frio": liste aqui o que precisa estar no prompt de
invocação para ele funcionar sem depender de memória da conversa anterior
— caminhos de arquivo, formato de saída esperado, restrições.>

## Ferramentas e por que cada uma é necessária

- `<ferramenta>` — <motivo, validado na Fase 2>

## Formato de saída esperado

<Descreva o formato que o agente deve devolver ao chamador (texto corrido,
JSON, lista de findings, etc.) para que o resultado seja consumível sem
reformatação manual.>

## Falhas conhecidas e como evitá-las

<Copiado das correções feitas durante a Fase 2 de validação manual deste
agente. Exemplo: "se a API key X não estiver no ambiente, não tente
adivinhar — reporte a ausência e pare".>
