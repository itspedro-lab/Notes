---
title: 'Overengineering em Projetos Pequenos: Lições de Arquitetura com um Arduino'
date: '2026-01-29'
description: Um relato sobre como um projeto pessoal com Arduino e elevador acabou se tornando um exercício prático de arquitetura, usando eventos, filas e máquina de estados.
lang: pt-br
tags:
    - arduino
    - event-driven
    - fsm
---
## Ideia Central

Em projetos pessoais com hardware, é muito comum começar pelo caminho mais direto: abrir a serial, mandar comandos e esperar respostas, no início funciona mas conforme o projeto cresce, começam a aparecer comportamento imprevisíveis e dificuldade de manutenção.

Esse projeto surgiu exatamente nesse contexto, durante o desenvolvimento de um sistema de controle para um elevador usando Arduino. A ideia inicial era simples: enviar comandos a partir de um servidor e executar no hardware porém na prática, isso acabou virando um pequeno exercício de arquitetura.

O objetivo era criar uma "ponte" entre três partes diferentes:

- Servidor, responsável por emitir comandos  
- Aplicação intermediária, responsável por organizar o fluxo  
- Arduino, responsável pela execução física do elevador  

Em vez de tudo se comunicar diretamente, a proposta foi estruturar esse fluxo de forma assíncrona, usando eventos, filas e estados bem definidos, trazendo mais previsibilidade.

## Como Funciona

De forma resumida, o fluxo segue este caminho:

Comando -> Evento -> Fila -> Validação -> Serial -> Execução -> Status

Servidor publica um comando. 
Aplicação recebe esse evento.
Comando entra em uma fila interna.
É validado e formatado.
É enviado pela porta serial.
Arduino executa a ação.
Status retorna para o servidor.

No contexto do elevador, isso significava transformar ações como subir, descer, abrir portas e parar em eventos bem definidos, evitando comandos concorrentes ou fora de ordem.

## Máquina de Estados

Para controlar esse fluxo, implementei uma máquina de estados finitos (FSM). Cada etapa do processo era representada por um estado, como:

Idle | Receiving | Processing | Sending | Waiting | Done | Error

No caso do elevador, a maquina de estados ajudava a evitar que o elevador entrasse em estados incosistentes como tentar subir com a porta aberta, mudar de direção no meio do movimento ou aceitar comandos enquanto ainda estava em execução e acabou sendo uma das partes mais importantes do projeto.

## Lidando com Concorrência e Bloqueios

Um dos principais problemas era a comunicação serial.

Se dois comandos tentassem ser enviados ao mesmo tempo, o sistema quebrava, a solução foi criar uma fila única de processamento com um worker dedicado para a serial, com isso, os comandos passaram a ser executados em ordem, sem conflitos. Em vez de tratar erros como exceções raras, o projeto passou a encará-los como parte natural do fluxo, o que deixou o sistema muito mais previsível.

Sendo honesto: hoje, usar um ESP32 com Wi-Fi e MQTT provavelmente resolveria metade disso em muito menos tempo.

Uma arquitetura baseada em WebSocket ou MQTT eliminaria a necessidade da aplicação intermediária, mas isso também teria eliminado boa parte do aprendizado. Esse projeto foi um laboratório para entender como organizar sistemas que interagem com o mundo físico.

Ele reforçou algo que venho percebendo com o tempo: não é o tamanho do projeto que define o quanto ele ensina, mas o quanto você se preocupa em estruturá-lo bem.

Projetos pessoais, quando levados a sério, acabam sendo excelentes professores.

---

*Se você chegou até aqui, provavelmente também gosta de transformar ideias simples em sistemas desnecessariamente complexos e aprender com isso. 😅 Se tiver sugestões, críticas ou ideias, vou gostar de ouvir.*
