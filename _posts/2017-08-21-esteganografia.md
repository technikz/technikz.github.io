---
layout: post
comments: true
title: Esteganografia em imagens
---

Dá-se o nome de esteganografia à tarefa de ocultar uma informação em
um arquivo de texto, imagem ou vídeo de modo que apenas o receptor que
conhece como a ocultação foi realizada saiba como recuperar a
informação inserida. Na esteganografia, usa-se o princípio da
ocultação por obscuridade, onde se pressupõe que apenas o remetente e
o destinatário sabem como decifrar o segredo enviado.
<!--break-->

Essa prática é bem antiga. Os gregos costumavam enviar informações por
mensageiros que poderiam ser capturados ou mortos no seu trajeto ao
destino. Logo, nada mais natural de que utilizar alguma forma
considerada segura de transmitir segredos. Naquela época usava-se
esteganografia de texto apenas, posto que não se dispunha dos recursos
modernos dos quais dispomos.

Definitivamente, não é o nosso caso. Temos à mão toda sorte de
computadores para fazer esteganografia. Embora esteja um pouco em
desuso pelos algoritmos modernos de criptografia, ainda é possível se
divertir um pouco com isso. Neste post, trataremos de uma das formas
de esteganografia usando imagens digitais. A ideia é esconder uma
imagem no arquivo de outra imagem sem alterar significativamente a
aparência da portadora.

## Codificação em bits

Todas as imagens são armazenadas na forma de matrizes. Os elementos
dessas matrizes, por sua vez, armazenam as cores dos pixels da
imagem. Para cada posição (x,y) na matriz, uma cor é associada a três
quantidades (geralmente inteiras) que indicam as proporções das cores
primárias Verde, Vermelho e Azul (Red, Green, Blue) da cor
específica. Considere, por exemplo a imagem a seguir.

![Imagem exemplo]({{ site.url }}/images/raposao.png)

É fácil verificar com algum software de tratamento de imagens que a
cor do pixel localizado na posição (linha, coluna) = (200,200)
referenciada em relação ao canto superior esquerdo é marrom.
Matematicamente falando, o pixel marrom correspondente à combinação de
três números inteiros (Red ,Green, Blue) = (133, 96, 77) que são
usados para as representar as componentes de cor.

Cada um dos inteiros de um pixel é normalmente armazenado em um
variável de 8 bits (char, em linguagem C). Com apenas 8 bits é
possível representar cada componente com uma faixa de variação de 0
a 255. Apesar ter apenas um byte de tamanho, essa quantidade permite
enganar com maestria o olho humano e ainda possibilita a geração de um
total de aproximadamente 16 milhões de tonalidades de cores para uma
imagem colorida.

Ocorre que o total dos 8 bits usados para cada componente pode ser
manipulado para esconder informações importantes. Os bits menos
significativos de um pixel guardam pouca informação para olhos
medianos. Observe, por exemplo, a seguinte sequência de imagens. Ela
foi composta manipulando os bits de cada pixel de forma que os N bits
menos significativos de cada componente de cor fossem deixados com
valores iguais a zero para seis valores distintos de N, variando de 1
(canto superior esquerdo) a 6 (canto inferior direito).

|![Imagem exemplo 1]({{ site.url }}/images/raposao1bits.png)| ![Imagem exemplo 2]({{ site.url }}/images/raposao2bits.png) | ![Imagem exemplo 3]({{ site.url }}/images/raposao3bits.png)| 
|---|---|---|
|(1 bit) | (2 bits) | (3 bits) |
|---|---|---|
|![Imagem exemplo 4]({{ site.url}}/images/raposao4bits.png) | ![Imagem exemplo 5]({{ site.url}}/images/raposao5bits.png) | ![Imagem exemplo 6]({{ site.url}}/images/raposao6bits.png) | 
|---|---|---|
|(4 bit) | (5 bits) | (6 bits) |

Perceba como ocorre a degradação das cores da imagem na medida em que
os bits são descartados. Para a imagem do exemplo, a degradação começa
a ser percebida com a perda de 3 ou 4 bits menos significativos. É aí
que entra a possibilidade de ocultação da informação...

Descartando-se uma quantidade de bits suficiente para não perder a
qualidade visual da imagem, pode-se usar posições dos bits perdidos
para esconder informação. Dá-se a essa prática o nome de
esteganografia de imagens usando bits menos significativos, ou **Least
Significant Bit** _steganography_.

## Usando o LSB-Steganography

Na Internet, é possível encontrar alguns exemplos de código que
realizam esteganografia de imagens com relativa facilidade. Neste
post, usarei como exemplo o projeto
[LSB-Steganography](https://github.com/RobinDavid/LSB-Steganography/)
que achei no GitHub.

O programa foi desenvolvido em OpenCV e é bem simples de ser instalado
e utilizado, posto que as ferramentas necessárias já tenham sido
instaladas no computador. Para clonar o repositório da aplicação,
basta abrir o terminal e digitar o seguinte comando:

```console
$ git clone https://github.com/RobinDavid/LSB-Steganography.git
```

Feito isso, o software está disponível via linha de comando para
realizar atividades de ocultação e recuperação de arquivos em uma
imagem fornecida.

No nosso exemplo, procuramos esconder a imagem do arquivo
`android.png` logo abaixo no arquivo da imagem da raposa.

![Android Logo]({{ site.url }}/images/android.png)

Para esconder o arquivo `android.png` no arquivo raposao.png, execute o comando

```console
$ md5sum android.png 
9d911f64d2907bd34a33bab2422d6c09  android.png
$ python LSBSteg.py -image raposao.png -binary android.png -steg-out raposao-steg.png
```

A imagem `raposao-steg.png` criada no processo é mostrada a
seguir. Perceba que não há diferenças visuais notáveis no arquivo que
contém a imagem escondida.

![Raposao Steg]({{ site.url }}/images/raposao-steg.png)

Para revelar o arquivo `android.png` escondido no arquivo `raposao-steg.png`,
execute o seguinte comando:

```console
$ python LSBSteg.py -steg-image raposao-steg.png -out reveal.out
$ md5sum reveal.out 
9d911f64d2907bd34a33bab2422d6c09  reveal.out
```

No Linux, é possível verificar que tipo de arquivo é `reveal.out` usando
o comando `file`.

```console
$ file reveal.out
reveal.out: PNG image data, 64 x 72, 8-bit/color RGBA, non-interlaced
```

Perceba que é uma imagem PNG com as mesmas dimensões
do arquivo original. E, como também percebi, possui o mesmo conteúdo.

Um cuidado deve ser tomado para que o processo funcione bem: a imagem
portadora deve ser salva em formato de compressão sem perdas. No
experimento, usei o formato PNG. Formatos de compressão com perdas
degradam os valores dos pixels, de sorte que a informação escondida é
completamente perdida no processo.

No exemplo do post foi realizada a ocultação de um arquivo de imagem,
mas quaisquer arquivos regulares também podem ser ocultados. O autor
sugere apenas como boa prática usar uma portadora cujo tamanho seja
pelo menos oito vezes maior que a imagem que se pretende
esconder. Isso é de se esperar, posto que o uso excessivo de bits
degrada a qualidade da imagem onde se deseja ocultar a informação.
