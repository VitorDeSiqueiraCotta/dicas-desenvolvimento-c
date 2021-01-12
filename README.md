![GitHub License](https://img.shields.io/github/license/VitorDeSiqueiraCotta/dicas-desenvolvimento-c.svg?color=Blue&label=License&style=flat-square)  ![GitHub Last Commit](https://img.shields.io/github/last-commit/VitorDeSiqueiraCotta/dicas-desenvolvimento-c.svg?color=Blue&label=Last%20Commit&style=flat-square)  ![GitHub Views](https://img.shields.io/github/search/VitorDeSiqueiraCotta/dicas-desenvolvimento-c/dicas-desenvolvimento-c.svg?color=Blue&label=Views&style=flat-square)

# Dicas desenvolvimento C

Guia básico de desenvolvimento na linguagem C, contendo compilação de boas práticas para maximizar a produtividade, confiabilidade, segurança e qualidade do software.

A linguagem C tem tipificação forte, ou seja, o compilador realiza rotinas de verificação para prevenir falhas ao manipular as variáveis de forma errônea. O programador precisa especificar para o compilador como o software irá funcionar (removendo as ambiguidades), pois, muitos problemas são indecidíveis, e alguns destes podem ser mitigados. 

# Parâmetro `void`

```C
#include<stdio.h>
void foo() {
    printf("Hello world!");
}
int main() {
    foo('O','i',"a resposta da vida eh:", 42, "nao deu erro.");    
    return 0;
}
```

O código acima foi compilado com sucesso, mesmo a assinatura da função não tendo algum parâmetro, a implementação não deixa explícito a ausência deles, **isto é perigoso, a função pode receber dezenas variáveis sem saber o seu tipo ([Variadic Function](https://en.wikipedia.org/wiki/Variadic_function)) um erro potencial não detectado pelo compilador**. A solução é usar `void`, nesta situação representa vazio ou ausência de variáveis na assinatura da função:

```c
void foo(void) {
    printf("Hello world!");
}
```

> Observação: em C++ as assinaturas das funções sem parâmetros é entendida por padrão como  `void` implícito, neste caso não faz diferença adicionar `void`.
>

# Casting explícito na alocação

Funções que manipulam a alocação de memória como: `malloc(), calloc(), realloc()` retornam ponteiro genérico do tipo `void`, o compilador não é capaz de validar o tipo de dado. O exemplo a seguir mostra um ponteiro  `int * ` recebendo alocação de `char *`, essa operação é inválida, o compilador detectará o erro através do casting explícito:

```C
int *numero = malloc(sizeof(char)); /* Erro não detectado */
int *numero = (char *)malloc(sizeof(char)); /* Casting explícito erro detectado */
```

# Qualificador `const`

Qualificador de tipo `const` serve para melhorar a semântica do código, prevenindo operações ilegais e acidentais. Quando o `const` é adicionado, a variável será acessada e não alterada, transformando-a numa constante, o compilador emitirá erros ao ser modificada. Outra vantagem do `const` é na melhora da confiabilidade e segurança contra os vazamentos de memória.

```C
void foo(void) {
    int sentidoDaVida = 42;
    sentidoDaVida++; /* Não emitirá erro */
    printf("%d", sentidoDaVida);
}
void foo(void) {
    int const SENTIDO_DA_VIDA = 42;
    SENTIDO_DA_VIDA++; /* Erro de compilação */
    printf("%d", SENTIDO_DA_VIDA);
}
```

# Instanciar estruturas diretamente

Um atalho de desenvolvimento é definir os valores das estruturas diretamente na declaração, economizando tempo e linhas de códigos, conforme o exemplo a seguir:

```C
#include <stdio.h>
#include <stdlib.h>

struct eixo {
    int x;
    int y;
};

int main(void) {
    struct eixo p1 = {.x = 1, .y = 2}; /* Declaração direta */
    struct eixo p2 = {3,4};            /* Declaração direta conforme a ordem dos membros */
    struct eixo p3;                    /* Declaração indireta */
   
    p3.x = 5;
    p3.y = 6;
    
    printf("P1(%d, %d)\n", p1.x, p1.y);
    printf("P2(%d, %d)\n", p2.x, p2.y);
    printf("P3(%d, %d)", p3.x, p3.y);
    
    return EXIT_SUCCESS;
}
```

# Aterrar ponteiros livres

Após acionar a função `free()`, é recomendado aterrar o ponteiro com `NULL`, caso contrário, se o ponteiro sofrer outro `free()`, haverá corrupção no gerenciamento de memória e o código será vulnerável. Há duas soluções, deixar explícito o aterramento ou adaptar a função `free()`.

```c
char* p = (char *)malloc(sizeof(char));
*p = 'x';
free(p);
p = NULL;	/* Aterramento do Ponteiro */
free(p);	/* Double Free sem efeito */

void free_s(void **p) {	/* Alternativa do free() seguro */
    if (p != NULL) {
        free(*p);
        *p = NULL;
    }
}
```

# Buffer do teclado :keyboard:

```C
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    int a;
    char b;

    scanf("%d", &a);
    scanf("%c", &b);
    
    printf("INT: [ %d ], CHAR: [ %c ]", a, b);
    
    return EXIT_SUCCESS;
}
```

O buffer do teclado pode capturar lixo de memória caso não seja manipulada de forma adequada, o código acima exemplifica a situação. O usuário insere um inteiro na variável "A", entretanto, quando o  `ENTER` é pressionado, o programa interpretará também o caractere `\n` (quebra de linha), o próximo `scanf()` vai receber o lixo de memória, causando inconsistência. Há uma variedade de soluções, mas dependendo do contexto, uma será será mais adequada.

## Tratar `scanf()`

As pequenas modificações a seguir no `scanf()` soluciona o `\n` do buffer do teclado, o `%*c` sinaliza que o próximo caractere deverá ser descartado, ou seja, esta solução evita a propagação de lixo de memória do teclado para os próximos `scanf()`.

```C
/* Adicionar %*c no final para evitar a propagação do '\n' */
scanf("%c%*c", &caractere);
scanf("%d%*c", &numero);
scanf("%s%*c", string);
```

> Observação: se o  `scanf()` for acionado com lixo de memória, a solução irá falhar.

Adicionar espaço antes para ignorar um possível `\n`.

```C
/* Adicionar um espaço antes do %c */
scanf(" %c", &var);
```

Combinação das duas técnicas evita a propagação de lixo de memória e ignora o buffer inválido.

```C
/* Adicionar espaço no começo e adiconar "%*c" no final */
scanf(" %c%*c", &var);
```

## Limpar `stdin`

As entradas dos teclados são representados pelo `stdin` (standard input), e é possível limpar diretamente o buffer. A seguir será mencionado três funções que limpam o buffer,  `setbuf()` é a melhor opção por ser portátil tanto no Windows e Linux/UNIX.

```C
/* Linux/UNIX/Windows */
setbuf(stdin, NULL);

/* Windows */
fflush(stdin);

/* Linux/UNIX */
__fpurge(stdin);
```

## Excesso de entradas

Caso o `scanf()` receba muitas de entradas inválidas, é possível tratar para resolver o lixo do buffer. Por exemplo, o `scanf("%d", &numero)` armazena apenas uma entrada que corresponde um número, mas caso seja inserido "1234 A B C D", os outros `scanf()` irão receber o "A B C D" mas sem a possibilidade de validar a entrada. A solução é descartar o que não era previsto por meio de macro, ou implementar uma função.

```C
/* Macro */
#define CLEAN_BUFF do{int c; while((c = getchar()) != '\n' && c != EOF);}while(0);

/* Função */
void limparBuffer(void) {
    do{
        int c;
        while((c = getchar()) != '\n' && c != EOF);
    }
    while(0);
}
```

# Operador incremento `++`  e decremento  `--`

Nas situações em que os operadores  `++` e `--` que não altera o resultado por ser PRÉ-(incrementando/decrementado) ou PÓS-(incrementando/decrementado), é indicado utilizar o PRÉ, pois, não precisa de criar uma variável auxiliar na operação, sendo mais performática. Conforme o exemplo a seguir:

```C
/* Lento */
for(int i = 0; i < 1000000; i++){}

/* Rápido*/
for(int i = 0; i < 1000000; ++i){}
```

# Operador ternário `?:`

Operador ternário é uma alternativa elegante do `if-else` e `switch-case` quando o resultado da condição precisa executar apenas uma ação, isto economiza linhas de código e melhora a legibilidade. A sintaxe do operador ternário é `condição ? ação verdadeiro : ação falso ;`.

```C
/* If-else comum, 4 linhas utilizadas */
if(numero % 2 == 0)
    printf("PAR");
else
    printf("IMPAR");

/* Operador ternário, 1 linha utilizada */
(numero % 2 == 0) ? printf("PAR") : printf("IMPAR");

/* Switch-case, 13 linhas utilizadas */
switch(numero) {
    case 1:
        printf("Acao 1");
        break;
    case 2:
        printf("Acao 2");
        break;
    case 3:
        printf("Acao 3");
        break;
    default:
        printf("Erro");
}

/* Operador ternário encadeado, 3 linhas utilizadas */
numero == 1 ? printf("Acao 1") :
numero == 2 ? printf("Acao 2") :
numero == 3 ? printf("Acao 3") : printf("Erro");
```

# Operador goes to `-->`

Operador goes to é combinação dos operadores `--` e `>`. A cada iteração é decrementado uma unidade até a condição seja satisfeita.

```C
int numero = 5;
while(numero --> -5) {	/* Exibe 4 até -5 */
	printf("%d ", numero);
}
```

# Loop Infinito

Há quatro maneiras de construir um loop infinito, através de: `while`, `do-while`, `for` e `goto`, estas opções geram o mesmo resultado, entretanto, há algumas peculiaridades em cada uma. O loop `while` e `do-while` são idênticas, entretanto, o código `while` é legível, enquanto o `for` é a pior escolha na legibilidade. Na condição de repetição, qualquer valor diferente de zero é verdadeiro, mas para melhorar a semântica é recomendado usar a biblioteca `stdbool.h` (C++ não precisa da biblioteca) que permite utilizar os tipos booleanos `true` e `false`.

No surgimento da linguagem C  `goto`  foi a primeira forma de criar laço de repetição (essa afirmação carece de referência), depois surgiu outras alternativas que tem a semântica melhor, ademais, o uso de `goto` dificulta a depuração, e nos códigos atuais é depreciada, considerada uma má prática legado. Em poucas ocasiões o `goto` pode ser utilizado como atalho na depuração.

```C
/* Legível */
while(true) {
	/* Código */
}

/* Menos legível  */
do {
	/* Código */
} while(true);

/* Ilegível */
for(;;) {
	/* Código */    
}

/* Não recomendado, legado */
LOOP:
	/* Código */
goto LOOP;
```

> Observação: compiladores e IDEs podem sinalizar warning de loop infinito.

# Enumeração de tipo `enum`

Enumeração de tipo ou `enum`,  é uma declaração de variável que receba apenas constante de inteiro. O objetivo do `enum` é melhorar a semântica e legibilidade do código, normalmente é aplicado juntamente com `switch`. Por padrão, ao instanciar o `enum`, os valores das constantes começam a partir de 0 e irá incrementando consecutivamente, mas é possível customizar o valor das constantes, conforme o exemplo a seguir:

```C
#include <stdio.h>
#include <stdlib.h>

enum BATERIA {
    vazia,         /* Valor 0, padrão */
    metade = 50,   /* Valor definido */
    cheia = 100,   /* Valor definido */
    sobrecarregado /* Valor 101, incrementado automaticamente */
};

int main(void) {
    enum BATERIA carga;
    
    carga = cheia;
    
    switch(carga) {
        case sobrecarregado:
            printf("Carga 101%%\n");
            break;
        case cheia:
            printf("Carga 100%%\n");
            break;
        case metade:
            printf("Carga 50%%\n");
            break;
        case vazia:
            printf("Carga 0%%\n");
            break;
    }
   
    printf("Sobrecarga: %d\n", sobrecarregado);
    printf("Cheia:      %d\n", cheia);
    printf("Metade:     %d\n", metade);
    printf("Vazia:      %d\n", vazia);
    
    return EXIT_SUCCESS;
}
```

# Estrutura `union`

No passado dos computadores, a memória volátil (RAM) era um recurso escarço e caro, o `union` desempenhou o importante papel de economizar e otimizar o mesmo espaço de memória por permitir que a memória alocada seja utilizado por diferentes variáveis\tipos de dados, tais como: `char`, `int`, `int[]`, `float`, etc. O tamanho do espaço alocado será correspondente ao espaço da maior variável\tipo de dado da estrutura `union`. Os membros da estrutura `union` são acessadas da mesma forma do `struct`,  segue um exemplo prático.

```C
#include <stdio.h>
#include <stdlib.h>
#include <float.h>
#define PI 3.14159265358979323846264338327950288419716939937510582
/*           float^  double^ */

typedef struct {
    int baixaPrecisao;      /* Usa 4 bytes */
    float mediaPrecisao;    /* Usa 4 bytes */
    double altaPrecisao;    /* Usa 8 bytes */
}   struct_numero;          /* Total 16 bytes */

typedef union {
    int baixaPrecisao;      /* Usa 4 bytes */
    float mediaPrecisao;    /* Usa 4 bytes */
    double altaPrecisao;    /* Usa 8 bytes, maior alocação */
}   union_numero ;          /* Total 8 bytes*/

int main(void) {
    struct_numero X;
    union_numero Y;

    printf("Tamanho alocacao STRUCTC: %d bytes\n", sizeof(struct_numero));
    printf("Tamanho alocacao UNION: %d bytes\n\n", sizeof(union_numero));

    X.altaPrecisao = (double)PI;
    Y.altaPrecisao = (double)PI;

    printf("Precisao DOUBLE %d casas decimais.\n", DBL_DIG);
    printf("Alta precisao X:\t %.*lf\n", DBL_DIG, X.altaPrecisao);
    printf("Alta precisao Y:\t %.*lf\n\n", DBL_DIG, Y.altaPrecisao);

    X.mediaPrecisao = (float)PI;
    Y.mediaPrecisao = (float)PI;

    printf("Precisao FLOAT %d casas decimais.\n", FLT_DIG);
    printf("Media precisao X:\t %.*f\n", FLT_DIG, X.mediaPrecisao);
    printf("Media precisao Y:\t %.*f\n\n", FLT_DIG, Y.mediaPrecisao);

    X.baixaPrecisao = (int)PI;
    Y.baixaPrecisao = (int)PI;

    printf("Precisao INT %d casas decimais.\n", 0);
    printf("Baixa precisao X:\t %.d\n", X.baixaPrecisao);
    printf("Baixa precisao Y:\t %.d\n\n", Y.baixaPrecisao);

    return EXIT_SUCCESS;
}
```

# Limpar o terminal Windows/Linux

O comando que limpa a tela do terminal é feita através de chamada de sistema (funções do tipo `system()`), infelizmente o uso destas funções não são portáteis entre os sistemas operacionais. A solução seria usar a definição de macro para detectar o sistema operacional utilizado, e customizar a entrada do comando com a constante `CLEAR` que limparia a tela, `cls` é usado pelo Windows, enquanto `clear` é usado pelo Linux. O código abaixa exemplifica o uso de macros.

```C
#include <stdio.h>
#include <stdlib.h>

#ifdef _WIN32
    #define CLEAR "cls"
#else
    #define CLEAR "clear"
#endif

int main(void) {
    printf("Texto 1.");
    
    getchar();		/* Pausa */
    system(CLEAR); 	

    printf("Texto 2.");

	return EXIT_SUCCESS;
}
```

# Licença

**"[Dicas desenvolvimento C](https://github.com/VitorDeSiqueiraCotta/dicas-desenvolvimento-c/)" by [Vitor de Siqueira Cotta](https://flickr.com/photos/lschlagenhauf/) is licensed under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/).** 

[![CC BY-SA 4.0][cc-by-sa-image]][cc-by-sa]

# Contato

<div align="center"><a href="https://www.linkedin.com/in/vitor-de-siqueira-cotta-analista-programador/"><img src="https://img.shields.io/badge/-Github-181717?&style=for-the-badge&logo=github&logoColor=white"/></a> <a href="https://github.com/VitorDeSiqueiraCotta"> <img src="https://img.shields.io/badge/-Linkedin-0077B5?&style=for-the-badge&logo=linkedin&logoColor=white"/></a> <a href="mailto:vitorsiqueira95@outlook.com"><img src="https://img.shields.io/badge/-Outlook-0078D4?&style=for-the-badge&logo=microsoft-outlook&logoColor=white&link=mailto:vitorsiqueira95@outlook.com"/></a> <a href="https://www.udemy.com/user/vitor-siqueira-2/"> <img src="https://img.shields.io/badge/-Udemy-EC5252?&style=for-the-badge&logo=udemy&logoColor=white&link=https://www.udemy.com/user/vitor-siqueira-2/"/></a></div>

[cc-by-sa]: http://creativecommons.org/licenses/by-sa/4.0/
[cc-by-sa-image]: https://licensebuttons.net/l/by-sa/4.0/88x31.png
[cc-by-sa-shield]: https://img.shields.io/badge/License-CC%20BY--SA%204.0-lightgrey.svg

