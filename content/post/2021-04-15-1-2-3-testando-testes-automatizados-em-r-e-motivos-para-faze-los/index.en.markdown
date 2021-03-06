---
title: '1,2,3, Testando! - Testes automatizados em R, e motivos para fazê-los (pt. 1)'
description: 'Nem sempre as coisas são o que aparentam ser...'
author: João Pedro Oliveira
date: '2021-04-15'
slug: []
categories:
  - R
tags:
  - engineering
Description: ''
Tags: []
Categories: []
comment: no
toc: no
autoCollapseToc: no
postMetaInFooter: no
hiddenFromHomePage: no
contentCopyright: no
reward: no
mathjax: yes
mathjaxEnableSingleDollar: yes
mathjaxEnableAutoNumber: no
hideHeaderAndFooter: no
---

## git commit -m "agora_vai_7.R - fix corrigido n.5"

Provavelmente você já esteve na situação em que não confia em seu próprio código. Depois dele falhar uma dezena de vezes, é dificil olhar para algo que escreveu e que mesmo que tenha revisado dezenas de vezes difícil acreditar que nenhum erro passou. Um tipo que foi convertido erroneamente e preencheu sua base de dados com `NA`, uma vírgula que adiciona um argumento extra e que não deveria estar ali, um arquivo errado que foi importado (Será que aquele `dados*2` não foi escrito como `data*2` em um surto de desatenção?). Muitos erros possíveis, como evitar? Como impedir de que cheguem no seu cliente? Como impedir que infectem sua pesquisa e tornem seus resultados inválidos? Testes! Nesse post vamos aprender um pouco sobre os motivos para testar e dar uma sobrevoada pela biblioteca `{testthat}`.

Testes automatizados são formas de você garantir que seu código está se comportando exatamente da forma como você deseja e retornando o resultado esperado. Eles tem como objetivo garantir que nenhuma distorção está acontecendo na pipeline e que as informaçõessao confiáveis o suficiente para serem entregues ao cliente. Neles você alimenta o computador com uma função a ser testada e um resultado esperado quando inserido determinado valor, como boas funções tendem a retornar `\(f(x)\)` quando alimentadas com `\(x\)`, é de se esperar que os testes passem quando tudo está ocorrendo perfeitamente.

Em R, convencionou-se que os testes são feitos utilizando a biblioteca `{testthat}`, que fornece uma excelente API tanto para testes unitários, quanto para testes deintegração**.** Nesse post eu focarei em testes unitários, no futuro eu voltarei para falar mais sobre testes de integração. Ambos são imensamente importantes, mas aqui eu tento manter um assunto por post, ou acabo me extendendo demais.

Com o perdao pela repetição, cito aqui o The Rust Programming Language (RPL), que diz que "Testes [...] sãofunções que verificam que o código que não é teste está funcionando da maneira esperada, testes tipicamente perfomam as seguintes ações:

1.  Preparam qualquer dado ou estado necessário.

2.  Executam o código a ser verificado.

3.  Conferem se o resultado do código é o esperado pelo desenvolvedor."

Embora o Rnãotenha a mesma suite de testes *built-in* que o Rust possui, sempre conseguimos dar um jeitinho.

### Nosso toolkit

Como dito anteriormente, a principal biblioteca que utilizaremos é a `testthat`, que deve estar acompanhada da `usethis`, então para reproduzir os exemplos aqui dispostos deve possuir ambas instaladas. Também adotarei uma sintaxe diferente, como em alguns momentos as funções geram efeitos colaterais, irei descrever o efeito gerado em itálico logo após o código executado.

Digamos que eu possua um projeto[^1] que está localizado no diretório que eu chamarei de `$JOBDIR/`, dentro dessa pasta eu possuo todos os arquivos essenciais para o meu trabalho. Dentro dessa pasta também tenho a pasta `R/`, onde estao os meus arquivos `.R`, que contém todo o código a ser executado.

[^1]: Aproveito aqui para lembrar o leitor de sempre usar os RStudio Projects em suas atividades, insular o código é importante muitas vezes.

### Quando é 2+3?

Em um cenário que unico arquivo dentro de `R/`, é o `1_somador.R`, e dentro desse arquivo eu tenho somente uma `function` chamada `soma()`, como definida abaixo:


```r
soma <- function(x, y) {
  x + y
}
```

Se eu já tenho uma função funcional, a criação de um teste simples passa para a etapa de escrever o teste em si. Para fazer isso, executamos a função `usethis::use_testthat()` e logo depois `usethis::use_test("somador")`, onde o primeiro argumento é o nome do arquivo do teste a ser executado. O nome deve ser sempre descritivo da`função (ou bloco de funções`) a ser testada. Executar:


```r
usethis::use_test("somador")
```

Irá gerar os seguintes efeitos colaterais:

-   //\> Cria o diretório `tests/testthat/`

-   //\> Dentro do diretório é criado o arquivo `test-somador.R`

Ao abrir o arquivo, irá se deparar com algo diferente, um template automático será gerado, olhe um pouco, entenda como funciona e depois apague tudo, iremos refazer ele aqui.

O primeiro passo é dizer para o o computador algo como "ei, aqui eu vou testar uma coisinha!", e no `R`, nós fazemos isso por meio da função `testthat::test_that()`. Usando `test_that("testa a soma de valores decimais", {...})`, eu consigo fazer a sinalização necessária para a linguagem. Para efetivamente testar se minha `função` funciona, eu devo escrever o seguinte


```r
library(testthat)
test_that("testa a soma de valores decimais", # Aqui uma pequena descrição do teste
                    {
                      # Aqui o teste a ser executado.
                      valor_retornado <- soma(0.1 + 0.3)
                      
                      expect_identical(valor_retornado, 0.4)
                      
                    })
```

Opa, perceba que nós temos algo novo aí, a função `expect_identical` aparece pela primeira vez, possuindo o nome bem sugestivo, o que ela diz é que ao executar `valor_retornado` e capturar seu resultado, o `R` deverá comparar este com `0.4`, se os resultados forem iguais o teste passou, caso contrário houve uma falha. Na ocorrencia de falhas o programador deve voltar a função e verificar qual foi o problema.

Curiosamente, ao contrário do que voce pode pensar, esse teste irá falhar. A falha na maioria dos casos é culpa do programador que não percebeu algum problema no decorrer do processo. No exemplo acima a falha tem uma origem, a [**IEEE 754**](https://en.wikipedia.org/wiki/IEEE_754 "Wikipedia - IEEE 754"), que é uma norma técnica que define como devem ser feitas operações com valores de ponto flutuante (valores decimais), ou seja, os falores diferem na casa dos decimais por conta de regras de arredondamento. Até mesmo o erro gerado é estranho, veja...


```r
testthat::expect_identical(0.1+0.2, 0.3)
```

```
## Error: 0.1 + 0.2 not identical to 0.3.
## Objects equal but not identical
```

Iguais mais nao identicos? Bom, realmente, o resultado teoricamente é igual:


```r
0.1+0.2
```

```
## [1] 0.3
```

Mas no fundo, para o computador, eles não são identicos e só um teste poderia lhe dizer isso pois a diferença não é nem exibida para nós.

A propósito, uma forma de só conferir se são iguais (deixando para lá o fator *identico*), é por meio da função `testthat::expect_equal()`, que tem tarefa semelhante, mas esse post já está ficando longo demais, entao fica para a próxima.

Vale lembrar que como estamos falando de testes unitários, os testes são feitos unidade por unidade, ou seja, umafunção de cada vez. O que eu recomendo é que cada conjunto de funções necessários para executar uma etapa da tarefa seja testada por arquivo, emboranãohaja uma obrigação disso ser seguido, eu acredito que o ganho em organização seja grande o suficiente para justificar a quantidade de arquivos. Portanto, nada de fazer os testes para um projeto inteiro todos em um arquivo só!

Termino esse post perguntando se ainda possui tanta certeza de que todos os códigos que escreveu anteriormente, e não testou, estão corretos.

##### Agradecimento.

Por fim eu gostaria de agradecer Steve Klabnik, Carol Nichols e toda a comunidade de Rust, faço esse agradecimento pois esse post é **fortemente** inspirado no capítulo 11 do livro "The Rust Programming Language" (RPL), e é uma tentativa de adaptar para o R os ensinamentos que ele provem. Acredito que o RPL é uma das melhores obras de computação que eu já tive o prazer de ler, de lá tirei muitas das práticas que atualmente uso. Além disso, é um livro tao completo que até fazer testes ensinam. Obrigado Steve, Carol e comunidade Rust, agradeço principalmente pelas noites em que explodiram minha cabeça com algum conhecimento novo.
