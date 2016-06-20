+++
author = "Andrei Formiga"
date = "2016-06-19T16:23:58-03:00"
description = "Comparamos Swift e OCaml em um exemplo típico para programação funcional"
draft = true
tags = ["swift", "ocaml", "funcional"]
title = "Interpretador e Compilador de expressões em Swift"
topics = ["programação"]
type = "post"

+++

Quando se fala em programação funcional, uma pergunta importante dos que já não estão
convencidos é "por que aprender programação funcional?" Autores de livros e palestrantes
normalmente já se antecipam e respondem essa pergunta no começo.

Os motivos são vários mas uma boa parte deles está focada em um objetivo maior: se tornar
um programador melhor. Esse objetivo está relacionado a motivos mais filosóficos e
de longo prazo, como a ideia que um bom programador deve conhecer diferentes paradigmas
e maneiras de organizar e resolver os problemas (vide o [ótimo texto de Peter Norvig sobre
como se tornar um bom programador](https://pihisall.wordpress.com/2007/03/15/aprenda-a-programar-em-dez-anos/)).

Mas o objetivo de subir de nível na programação também está relacionado a motivos mais diretos
e pragmáticos: no caso da programação funcional, muitas novas linguagens têm sido criadas com
forte influência do paradigma funcional, mesmo que não sejam classificadas dentro do paradigma.

Entre essas novas linguagens criadas com um forte sabor funcional está a linguagem
[Swift](https://swift.org/) da Apple. Anunciada publicamente há apenas dois anos, Swift
tem várias característiscas diretamente inspiradas pelas linguagens funcionais. Neste texto
eu vou mostrar como traduzir um exemplo do [livro de OCaml](http://andreiformiga.com/livro/ocaml)
para Swift, e vamos ver que algumas partes do código são uma tradução direta, mudando
apenas a sintaxe; como Swift não é uma linguagem primariamente funcional outras partes
ficam mais naturais com uma abordagem imperativa e são diferentes da versão em OCaml. Mesmo
assim, as ideias principais para resolver o problema são definitivamente as mesmas, e resultam
em uma solução compacta e elegante que nem sempre é óbvia para quem não tem experiência
com programação funcional.

O exemplo que será traduzido vem do Capítulo 6 [do livro](https://www.casadocodigo.com.br/pages/sumario-ocaml),
que é um exemplo um pouco maior de programação funcional pura: uma linguagem de expressões
aritméticas, junto com um interpretador para a linguagem, uma pequena máquina virtual de pilha,
e um compilador da linguagem de expressões para a máquina virtual.

Para quem leu o meu livro ou já sabe OCaml (ou alguma linguagem similar como Haskell),
a versão em Swift será simples de entender. Para quem sabe Swift, a influência funcional
em características como as `enums` da linguagem ficarão claras, embora programadores de
experiência principalmente imperativa muitas vezes nunca pensaram em usar `enums` da forma
que veremos aqui (já ouvi isso de algumas pessoas). Para quem ainda não conhece nenhuma das
duas linguagens, todos os conceitos principais serão explicados aqui.

<!--
Criada por Chris Lattner (criador também do [LLVM](https://llvm.org),
um conjunto de ferramentas para criação de compiladores e geradores de código), a linguagem
foi anunciada publicamente em 2014 pela Apple como a linguagem que iria futuramente substituir
Object-C como linguagem principal para criar aplicativos para o iOS (além do Mac OS X e outros
produtos similares como o tvOS). Em 2015, quando da versão 2.0 da linguagem, a Apple anunciou que
a linguagem Swift e seu ferramental seria aberto como _open source_.
-->

~~~ocaml
type exp =
  | Const of int
  | Soma of exp * exp
  | Sub of exp * exp
  | Mult of exp * exp

~~~

~~~swift
enum Exp {
	case Const(Int)
	indirect case Soma(Exp, Exp)
	indirect case Sub(Exp, Exp)
	indirect case Mult(Exp, Exp)
}

~~~
