---
layout:     post
title:      "Server-side: características de um servidor"
subtitle:   "Um pouco sobre a arquitetura dos servidores"
date:       2017-02-15 00:00:00
author:     "Ronaldo Faria Lima"
header-img: "img/ronflima/background.jpg"
category:   server
---

# O que é o servidor?

O conceito de servidor não é novo. Apareceu na série 360 da IBM em 1964. Desde
então vem sofrendo transformações até chegar no modelo em nuvem conforme temos
hoje. Mas, o que raios é o servidor?

Olhando pela ótica do software, que é a premissa deste artigo, o servidor é um
artefato de software cuja função é prover serviços outros artefatos de software,
chamados de _clientes_. Esta não é uma definição formal nem muito menos
precisa. A intenção é partirmos desta definição para entender como projetar um
servidor de software de maneira a usar as principais características do sistema
operacional no qual estará hospedado.

Hoje em dia o papel principal do servidor é prover serviços a clientes leves, ou
_thin clients_, normalmente aplicativos para celulares. Todo o processamento
pesado e o armazenamento fica por conta do servidor, deixando o cliente com a
tarefa de organizar, logicamente, os fluxos de trabalho disponíveis para que o
usuáio seja atendido em determinada funcionalidade.

O servidor pode ser uma ou mais peças de software trabalhando em conjunto,
formando um sistema que pode ser agregado ou distribuído. Para o cliente, o
servidor é um só, mesmo que sejam várias máquinas físicas diferentes, vários
sistemas de software criados por várias linguagens diferentes, interagindo entre
si através das mais diversas formas de integração.

