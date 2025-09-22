---
layout: post
title: "Imergindo na IA: notas de um desenvolvedor estudando Aprendizado de Máquina e Ética e Dados"
description: "Aproximações entre o curso de Inteligência Artificional e o trabalho como desenvolvedor de projetos com IA"
tags: programming learning
categories: misc
youtubeId:
image:
---

Escolhi neste semestre as disciplinas de Aprendizado de Máquina Supervisionado e Ética e Dados como minhas primeiras do curso de [Inteligência Artificial (UFRN)](https://pes.imd.ufrn.br/pes/index). A primeira trata de algoritmos que aprendem a partir de exemplos, desde a coleta, sistematização e ajustes de dados até a construção de árvores de decisão e redes neurais artificiais. A proposta é que, ao final do semestre, sejamos capazes de construir modelos preditivos a partir de conjuntos de dados reais. Por exemplo, a máquina receber os dados de exame de um paciente e prever se ele tem ou não predisposição a uma certa doença. Já a segunda trata de dimensões históricas, filosóficas e legais do uso de dados e algoritmos, como vieses, privacidade, transparência e responsabilidade.

Nesses primeiros meses de curso fiz diversas pontes com o trabalho de backend que venho desenvolvendo nos últimos anos, especialmente com projetos de IA que estou trabalhando na [JetRockets](https://www.linkedin.com/company/jetrockets/). Parte dessas pontes diz respeito a questões matemáticas; outras, a modos de ajustar dados enviados e recebidos por modelos de IA; e algumas tratam diretamente de ética e privacidade.

Seguem as principais conexões que tenho observado:

**Comunicação com sistemas e modelos externos**: um aspecto comum em grandes projetos backend é a integração com diversos sistemas terceiros via filas, APIs e webhooks. Além dos detalhes puramente técnicos, realizar essa comunicação requer desenvolver abstrações que consigam produzir e consumir informações pensadas para outras regras de negócio — por exemplo, meios de pagamento, sistemas de ERP e CRMs. A combinação, suavização e transformação de dados vindos de diferentes fontes é um desafio constante.

No Aprendizado de Máquina, durante o treinamento que dá origem aos modelos, estudamos e praticamos como interpretar e preparar dados vindos de fontes variadas para utilizá-los nos treinamentos e testes. É necessário entender o formato, a estrutura e o significado dos dados, além de identificar e tratar possíveis problemas como valores ausentes, inconsistências e ruídos.

Algo semelhante ocorre na comunicação com modelos de IA via API (como Claude, OpenAI e Gemini): é preciso interpretar e preparar os dados de entrada e saída para garantir que estejam no formato correto e atendam aos requisitos do modelo. Em ambos os casos, é fundamental compreender bem os dados e as necessidades do sistema para garantir uma comunicação eficaz.

**Entendendo a vetorização**: em projetos de IA, a vetorização é o processo de transformar dados em vetores numéricos para que possam ser processados por algoritmos de aprendizado de máquina. Isso é especialmente importante quando se trabalha com dados não estruturados, como texto e imagens. A vetorização permite que os algoritmos extraiam características relevantes e facilitem a análise e a tomada de decisões.

Em projetos backend, a vetorização também pode otimizar armazenamento e consulta de dados, especialmente quando se trabalha com grandes volumes de informações. Entender seus conceitos e técnicas tem sido fundamental nos projetos em que atuo; estudá-los e praticá-los faz parte importante do meu aprendizado em IA.

**Normalizando dados**: diversos processos de fusão, comparação e transformação de dados envolvem a normalização — ajustar os valores para uma escala comum sem distorcer diferenças nas faixas de valores. Isso garante que os dados sejam tratados de forma consistente, especialmente em algoritmos sensíveis à escala.

No Aprendizado de Máquina, a normalização é etapa crucial de pré-processamento: melhora performance e convergência dos algoritmos. Em projetos de IA, etapas como a preparação de dados para RAG (Retrieval-Augmented Generation) também envolvem normalização para atender requisitos do modelo. Por isso, tem sido útil revisitar conceitos como média, mediana, moda, variância e desvio padrão, aplicando-os nos projetos da disciplina e nos projetos de IA que venho desenvolvendo.

**Entendendo e controlando vieses em dados e algoritmos**: os vieses são inclinações políticas, sociais ou de gênero presentes nas fontes de dados. Podem ter raízes históricas e culturais ou ser produto de decisões humanas conscientes ou inconscientes. Eles influenciam desde a seleção inadequada de dados até a representação desigual de diferentes grupos.

No Aprendizado de Máquina, o controle de vieses é preocupação constante, pois podem levar a resultados injustos ou imprecisos, especialmente quando se trabalha com dados sensíveis. O controle de vieses também aparece na preparação de prompts para modelos de IA, onde é preciso garantir que o retorno do modelo seja adequado àquela chamada, muitas vezes mitigando vieses indesejados. Tenho estudado técnicas e estratégias para identificar, mitigar e controlar vieses, garantindo modelos mais justos, transparentes e responsáveis.

**Gerindo dados privados**: ao lidar com informações pessoais e sensíveis, garantir a privacidade e a segurança dos dados é essencial. Isso envolve adotar práticas e tecnologias para proteger contra acessos não autorizados, vazamentos e usos indevidos.

Na relação com modelos de IA, o gerenciamento de dados privados é uma preocupação constante. Também é fundamental garantir conformidade com leis e regulamentos de privacidade, como a LGPD no Brasil. Tenho estudado legislação e práticas, adotando técnicas e estratégias para proteger a privacidade dos dados com abordagens seguras e responsáveis.

![image](https://github.com/0jonjo/0jonjo.github.io/assets/64807181/9364065b-f8c8-489c-a2e3-55bff1acd8c1)
>Imagem: "Mergulho" de Anderson Freire. [Open Verse, Creative Commons 2.0](hhttps://openverse.org/image/b952887f-8c48-4bb7-a5b6-e9ea9f97a00d)
