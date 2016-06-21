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

### Expressões e sintaxe abstrata

A linguagem objeto que iremos tratar é uma linguagem de expressões aritméticas,
com operandos constantes, usando três operações aritméticas: soma, subtração e
multiplicação. É uma linguagem simples e que todo mundo deve entender da
matemática do colégio. Um exemplo é a expressão `2 + 3 * 4`.

Se o objetivo fosse criar um interpretador para usuários finais, teríamos que
definir uma sintaxe para a linguagem, provavelmente incluindo parênteses. No
jargão dos processadores de linguagens (como compiladores e interpretadores)
essa sintaxe externa, vista pelo usuário final, é chamada de _sintaxe concreta_.
Mas como o objetivo é ver o funcionamento de um interpretador e de um compilador
simples, evitamos a complicação de analisar a sintaxe concreta e trabalhamos
diretamente com a _sintaxe abstrata_. A sintaxe abstrata é uma representação
interna da entrada, geralmente uma estrutura de dados que representa o programa
ou expressão na entrada.

A sintaxe abstrata pode ser representada em uma estrutura de árvore, chamada
de _árvore de sintaxe abstrata_; em inglês, _Abstract Syntax Tree_ (AST).
Árvores representam muito bem as relações estruturais presentes na sintaxe
da maioria das linguagens, incluindo aninhamento e hierarquia.

Em uma AST para expressões, os nós internos são operadores e as folhas são os
operandos atômicos, aqui sempre números inteiros. A árvore para a expressão
`2 + 3 * 4` é
<center>
![Árvore para 2+3*4](/img/arv3.png)
</center>

E, para comparação, a árvore para a expressão `(2 + 3) * 4` é
<center>
![Árvore para (2+3)*4](/img/arv2.png)
</center>

Escrevi sobre árvores sintáticas em outro texto recente: [Representação de
fórmulas da lógica](/post/repformlog). Lá eu falo sobre como representar
as árvores sintáticas na linguagem C. Aqui veremos como fazer isso em
OCaml e Swift, linguagens mais modernas. O Capítulo 6 [do livro](/livro/ocaml)
também tem uma discussão maior sobre árvores sintáticas e sintaxe abstrata.

### Representação das árvores sintáticas

Um programador com experiência em alguma linguagem funcional como OCaml ou
Haskell não precisa pensar muito sobre como representar as árvores sintáticas; é
_óbvio_ que a melhor forma é usando um tipo variante ou _tipo de dado algébrico_
(no inglês, _Algebraic Data Type_ ou ADT). Em OCaml isso fica da seguinte
forma:

~~~ocaml
type exp =
  | Const of int
  | Soma of exp * exp
  | Sub of exp * exp
  | Mult of exp * exp

~~~

A variante `Const` representa constantes inteiras (tendo um valor associado do
tipo `int`), e as variantes `Soma`, `Sub` e `Mult` representam as operações
indicadas pelo nome. Cada operação tem dois operandos que são expressões,
o que significa que o tipo é recursivo. A expressão `2 + 3 * 4` é representada
em código como

~~~ocaml
Soma (Const 2, Mult(Const 3, Const 4))

~~~

Este valor também pode ser visto graficamente, em um diagrama que deixa claro
porque usamos tipos variantes para representar árvores sintáticas (note a
semelhança com a árvore sintática vista anteriormente):
<center>
![Valor em OCaml para 2 + 3 *4](/img/arvml2.png)
</center>

Em Swift podemos criar um ADT usando uma `enum` na qual os casos (ou variantes)
podem ter valores associados:

~~~swift
enum Exp {
	case Const(Int)
	indirect case Soma(Exp, Exp)
	indirect case Sub(Exp, Exp)
	indirect case Mult(Exp, Exp)
}

~~~

O caso `Const` representa constantes inteiras, contendo um valor associado do
tipo `Int`, enquanto que os outros casos representam operações binárias
Uma particularidade é que Swift divide os tipos em tipos de referência e tipos
de valor (_value types_), e as `enum`s são tipos de valor. Isso significa que
os valores associados a cada `case` são armazenados diretamente no espaço de
memória reservado para o `enum`, sem uso de referência. Obviamente, seguir
isso à risca tornaria impossível ter `enum`s recursivas, pois a quantidade
de memória necessária seria infinita. Para resolver isso, em Swift é possível
declarar casos recursivos como `indirect`, ou seja, indiretos. Por baixo dos
panos, o compilador Swift vai adicionar o uso de referências quando necessário,
mas de resto os casos indiretos funcionam da mesma forma que os outros.

Em Swift a expressão `2 + 3 * 4` é representada em código como:
~~~swift
Exp.Soma(Exp.Const(2),
         Exp.Mult(Exp.Const(3), Exp.Const(4)))

~~~

A única diferença para a mesma expressão em OCaml é que em Swift é necessário
incluir antes de cada caso o nome da `enum` (algo parecido ocorre em OCaml
quando se usa um tipo variante em um módulo diferente de onde ele foi definido).

Agora que sabemos como representar as expressões, vamos calcular seus valores.

### Interpretador

O interpretador em OCaml segue uma regra clássica da programação funcional: uma
função que processa um valor de um certo tipo deve ter a mesma estrutura do tipo.
Se o tipo é recursivo, a função será recursiva nos mesmos pontos (obrigado,
[HtDP](http://htdp.org/)):

~~~ocaml
let rec eval e =
  match e with
    Const n -> n
  | Soma (e1, e2) -> eval e1 + eval e2
  | Sub (e1, e2) -> eval e1 - eval e2
  | Mult (e1, e2) -> eval e1 * eval e2

~~~

Para o caso `Const`, o valor da expressão é o valor da constante. Para os outros
casos, é preciso obter recursivamente o valor das sub-expressões da operação,
e aplicar a operação correspondente aos resultados. É uma daquelas coisas que
ficam incrivelmente óbvias depois que você já conhece a resposta.

Em Swift o código é praticamente o mesmo, tirando as diferenças sintáticas:

~~~swift
func eval(e: Exp) -> Int {
    switch e {
        case let .Const(i):
            return i
        case let .Soma(e1, e2):
            return eval(e1) + eval(e2)
        case let .Sub(e1, e2):
            return eval(e1) - eval(e2)
        case let .Mult(e1, e2):
            return eval(e1) * eval(e2)
    }
}

~~~

A versão Swift é um pouco mais prolixa em termos de sintaxe, mas representa
diretamente as mesmas ideias.

Mas Swift não é uma linguagem funcional, então eventualmente vemos algumas
diferenças quando consideramos uma máquina virtual para expressões.

### Máquina de pilha

Um compilador é basicamente um tradutor, convertendo programas escritos em
uma linguagem (a linguagem _fonte_) para uma outra linguagem (a linguagem de
_destino_). Muitas vezes a linguagem fonte é de alto nível, e a linguagem
de destino é alguma linguagem de nível mais baixo que pode ser executada
diretamente por algum processador ou máquina virtual. Para discutir
compilação sem cair nas complexidades das arquiteturas reais, vamos
aqui definir uma máquina de pilha bem simples para avaliar expressões
aritméticas. Plagiando um pouco de mim mesmo:

> Existem dois tipos principais de arquiteturas de máquinas de execução: máquinas de
> pilha e máquinas de registrador. Podemos dizer que uma máquina de pilha executa
> instruções que obtêm os dados de entrada de uma _pilha_, e armazenam os dados
> de saída na mesma pilha. Essa pilha é um dispositivo de memória que está
> organizada para seguir a estratégia LIFO (_Last In, First Out_ ou o último a
> entrar é o primeiro a sair) de armazenamento.
>
> Já uma máquina de registradores executa instruções que obtêm dados de entrada dos
> registradores da máquina e armazenam os dados de saída também em registradores.
> Os registradores são memórias especializadas que normalmente armazenam uma pequena
> quantidade de dados.

Nossa máquina de pilha para expressões terá quatro instruções: a operação de
empilhar, que coloca um valor inteiro no topo da pilha, e as três operações
aritméticas da linguagem de expressões. As instruções que representam operações
retiram os seus dois operandos da pilha, e empilham o resultado da operação.
Por exemplo, suponha que a pilha contenha os valores 9 e 1 (nesta ordem, do
topo) e a máquina executa três instruções: empilha 5, empilha 3, e soma.
O estado da pilha em cada passo é:
<center>
![Execução na máquina de pilha](/img/maqpilha.png)
</center>

Quando a máquina executa a instrução `soma`, os operandos 3 e 5 são retirados
da pilha e o resultado da soma deles, 8, é empilhado em seguida.

~~~swift
enum Instrucao {
    case Empilha(Int)
    case Oper(Operacao)
}

~~~

No código Swift (assim como em OCaml, não mostrado aqui) declaramos uma
instrução `Oper` que representa todas as operações binárias; a operação
específica é definida pelo tipo `Operacao`:

~~~swift
enum Operacao {
    case OpSoma
    case OpSub
    case OpMult
}

~~~
