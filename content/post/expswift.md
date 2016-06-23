+++
author = "Andrei Formiga"
date = "2016-06-19T16:23:58-03:00"
description = "Comparamos Swift e OCaml em um exemplo típico para programação funcional"
draft = false
tags = ["swift", "ocaml", "funcional"]
title = "Interpretador e Compilador de expressões em Swift"
topics = ["programação"]
type = "post"

+++

Quando se fala em programação funcional, uma pergunta importante é
"por que aprender programação funcional?" Autores de livros e palestrantes
normalmente já se antecipam e respondem a essa pergunta no começo.

Os motivos são vários mas uma boa parte deles está focada em um
objetivo maior: se tornar um programador melhor. Esse objetivo está
relacionado a motivos mais filosóficos e de longo prazo, como a ideia
que um bom programador deve conhecer diferentes paradigmas e maneiras
de organizar e resolver os problemas (vide o
[ótimo texto de Peter Norvig sobre como se tornar um bom programador](https://pihisall.wordpress.com/2007/03/15/aprenda-a-programar-em-dez-anos/)).

Mas o objetivo de subir de nível na programação também está
relacionado a motivos mais diretos e pragmáticos: no caso da
programação funcional, muitas novas linguagens têm sido criadas com
forte influência do paradigma funcional, mesmo que não sejam
classificadas dentro do paradigma.

Entre essas novas linguagens criadas com um forte sabor funcional está
a linguagem [Swift](https://swift.org/) da Apple. Anunciada
publicamente há apenas dois anos, Swift tem várias característiscas
diretamente inspiradas pelas linguagens funcionais. Neste texto eu vou
mostrar como traduzir um exemplo do
[livro de OCaml](http://andreiformiga.com/livro/ocaml) para Swift, e
vamos ver que algumas partes do código são uma tradução direta,
mudando apenas a sintaxe; como Swift não é uma linguagem primariamente
funcional, outras partes ficam mais naturais com uma abordagem
imperativa e são diferentes da versão em OCaml. Mesmo assim, as ideias
principais para resolver o problema são definitivamente as mesmas, e
resultam em uma solução compacta e elegante que nem sempre é óbvia
para quem não tem experiência com programação funcional.

O exemplo que será traduzido vem do Capítulo 6
[do livro](https://www.casadocodigo.com.br/pages/sumario-ocaml), que é
um exemplo um pouco maior de programação funcional pura: uma linguagem
de expressões aritméticas, junto com um interpretador para a
linguagem, uma pequena máquina virtual de pilha, e um compilador da
linguagem de expressões para a máquina virtual.

Para quem leu o meu livro ou já sabe OCaml (ou alguma linguagem
similar como Haskell), a versão em Swift será simples de
entender. Para quem sabe Swift, a influência funcional em
características como as `enums` da linguagem ficarão claras, embora
programadores de experiência principalmente imperativa muitas vezes
nunca pensaram em usar `enums` da forma que veremos aqui (já ouvi isso
de algumas pessoas). Para quem ainda não conhece nenhuma das duas
linguagens, todos os conceitos principais serão explicados aqui.

O código Swift mostrado neste texto não é necessariamente idiomático,
por dois motivos: primeiro que ainda sou _noob_ na linguagem, e segundo
porque o código foi traduzido de outra linguagem. De toda forma, começamos
discutindo como representar expressões em um programa.


### Expressões e sintaxe abstrata

A linguagem objeto que iremos tratar é uma linguagem de expressões aritméticas,
com operandos constantes, usando três operações: soma, subtração e
multiplicação. É uma linguagem simples e que todo mundo deve entender da
matemática do colégio. Um exemplo é a expressão `2 + 3 * 4`.

Se o objetivo fosse criar um interpretador para usuários finais, teríamos que
definir uma sintaxe para a linguagem, provavelmente incluindo parênteses. No
jargão dos processadores de linguagens (como compiladores e interpretadores)
essa sintaxe externa, vista pelo usuário final, é chamada de _sintaxe concreta_.
Mas como o objetivo é ver o funcionamento de um interpretador e de um compilador
simples, evitamos a complicação de analisar a sintaxe concreta e trabalhamos
diretamente com a _sintaxe abstrata_. A sintaxe abstrata é uma representação
interna da entrada, geralmente uma estrutura de dados que representa a
expressão que deve ser processada.

A sintaxe abstrata pode ser representada em uma estrutura de árvore, chamada
de _árvore de sintaxe abstrata_; em inglês, _Abstract Syntax Tree_ (AST).
Árvores representam muito bem as relações estruturais presentes na sintaxe
da maioria das linguagens, incluindo a capacidade de representar
relações de aninhamento e hierarquia.

Em uma AST para expressões, os nós internos são operadores cujos filhos são
os operandos; as folhas representam os
operandos atômicos, aqui sempre números inteiros. A árvore para a expressão
`2 + 3 * 4` é
<center>
![Árvore para 2+3*4](/img/arv3.png)
</center>

Note que a árvore representa explicitamente a precedência entre operadores.
Para comparação, a árvore para a expressão `(2 + 3) * 4` é
<center>
![Árvore para (2+3)*4](/img/arv2.png)
</center>

Os parênteses não são necessários na árvore pois sua própria estrutura
representa a precedência desejada. Essa é uma vantagem de usar uma
representação hierárquica, ao invés de uma sequência linear de
símbolos.

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
semelhança com a primeira árvore sintática vista anteriormente):
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
tipo `Int`, enquanto que os outros casos representam operações binárias.
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
casos, é preciso obter recursivamente o valor das sub-expressões que são
operandos da operação,
e aplicar a operação correspondente aos resultados. É uma daquelas coisas que
ficam incrivelmente óbvias depois que você já sabe como fazer, mas nem tanto
quando você nunca viu antes.

Em Swift o código é praticamente o mesmo, tirando as diferenças sintáticas
(Swift é mais prolixa):

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

Neste ponto já podemos fazer alguns testes no REPL ou incluindo em um arquivo
e compilando. Os testes consistem em definir expressões no tipo `Exp` e
executar o interpretador para verificar se o resultado está correto (aqui
testando no REPL):

~~~swift
1> let e1 = Exp.Soma(Exp.Const(2),
                     Exp.Mult(Exp.Const(3),
                              Exp.Const(4)))
e1: Exp ...

2> eval(e1)
$R0: Int = 14

~~~

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

As instruções da máquina são representadas em Swift com o seguinte tipo:
~~~swift
enum Instrucao {
    case Empilha(Int)
    case Oper(Operacao)
}

~~~

(Daqui em diante não vou mostrar o código OCaml equivalente a cada trecho
em Swift; o
[código completo desse exemplo em
OCaml](https://github.com/tautologico/opfp/blob/master/exp/src/exp.ml)
pode ser visto no github).

No código Swift (assim como em OCaml) declaramos uma
instrução `Oper` que representa todas as operações binárias; a operação
específica é definida pelo tipo `Operacao`:

~~~swift
enum Operacao {
    case OpSoma
    case OpSub
    case OpMult
}

~~~

O coração de uma máquina de pilha é, obviamente, a pilha. O código em OCaml
é puramente funcional: a função que executa uma instrução da máquina recebe
a instrução e a pilha atual, e retorna a pilha resultante após a execução
da instrução dada; sem efeitos colaterais. Isso funciona e é relativamente
eficiente em OCaml pelo uso das listas funcionais, que são estruturas
persistentes que permitem manter versões alteradas de uma lista original
sem copiar a lista inteira em cada modificação.

Swift, por padrão, não inclui nenhuma estrutura similar. Poderíamos
definir um tipo de referência usando classes, ou poderíamos usar
[uma biblioteca já pronta](https://github.com/typelift/Swiftz)
com listas funcionais, mas eu achei melhor me ater à linguagem básica.

Por isso, a pilha é imperativa na versão Swift. Definimos um tipo para a
pilha, agrupando a funcionalidade requerida em métodos:

~~~swift
struct Pilha {
    var itens = [Int]()

    mutating func empilha(i: Int) {
        self.itens.append(i)
    }

    func topo() -> Int? {
        return self.itens.last
    }

    mutating func operandos() -> (Int, Int)? {
        if self.itens.count < 2 {
            return nil
        } else {
            let r1 = self.itens.removeLast()
            let r2 = self.itens.removeLast()
            return (r1, r2)
        }
    }
}

~~~

A pilha é implementada usando um _array_ de inteiros; _arrays_ em
Swift são dinâmicos. Definimos um método para empilhar, um para
verificar o valor no topo da pilha, e um que obtêm os dois operandos
de uma operação, se presentes na pilha. Métodos que alteram o valor
de um tipo `struct` devem ser marcados com a palavra-chave `mutating`.
O tipo de retorno do método `topo` é `Int?`, que é um _tipo opcional_
(em OCaml seria `int option`). Um valor do tipo `Int?` pode conter
um inteiro ou o valor especial `nil`, que representa a falta de um
valor. No caso do método `topo`, o valor é opcional porque se a
pilha estiver vazia, não há nada para retornar. O mesmo ocorre
no método `operandos`, que retorna os dois inteiros no topo da
pilha, se a pilha tiver pelo menos dois elementos.

Com a pilha definida, a execução de instruções da máquina é simples:
se recebe a instrução de empilhar, a máquina empilha o número contido
na instrução; se recebe uma instrução de operação, retira dois operandos
da pilha e aplica a operação. Para simplificar a aplicação da operação,
usamos uma função que mapeia cada valor `Operacao` na função correspondente.
Em Swift podemos extender tipos já existentes, por exemplo adicionando
novos métodos; vamos adicionar o mapeamento dos valores `Operacao` como
um método no tipo:

~~~swift
extension Operacao {
    func oper() -> (Int, Int) -> Int {
        switch self {
            case .OpSoma:
                return (+)
            case .OpSub:
                return (-)
            case .OpMult:
                return (*)
        }
    }
}

~~~

Com toda a estrutura montada, definimos a função que executa uma instrução
da máquina, dada a instrução e a pilha inicial:

~~~swift
func execInst(i: Instrucao, inout pilha: Pilha) {
    switch i {
        case let .Empilha(i):
            pilha.empilha(i)
        case let .Oper(op):
            if let (v1, v2) = pilha.operandos() {
                pilha.empilha(op.oper()(v1, v2))
            }
    }
}

~~~

A função `execInst` segue a ideia discutida antes: se a instrução for para
empilhar, empilhe; se for uma operação, verifique se é possível desempilhar
os dois operandos, obtenha a função adequada (`op.oper()`) e aplique essa
função aos operandos, empilhando o resultado. O uso de ADTs, _pattern matching_
e funções como valores torna o código compacto e elegante.

Um programa da máquina é uma sequência de instruções, representada no código
por um _array_ `[Instrucao]`. Para executar um programa basta começar com uma
pilha vazia e executar cada instrução em sequência:

~~~swift
func execProg(prog: [Instrucao]) -> Pilha {
    var p = Pilha()
    for i in prog {
        execInst(i, pilha: &p)
    }

    return p
}

~~~

Geralmente estamos interessados apenas no valor final do programa; se não houver
nenhum problema na execução, esse valor estará no topo da pilha, ao final da
execução. A função `executa` é definida para esse caso de uso:

~~~swift
func executa(prog: [Instrucao]) -> Int? {
    let pilha = execProg(prog)
    return pilha.topo()
}

~~~

Podemos fazer um pequeno teste, no REPL ou incluindo no arquivo fonte:

~~~swift
let p1 = [Instrucao.Empilha(5),
          Instrucao.Empilha(7),
          Instrucao.Oper(Operacao.OpSoma),
          Instrucao.Empilha(10),
          Instrucao.Oper(Operacao.OpMult)]

if let res = executa(p1) {
    print("resultado do programa: \(res)")
} else {
    print("programa sem resultado")
}

~~~

Quando executado, esse código deve imprimir o resultado do programa, que é
120.

Agora só falta o compilador que vai traduzir da linguagem de expressões
para a linguagem da máquina de pilha.

### Compilador

Felizmente, esse processo de compilação é simples. Para uma expressão `Const`,
empilhe o valor constante; se a expressão for uma operação, recursivamente
traduza as duas sub-expressões, concatenando as sequências de instruções
resultantes, e adicione ao final a operação adequada. O único cuidado é
com a ordem: o código para o segundo operando (operando direito) das operações
deve ser adicionado primeiro, para deixar os argumentos na pilha na ordem
certa para a operação que virá depois (não faz diferença para soma e
multiplicação, mas faz para a subtração). O código é:

~~~swift
func compila(e: Exp) -> [Instrucao] {
    switch e {
        case let .Const(i):
            return [Instrucao.Empilha(i)]
        case let .Soma(e1, e2):
            var prog = compila(e2)
            prog.appendContentsOf(compila(e1))
            prog.append(Instrucao.Oper(Operacao.OpSoma))
            return prog
        case let .Sub(e1, e2):
            var prog = compila(e2)
            prog.appendContentsOf(compila(e1))
            prog.append(Instrucao.Oper(Operacao.OpSub))
            return prog
        case let .Mult(e1, e2):
            var prog = compila(e2)
            prog.appendContentsOf(compila(e1))
            prog.append(Instrucao.Oper(Operacao.OpMult))
            return prog
    }
}

~~~

E um último teste: vamos definir uma expressão e verificar se avaliar seu
resultado com o interpretador dá no mesmo que compilar e executar o código
compilado:

~~~swift
let e1 = Exp.Mult(Exp.Const(3),
                  Exp.Soma(Exp.Const(4),
                           Exp.Const(2)))
print(eval(e1))
print(executa(compila(e1))!)

~~~

Isso deve mostrar o mesmo resultado, 18, nas duas impressões.

### Programação funcional é programar com funções, certo?

No final, a versão completa em Swift é bastante similar ao
código original em OCaml. A única ideia diferente é o uso
de uma pilha imperativa em Swift ao invés da pilha
puramente funcional em OCaml; de resto as diferenças são
apenas sintáticas, com a sintaxe de Swift se mostrando mais
prolixa: o programa completo em Swift tem cerca de 130 linhas,
a versão OCaml tem 96, contando apenas as funções que foram
traduzidas para Swift (mas a versão em
OCaml ainda tem mais comentários). Isso indica que Swift tem
pelo menos algum suporte para código escrito em estilo funcional.

Mas isso vai além do que é mostrado nesse exemplo pequeno, incluindo
a ênfase da linguagem em programar com _protocolos_ (similares às
_type classes_ em Haskell) e valores imutáveis. Vide por exemplo
[esta palestra recente](https://developer.apple.com/videos/play/wwdc2016/419/)
do pessoal da Apple.

Esses conceitos (e como usá-los efetivamente) nem sempre são óbvios
para quem não tem experiência com programação funcional. Uma reação
comum de alguns programadores ao exemplo deste texto é
"quando vejo `enum` eu penso no conceito da linguagem C e similares;
eu nunca pensaria em resolver esse problema desta forma".

Curiosamente, ao ler coisas sobre Swift na web eu encontrei
[um texto](https://medium.com/@Jernfrost/functional-design-patterns-in-swift-interpreter-169fa164a6ec#.15vd40iv2)
que também se propõe a criar um interpretador de expressões usando
programação funcional. É uma leitura interessante e serve como um
exemplo de uso de funções de alta ordem (funções como valores de
primeira classe), que o autor usa para representar as árvores de
sintaxe abstrata. Mas dificilmente um programador funcional
com alguma experiência usaria uma solução desse tipo. Representar
a AST com funções é interessante como curiosidade, mas não faz
sentido na prática; usamos estruturas de dados para a AST porque
um interpretador ou compilador mais sofisticado não vai apenas
executar ou traduzir o código diretamente; vai varrer a árvore
várias vezes, realizando vários tipos de análise no código. Isso
é difícil, pouco prático ou impossível de fazer com _closures_,
na maioria das linguagens.

(No final do código em OCaml tem uma função que faz uma otimização
simples nas expressões, removendo somas em que um dos operandos é
zero. Isso só é possível porque a AST está representada como uma
estrutura de dados. Aliás, a versão em Swift desta função fica
como exercício para o leitor.)

Programação funcional não é apenas "programar com funções". Assim como
com qualquer paradigma, existem alguns conceitos-chave que são usados
como resumo para explicar do que se trata a programação funcional, mas
também existe uma cultura, tradições, formas recomendadas de resolver
os problemas.  Isso não se aprende de ouvir falar, mas na prática,
escrevendo e lendo código, falando com pessoas mais experientes no
paradigma, e lendo o que eles escrevem.  Usar uma linguagem do
paradigma é uma forma de ficar imerso nessa cultura, de se forçar a
resolver os problemas de outra forma; é como um antropólogo que vai
viver no meio do povo que ele quer entender.

Sem essa imersão, é difícil realmente aprender um paradigma, por mais
que se leia sobre ele. E aprender um paradigma significa ampliar seu
repertório de técnicas para resolver problemas, mesmo que não use o
paradigma diretamente na sua vida profissional.

O [código original em OCaml](https://github.com/tautologico/opfp/blob/master/exp/src/exp.ml)
está no github, em um [repositório que contém todos os exemplos](https://github.com/tautologico/opfp)
do livro.

O [código completo em Swift](https://gist.github.com/tautologico/e93ce90c8fe8bf31c9e867d24b1b8fec)
está disponível como um gist.
