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
## A Ideia Central

Em projetos pessoais com hardware, é muito comum começar pelo caminho mais direto: abrir a serial, mandar comandos e esperar respostas. No início funciona. Mas, conforme o projeto cresce, começam a aparecer bloqueios, perda de mensagens, comportamento imprevisível e dificuldade de manutenção.

Esse projeto surgiu exatamente nesse contexto, durante o desenvolvimento de um sistema de controle para um elevador usando Arduino. A ideia inicial era simples: enviar comandos a partir de um servidor e executar no hardware. Na prática, isso acabou virando um pequeno exercício de arquitetura.

O objetivo era criar uma “ponte” entre três partes diferentes:

- Um servidor, responsável por emitir comandos  
- Uma aplicação intermediária, responsável por organizar o fluxo  
- Um Arduino, responsável pela execução física do elevador  

Em vez de tudo se comunicar diretamente, a proposta foi estruturar esse fluxo de forma assíncrona, usando eventos, filas e estados bem definidos, trazendo mais previsibilidade e reduzindo problemas comuns em projetos desse tipo.

## Como o Sistema Funciona

De forma resumida, o fluxo segue este caminho:

O servidor publica um comando.  
A aplicação recebe esse evento.  
O comando entra em uma fila interna.  
É validado e formatado.  
É enviado pela porta serial.  
O Arduino executa a ação.  
O status retorna para o servidor.

No contexto do elevador, isso significava transformar ações como subir, descer, abrir portas e parar em eventos bem definidos, evitando comandos concorrentes ou fora de ordem.

Esse desacoplamento foi essencial para manter o sistema estável. Cada parte tinha uma responsabilidade clara, e o uso de eventos permitia que elas operassem de forma independente.

## A Importância da Máquina de Estados

Para controlar esse fluxo, implementei uma máquina de estados finitos (FSM).

Cada etapa do processo era representada por um estado, como:

- Ocioso  
- Recebendo comando  
- Processando  
- Enviando  
- Aguardando resposta  
- Finalizado  
- Erro  

No caso do elevador, a FSM ajudava a evitar situações como tentar subir com a porta aberta, mudar de direção no meio do movimento ou aceitar comandos enquanto ainda estava em execução.

Isso impedia que o sistema entrasse em estados inconsistentes. A FSM acabou sendo uma das partes mais importantes do projeto.

## Lidando com Concorrência e Bloqueios

Um dos principais problemas era a comunicação serial.

Se dois comandos tentassem ser enviados ao mesmo tempo, o sistema quebrava. No elevador, isso podia significar movimentos inesperados ou perda de controle. A solução foi criar uma fila única de processamento com um worker dedicado para a serial, com isso, os comandos passaram a ser executados em ordem, sem conflitos. Em vez de tratar erros como exceções raras, o projeto passou a encará-los como parte natural do fluxo, o que deixou o sistema muito mais previsível.

O mais interessante desse projeto não foi o resultado final, mas o processo. Ele mostrou, na prática, que:

- Projetos pequenos também precisam de arquitetura  
- Eventos reduzem acoplamento  
- Filas organizam concorrência  
- Estados evitam comportamentos inesperados  
- Erros precisam ser modelados  

Conceitos que aparecem em sistemas maiores começaram a surgir naturalmente, mesmo em um projeto pessoal.

## Limitações e Caminhos Mais Simples

Sendo honesto: hoje, usar um ESP32 com Wi-Fi e MQTT provavelmente resolveria metade disso em muito menos tempo.

Uma arquitetura baseada em WebSocket ou MQTT eliminaria a necessidade da aplicação intermediária, mas isso também teria eliminado boa parte do aprendizado. Esse projeto foi, acima de tudo, um laboratório para entender como organizar sistemas que interagem com o mundo físico.

Mais do que controlar um elevador com Arduino, esse projeto serviu para experimentar, errar, ajustar e entender melhor como sistemas orientados a eventos funcionam na prática.

Ele reforçou algo que venho percebendo com o tempo: não é o tamanho do projeto que define o quanto ele ensina, mas o quanto você se preocupa em estruturá-lo bem.

Projetos pessoais, quando levados a sério, acabam sendo excelentes professores.

---

*Se você chegou até aqui, provavelmente também gosta de transformar ideias simples em sistemas desnecessariamente complexos e aprender com isso. 😅*
