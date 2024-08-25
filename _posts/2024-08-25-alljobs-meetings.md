---
layout: post
title: "A beleza do erro: primeiros passos em um app Spring Boot"
description: "Aprendizados projetando e implementando um sistema de reuniões para um balcão de empregos"
tags: programacao aprendizado java
categories: misc
youtubeId:
image: https://live.staticflickr.com/1460/24941500440_abc92ac95c_b.jpg
---

Nas últimas semanas venho me dedicando a um projeto de estudo em Spring Boot, um app de gestão de reuniões online para um sistema de balcão de empregos que vem sendo desenvolvido nos últimos 3 anos, o [Alljobs](https://github.com/0jonjo/alljobs).

A ideia é poder marcar reuniões entre recrutador e candidato a partir de uma chamada de API. Para isso, o [Alljobs Meetings](https://github.com/0jonjo/alljobs-meetings) deve realizar uma série de operações:

1. Verificar se o usuário (user) e recrutador (headhunter) existem no banco de dados;
2. Verificar a disponibilidade de horários registrada pelo recrutador e candidato;
3. Encontrar o primeiro dia e horário disponível para a reunião na agenda de ambos;
4. Criar um link para o encontro online.
5. Enviar um e-mail de confirmação para o recrutador e candidato.
6. Salvar a reunião no banco de dados.

Essas ações expressam regras de negócio que complementam o sistema original da Alljobs. Ele tem foco no cadastro, currículo e interações entre recrutador e candidato a partir de aplicações em vagas. Já o Meetings é para o momento em que o recrutador já triou candidatos e deseja falar com algum deles numa chamada. As seis ações do app visam que a reunião seja marcada o mais breve possível, dentro das preferências de cada usuário, sem conflitos de horários com outras reuniões e que eles sejam notificados antecipadamente delas.

Para dar conta desses processos, foram rascunhados diversas formas de organização das tabelas no bancos de dados. Depois de algumas conversas com colegas e pesquisas - viva as comunidades! - foram definidas duas tabelas, uma para as próprias reuniões (meetings) e outra para preferências de horários dos usuários (available_times). A tabela de reuniões armazena os dados da reunião, como data, horário, link da chamada e status. A tabela de preferências armazena os horários disponíveis de cada usuário, com o dia da semana, início e fim de uma janela. Desse modo, um mesmo usuário pode ter várias janelas de horários disponíveis ao longo da semana, e um horário pode ser ou não disponível para dois usuários estarem em uma reunião.

Antes da etapa 1, foi necessário revisar o básico de Java e aprender como criar um projeto em Spring Boot. No primeiro passo utilizei o curso [Java Essentials](https://www.linuxtips.io/course/java-essentials) da Linuxtips ministrado pela instrutora [Kamila Santos](https://www.linkedin.com/in/kamila-santos-oliveira/). A partir daí, comecei a desenvolver o projeto, seguindo os passos do ebook e curso base de [Spring Boot da Michelli Brito](https://www.youtube.com/watch?v=wlYvA2b1BWI).

O objetivo não foi seguir um livro de receitas, mas sim começar a pensar e escrever com a cabeça de um desenvolvedor dessas tecnologias. Entender como são pensados os objetos, as classes e interfaces, e como eles se comunicam entre si. No processo, tive que me questionar diversas vezes: - Como as pessoas fazem isso no Spring Boot? Onde colocam essa checagem? Qual a melhor forma de persistir esses dados no banco? Algo que seria trivial em Ruby ou Python se torna um pequeno desafio na nova linguagem e framework. Ai está a beleza da coisa: a repetição de erros que leva a aprender.

Entre a etapa 1 e 2, chega o momento dos jardins que se bifurcam - como bem previu [Borges](https://brasil.elpais.com/brasil/2018/09/11/cultura/1536655170_142491.html). Não existe um único modo de implementar checagens de existência de usuários no banco de dados, e não há um modo estritamente melhor ou pior, mas sim o que é mais adequado para o contexto do projeto. É uma oportunidade para ["gastar tempo" estudando](../aprendiz/) e implementando diferentes abordagens Clicar para o projeto realizar o build e ver ele quebrar de novo e de novo, cada vez com um erro diferente. Ou então funcionar e nos testes manuais descobrir que o resultado não é o esperado. Foi o momento de explorar as possibilidades do Spring Boot, as anotações, os métodos, os testes, etc.

Para salvar as preferências de horários dos usuários foi necessário de um lado estudar os modos pelos quais é possível gerir datas e horários nos bancos de dados  - utilizado no ecossistema Alljobs - e de outro, como é possível manipular esses dados no Spring Boot.

A primeira descoberta interessante foi que o [Postgres](https://www.postgresql.org/) - utilizado nos demais projetos Alljobs - tem tipos de colunas específicos para salvar intervalos de datas, horários e números, o que permite trabalhar mais facilmente com eles. Utilizar esses "ranges", nativos do banco de dados a partir da versão 12, foi uma indicação de colega quando expliquei sobre como pretendia salvar os horários disponibilizados pelos usuários. Mais uma vez, viva as comunidades e seu compartilhamento de conhecimento!

Depois disso, foi necessário destrinchar como o Java e o Spring Boot trabalham com datas e horários. Sem surpresa vi que os modos de fazer isso foram evoluindo ao longo do tempo. Aprendi que o Java 8 introduziu uma nova API para manipulação de datas e horários, a [java.time](https://www.baeldung.com/java-8-date-time-intro), que é mais robusta e segura que a antiga [java.util.Date](https://www.baeldung.com/java-date-time). No Spring Boot, a manipulação de datas e horários é feita através das anotações [@LocalDate](https://www.baeldung.com/spring-boot-date-fields) e [@LocalTime](https://www.baeldung.com/spring-boot-date-fields) que facilitam a conversão de dados entre o banco de dados e a aplicação.

No fim, não consegui ainda conciliar as ranges do Postgres com os horários do Spring e Java no momento de salvar as preferências dos usuários. Tive que dividir inicialmente as colunas de início e fim de uma janela de horários disponíveis em duas colunas timestamp (data e hora). Dias de estudo que resultaram em fracasso? Só se eu não soubesse que a beleza do erro é a felicidade do aprendiz, a junção de saberes e experiências que não seriam possíveis se eu não estivesse errando e aprendendo continuamente. Que venham os próximos erros nas demais etapas do projeto!

Para conhecer o Alljobs Meetings, acesse o repositório no Github:
[www.github.com/0jonjo/alljobs-meetings](https://www.github.com/0jonjo/alljobs-meetings)

Para saber mais sobre ranges no Postgres e manipulação de datas e horários no Java e Spring Boot:

- Ranges no Postgres: [https://www.postgresql.org/docs/12/rangetypes.html](https://www.postgresql.org/docs/12/rangetypes.html)
- LocalTime e LocalDate no Spring Boot: [https://www.baeldung.com/spring-boot-date-fields](https://www.baeldung.com/spring-boot-date-fields)
- Manipulando datas e horários no Java: [https://www.baeldung.com/java-date-time](https://www.baeldung.com/java-date-time)

---
>Image: ["Spring Flowers & Rain Boots, David Kesler"](https://openverse.org/image/9e127bcc-ccf4-406f-b02f-2e26e816c8c9?). [Open Verse, Creative Commons 2.0](https://openverse.org/)
